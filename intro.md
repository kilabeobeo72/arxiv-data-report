# arXiv Data Science Project — Report Book

Trang báo cáo **Jupyter Book** của nhóm cho đồ án môn **Nhập môn Khoa học Dữ liệu**.

Dự án xây dựng một đường ống (pipeline) thu thập, phân tích cấu trúc phân cấp và chuẩn hóa kho bài báo khoa học **arXiv** từ nguồn LaTeX gốc qua các giai đoạn:

1. **Milestone 1 — Data Crawling**:
   - Discovery danh sách các bài báo được phân công qua **arXiv Atom API** và xác định đầy đủ các phiên bản (`v1..vN`) của từng công bố.
   - Tải tuần tự các gói mã nguồn từ mirror chuyên dụng `https://export.arxiv.org/src/{version_id}` với cơ chế kiểm soát tốc độ ($\ge 3\text{s}$/request), kiểm tra toàn vẹn `SHA-256` và phục hồi trạng thái (`--resume`).
   - Giải nén an toàn và lọc chỉ giữ lại các file `.tex` và `.bib`, loại bỏ hoàn toàn file hình ảnh/binary để tối ưu dung lượng lưu trữ.
   - Thu thập siêu dữ liệu trích dẫn (Reference Metadata) qua **Semantic Scholar Graph API** và chuẩn hóa thành hai bảng `papers` và `references`.
   - Đo lường hiệu năng (Runtime, Average/Peak RAM, Peak/Final Disk) bằng bộ lấy mẫu nền `psutil`.
2. **Milestone 2 — Hierarchical Parsing & Standardization**:
   - Xác định đường biên dịch chính từ file gốc chứa `\documentclass` và gom đệ quy các file con qua `\input` / `\include`.
   - Phân tích cấu trúc văn bản LaTeX thành cây phân cấp (Document $\rightarrow$ Section $\rightarrow$ Subsection $\rightarrow$ Paragraph $\rightarrow$ Sentence / Equation / Figure / Table).
   - Chuẩn hóa công thức toán học, chuyển đổi `\bibitem` sang chuẩn BibTeX, và khử trùng lặp (Deduplication) nội dung xuyên suốt các phiên bản `v1..vN` vào 3 bảng `elements`, `hierarchy`, `bibtex`.

---

## Phân công phạm vi dữ liệu và nhiệm vụ trong nhóm

Mỗi thành viên được cấp một khoảng **5.000 mã arXiv ID** riêng biệt (tổng cộng **25.000 ID** cho toàn nhóm 5 thành viên) và phụ trách một module cốt lõi trong kiến trúc chung:

| Thành viên | Khoảng phân công (`start_month start_id end_month end_id`) | Dải mã arXiv ID (`YYMM.NNNNN`) |
| :---: | :--- | :--- | :--- |
| **Thành viên 1 (Kiệt)** | `2510.14401` $\rightarrow$ `2510.19400` (5.000 ID) |
| **Thành viên 2 (Dương)** | `2025-10 9401 2025-10 14400` | `2510.09401` $\rightarrow$ `2510.14400` (5.000 ID) |
| **Thành viên 3 (Hà)** | `2510.04401` $\rightarrow$ `2510.09400` (5.000 ID) |
| **Thành viên 4 (Kiên)** | `2025-09 26047 2025-10 4400` | `2509.26047` $\rightarrow$ `2509.26646` + `2510.00001` $\rightarrow$ `2510.04400` (5.000 ID) |
| **Thành viên 5 (Anh)** |  `2025-09 11047 2025-09 16046` | `2509.11047` $\rightarrow$ `2509.16046` (5.000 ID) |

---

## Liên kết Repository

- **Source Code Repository**: [https://github.com/theanhnek/arxiv-data-project](https://github.com/theanhnek/arxiv-data-project)
- **Cơ chế tạo bảng biểu & đồ thị**: Toàn bộ bảng số liệu và biểu đồ trong cuốn Jupyter Book này được **sinh tự động bằng mã Python trong quá trình biên dịch (`execute_notebooks: force`)** từ các tệp metrics chuẩn (`metrics/benchmark_summary.json` và `metrics/per_paper_metrics.json`).
