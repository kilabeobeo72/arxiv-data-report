# arXiv Data Project — Jupyter Book Report (Milestone 1 & 2)

Đây là mã nguồn **Jupyter Book** của nhóm cho đồ án môn **Nhập môn Khoa học Dữ liệu (HCMUS)**, tự động sinh bảng biểu và đồ thị từ file `metrics/benchmark_summary.json` và `metrics/per_paper_metrics.json` trong lúc build.

Cách cập nhật số liệu mới sau này (trước hạn nộp Poster)

Mỗi khi chạy thêm dữ liệu mới hoặc cập nhật Milestone 2:
1. Copy 2 file mới từ `data/metrics/benchmark_summary.json` và `data/metrics/per_paper_metrics.json` đè vào thư mục `metrics/` của repo báo cáo.
2. Chạy `git add . && git commit -m "Update metrics" && git push`.
3. GitHub Actions sẽ tự động chạy lại các notebook để vẽ lại biểu đồ mới mà **đường link `github.io` không hề thay đổi**.
