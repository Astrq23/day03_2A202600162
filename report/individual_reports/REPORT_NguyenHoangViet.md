# Individual Report: Lab 3 - Chatbot vs ReAct Agent

- **Student Name**: Nguyễn Hoàng Việt
- **Role**: Fullstack & Telemetry Engineer
- **Date**: 2026-04-06

---

## I. Technical Contribution (15 Points)

Em phụ trách trải nghiệm thị giác và thu thập Telemetry đo lường hiệu suất mô hình. Em đã dựng ứng dụng Flask tích hợp HTML/CSS Web UI với giao diện Glassmorphism tài chính thực thời thay cho console dòng lệnh thô kệch.

- **Modules Implemented**: `app.py`, `ui/index.html` và file `src/telemetry/logger.py`
- **Code Highlights**:
```python
logger.log_event("LLM_RESPONSE", {"usage": usage, "latency_ms": latency})
```
```html
<div class="stat-card">
  <label>Ngân sách tháng</label>
  <!-- SVG Circular chart updating magically -->
</div>
```
- **Documentation**: Tạo Server Flask có các API Endpoints `/chat` và `/stats`, móc nối tín hiệu Async của JavaScript và lưu vết P50 Latency / Request Token. Thiết kế hoạt ảnh mô phỏng Typing để chặn chết giao diện khi LLM xử lý.

---

## II. Debugging Case Study (10 Points)

- **Problem Description**: Giao diện treo cứng cục bộ từ 5-10 giây chờ câu trả lời, không biết Agent đang hoạt động hay đã bị crash ở đâu. Sidebar biểu đồ vòng tròn % ngân sách luôn hiển thị 0%.
- **Diagnosis**: 
  1. API Backend Flask gọi ReAct đang chạy hoàn toàn synchronous (chặn đứng UI JS trong thời gian chờ vòng lặp While hoàn tất).
  2. Dữ liệu Observation về số tiền trả dưới định dạng string hỗn tạp: `Tổng chi là: 50.000 VND`. Front-end JS không thể cast kiểu dữ liệu để vẽ đường tròn.
- **Solution**: 
  1. Bổ sung `typing indicator` (dấu 3 chấm nảy) bằng JS animation để báo hiệu server đang suy luận.
  2. Bổ sung Regex Parse số học bằng `re.findall(r"[\d.]+")` ở Flask route `/stats` để ép chuỗi ra duy nhất kiểu float trước khi pass xuống Frontend vẽ SVG.

---

## III. Personal Insights: Chatbot vs ReAct (10 Points)

1. **Reasoning**: UI Chatbot chỉ cần chờ API nhả đáp án ra rồi in, UI Agent đòi hỏi người Front-end hiểu rằng hệ thống bên dưới đang chạy rất nhiều tác vụ song song, sinh ra khái niệm thời gian phản hồi (TTFT) bất định.
2. **Reliability**: Sự tin cậy đến từ Telemetry (việc thống kê và ghi sổ được số Token sử dụng - ở mức 1500 tokens / 1 yêu cầu) giúp kiểm soát hoàn toàn ngân sách server OpenAI — Nếu là một PM, điều kiện kiên quyết là tính toán giá vận hành bằng Metric này.
3. **Observation**: Người dùng sẽ thấy Agent giống một nhân viên "Ngân hàng thật thụ" do nó tích hợp Web App để trực quan hoá được dữ liệu thay vì nói suông.

---

## IV. Future Improvements (5 Points)

- **Scalability**: Chuyển API HTTP Fetch thành WebSockets để thực hiện Streaming Real-time Token Output trên màn hình theo phong cách ChatGPT, đồng thời in cả các suy nghĩ "Thinking..." ở khung chat thay vì giấu ở server terminal.
- **Safety**: CORS và API Rate Limiting (Token Bucket) ở tầng Flask để ngăn chặn IP spam bot ngốn tiền API OpenAI của nhà phát hành.
- **Performance**: Phân tải bằng Asynchronous Celery, các tool thu thập Data nặng để vẽ biểu đồ sẽ caching Redis thay vì cứ mỗi tin nhắn lại đọc đi đọc lại file CSV `_read_db()`.
