# Phương pháp triển khai — Milestone 1: Data Crawling

## 1. Kiến trúc tổng thể của hệ thống

Hệ thống thu thập dữ liệu của nhóm được chia thành **5 module tách biệt** nhưng dùng chung cấu hình (`CrawlConfig`), cơ chế điều tiết nhịp gửi yêu cầu (Rate-limit Pacing) và ghi trạng thái nguyên tử (Atomic State Snapshots):

1. **Module 1 — Entry Discovery (`src/discovery.py`)**:
   - Đọc bảng khoảng phân công (`start_month start_id end_month end_id`) hoặc danh sách ID đơn lẻ, kiểm tra tính hợp lệ và chuẩn hóa về định dạng gốc (`YYMM.NNNNN` hoặc `archive/YYMMNNN`).
   - Gọi **arXiv Atom API** (`https://export.arxiv.org/api/query`) với giãn cách tối thiểu $\ge 3\text{ giây}$ giữa các request.
   - Đọc hậu tố phiên bản mới nhất (ví dụ `v2`, `v4`) để sinh danh sách toàn bộ các phiên bản từ `v1` đến `vN` cần thu thập và ghi vào `data/state/discovery.json`.
2. **Module 2 — Versioned Source Downloader (`src/downloader.py`)**:
   - Đọc danh sách các phiên bản đã được xác nhận trong `data/state/discovery.json` và tải tuần tự từ mirror chuyên dụng `https://export.arxiv.org/src/{version_id}`.
   - Chia sẻ chung bộ đếm thời gian `last_request` với `ArxivClient` để đảm bảo không vượt quá giới hạn 1 request / 3 giây.
   - Stream dữ liệu theo từng khối 64 KB xuống file tạm `.part`, tính mã băm `SHA-256` trên luồng, kiểm tra magic bytes để chặn trang HTML báo lỗi hoặc bị chặn rate-limit (ngay cả khi trả mã `HTTP 200`), và chỉ đổi tên nguyên tử khi tải hoàn tất.
3. **Module 3 — Safe TeX/BibTeX Extractor (`src/extractor.py`)**:
   - Nhận diện định dạng gói nguồn thực tế (`tar_archive`, `plain_source`, `corrupt_archive`, `unsupported_format`).
   - Chỉ trích xuất các file thông thường có phần mở rộng `.tex` hoặc `.bib` vào `data/corpus/<paper_id>/v<N>/source/`, loại bỏ hoàn toàn các file hình ảnh (`.png`, `.jpg`, `.pdf`, `.eps`) và file binary.
   - Bảo toàn cấu trúc thư mục con tương đối để các lệnh `\input{...}` và `\include{...}` hoạt động chính xác ở Milestone 2, đồng thời chặn mọi tấn công Path Traversal (`..`, đường dẫn tuyệt đối, symlink/hardlink).
   - Đo lường độc lập 3 chỉ số dung lượng: `input_bytes` (gói nén tải về), `archive_uncompressed_bytes` (tổng dung lượng giải nén trước khi lọc bỏ ảnh) và `retained_bytes` (dung lượng `.tex`/`.bib` giữ lại).
4. **Module 4 — Semantic Scholar References & Storage (`src/references.py`, `src/storage.py`)**:
   - Truy vấn **Semantic Scholar Graph API** (`https://api.semanticscholar.org/graph/v1/paper/arXiv:{arxiv_id}`) theo mức công bố (publication-level), tuân thủ đồng thời giới hạn 1 request/giây và 100 requests/5 phút.
   - Lưu giữ toàn bộ danh sách trích dẫn thô (bao gồm cả các tài liệu không có mã arXiv) trong kho cục bộ và xuất 2 bảng dữ liệu chuẩn: `papers` và `references`.
5. **Module 5 — Resource Monitoring & Corpus Metrics (`src/metrics.py`)**:
   - Chạy luồng nền `ResourceMonitor` (sử dụng `psutil`) lấy mẫu định kỳ mỗi `0.5 giây` tổng bộ nhớ RSS của tiến trình chính cộng các tiến trình con (`avg_ram_mb`, `peak_ram_mb`) và dung lượng đĩa (`peak_working_disk_mb`, `final_corpus_disk_mb`).
   - Tổng hợp toàn bộ số liệu chất lượng và hiệu năng ra `data/metrics/benchmark_summary.json` và `data/metrics/per_paper_metrics.json`.

---

## 2. Công cụ và Thư viện sử dụng

| Thành phần | Công cụ / Giao thức | Thư viện Python | Mục đích sử dụng |
| :--- | :--- | :--- | :--- |
| **Entry Discovery** | arXiv Atom API (`/api/query`) | `requests`, `xml.etree.ElementTree` | Tra cứu metadata và xác định số phiên bản `v1..vN` |
| **Source Download** | Mirror `export.arxiv.org/src/` | `requests`, `hashlib` | Tải streaming gói nguồn theo phiên bản & kiểm tra `SHA-256` |
| **Source Extraction** | Tar/Gzip & Plain TeX | `tarfile`, `tempfile`, `pathlib` | Giải nén an toàn `.tex`/`.bib` và loại bỏ hình ảnh |
| **Reference Crawling** | Semantic Scholar Graph API v1 | `requests`, `collections.deque` | Thu thập danh sách trích dẫn và mã định danh `externalIds.ArXiv` |
| **Performance Audit** | OS Process & Filesystem | `psutil`, `shutil`, `time` | Đo thời gian thực thi, RAM trung bình/cực đại và dung lượng đĩa |

---

## 3. Cơ chế chống lỗi và Phục hồi (Fault Tolerance & Resume)

- **Ghi file nguyên tử (Atomic Writes)**: Mọi file trạng thái JSON và file mã nguồn đều được ghi qua file tạm (`.tmp` / `.part`) trước khi gọi `os.replace`, đảm bảo nếu chương trình bị ngắt đột ngột giữa chừng sẽ không bao giờ sinh ra file hỏng nhưng mang tên hợp lệ.
- **Khôi phục thông minh (`--resume`)**: Khi chạy lại với cờ `--resume`, hệ thống kiểm tra đồng thời trạng thái trong `data/state/`, sự tồn tại thực tế của file đích và mã băm `SHA-256` để bỏ qua các phiên bản đã thành công và chỉ thực hiện lại các phiên bản bị lỗi mạng tạm thời (`ChunkedEncodingError`, `HTTP 429`, `HTTP 5xx`).
