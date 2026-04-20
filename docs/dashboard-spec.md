# Dashboard Specification (Layer-2 Observation)

Bản đặc tả này mô tả cấu hình cho Dashboard 6-panel dùng trong quá trình giám sát hệ thống AI Agent.

## 1. Latency (P50/P95/P99)
- **Loại biểu đồ**: Time-series Line Chart.
- **Đơn vị**: Milliseconds (ms).
- **SLO Line**: Có đường kẻ đỏ ở mức **1500ms** (SLO target tinh chỉnh theo thực tế).
- **Mục tiêu**: Giám sát độ trễ phản hồi. Nếu P95 > 1500ms, cần kiểm tra ngay vì bình thường hệ thống chạy dưới 800ms.

## 2. Traffic & Throughput
- **Loại biểu đồ**: Bar Chart (Stacked by status code).
- **Đơn vị**: Requests per Minute (RPM).
- **Mục tiêu**: Theo dõi lưu lượng tải. Phân biệt các request thành công (200) và lỗi (4xx, 5xx).

## 3. Error Rate Breakdown
- **Loại biểu đồ**: Pie Chart hoặc Donut Chart.
- **Đơn vị**: Percentage (%) / Count.
- **Phân nhóm**: Group theo `error_type` (ví dụ: `ConnectTimeout`, `RateLimitError`, `PiiRedactionError`).
- **Mục tiêu**: Xác định nhanh nguyên nhân gây lỗi phổ biến nhất.

## 4. Cost Over Time (Accumulated)
- **Loại biểu đồ**: Area Chart.
- **Đơn vị**: USD ($).
- **Mục tiêu**: Theo dõi ngân sách. Cảnh báo nếu độ dốc của biểu đồ tăng đột biến (cost spike).

## 5. Token Usage (In vs Out)
- **Loại biểu đồ**: Multiline Chart.
- **Đơn vị**: Tokens.
- **Mục tiêu**: Giám sát tỷ lệ Token Input vs Output. Giúp tối ưu hóa prompt (nếu input quá lớn) hoặc kiểm soát hiện tượng lặp từ (nếu output quá lớn).

## 6. Quality Proxy Index
- **Loại biểu đồ**: Gauge Chart.
- **Đơn vị**: Score (0.0 - 1.0).
- **Ngưỡng**: Vùng xanh (> 0.7), Vùng vàng (0.4 - 0.7), Vùng đỏ (< 0.4).
- **Mục tiêu**: Đánh giá nhanh chất lượng phản hồi từ hệ thống dựa trên `heuristic_quality` score.

---

## Tiêu chuẩn trình bày (Quality Bar)
- **Time range**: Mặc định 1 giờ gần nhất.
- **Auto-Refresh**: Mỗi 15 giây.
- **Labels**: Tất cả các trục X, Y phải có nhãn và đơn vị rõ ràng.
- **Dashboard Layout**: Sắp xếp Latency và Error Rate ở các vị trí dễ thấy nhất (góc trên bên trái).
