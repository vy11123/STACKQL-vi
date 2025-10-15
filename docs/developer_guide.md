# Bản dịch
- Đây là bản dịch file: [developer_guide.md](https://github.com/stackql/stackql/blob/main/docs/developer_guide.md) thuộc dự án [StackQL](https://github.com/stackql/stackql).
- Mục đích: Hỗ trợ người Việt Nam có thể hiểu rõ hơn về StackQL.
- Tài liệu chỉ mang tính chất tham khảo, có thể sai sót trong quá trình biên dịch, rất mong được góp ý để có thể cả thiện hơn.


# Hướng dẫn dành cho nhà phát triển (StackQL Developer Guide)

## Đóng góp 

Vui lòng xem [tài liệu đóng góp](/CONTRIBUTING.md).

## CICD (Continuous Integration / Continuous Deployment)

Xem [tài liệu CICD](/docs/CICD.md).

## Khởi đầu nhanh dành cho nhà phát triển

Tóm tắt ngắn gọn là để thực hiện việc build cơ bản và kiểm thử đơn vị, bạn cần những điều sau:

- Cài đặt `golang` trên hệ thống của bạn **nế bạn chưa có phiên bản >= 1.21**, theo [tài liệu `golang`](https://go.dev/doc/install).
- Cài đặt `python` trên hệ thống của bạn **nếu bạn chưa có phiên bản >= 3.11**, có thể tải tài liệu từ [website chính thức của `python`](https://www.python.org/downloads/) hoặc qua nhiều trình quản lí gói.
- Sử dụng `venv` hoặc cách khác, cài đặt các gói python cần thiết, ví dụ: (nếu hệ thống cho phép) từ thư mục gốc của repository: `pip install -r cicd/requirements.txt`.

Sau đó, mỗi lệnh sau nên được chạy từ thư mục gốc của kho mã:

    - Lấy các thư viện `go` cần thiết: `go get ./...`.
    - Chạy kiểm thử đơn vị:  `python cicd/python/build.py --test`.
    - Biên dịch ứng dụng:  `python cicd/python/build.py --build` (lệnh này sẽ tạo ra một tệp thực thi tại `./build/stackql`).


Đối với phát triển nghiêm túc, các kiểm thử tích hợp mô phỏng là rất cần thiết. Vì vậy, sẽ có thêm các phụ thuộc sau:

- Cài đặt `psql`.  - Trên một số hệ thống, bạn có thể chỉ cần cài đặt phần client và/hoặc sử dụng các trình quản lý gói khác nhau; nếu không được, bạn có thể [cài đặt PostgreSQL thủ công](https://www.postgresql.org/download/).

Sau khi đã cài đặt tất cả các phụ thuộc, các bài kiểm thử `robot` nên được chạy từ thư mục gốc của kho mã (dựa vào tệp thực thi tại `./build/stackql` đã được biên dịch ở bước trước):
    - Chạy kiểm thử chức năng mô phỏng: `python cicd/python/build.py --robot-test`.  - Lệnh này sẽ đưa tệp thực thi vào quy trình kiểm thử tự động.

Chạy kiểm tra linting cục bộ cũng là một điều hữu ích, và có thể thực hiện như sau:

- Cài đặt `golangci-lint` phiên bản `v1.59.1` (phiên bản hiện đang được sử dụng trong CI), theo  [tài liệu của `golangci-lint`](https://golangci-lint.run/welcome/install/).
- - Từ thư mục gốc của kho mã: `golangci-lint run`.


## xml

Khó khăn vốn có trong việc tuần tự hóa `xml` một cách tổng quát đã được cộng đồng phát triển golang diễn đạt rõ ràng trong [tài liệu `encoding/xml`](https://pkg.go.dev/encoding/xml#pkg-note-BUG):

> Việc ánh xạ giữa các phần tử XML và cấu trúc dữ liệu vốn dĩ có khiếm khuyết: một phần tử XML là tập hợp các giá trị ẩn danh phụ thuộc vào thứ tự, trong khi cấu trúc dữ liệu là tập hợp các giá trị có tên không phụ thuộc vào thứ tự. Xem [encoding/json](https://pkg.go.dev/encoding/json) để biết định dạng văn bản phù hợp hơn với cấu trúc dữ liệu.

Hiện tại, `stackql` xử lý việc tuần tự hóa và giải tuần tự hóa `xml` (SERDE) thông qua phần lõi, và không chuyển hướng việc này sang các SDK. Tùy theo mức độ ưu tiên, điều này có thể được xem xét lại *một cách cẩn trọng*.

## Biên dịch cục bộ

```bash
env CGO_ENABLED=1 PLANCACHEENABLED=false go build \
  --tags "sqlite_stackql" \
  -ldflags "-X github.com/stackql/stackql/internal/stackql/cmd.BuildMajorVersion=${BUILDMAJORVERSION:-1} \
  -X github.com/stackql/stackql/internal/stackql/cmd.BuildMinorVersion=${BUILDMINORVERSION:-1} \
  -X github.com/stackql/stackql/internal/stackql/cmd.BuildPatchVersion=${BUILDPATCHVERSION:-1} \
  -X github.com/stackql/stackql/internal/stackql/cmd.BuildCommitSHA=$BUILDCOMMITSHA \
  -X github.com/stackql/stackql/internal/stackql/cmd.BuildShortCommitSHA=$BUILDSHORTCOMMITSHA \
  -X \"github.com/stackql/stackql/internal/stackql/cmd.BuildDate=$BUILDDATE\" \
  -X \"stackql/internal/stackql/planbuilder.PlanCacheEnabled=$PLANCACHEENABLED\" \
  -X github.com/stackql/stackql/internal/stackql/cmd.BuildPlatform=$BUILDPLATFORM" -o ./build ./stackql
```

## Kiểm thử cục bộ

### Kiểm thử đơn vị

Hiện tại, chúng tôi không quá cứng nhắc về cách triển khai kiểm thử đơn vị. Về mặt định hướng, các bài kiểm thử đơn vị có thể được triển khai tương tự như [tài liệu chính thức về gói kiểm thử](https://pkg.go.dev/testing), đặc biệt là [phần tổng quan](https://pkg.go.dev/testing#pkg-overview).

Để chạy tất cả các bài kiểm thử đơn vị:

```bash
go test -timeout 1200s --tags "sqlite_stackql" ./...
```

### Kiểm thử Robot

**Lưu ý**: bước này yêu cầu bản build cục bộ (ở trên) đã được thực hiện thành công, tạo ra tệp thực thi tại `./build/`.

```bash
env PYTHONPATH="$PYTHONPATH:$(pwd)/test/python" robot -d test/robot/reports test/robot/functional
```

Hoặc tốt hơn, nếu bạn có Docker Desktop và ảnh `postgres` được đề cập trong các tệp docker-compose:

```bash
robot --variable SHOULD_RUN_DOCKER_EXTERNAL_TESTS:true -d test/robot/functional test/robot/functional
```

### Kiểm thử thủ công

Vui lòng xem [tài liệu kiểm thử mô phỏng](/test/python/stackql_test_tooling/flask/README.md).


## Trình gỡ lỗi (Debuggers)

Cấu hình công cụ `vscode` hầu như đã sẵn sàng để sử dụng, như được thấy trong thư mục `.vscode`. Bạn sẽ cần tạo một tệp tại vị trí đã bị `.gitignore` là `.vscode/.env`.  Cách đơn giản nhất là sao chép tệp ví dụ để bắt đầu: `cp .vscode/example.env .vscode/.env`.

Cấu hình trình gỡ lỗi hiện tại khá lộn xộn, và có lẽ theo thời gian chúng tôi sẽ tinh gọn nó. Tuy nhiên, nó vẫn là một ví dụ hữu ích.

## Phát triển nhà cung cấp (Provider)

Bạn muốn mở rộng thêm chức năng mới thông qua `stackql`?  Chúng tôi rất hoan nghênh điều đó! 

Vui lòng xem [registry_contribution.md](/docs/registry_contribution.md).

## Xác thực nhà cung cấp (Provider Authentication)

Ở giai đoạn này, cấu hình xác thực cần được chỉ định cho từng nhà cung cấp, kể cả những nhà cung cấp không yêu cầu xác thực. Các loại xác thực được hỗ trợ gồm:

- `api_key`.
- `basic`.
- `interactive` dành cho oAuth tương tác, hiện tại chỉ hỗ trợ Google thông qua công cụ dòng lệnh `gcloud`.
- `service_account` dành cho khóa riêng dạng json (ví dụ: tài khoản dịch vụ của Google).
- `null_auth` dành cho các nhà cung cấp không yêu cầu xác thực.

Nếu bạn muốn thêm các loại xác thực khác hoặc phát hiện lỗi, vui lòng tạo một issue.

Các ví dụ có thể được tìm thấy tại [đây](/docs/examples/examples.md).


## Chế độ máy chủ (Server mode)

**Lưu ý: tính năng này đang ở giai đoạn alpha.**.  Chúng tôi sẽ cập nhật thời gian phát hành chính thức sau một thời gian phân tích và thử nghiệm. Tại thời điểm hiện tại, chế độ máy chủ chủ yếu hữu ích cho mục đích R&D:
- thực nghiệm.
- tích hợp công cụ / hệ thống và thiết kế liên quan.
- phát triển chính `stackql`.
- phát triển các trường hợp sử dụng cho sản phẩm.

Máy chủ `stackql` sử dụng giao thức truyền dữ liệu của `postgres` và có thể được sử dụng với client `psql`, bao gồm cả xác thực mTLS và mã hóa trong quá trình truyền. Vui lòng xem [các ví dụ liên quan](/docs/examples/examples.md#running-in-server-mode) để biết thêm chi tiết.

## Cân nhắc về đồng thời (Concurrency considerations)

Ở chế độ máy chủ, một nhóm luồng (thread pool) sẽ cấp phát một luồng để xử lý mỗi kết nối.

Các bước sau đây được thực hiện theo luồng đơn (single-threaded):

  - Phân tích từ vựng và cú pháp (Lexical and Syntax Analysis)
  - Phân tích ngữ nghĩa (Semantic Analysis)
  - Thực thi một primitive đơn lẻ không có con
  - Thực thi các primitive a, b trong đó a > b hoặc b < a theo thứ tự một phần của biểu đồ kế hoạch (plan DAG). Mặc dù không nhất thiết phải là cùng một luồng thực thi, chúng sẽ được thực hiện tuần tự nghiêm ngặt.

Các bước sau đây có khả năng được thực hiện theo luồng đa (multi-threaded):

  - Tối ưu hóa kế hoạch (Plan optimization)
  - Thực thi các primitive cùng cấp (sibling primitives)

## Biên dịch lại trình phân tích cú pháp (Rebuilding Parser)

Vui lòng tham khảo [kho mã parser](https://github.com/stackql/stackql-parser).


## Các nâng cấp cần thiết còn tồn đọng (Outstanding required Uplifts)

### Nợ kỹ thuật / lỗi cấp cao (High level Tech debt / bugs for later)

Những vấn đề cấp cao cần xem xét sau:

  - Hệ thống bộ nhớ đệm → cơ sở dữ liệu (redis????)

### Bộ nhớ đệm (Cache)

  - Giới hạn kích thước bộ nhớ đệm và chính sách xoay vòng
  - Định dạng lưu trữ bộ nhớ đệm: từ json đơn giản → cơ sở dữ liệu (redis????)
  - Tái sử dụng bộ nhớ đệm LRU của Vitess???

### Mô hình dữ liệu (Data Model)

  - Cần có góc nhìn hợp lý về bảng / phép nối / hàng dữ liệu
  - Di chuyển phản hồi sang kiểu máy chủ MySQL a la Vitess
  - Các thao tác DML cần sử dụng bộ lọc phản hồi tương tự như các thao tác metadata

### Mô hình thực thi (Execution model)

  - Chế độ lỗi và khả năng xảy ra nhiều lỗi… cần có cách truyền đạt nguyên nhân và trạng thái cuối cùng đến người dùng. Cần một triết lý tổng thể có thể mở rộng cho các giao dịch.
  - Cần có góc nhìn hợp lý về các primitive và tối ưu hóa, sử dụng phương pháp ánh xạ hàm mặc định có thể mở rộng.
  - Thực thi song song các thao tác DML “nguyên tử”.

## Kiểm thử

Thực tế, các tệp GitHub Actions là nguồn tham chiếu chính xác nhất cho quy trình build và kiểm thử, và chúng tôi khuyến khích bạn xem qua chúng. Tuy nhiên, để ngắn gọn, dưới đây là hướng dẫn kiểm thử dành cho nhà phát triển.

Yêu cầu hệ thống được trình bày tại [README gốc](/README.md#system-requirements). 

Kiểm thử ứng dụng cục bộ:

1. Chạy các bài kiểm thử Go `go test --tags "sqlite_stackql" ./...`.
2. Biên dịch tệp thực thi [theo hướng dẫn tại README gốc](/README.md#build)
3. Thực hiện ghi đè registry để mô phỏng: `python3 test/python/stackql_test_tooling/registry_rewrite.py --srcdir "$(pwd)/test/registry/src" --destdir "$(pwd)/test/registry-mocked/src"`.
3. Chạy các bài kiểm thử robot:
    - Kiểm thử chức năng (có thể mô phỏng) `robot -d test/robot/reports test/robot/functional`.
    - Kiểm thử tích hợp `robot -d test/robot/reports test/robot/integration`.  Với các bài kiểm thử này, bạn cần thiết lập các biến môi trường theo cấu hình trong GitHub Actions.
4. Chạy các bài kiểm thử Python thủ công (đã ngừng hỗ trợ):
    - Chuẩn bị: `cp test/db/db.sqlite test/db/tmp/python-tests-tmp-db.sqlite`.
    - Thực thi: `python3 test/deprecated/python/main.py`.

[Bài viết này](https://medium.com/cbi-engineering/mocking-techniques-for-go-805c10f1676b)  cung cấp cái nhìn tổng quan hữu ích về kỹ thuật mô phỏng trong Golang.

### go test

Độ phủ kiểm thử hiện tại còn hạn chế. Các lỗi hồi quy được giảm thiểu thông qua kiểm thử tích hợp bằng `go test` trong các gói [driver](/internal/stackql/driver/driver_integration_test.go) và [stackql](/stackql/main_integration_test.go).  Một số chức năng kiểm thử được hỗ trợ thông qua các tiện ích trong [gói test](/internal/test).

#### Độ phủ kiểm thử gotest tại một thời điểm

Nếu chưa thực hiện, hãy cài đặt công cụ 'cover' bằng lệnh: `go get golang.org/x/tools/cmd/cover`.  
Sau đó chạy: `go test --tags "sqlite_stackql" -cover ../...`.

### Kiểm thử chức năng và tích hợp

Việc kiểm thử chức năng và tích hợp tự động chủ yếu được thực hiện thông qua framework Robot. Vui lòng xem [tài liệu hướng dẫn kiểm thử Robot](/test/robot/README.md).

Có một phần [kiểm thử python thủ công](/test/deprecated/python/main.py) cũ, đã bị ngừng sử dụng, sẽ được chuyển sang dùng Robot Framework và bị loại bỏ.

### Kiểm tra mã (Linting)

Chúng tôi sử dụng `golangci-lint`.

Việc kiểm tra lint cho các tệp Go (và cả các tác vụ GitHub Actions) trong CI được định nghĩa tại [.github/workflows/lint.yml](/.github/workflows/lint.yml).

Để chạy linter trên máy cục bộ, trước tiên hãy đảm bảo bạn đang sử dụng đúng phiên bản `golangci-lint` giống như CI, sau đó có thể:

-  Chạy `golangci-lint run` để hiển thị toàn bộ kết quả ra console, hoặc...
-  Chạy `golangci-lint run > cicd/log/lint.log 2>&1` để ghi toàn bộ đầu ra vào tệp `cicd/log/lint.log` (tính từ thư mục gốc của repository).

## Biên dịch chéo (Cross Compilation) trên máy cục bộ

### Từ máy Mac

Để hỗ trợ biên dịch cho Windows:

```
brew install mingw-w64
```

Để hỗ trợ biên dịch cho Linux:

```
export HOMEBREW_BUILD_FROM_SOURCE=1
brew install FiloSottile/musl-cross/musl-cross
```

## Kiểm thử bản dựng mới nhất từ hệ thống CI

### Trên máy Mac

Tải về và giải nén. Ví dụ, ta sẽ sử dụng tệp thực thi `~/Downloads/stackql`.

Đầu tiên:
```
chmod +x ~/Downloads/stackql
```

Sau đó, trên macOS phiên bản > 10, bạn cần cho phép tệp thực thi này được chạy, vì nó không được ký bởi nhà phát triển đã xác thực. Cách ít gây ảnh hưởng nhất là thử chạy một lệnh (ví dụ bên dưới), rồi mở `System Settings` > `Security & Privacy` sẽ có giao diện cho phép chạy tệp `stackql` không đáng tin cậy.  Cách này hoạt động ít nhất trên High Sierra `v1.2.1`.

Sau đó, chạy các lệnh kiểm thử, ví dụ:
```
~/Downloads/stackql --credentialsfilepath=$HOME/stackql/stackql-devel/cicd/keys/sa-key.json exec "select group_concat(substr(name, 0, 5)) || ' lalala' as cc from google.compute.disks where project = 'lab-kr-network-01' and zone = 'australia-southeast1-b';" -o text
```

## Phân tích hiệu năng (Profiling)


```
time ./stackql exec --cpuprofile=./select-disks-improved-05.profile --auth='{ "google": { "credentialsfilepath": "'${HOME}'/stackql/stackql-devel/cicd/keys/sa-key.json" }, "okta": { "credentialsfilepath": "'${HOME}'/stackql/stackql-devel/cicd/keys/okta-token.txt", "type": "api_key" } } ' "select name from google.compute.disks where project = 'lab-kr-network-01' and zone = 'australia-southeast1-a';"
```


## Ký yêu cầu HTTP của AWS

https://docs.aws.amazon.com/sdk-for-go/api/aws/signer/v4/

## Truy vấn dữ liệu từ các câu lệnh DML không phải SELECT

Câu lệnh `INSERT RETURNING` có thể hoạt động theo hai cơ chế:

- Phản hồi đồng bộ, chẳng hạn như [`google.storage.buckets`](https://cloud.google.com/storage/docs/json_api/v1/buckets/insert).  Mệnh đề returning là một phép chiếu lên phần thân phản hồi có sẵn ngay lập tức.
- - Phản hồi bất đồng bộ, chẳng hạn như [`google.compute.instances`](https://cloud.google.com/compute/docs/reference/rest/v1/instances/insert) và [`google.compute.networks`](https://cloud.google.com/compute/docs/reference/rest/v1/networks/insert).  - Mệnh đề returning là một phép chiếu lên phần thân phản hồi **sau khi** quá trình chờ hoàn tất.

Các trường hợp sử dụng trong tương lai cho `UPDATE RETURNING`, `REPLACE RETURNING` và `DELETE RETURNING` sẽ hoạt động theo cách quan sát tương tự.