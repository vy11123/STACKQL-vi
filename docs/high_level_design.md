# Bản dịch
- Đây là bản dịch file: [high_level_design.md](https://github.com/stackql/stackql/blob/main/docs/high-level-design.md) thuộc dự án [StackQL](https://github.com/stackql/stackql).
- Mục đích: Hỗ trợ người Việt Nam có thể hiểu rõ hơn về StackQL.
- Tài liệu chỉ mang tính chất tham khảo, có thể sai sót trong quá trình biên dịch, rất mong được góp ý để có thể cả thiện hơn.


# Thiết kế cấp cao/Thiết kế tổng thể

## Kiến trúc

Kiến trúc của `stackql` có thể được xem như một sự kết hợp của các thành phần sau:

1. Một trình phân tích cú pháp và viết lại câu lệnh SQL.
2. Một ORM (Object-Relational Mapping) giữa các API tùy ý và hệ quản trị cơ sở dữ liệu SQL truyền thống. Trên thực tế, tất cả các API hiện tại đều là API HTTP(S).
3. Một bộ lập kế hoạch cho các đồ thị DAG được xác định bởi các lời gọi API và các mối phụ thuộc của chúng.
4. Một bộ thực thi cho mục (3).
5. Một kết nối tới hệ quản trị cơ sở dữ liệu SQL, giúp ứng dụng hỗ trợ đầy đủ các ngữ nghĩa của SQL.

---

`stackql` tổng quát hóa khái niệm về hạ tầng / tài nguyên tính toán thành một hệ phân cấp gồm `provider`, `service`, `resource` có thể được truy vấn bằng cú pháp SQL, cùng với một số thao tác mệnh lệnh không thuộc SQL chuẩn. Về tiềm năng, bất kỳ loại hạ tầng nào — từ tính toán, điều phối, lưu trữ, đến các dịch vụ SAAS, PAAS, v.v. — đều có thể được quản lý bằng  `stackql`, mặc dù mục tiêu chính vẫn là quản lý hạ tầng đám mây.
Các truy vấn đa nhà cung cấp (multi-provider queries) là một tính năng hạng nhất trong `stackql`.

---

Xét theo cách thực thi truy vấn từ dưới lên — bắt đầu từ phần thực thi backend đến xử lý mã nguồn frontend — thiết kế chiến lược của `stackql` là:

  - **Thực thi Backkend** các truy vấn thông qua các giao diện `Primitive` giao diện này bao bọc các thao tác truy cập và thay đổi dữ liệu đối với các API tùy ý. Các `Primitive`có thể hoạt động trên bất kỳ API cụ thể nào, ví dụ: HTTP, SDK, IPC, hoặc giao thức truyền dữ liệu chuyên biệt. Có thể kết hợp đa dạng (ví dụ: một phần là API HTTP, một phần là SDK).
  - Một đối tượng `Plan` bao gồm một [DAG](https://en.wikipedia.org/wiki/Directed_acyclic_graph) có các `Primitive`.  Các `Plan` có thể được tối ưu hóa và lưu vào bộ nhớ đệm theo cách giống như [vitess](https://github.com/vitessio/vitess).  Về mặt logic, một `Plan`, sau khi được khởi tạo sẽ được hoàn thiện theo các giai đoạn tuần tự sau:
    1. **Sinh mã trung gian (Intermediate Code Generation)**; `//TODO` hiện tại chưa có ngôn ngữ chính thức nào được định nghĩa. Đơn giản chỉ là các đối tượng và con trỏ hàm của `stackql`, được đóng gói trong các `Primitives`.
    2. **Tối ưu hóa mã (Code Optimization)**; thực hiện song song các thao tác độc lập, loại bỏ các thao tác dư thừa.
    3. **Sinh mã cuối cùng (Code Generation)**; thực hiện các lời gọi cuối cùng tới backend bất kỳ, ví dụ như API HTTP.
  - **Phân tích ngữ nghĩa (Semantic Analysis)** của truy vấn là một giai đoạn nhận đầu vào là AST (cây cú pháp trừu tượng) và:
    - tạo bảng ký hiệu (symbol table).
    - phân tích hệ phân cấp của các nhà cung cấp và các API cần thiết để hoàn thành truy vấn. Thông thường, các thông tin này được lấy bằng cách tải xuống và lưu vào bộ nhớ đệm các tài liệu khám phá của nhà cung cấp.
    - thực hiện kiểm tra kiểu dữ liệu, phân tích phạm vi (nhãn).
    - tạo một đối tượng `Planbuilder` và bổ sung thông tin cho nó trong quá trình phân tích.
    - **có thể** sinh ra một số primitives.
    - - ít nhất sẽ sinh ra một bản nháp `Plan`.
  - **Phân tích từ vựng và cú pháp (Lexical and Syntax analysis)**; sử dụng công cụ từ Vitess, vốn là ngữ pháp kiểu lex / yacc, được xử lý bằng các thư viện golang để mô phỏng lex và yacc. Mô-đun [sqlparser](https://github.com/stackql/stackql-parser/tree/main/go/vt/sqlparser) ban đầu từ [vitess](https://github.com/vitessio/vitess) chứa phần triển khai. Kết quả đầu ra là một AST (cây cú pháp trừu tượng).

Phân tích ngữ nghĩa và các giai đoạn sau đó đều nhạy cảm với loại và cấu trúc của các backend nhà cung cấp.

---

## Các thành phần

Ở cấp độ tổng quát, mối quan hệ giữa các thành phần được thể hiện như trong Hình A1.
Để đơn giản, biểu đồ đã được lược bớt và nhiều chi tiết bị bỏ qua.
Thông tin kết nối cơ sở dữ liệu được khởi tạo từ ngữ cảnh do người dùng cung cấp, sau đó được gắn vào `HandlerContext`, và được truyền đi khi cần thiết.

![Kiến trúc thành phần cấp cao](/docs/diagrams/plantuml/hld-components.png)

**Hình A1**: Kiến trúc thành phần cấp cao

## Cơ sở dữ liệu và bảng

Ứng dụng sử dụng hệ quản trị cơ sở dữ liệu quan hệ (RDBMS) để triển khai các ngữ nghĩa của SQL.
Hiện tại, phiên bản mặc định là một tệp nhị phân `SQLite3` nhúng, được truy cập thông qua `CGO`.
Theo mặc định, phiên bản cơ sở dữ liệu được lưu trong bộ nhớ, nhưng có thể được lưu trữ lâu dài nếu được chỉ định qua các tham số khi chạy.

Đối với mỗi loại phản hồi từ API, một bảng trong cơ sở dữ liệu có thể được tạo ra một cách lười biếng (lazy creation). Việc tạo bảng chủ động (eagerly) sẽ chậm hơn, nhưng không loại trừ khả năng sẽ có các tùy chọn kích hoạt trong tương lai cho các cơ sở dữ liệu lưu trữ lâu dài.  
Tên bảng sử dụng quy tắc phân tách bằng dấu chấm (`.`) bao gồm các phần: nhà cung cấp (provider), dịch vụ (service), tài nguyên (resource), kiểu lược đồ (schema type) và `generation`; phần cuối cùng là một số nguyên dương đại diện cho phiên bản của bảng. Generation hỗ trợ phân tích các bản ghi đã bị loại bỏ hoặc nhiều phiên bản của một API.

Mỗi bảng chứa các cột điều khiển để xác định truy vấn và phiên làm việc mà các bản ghi thuộc về.
Điều này sẽ hỗ trợ kiểm toán, xử lý đồng thời và thu gom rác.

### Thu gom rác (Garbage Collection)

Hiện tại, chức năng thu gom rác chưa được triển khai.
Trong ngắn hạn, sẽ có tùy chọn thu gom thủ công toàn bộ.

**Bảng GC1**: Đề xuất thời gian tồn tại của các đối tượng trong cơ sở dữ liệu.
| Đối tượng | Thời gian tồn tại tối thiểu đề xuất | Thời gian tồn tại tối đa đề xuất |
| --- | ----------- | ----------- |
| Cơ sở dữ liệu |  |  |
| Bảng | Giống như thời gian tồn tại của tài liệu API | Giống như thời gian tồn tại của tài liệu API |
| Bản ghi | Truy vấn hoặc phiên làm việc (phiên có thể hữu ích cho SELECT) | Phiên làm việc hoặc một giới hạn tùy ý, theo khả năng tốt nhất |