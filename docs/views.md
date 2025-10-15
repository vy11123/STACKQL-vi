# Bản dịch
- Đây là bản dịch file: [views.md](https://github.com/stackql/stackql/blob/main/docs/views.md) thuộc dự án [StackQL](https://github.com/stackql/stackql).
- Mục đích: Hỗ trợ người Việt Nam có thể hiểu rõ hơn về StackQL.
- Tài liệu chỉ mang tính chất tham khảo, có thể sai sót trong quá trình biên dịch, rất mong được góp ý để có thể cả thiện hơn.


# Các View

## *a priori (trước khi thực thi)* 

Tại thời điểm định nghĩa, có thể xác định rõ:

- Các hoán vị (lưu ý số nhiều) có thể có của các tham số bắt buộc để hỗ trợ thực thi.
- Các tham số tùy chọn.
- Lược đồ của view:
    - Lược đồ `openapi`.
    - Lược đồ quan hệ (relational schema).

## Thời gian chạy

Biểu diễn của view tại thời gian chạy phải hỗ trợ:

- View có thể được đặt bí danh giống như bảng.
- Các cột trong view có thể được đặt bí danh giống như các cột trong bảng (ngay cả và **đặc biệt** là những cột đã được đặt bí danh bên trong chính view đó).

## Ý tưởng

- Các câu lệnh DDL của view trong StackQL được lưu trong một bảng đặc biệt dành riêng cho mục đích này.
    - Tên bảng vật lý ví dụ như `__iql__.views`.
    - Các view không cần tồn tại cho đến khi phần `SELECT ... FROM <view>` - của truy vấn được thực thi.
      Điều này có lợi trên các hệ thống RDBMS nơi việc tạo view sẽ thất bại nếu các bảng vật lý chưa tồn tại.
    - Có thể cần một lớp gián tiếp để thực thi view, liên quan đến tên bảng chứa ID thế hệ.
      Lựa chọn đơn giản nhất là sử dụng tên bảng đầu vào.
- Các định nghĩa view SQL (được chuyển thành bảng vật lý) được lưu trong RDBMS.
    - Điều này ngụ ý rằng ngay từ giai đoạn phân tích ban đầu, cần xác định rõ rằng một view đang được tham chiếu.
    - Một phần của không gian tên cần được dành riêng cho các view này; có thể cấu hình bằng regex / mẫu không gian tên hiện có?
    - Có khả năng cần một hoặc nhiều đối tượng chuyên biệt hoặc phần mở rộng của giao diện `table` để phục vụ phân tích view và định tuyến tham số.
- Khi phân tích hoàn tất:
    - Quá trình thu thập dữ liệu diễn ra như bình thường thông qua DAG nguyên thủy.
    - Giai đoạn lựa chọn sử dụng các view vật lý.


## Truy vấn con (Subqueries)

Một số khía cạnh của việc phân tích và thực thi truy vấn con sẽ giống với view, nhưng không phải tất cả. Những yếu tố cần cân nhắc khi triển khai view trong ngắn hạn để việc triển khai truy vấn con sau này được nhanh chóng và tự nhiên:

Còn tiếp tục...