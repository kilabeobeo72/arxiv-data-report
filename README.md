# arXiv Data Project — Jupyter Book Report (Milestone 1 & 2)

Đây là mã nguồn **Jupyter Book** của nhóm cho đồ án môn **Nhập môn Khoa học Dữ liệu (HCMUS)**, tự động sinh bảng biểu và đồ thị từ file `metrics/benchmark_summary.json` và `metrics/per_paper_metrics.json` trong lúc build.

## 1. Cách đưa lên GitHub Pages (`github.io`) để lấy link nộp Moodle

Theo quy định đề bài, báo cáo Jupyter Book được đặt trong **một repository public riêng** (ví dụ: `arxiv-data-report`). Thực hiện đúng 3 bước sau:

### Bước A: Tạo repository mới trên GitHub
1. Vào GitHub $\rightarrow$ bấm **New repository**.
2. Đặt tên repository: `arxiv-data-report` (chọn **Public**).
3. Vào mục **Settings $\rightarrow$ Pages** của repo mới tạo $\rightarrow$ tại mục **Build and deployment > Source**, chọn **GitHub Actions**.

### Bước B: Đẩy thư mục `report_book/` này lên repo mới
Mở PowerShell tại thư mục `D:\arxiv-data-project\report_book` và chạy:

```powershell
cd D:\arxiv-data-project\report_book
git init
git branch -M main
git add .
git commit -m "Initialize Jupyter Book report for Milestone 1 and 2"
git remote add origin https://github.com/<TEN_GITHUB_CUA_BAN>/arxiv-data-report.git
git push -u origin main
```

### Bước C: Lấy link `github.io` nộp lên Moodle
- Sau khi `git push`, tab **Actions** trên GitHub sẽ tự động chạy workflow `Deploy Jupyter Book to GitHub Pages` (mất khoảng 1–2 phút).
- Khi hiện dấu tích xanh, trang web Jupyter Book của nhóm sẽ hoạt động tại địa chỉ cố định:
  👉 **`https://<TEN_GITHUB_CUA_BAN>.github.io/arxiv-data-report/`**
- Đại diện nhóm nộp đường link này (cùng link repo code `https://github.com/theanhnek/arxiv-data-project`) lên Moodle.

---

## 2. Cách cập nhật số liệu mới sau này (trước hạn nộp Poster)

Mỗi khi nhóm chạy thêm dữ liệu mới hoặc cập nhật Milestone 2:
1. Copy 2 file mới từ `data/metrics/benchmark_summary.json` và `data/metrics/per_paper_metrics.json` đè vào thư mục `metrics/` của repo báo cáo.
2. Chạy `git add . && git commit -m "Update metrics" && git push`.
3. GitHub Actions sẽ tự động chạy lại các notebook để vẽ lại biểu đồ mới mà **đường link `github.io` không hề thay đổi**.
