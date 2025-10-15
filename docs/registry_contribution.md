# Bản dịch
- Đây là bản dịch file: [registry_contribution.md](https://github.com/stackql/stackql/blob/main/docs/registry_contribution.md) thuộc dự án [StackQL](https://github.com/stackql/stackql).
- Mục đích: Hỗ trợ người Việt Nam có thể hiểu rõ hơn về StackQL.
- Tài liệu chỉ mang tính chất tham khảo, có thể sai sót trong quá trình biên dịch, rất mong được góp ý để có thể cả thiện hơn.


# Đóng góp vào Registry Nhà cung cấp (Provider Registry)

Việc cộng đồng cùng phát triển chức năng nhà cung cấp (provider) là yếu tố then chốt đối với `stackql` và sứ mệnh của chúng tôi.
Tại đây, chúng tôi mô tả quy trình để phát triển, kiểm thử và tích hợp một nhà cung cấp mới — từ con số không đến khi sẵn sàng phát hành chính thức (General Availability).

Chúng tôi bao gồm một ví dụ tích hợp cho một nhà cung cấp đơn giản, `publicapis`.

## Quy trình làm việc

1. Tạo tài liệu định nghĩa nhà cung cấp (Provider definition document).
2. Tạo ít nhất một tài liệu định nghĩa dịch vụ (Service definition document).
3. Chạy `stackql` với cấu hình hỗ trợ phát triển nhà cung cấp cục bộ.
4. Lặp lại và xác minh rằng `stackql` - hoạt động như mong đợi với nhà cung cấp mới của bạn.
5. Gửi Pull Request vào kho lưu trữ Provider Registry.


### 1. Tạo tài liệu định nghĩa nhà cung cấp (Provider).

Đây là một tài liệu định dạng YAML hoặc JSON, chứa siêu dữ liệu của nhà cung cấp và các tham chiếu đến tài liệu dịch vụ (Service) thông qua đó nhà cung cấp sẽ cung cấp chức năng.

Ví dụ theo [docs/examples/registry/src/publicapis/v1/provider.yaml](/docs/examples/registry/src/publicapis/v1/provider.yaml).

### 2. Tạo ít nhất một tài liệu định nghĩa dịch vụ (Service).

Đây là một tài liệu đặc tả [OpenAPI](https://swagger.io/specification/) ở định dạng YAML hoặc JSON, **kèm theo** một chú thích hợp lệ `components.x-stackQL-resources`, dùng để định nghĩa phần Tài nguyên (Resource) trong hệ phân cấp của `stackql`.

Ví dụ theo [docs/examples/registry/src/publicapis/v1/services/api-v1.yaml](/docs/examples/registry/src/publicapis/v1/services/api-v1.yaml).


### 3. Chạy `stackql` với cấu hình hỗ trợ phát triển nhà cung cấp cục bộ

Cấu hình `stackql` để sử dụng tài liệu của bạn thông qua tham số dòng lệnh `--registry`.  Phần [ví dụ tích hợp sẽ trình bày chi tiết hơn](#Configuring-StackQL-to-consume-a-local-development-registry) expands upon this.

### 4. Lặp lại và xác minh rằng `stackql` hoạt động như mong đợi với nhà cung cấp mới của bạn.

Lặp lại các bước 2 và 3 cho đến khi phạm vi API và chức năng đáp ứng đầy đủ yêu cầu của bạn.

### 5. Gửi Pull Request vào kho lưu trữ Provider Registry

Theo [hướng dẫn đóng góp vào Provider Registry](https://github.com/stackql/stackql-provider-registry/blob/main/.github/CONTRIBUTING.md).  Nhóm phát triển sẽ xem xét nhanh nhất có thể và phối hợp với bạn để hoàn tất việc tích hợp. Sau khi hoàn tất, chức năng của bạn sẽ khả dụng thông qua lệnh `registry pull...`

## Ví dụ tích hợp

Trong phần hướng dẫn này, chúng tôi trình bày những bước cơ bản để tích hợp một nhà cung cấp đơn giản, `publicapis`, vốn đã được đưa vào hệ thống quản lý mã nguồn tại đây.

Các giả định cho phần hướng dẫn này bao gồm:

1. - Bạn đã xây dựng (ví dụ: [the readme](/README.md#build)) hoặc sao chép tệp thực thi `stackql` - phù hợp với nền tảng của bạn vào thư mục [build](/build).
2. - Thư mục làm việc hiện tại trùng với vị trí của tài liệu này, nếu không thì các đường dẫn tương đối theo `pwd` sẽ cần được điều chỉnh.

### Cấu hình StackQL để sử dụng registry phát triển cục bộ

Cấu hình truy cập registry:

```bash
PROVIDER_REGISTRY_ROOT_DIR="$(pwd)/../docs/examples/registry"

REG_STR='{ "url": "file://'${PROVIDER_REGISTRY_ROOT_DIR}'", "localDocRoot": "'${PROVIDER_REGISTRY_ROOT_DIR}'",  "verifyConfig": {"nopVerify": true } }'
```

Cấu hình xác thực nhà cung cấp theo [hướng dẫn dành cho nhà phát triển:](/docs/developer_guide.md#provider-authentication):

```bash
AUTH_STR='{ "publicapis": { "type": "null_auth" }  }'
```

### Tương tác với registry phát triển cục bộ

```bash
$(pwd)/../build/stackql --auth="${AUTH_STR}" --registry="${REG_STR}" exec "select API from publicapis.api.apis where API like 'Dog%' limit 10;"

$(pwd)/../build/stackql --auth="${AUTH_STR}" --registry="${REG_STR}" exec "select API from publicapis.api.random where title =  'Dog';"

## The single, anonmyous column returned from selecing an array of strings is a core issue to fix separate to this
$(pwd)/../build/stackql --auth="${AUTH_STR}" --registry="${REG_STR}" exec "select * from publicapis.api.categories limit 5;"
```