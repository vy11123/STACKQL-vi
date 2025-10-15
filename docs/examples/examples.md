# Bản dịch
- Đây là bản dịch file: [examples.md](https://github.com/stackql/stackql/blob/main/docs/examples/examples.md) thuộc dự án [StackQL](https://github.com/stackql/stackql).
- Mục đích: Hỗ trợ người Việt Nam có thể hiểu rõ hơn về StackQL.
- Tài liệu chỉ mang tính chất tham khảo, có thể sai sót trong quá trình biên dịch, rất mong được góp ý để có thể cả thiện hơn.

# Ví dụ

Thư mục này chứa các ví dụ về:

- [Việc sử dụng trực tiếp stackql trong thư mục con stackql](/docs/examples/stackql)
- Các ví dụ về registry:
    - [Registry trống](/docs/examples/empty-registry).
    - [Một triển khai tham chiếu của registry đơn giản](/docs/examples/registry).
- Các ví dụ về việc [gọi stackql bằng các công cụ khác nhau trong thư mục scripts](/docs/examples/scripts).

## Giả định cho các ví dụ gọi trực tiếp stackql

- `stackql` đã được thêm vào `${PATH}` của bạn.
- Thông tin xác thực được cung cấp dưới dạng chuỗi JSON thông qua đối số `--auth`.  Với mỗi nhà cung cấp, bạn cung cấp một cặp khóa/giá trị. Giá trị là một chuỗi JSON, có thể chỉ định `type` (mặc định là `service_account`, đại diện cho khóa tài khoản dịch vụ của Google). Giá trị này tối thiểu phải chứa một trong hai cách sau:
    - Một tệp khóa hợp lệ tại vị trí tệp `{ "credentialsfilepath": "/PATH/TO/KEY/FILE" }`.  Ví dụ, với nhà cung cấp Google, bạn có thể sử dụng tệp JSON khóa tài khoản dịch vụ.
    - Một chuỗi khóa hợp lệ được lưu trong biến môi trường (đã được export). Ví dụ: `{ "credentialsenvvar": "OKTA_SECRET_KEY" }`.  Với nhà cung cấp Google, bạn cũng có thể sử dụng khóa tài khoản dịch vụ theo cách này.

Nếu sử dụng xác thực kiểu `service account` với nhà cung cấp `google`, thì không cần thêm thông tin phụ trợ nào. Tuy nhiên, nếu bạn sử dụng kiểu khóa khác hoặc nhà cung cấp khác, thì sẽ cần thêm thông tin tại thời gian chạy, ví dụ như: (còn tiếp tục...)

## Chạy stackql

Ví dụ đơn giản nhất là sử dụng shell tương tác.

Google:

```sh

export OKTA_SECRET_KEY="$(cat ${HOME}/stackql/stackql-devel/cicd/keys/okta-token.txt)"

export AUTH_STR='{ "google": { "credentialsfilepath": "'${HOME}'/stackql/stackql-devel/cicd/keys/sa-key.json", "type": "service_account" }, "okta": { "credentialsenvvar": "OKTA_SECRET_KEY", "type": "api_key" } }'

./stackql shell --auth="${AUTH_STR}"


```

## Truy vấn

### SELECT

```
stackql \
  --auth="${AUTH_STR}" exec  \
  "select * from google.compute.instances WHERE zone = '${YOUR_GOOGLE_ZONE}' AND project = '${YOUR_GOOGLE_PROJECT}' ;" ; echo

```

Hoặc...

```
stackql \
  --auth="${AUTH_STR}" exec  \
  "select selfLink, projectNumber from google.storage.buckets WHERE location = '${YOUR_GOOGLE_ZONE}' AND project = '${YOUR_GOOGLE_PROJECT}' ;" ; echo

```

Cho ví dụ:
```sql
select d1.name, d1.id from google.compute.disks d1 where d1.project = 'lab-kr-network-01' and d1.zone = 'australia-southeast1-a' ;
```

### Joins

- Vui lòng [xem ví dụ self join tại đây](/docs/examples/stackql/self-join.sql).
- Vui lòng [xem ví dụ three way join tại đây](/docs/examples/stackql/three-way-join.sql).
- Vui lòng [xem ví dụ cross-provider join tại đây](/docs/examples/stackql/cross-provider-join.sql).

### Lệnh SHOW SERVICES

```
stackql --approot=../test/.stackql \
  --configfile=../test/.stackqlrc exec \
  "SHOW SERVICES from google ;" ; echo

```

### Lệnh INSERT phức tạp

```
insert into google.compute.disks(project, zone, data__name) SELECT 'lab-kr-network-01', 'australia-southeast1-a', name || '-new-disk01' as name from google.compute.disks where project = 'lab-kr-network-01' and zone =  'australia-southeast1-a' limit 2;
```

## Ví dụ truy vấn Okta

### Chèn ứng dụng (app insert)

```
insert into okta.application.apps(subdomain, data__name, data__label, data__signOnMode, data__settings) SELECT 'dev-79923018-admin', 'template_basic_auth', 'some other4 new app', 'BASIC_AUTH', '{ "app": { "authURL": "https://example.com/auth.html", "url": "https://example.com/bookmark.html" } }';
```

### Truy vấn bảng có bí danh

```
select * from okta.application.apps;
```

## Chạy ở chế độ máy chủ

**Lưu ý rằng tính năng này đang ở giai đoạn alpha**, như đã đề cập trong [hướng dẫn dành cho nhà phát triển.](/docs/developer_guide.md#server-mode).


Để chạy máy chủ `stackql` qua giao thức wire của `postgres` (không cần xác thực client), từ thư mục `build`.

```bash
./stackql --auth="${AUTH_STR}" --registry="${REG_STR}" srv
```

Sau đó, sử dụng client `psql`:

```bash
psql -d "host=127.0.0.1 port=5466 user=silly dbname=silly"
```

Để chạy với xác thực mTLS, trước tiên hãy chuẩn bị các tệp cần thiết theo [README thiết lập mTLS](/test/server/mtls/README.md).  Quan trọng: cần định nghĩa biến môi trường `CLIENT_CERT` trong phiên shell bạn sẽ dùng để chạy máy chủ.

Sau đó:

```bash
STACKQL_SRV_TLS_CFG='{ "keyFilePath": "../test/server/mtls/credentials/pg_server_key.pem", "certFilePath": "../test/server/mtls/credentials/pg_server_cert.pem", "clientCAs": [ "'${CLIENT_CERT}'" ] }'

./stackql --auth="${AUTH_STR}" --registry="${REG_STR}"  --pg.tls="${STACKQL_SRV_TLS_CFG}" 
```

Và sau đó, sử dụng client `psql` (từ cùng thư mục; `build`):

```bash
psql -d "host=127.0.0.1 port=5466 user=silly dbname=silly sslmode=verify-full sslcert=../test/server/mtls/credentials/pg_client_cert.pem sslkey=../test/server/mtls/credentials/pg_client_key.pem sslrootcert=../test/server/mtls/credentials/pg_server_cert.pem"
```

### Truy cập từ python

- Truy cập từ Python yêu cầu một máy chủ đang chạy, đơn giản nhất là dùng `stackql srv, sẽ phục vụ trên cổng mặc định mà không cần xác thực.
- Đối với kiểm thử tích hợp, chúng tôi sử dụng  `psycopg` hiện tại là phiên bản `v3`.
    - Chạy / điều chỉnh [script này](/docs/examples/scripts/python/psycopg3_scratchpad.py) để xử lý sự cố với `psycopg` `v3`. 
- `superset` sử dụng `sqlalchemy` vốn dùng `psycopg2` **lưu ý phiên bản khác biệt**.
    - Chạy / điều chỉnh [script này](/docs/examples/scripts/python/psycopg2_scratchpad.py) để xử lý sự cố với `psycopg2`. 
    - Chạy / điều chỉnh [script này](/docs/examples/scripts/python/sqlalchemy_scratchpad.py) để xử lý sự cố với `sqlalchemy`. 

### Với docker

Từ thư mục gốc của repository.

```bash
docker compose up stackqlsrv
```

```bash
psql -d "host=127.0.0.1 port=5576 user=silly dbname=silly sslmode=verify-full sslcert=./cicd/vol/srv/credentials/pg_client_cert.pem sslkey=./cicd/vol/srv/credentials/pg_client_key.pem sslrootcert=./cicd/vol/srv/credentials/pg_server_cert.pem"
```