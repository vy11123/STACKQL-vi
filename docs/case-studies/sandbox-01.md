# Bản dịch
- Đây là bản dịch file: [sandbox-01.md](https://github.com/stackql/stackql/blob/main/docs/case-studies/sandbox-01.md) thuộc dự án [StackQL](https://github.com/stackql/stackql).
- Mục đích: Hỗ trợ người Việt Nam có thể hiểu rõ hơn về StackQL.
- Tài liệu chỉ mang tính chất tham khảo, có thể sai sót trong quá trình biên dịch, rất mong được góp ý để có thể cả thiện hơn.



Self join (Tự kết nối):
```sql
select 
  d1.name as n, 
  d1.id, 
  d2.id as d2_id 
from 
  google.compute.disks d1 
  inner join 
  google.compute.disks d2 
  on d1.id = d2.id 
where 
  d1.project = 'stackql-dev-01' 
  and d1.zone = 'australia-southeast1-b' 
  and d2.project = 'stackql-dev-01' 
  and d2.zone = 'australia-southeast1-b'
;
```