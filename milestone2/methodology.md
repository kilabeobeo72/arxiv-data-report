# Phương pháp triển khai — Milestone 2: Hierarchical Parsing & Standardization

## 1. Mục tiêu của Milestone 2

Sau khi thu thập các file mã nguồn `.tex` và `.bib` ở Milestone 1, giai đoạn **Milestone 2** chuyển đổi văn bản LaTeX bán cấu trúc thành **cấu trúc cây phân cấp chuẩn hóa (Hierarchical Tree)** và **khử trùng lặp (Deduplication)** xuyên suốt các phiên bản `v1..vN` của mỗi công bố, tạo đầu vào cho bài toán Reference Matching ở Milestone 3.

---

## 2. Quy trình 4 bước của Parser

### 2.1 Tập hợp đa tệp (Multi-file Gathering)
- Xác định tệp gốc (root file) của mỗi phiên bản dựa trên khai báo `\documentclass`.
- Lần theo đệ quy các lệnh `\input{...}` và `\include{...}` (bỏ qua các dòng đã bị comment bằng `%`) để tái tạo chính xác luồng biên dịch LaTeX. Các file `.tex` phụ trợ không nằm trên nhánh biên dịch từ file gốc được loại bỏ.

### 2.2 Xây dựng Cây phân cấp (Hierarchy Construction)
Mỗi phiên bản `vN` được phân tích thành một cây cấu trúc:
- **Root**: `document` (gốc của phiên bản `vN`).
- **Internal Nodes**: Các cấp tiêu đề phân mục (`chapter`, `section`, `subsection`, `subsubsection`, `paragraph`) cùng các vùng chứa (`itemize`, `enumerate`).
- **Leaf Nodes**:
  - `sentence`: Từng câu văn đã được tách chuẩn xác sau khi làm sạch macro trang trí.
  - `equation`: Các khối công thức toán học (`equation`, `align`, `\[...\]`).
  - `figure` / `table`: Các môi trường bảng biểu và hình vẽ (giữ nguyên caption, label và tham chiếu).

### 2.3 Chuẩn hóa Nội dung & Chuyển đổi BibTeX (Standardization)
- **Làm sạch LaTeX & Chuẩn hóa Toán học**: Loại bỏ comment, macro định dạng trình bày (`\vspace`, `\hspace`, `\centering`), đồng nhất các công thức toán nội dòng về `$...$` và khối công thức về `\begin{equation}...\end{equation}`.
- **Hợp nhất Tài liệu tham khảo**: Chuyển đổi toàn bộ các mục `\bibitem{key}` viết tay trong môi trường `thebibliography` sang định dạng **BibTeX** chuẩn và hợp nhất với các mục từ file `.bib`.

### 2.4 Khử trùng lặp đa phiên bản (Cross-Version Deduplication)
- **Khử trùng lặp nội dung (`elements` & `hierarchy`)**: Các nút lá (`sentence`, `equation`, `heading`...) có nội dung sau chuẩn hóa giống hệt nhau giữa `v1`, `v2`, ... chỉ được lưu **một lần duy nhất** trong bảng `elements` (chung một `element_id`), trong khi bảng `hierarchy` lưu các cạnh `(version, element_id, parent_id)` biểu diễn cấu trúc cây của từng phiên bản.
- **Khử trùng lặp trích dẫn (`bibtex`)**: Hợp nhất các mục BibTeX trùng lặp giữa các phiên bản và đồng bộ lại khóa trích dẫn trong `\cite{...}`.
