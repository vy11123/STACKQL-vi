# Bản dịch
- Đây là bản dịch file: [type_system.md](https://github.com/stackql/stackql/blob/main/docs/type_system.md) thuộc dự án [StackQL](https://github.com/stackql/stackql).
- Mục đích: Hỗ trợ người Việt Nam có thể hiểu rõ hơn về StackQL.
- Tài liệu chỉ mang tính chất tham khảo, có thể sai sót trong quá trình biên dịch, rất mong được góp ý để có thể cả thiện hơn.


# Hệ thống kiểu của __`stackql`__ 

Tồn tại một quan hệ giữa kiểu trong tài liệu openapi (tài liệu discovery) và kiểu quan hệ, được gọi là "ánh xạ discovery-relational (DRM)":

$\ R_{drm}: \text{discovery-type} \to \text{relational-type}\ \ \ \ \ (1) $

Ngoài ra, cũng tồn tại quan hệ ánh xạ object-relational "truyền thống":

$\ Q_{orm}: \text{relational-type} \to \text{golang-type}\ \ \ \ \ (2) $

Các quan hệ này được ánh xạ trong:

- [internal/stackql/drm](/internal/stackql/drm).
- [internal/stackql/typing](/internal/stackql/typing).

Trình điều khiển `golang` của `sql` được sử dụng: 

```go
import (
    "database/sql"
)
    

var (
    _ *sql.ColumnType = (*sql.ColumnType)(nil) // This is the golang SQL driver type

)
```