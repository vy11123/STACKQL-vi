# Bản dịch
- Đây là bản dịch file: [dashboards.md](https://github.com/stackql/stackql/blob/main/docs/dashboards.md) thuộc dự án [StackQL](https://github.com/stackql/stackql).
- Mục đích: Hỗ trợ người Việt Nam có thể hiểu rõ hơn về StackQL.
- Tài liệu chỉ mang tính chất tham khảo, có thể sai sót trong quá trình biên dịch, rất mong được góp ý để có thể cả thiện hơn.


## Các đặc điểm phần mềm bảng điều khiển (Dashboard)

- Phần mềm bảng điều khiển có thể truy vấn các view. 
- Các bảng và view cần được định nghĩa trước để truy vấn hoạt động.
  Điều này ngụ ý rằng cần nạp trước (AOT) các bảng. Dưới đây là một số lựa chọn:
    - ~~(a) Dành riêng một không gian tên cho các truy vấn như vậy. Với bất kỳ truy vấn nào trong không gian tên đó, thực hiện tạo bảng và view trước rồi mới tiếp tục.~~ **Cách này, tự nó, sẽ KHÔNG hoạt động để tạo bảng điều khiển vì phần mềm bảng điều khiển sẽ không biết đến các đối tượng bảng / view cần thiết.**
    - (b) Nạp trước tất cả bảng và view khi tải về nhà cung cấp. Bề ngoài có vẻ rất tốn kém.
    - (c) **Hack (Cách lách)**: Chạy các truy vấn tùy ý (hoặc theo lô) để nạp trước từ bên trong bảng điều khiển (hoặc `stackql`) như một hoạt động ngoại tuyến của quản trị viên.
- Thực hiện (b) hoặc (c) sẽ yêu cầu `stackql` hỗ trợ truy vấn con một cách nguyên bản.
- Có thể xem xét cú pháp `precharge <table_name_list> ;` để hỗ trợ.


## Luồng dữ liệu (Data flow)

1. Quản trị viên (hoặc hành động tự động-automated) thực hiện nạp trước bảng.
1. - Người dùng quản trị nhập truy vấn `stackql` để nạp trước bảng.
2. `stackql` thực thi truy vấn và trả về kết quả.