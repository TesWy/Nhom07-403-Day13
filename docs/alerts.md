# Alert Rules and Runbooks

Tài liệu này hướng dẫn cách xử lý các sự cố dựa trên cảnh báo từ hệ thống giám sát.

## Quy trình xử lý chung (Triage Flow)

1. **Nhận cảnh báo**: Xác định Severity (P1/P2) và loại cảnh báo.
2. **Kiểm tra nhanh (Quick Check)**: Kiểm tra Dashboards để xác định phạm vi ảnh hưởng (toàn bộ hệ thống hay chỉ một tính năng).
3. **Phân tích sâu (Deep Dive)**: Sử dụng Langfuse Traces và JSON Logs để tìm root cause.
4. **Xử lý (Mitigation)**: Thực hiện các bước giảm thiểu theo hướng dẫn bên dưới.
5. **Hậu kiểm**: Xác nhận cảnh báo đã tắt và hệ thống ổn định.

---

## 1. High latency P95 (P2)
- **Tình trạng**: Độ trễ đuôi (tail latency) vượt quá 5s.
- **Tác động**: Trải nghiệm người dùng bị chậm, có thể gây timeout ở phía client.
- **Các bước kiểm tra**:
  1. Truy cập **Langfuse/Traces**, lọc các trace có latency > 5000ms.
  2. So sánh thời gian của `RAG span` vs `LLM span`. 
  3. Kiểm tra xem incident toggle `rag_slow` có đang bật không (`GET /health`).
- **Xử lý**:
  - Nếu do RAG: Tạm thời giảm `top_k` hoặc sử dụng cache.
  - Nếu do LLM: Kiểm tra độ dài prompt hoặc xem xét chuyển sang model nhanh hơn.

## 2. High error rate (P1) - CRITICAL
- **Tình trạng**: Tỷ lệ lỗi > 5% trong 5 phút.
- **Tác động**: Người dùng không nhận được câu trả lời.
- **Các bước kiểm tra**:
  1. Mở Log Viewer, lọc theo `level: "error"` và group theo `error_type`.
  2. Kiểm tra log `request_failed` để xem chi tiết exception.
  3. Xác định lỗi do hạ tầng (LLM API down) hay do ứng dụng (PiiScrubbing failure, Schema mismatch).
- **Xử lý**:
  - Nếu LLM API lỗi: Chuyển hướng sang model dự phòng (fallback).
  - Nếu do code mới: Rollback bản deploy gần nhất.

## 3. Cost budget spike (P2)
- **Tình trạng**: Chi phí giờ tăng > 2 lần bình thường.
- **Tác động**: Vượt ngân sách dự kiến của dự án.
- **Các bước kiểm tra**:
  1. Phân tách chi phí theo `feature` và `model`.
  2. Kiểm tra xem có user nào đang gửi message quá dài không (check `tokens_in` trong logs).
  3. Kiểm tra incident `cost_spike` có đang được kích hoạt giả lập không.
- **Xử lý**:
  - Áp dụng giới hạn độ dài input message (Prompt truncation).
  - Tắt các tính năng tiêu tốn nhiều token (như verbose RAG).

## 4. Token burn spike (P2)
- **Tình trạng**: Output tokens > 50,000/phút.
- **Tác động**: Tăng chi phí đột biến và có thể bị rate limit từ nhà cung cấp LLM.
- **Các bước kiểm tra**:
  1. Kiểm tra Langfuse để xem LLM có bị hiện tượng "lặp từ" (infinite loop) không.
  2. Kiểm tra xem có cuộc tấn công DoS nào đang diễn ra không.
- **Xử lý**:
  - Hạn chế `max_tokens` trong tham số gọi LLM.
  - Áp dụng Rate Limiting theo `user_id`.
