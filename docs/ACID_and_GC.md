# Bản dịch
- Đây là bản dịch file:[ACID_and_GC.md](https://github.com/stackql/stackql/blob/main/docs/ACID_and_GC.md) thuộc dự án [StackQL](https://github.com/stackql/stackql).
- Mục đích: Hỗ trợ người Việt Nam có thể hiểu rõ hơn về StackQL.
- Tài liệu chỉ mang tính chất tham khảo, có thể sai sót trong quá trình biên dịch, rất mong được góp ý để có thể cả thiện hơn.


# Thu Gom Bộ Nhớ, Bộ Nhớ Đệm, Xử Lý Đồng Thời và Giao Diện Hiển Thị

**Lưu ý**: Các tính năng được mô tả dưới đây đang ở giai đoạn ***alpha*** và **người sở hữu chịu trách nhiệm sử dụng**.


## Bối cảnh


### Quan sát về mối quan hệ giữa thu gom bộ nhớ (GC), bộ nhớ đệm (cache) và xử lý đồng thời (concurrency) 

Nếu chúng ta muốn triển khai một bộ đệm kết quả lớn hoặc bộ đệm phân tích, thì:

- Các bảng bộ đệm cần có cơ chế thu gom bộ nhớ tinh chỉnh, có thể với chính sách riêng có thể cấu hình, khác biệt với GC của kho dữ liệu tiêu chuẩn. *Điểm khác biệt duy nhất so với truy vấn thông thường có lẽ là vòng đời của đối tượng.*
- Có lẽ nên đặt không gian tên riêng cho các bảng bộ đệm.*Có thể chấp nhận sử dụng bảng bộ đệm cho mọi thứ trong không gian tên bộ đệm và chấp nhận hậu quả.*
- Khi một `truy vấn` có thể lưu vào bộ đệm được phân tích, các tham số điều khiển GC của bộ đệm phải được tiếp nhận và sử dụng. *Không nghĩ rằng điều này cần khác biệt so với truy vấn thông thường.*

- GC cần nhận biết ngôn ngữ truy vấn (dialect).
- Truy cập **đồng thời** vào dữ liệu có thể được điều phối thông qua các cột điều khiển.
    - GC cần biết bản ghi nào đang tham gia vào *giao dịch đang hoạt động*.
    - Về việc giới hạn lựa chọn tập kết quả trong xử lý đồng thời và GC:
        - **Nếu** dữ liệu hiện có được sử dụng, thì tập hợp điều kiện điều khiển phải được áp dụng.
        - **Nếu** dữ liệu mới được thêm vào, thì điều kiện điều khiển cũng được sử dụng tương tự.
        - Trong cả hai trường hợp, mối quan hệ giữa giao dịch đang hoạt động và tham số điều khiển phải được cập nhật một cách nguyên tử, và bộ phân tích phải tích lũy các tham số để sử dụng trong quá trình lựa chọn.
        - Giả định đơn giản là tập điều kiện chỉ áp dụng cho từng tập chèn riêng lẻ.
    - Điều này có nghĩa là các truy vấn phức tạp thực chất sở hữu danh sách các cặp `{ Bảng, Điều kiện điều khiển }`. `Bảng` được xác định sớm, còn `Điều kiện điều khiển` được xác định muộn.
    - Trong một `truy vấn` (bao gồm truy vấn SQL đơn giản, truy vấn con hoặc CTE), **tất cả** các bảng được phân tích phải có tham số điều khiển được lưu lại để sử dụng trong quá trình lựa chọn sau này. Tính năng này hiện chưa được triển khai. Việc đọc/ghi tham số điều khiển phải đảm bảo an toàn luồng, thông qua cơ chế khóa hoặc quy tắc thứ tự.


### Ý tưởng về bộ nhớ đệm (Cache ideation)

- ~~Chú thích khởi tạo truy vấn bất đồng bộ (theo cách gọi trong MySQL).~~ Nếu bộ đệm chưa được khởi tạo, thì các truy vấn ban đầu sẽ chạy trực tiếp (online).
- Việc lên lịch có thể được cấu hình (hoặc mở rộng theo cách tương tự).
- Truy vấn sẽ truy cập bộ đệm nếu được cho phép, thời gian sống (TTL) còn hiệu lực, và/hoặc có chú thích đi kèm.
- TTL, lịch trình, và chính sách truy cập đều có thể cấu hình.
- Tóm lại, quy trình gồm một bước khởi tạo bộ đệm, sau đó là xử lý phân tích dữ liệu (OLAP).


### Bằng chứng khái niệm (POC) cho việc đọc bộ đệm ban đầu

- Đưa dữ liệu vào bảng bộ đệm trống thông qua script thiết lập.
- Giai đoạn phân tích cần nhận biết tiền tố của bộ đệm.
- Xong!


### Cách sử dụng


Dưới đây là ví dụ sử dụng StackQL để lưu vào bộ đệm phản hồi từ GitHub trong vòng 1 giờ:

```
export NAMESPACES='{ "analytics": { "ttl": 86400, "regex": "^(?P<objectName>github.*)$", "template": "stackql_analytics_{{ .objectName }}" } }'


stackql ... --namespaces="${NAMESPACES}" ... shell
```

### MVCC và Postgres?

- [Tổng quan cấp cao về MVCC trong Postgres](https://devcenter.heroku.com/articles/postgresql-concurrency#:~:text=a%20hard%20problem.-,How%20MVCC%20works,statements%20together%20via%20BEGIN%20%2D%20COMMIT%20). Một số điểm thú vị từ bài viết này:
    - Các bộ đếm `t_xmin` và `t_xmax` có thể được quan sát thông qua truy vấn rõ ràng; `SELECT *, xmin, xmax FROM table_name`.
    - ID giao dịch cũng có thể được truy xuất; `SELECT txid_current();`.
- [Tài liệu chính thức về MVCC của Postgres](https://www.postgresql.org/docs/current/mvcc.html) 
- [Tài liệu chính thức về VACUUM và xử lý tràn ID của Postgres](https://www.postgresql.org/docs/current/routine-vacuuming.html)

Postgres triểnkhai [cơ chế khóa đọc/ghi (RW locking) trên các chỉ mục](https://www.postgresql.org/docs/current/locking-indexes.html) tùy thuộc vào loại chỉ mục và có thể gây ra các vấn đề về hiệu năng hoặc bế tắc. Chỉ mục kiểu `B-tree` là loại mặc định, phù hợp nhất cho dữ liệu kiểu `scalar` (dữ liệu đơn giá trị như số nguyên, chuỗi ngắn…).

Ở cấp độ bản ghi, MVCC đảm nhận, tận dụng [header của tuple ](https://www.postgresql.org/docs/current/storage-page-layout.html#STORAGE-TUPLE-LAYOUT).  Các trường chính ở đây là `t_xmin` cho giao dịch đã tạo bản ghi và `t_xmax`, ***usually*** ghi nhận giao dịch đã xóa bản ghi.  Cả `t_xmin` và `t_xmax` đều là bộ đếm 32 bit và các bất thường do tràn chỉ được ngăn chặn bởi hành động [garbage collection (thu gom rác) của `VACUUM`](https://www.postgresql.org/docs/current/routine-vacuuming.html).  Theo quy chuẩn, cập nhật sẽ tạo ra một dòng mới (xóa thì giống vậy, nhưng không tạo tuple mới) và phần đầu của tuple `t_max` được gán cho giao dịch đã thực hiện cập nhật. Một biến thể thú vị là trường hợp nhiều giao dịch cùng giữ khóa trên tuple; trong trường hợp đó, trường `t_infomask` [sẽ chỉ ra rằng `t_xmax` nên được hiểu là một `MultiXactId`](https://github.com/postgres/postgres/blob/ce20f8b9f4354b46b40fd6ebf7ce5c37d08747e0/src/include/access/htup_details.h#L208).  `MultiXactId` cũng là một bộ đếm 32-bit, nhưng hoạt động như một con trỏ gián tiếp đến danh sách các ID giao dịch được lưu trữ ở nơi khác. Ngoài ra còn có nhiều biến thể khác, và các thuật toán để duy trì tính nhất quán của toàn bộ dữ liệu liên quan là rất phức tạp.


## Phương pháp ngắn hạn cho StackQL

### Phiên bản v1

Trong giai đoạn đầu, việc sử dụng MVCC (Kiểm soát đồng thời đa phiên bản) có thể là quá mức cần thiết đối với StackQL. Lý do là vì StackQL thực tế **không cần** cập nhật các phần dữ liệu không thuộc kiểm soát trong bản ghi cơ sở dữ liệu, và các giao dịch **không bao giờ** trực tiếp xóa bản ghi. Do đó, các thông tin sau là đủ để suy luận xem một bản ghi có phải là ứng viên bị xóa hay không:

  - `txn_max_id` là ID giao dịch lớn nhất đã khóa bản ghi.  
  - `txn_running_min_id` là ID giao dịch nhỏ nhất vẫn đang chạy trong hệ thống.

Nếu `txn_running_min_id` > `txn_max_id` thì bản ghi có thể được xóa một cách an toàn. Đây chắc chắn không phải là phương pháp duy nhất hoặc tối ưu, nhưng nó đơn giản và chỉ yêu cầu lưu một ID giao dịch tối đa (Txn ID) cho mỗi bản ghi. Kỹ thuật đơn giản nhất là lưu Txn ID này dưới dạng một cột kiểm soát trong mỗi bộ bản ghi.

Một **yêu cầu bắt buộc**  của mô hình lưu trữ Txn ID trong từng dòng (in-row pattern) là việc cập nhật bản ghi trong hệ thống cơ sở dữ liệu phải được thực hiện một cách nguyên tử (atomically). Điều này chắc chắn đúng với các hệ quản trị cơ sở dữ liệu như SQLite, Postgres, MySQL, v.v., nhưng có thể không đúng với tất cả các backend trong tương lai. Tuy nhiên, điều này là đủ cho `phiên bản v1`.

Các giao dịch `phantom` Txns tồn tại lâu hoặc không hoạt động phải có khả năng bị loại bỏ thông qua một khía cạnh nào đó của quá trình thu gom rác (GC). Điều này là cần thiết bởi nếu không `txn_running_min_id` sẽ không thể tiến triển và cuối cùng sẽ làm hỏng tính bất biến của bộ đếm giao dịch đơn điệu. Ban đầu, chúng ta có thể xử lý vấn đề này bằng cách loại bỏ tất cả các giao dịch cũ hơn một ngưỡng thời gian nhất định hoặc thời gian tương ứng với ID giao dịch của một ngưỡng số lượng nhất định. Cách tiếp cận này có thể được cải thiện sau.

Vì vậy, một cách tiếp cận hợp lý cho `phiên bản v1` về xử lý đồng thời và thu gom rác (GC) trong `stackql` là:

- Txn ID, như có thể được yêu cầu trong nhiều khía cạnh của hệ thống, là một bộ đếm tăng đơn điệu. Cách triển khai được chọn là kiểu số nguyên có kích thước cố định, hoạt động như một vòng lặp và được đặt lại định kỳ về 0. **Cập nhật**: - điều này sẽ được triển khai ở tầng backend của cơ sở dữ liệu, thông qua SQL. **TBD (Chưa hoàn thiện)**: - logic đặt lại về 0 vẫn chưa được triển khai ở giai đoạn *alpha* ban đầu này.  
- Một danh sách toàn cục các bộ dữ liệu (Txn ID, thời điểm bắt đầu) đang hoạt động phải được duy trì. `txn_running_min_id` - có thể được suy ra từ danh sách này. Cách làm này tương tự như Postgres.  **TBD (Chưa hoàn thiện)**: - kho lưu trữ Txn ID đã hoàn thành, tuy nhiên các timestamp vẫn chưa được lưu lại, vì vậy các giao dịch cũ hiện đang bị ẩn.
- Giai đoạn `acquire` (ghi vào DB sau khi gọi REST hoặc đọc từ cache) phải cập nhật `txn_max_id` bằng logic SQL có điều kiện (chỉ cập nhật nếu giá trị mới lớn hơn giá trị hiện tại).
- **TBD (Chưa hoàn thiện)**: Các chu kỳ thu gom rác (GC) sẽ được kích hoạt bởi:
  - Ngưỡng số lượng giao dịch (Txn) đang hoạt động đạt mức quy định.
  - Lịch trình định sẵn.
  - ...
- Trong các chu kỳ GC:
  - **TBD (Chưa hoàn thiện)**: Không được phép bắt đầu giao dịch mới.
  - **Giả thuyết**: Các giao dịch hiện tại có thể ở trạng thái không hoạt động.
  - Nếu có quá nhiều giao dịch đang hoạt động và/hoặc một số đã quá cũ, thì cần hủy các giao dịch bất thường.
  - Nếu `txn_running_min_id` > `txn_max_id` thì cần hủy bản ghi tương ứng. Các hoạt động quản trị cần được hỗ trợ:
    - Can thiệp xóa bỏ giao dịch và bản ghi. Tính năng này có thể thực hiện thông qua cú pháp `PURGE`.
- **TBD (Chưa hoàn thiện)**: Cần triển khai các luồng xử lý có thể bị gián đoạn trước (pre-emptible handler threads).


This naive approach avoids deadlock and provides break glass.

Please **watch this space** on all items which are **TBD**-inclusive.

### Phiên bản v2

- Các hoạt động quản trị cần được hỗ trợ (có thể có trong `phiên bản v2`):
  - Can thiệp xóa dữ liệu (Purge).
  - Liệt kê các giao dịch (Txn) đang hoạt động.
  - Tạm dừng / Cho phép giao dịch mới.
  - - Hủy giao dịch (lọc hoặc không lọc).

## ACID


### Khi tính chất ACID của RDBMS bị phá vỡ

Dù đã nỗ lực để đảm bảo tính chất ACID, vẫn có những trường hợp hệ thống bị lỗi và cần sự can thiệp từ quản trị viên — ngay cả với các hệ quản trị cơ sở dữ liệu quan hệ (RDBMS) lưu trữ trên đĩa cục bộ, [tham khảo tài liệu về lỗ hỏng dữ liệu của  `postgres` tại đây](https://wiki.postgresql.org/wiki/Corruption).