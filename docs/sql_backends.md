# Bản dịch
- Đây là bản dịch file: [sql_banckends.md](https://github.com/stackql/stackql/blob/main/docs/sql_backends.md) thuộc dự án [StackQL](https://github.com/stackql/stackql).
- Mục đích: Hỗ trợ người Việt Nam có thể hiểu rõ hơn về StackQL.
- Tài liệu chỉ mang tính chất tham khảo, có thể sai sót trong quá trình biên dịch, rất mong được góp ý để có thể cả thiện hơn.


# Các Hệ Thống SQL Backend

Đối với các `stackql`, một `SQL backend` là một lớp trừu tượng thực thi các câu lệnh SQL. Khối lượng công việc chủ yếu nhẹ đối với các thao tác cập nhật và thậm chí còn nhẹ hơn đối với xung đột đọc-ghi.

Hiện tại, các hệ thống RDBMS truyền thống như  `sqlite` và `postgres` đang được sử dụng để triển khai các SQL backend. Tuy nhiên, không có lý do gì mà một hệ thống phi truyền thống, được tối ưu hóa cho phân tích, lại không thể được sử dụng — ít nhất là một phần.

## Các Hệ Thống Backend

### SQLite

Việc triển khai mặc định là SQLite **nhúng**.  SQLite **không** có giao thức wire hoặc phiên bản gốc TCP.

### Postgres

#### Postgres qua TCP

- [Sử dụng giao diện driver SQL của Golang](https://github.com/jackc/pgx/wiki/Getting-started-with-pgx-through-database-sql#hello-world-from-postgresql).
- [PGX native (hiệu năng cải thiện)](https://github.com/jackc/pgx/wiki/Getting-started-with-pgx).

#### Postgres nhúng

https://github.com/fergusstrange/embedded-postgres

#### Lỗi tích hợp Postgres

Danh sách các bài kiểm thử thất bại với Postgres:

- [ ] Google IAM Policy Agg                                                 
- [ ] Google Join kết hợp biểu thức Select có nối chuỗi              
- [ ] Okta Users Select đơn giản có phân trang                                    
- [ ] AWS EC2 Volumes Select đơn giản                                         
- [ ] GitHub SAML Identities Select sử dụng GraphQL                                 
- [ ] GitHub Repository With Functions Select                               
- [ ] Join GCP Okta giữa các nhà cung cấp có từ khóa phụ thuộc JSON trong tên bảng
- [ ] K8S Nodes Select sử dụng JSON Path                                 
- [ ] Data Flow Sequential Join Select With Functions Github                       
- [ ] Shell Session đơn giản                                                  
- [ ] PG Session Postgres Client truy vấn có kiểu dữ liệu                              
- [ ] PG Session Postgres Client V2 truy vấn có kiêir dữ liệu                       

## Ghi chú kỹ thuật

### Trình điều khiển SQL cho Golang

Cách sử dụng các trình điều khiển:

- https://go.dev/doc/database/open-handle

Danh sách các trình điều khiển:

- https://github.com/golang/go/wiki/SQLDrivers

### Chuỗi Data Source Name (DSN)

- [DSN cho SQLite theo golang](https://github.com/mattn/go-sqlite3#dsn-examples).
- [Chuỗi kết nối Postgres URI](https://www.postgresql.org/docs/current/libpq-connect.html#LIBPQ-CONNSTRING).