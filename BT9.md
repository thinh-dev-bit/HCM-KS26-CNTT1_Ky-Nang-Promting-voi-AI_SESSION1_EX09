# Phần 1: Giải thích hình tượng "Chiếc cốc nước" và Hiện tượng Truncation

Để giải thích cho nhân viên CSKH hiểu vì sao AI quên mất thông tin ở Lượt 1, chúng ta sử dụng hình ảnh ẩn dụ **"Chiếc cốc nước"**:

*   **Context Window (Cửa sổ ngữ cảnh) = Chiếc cốc nước có dung tích cố định:** Mỗi mô hình AI đều có một giới hạn bộ nhớ RAM nhất định cho mỗi phiên chat (ví dụ: chứa được tối đa 32.000 Tokens).
*   **Dữ liệu Chat (Tokens) = Những giọt nước:** Mỗi khi khách hàng đặt câu hỏi và AI trả lời, bạn đang đổ thêm nước vào cốc.
*   **Quá trình tràn cốc và Hiện tượng Truncation (Cắt gọt dữ liệu cũ):**
    *   Ở **Lượt 1**, thông tin *"Anh Trần Văn B, mã #MTX-9988"* là những giọt nước đầu tiên được rót vào đáy cốc. Lúc này cốc trống 95%, AI ghi nhớ rất tốt.
    *   Trong suốt **40 lượt tiếp theo**, các đoạn tư vấn dông dài về rau củ đã liên tục đổ nước vào cho đến khi chiếc cốc đầy tràn 100% dung tích (chạm ngưỡng giới hạn Token).
    *   Đến **Lượt 42**, khi khách hàng tiếp tục hỏi, hệ thống bắt buộc phải rót thêm nước mới vào. Nguyên lý hoạt động của AI là **Truncation** (Cắt bỏ phần ngọn/đáy): Để có chỗ chứa dữ liệu mới, hệ thống tự động "tràn" hoặc "đẩy văng" những giọt nước cũ nhất nằm ở dưới cùng của chiếc cốc (chính là thông tin Lượt 1) ra ngoài. 
    *   **Hậu quả:** Đoạn văn bản chứa tên *"Trần Văn B"* và mã *"#MTX-9988"* đã bị xóa hoàn toàn khỏi bộ nhớ RAM hiện tại. Do đó, AI rơi vào trạng thái "mất trí nhớ", nhầm lẫn thành "chị Mai" và phải hỏi lại mã đơn hàng.

---

# Phần 2: Phương án quản lý RAM bộ nhớ chống tràn cốc

Để đảm bảo luồng tư vấn CSKH diễn ra mượt mà dù kéo dài hàng trăm lượt, Siêu thị MartX cần áp dụng 2 phương án sau:

## 1. Phương án thao tác người dùng: Kỹ thuật "Chat Summary" (Tóm tắt định kỳ)
Thay vì để phiên chat chạy vô tận đến mức tràn cốc, nhân viên CSKH cần chủ động "chắt lọc" lại nước trong cốc.
*   **Thao tác:** Khi phiên chat kéo dài đến khoảng lượt 25-30 (trước khi cốc đầy), nhân viên gõ lệnh: *"Hãy tóm tắt lại thông tin cốt lõi của khách hàng này (Tên, mã đơn, vấn đề chính) thành 3 gạch đầu dòng ngắn gọn."*
*   **Khởi động lại (Reset):** Sau khi AI xuất ra bản tóm tắt, nhân viên mở ngay một phiên chat mới hoàn toàn (**New Chat** - chiếc cốc nước rỗng 100%), dán 3 gạch đầu dòng đó vào để làm bối cảnh đầu vào, rồi tiếp tục tư vấn. Cách này giải phóng toàn bộ các Tokens "rác" không cần thiết của 30 lượt trước đó.

## 2. Phương án thiết kế hệ thống (Dành cho Kỹ sư IT MartX): Cơ chế "Rolling Memory" (Cửa sổ trượt)
Nếu công ty tích hợp AI qua hệ thống API riêng biệt, đội ngũ lập trình viên cần thiết lập hệ thống tự động gạn lọc dữ liệu trước khi gửi lên API:
*   **Khóa bối cảnh cốt lõi (Pinned Context):** Luôn luôn chèn cố định thông tin định danh (Tên: Trần Văn B, Mã: #MTX-9988) vào đầu mỗi gói tin (System Prompt) gửi lên AI.
*   **Cửa sổ trượt (Sliding Window):** Hệ thống chỉ tự động gửi kèm 5 đến 10 lượt chat gần nhất cho AI đọc thay vì gửi toàn bộ lịch sử 40 lượt chat từ đầu. Các câu hỏi dông dài ở khúc giữa sẽ bị hệ thống tự động lược bỏ. 
*   **Kết quả:** Cốc nước của AI trên hệ thống sẽ không bao giờ bị đầy, tối ưu hóa được chi phí trả tiền API (do không phải gửi đi gửi lại lượng văn bản khổng lồ) mà AI vẫn giữ được độ minh mẫn tuyệt đối.