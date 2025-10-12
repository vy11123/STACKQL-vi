# Bản dịch
- Đây là bản dịch file: [CICD.md](https://github.com/stackql/stackql/blob/main/docs/CICD.md) thuộc dự án [StackQL](https://github.com/stackql/stackql).
- Mục đích: Hỗ trợ người Việt Nam có thể hiểu rõ hơn về StackQL.
- Tài liệu chỉ mang tính chất tham khảo, có thể sai sót trong quá trình biên dịch, rất mong được góp ý để có thể cả thiện hơn.


# CICD

Tóm tắt:

- Hiện tại, các bước kiểm tra pull request (PR), biên dịch và kiểm thử đều được thực hiện thông qua tệp [.github/workflows/build.yml](/.github/workflows/build.yml).
- Việc phát hành qua nhiều kênh (website, homebrew, chocolatey...) hiện đang được thực hiện thủ công.
- Các tác vụ xây dựng và đẩy Docker (Docker Build và Push Jobs) vẫn còn tiềm năng để cải thiện. 
    - Hiện tại, chúng được triển khai dựa một phần vào các mẫu được mô tả tại:
        - https://docs.docker.com/build/ci/github-actions/multi-platform/#distribute-build-across-multiple-runners
        - https://docs.docker.com/build/ci/github-actions/share-image-jobs/ 
    - Mẫu này thực hiện các bước sau:
        - (a) Xây dựng và đẩy ảnh Docker theo dạng digest.
        - (b) Tận dụng  để ghi các thẻ (tag) mong muốn [`docker buildx imagetools (công cụ xử lý hình ảnh của Docker buildx)`](https://docs.docker.com/reference/cli/docker/buildx/imagetools/)
    - Mẫu triển khai này chỉ cần thiết vì nếu thực hiện đẩy thẻ đồng thời, thì các thẻ đa kiến trúc giống nhau có thể bị ghi đè do điều kiện tranh chấp ngược (reverse race condition).
    - **Lưu ý**.  Cần cẩn trọng khi lựa chọn từ [danh sách runner của github actions](https://github.com/actions/runner-images).  Một số runner có thể không tương thích với bộ công cụ giả định.
- Các ảnh Docker (Docker images):
    - [Ảnh dùng trong môi trường sản xuất tại `stackql/stackql`](https://hub.docker.com/r/stackql/stackql/tags).
    - [Ảnh dùng cho môi trường phát triển (không có bất kỳ đảm bảo nào) tại `stackql/stackql-devel`](https://hub.docker.com/r/stackql/stackql-devel/tags).



## Thông tin bí mật (Secrets)

Thay vì triển khai đầy đủ theo [các thực tiễn tốt nhất của github actions](https://securitylab.github.com/research/github-actions-preventing-pwn-requests/), hiện tại:
- Các bước kiểm tra pull request sử dụng [the pull_request event](https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows#pull_request), theo đúng [thực tiễn tốt nhất](https://securitylab.github.com/research/github-actions-preventing-pwn-requests/).
- Các bước kiểm thử tích hợp (integration test), vốn yêu cầu thông tin bí mật (secrets), sử dụng [ngữ cảnh GitHub](https://docs.github.com/en/actions/learn-github-actions/contexts#github-context) để tránh chạy trong các trường hợp không có secrets.
    - Do đó, các pull request từ fork bên ngoài — vốn là mô hình đóng góp từ cộng đồng — sẽ không chạy kiểm thử tích hợp. Về mặt chiến lược, chúng tôi có thể điều hướng các đóng góp cộng đồng thông qua một nhánh staging và/hoặc áp dụng mô hình nhánh phát hành (release branches). Đây chưa phải là vấn đề cấp thiết, và chúng tôi sẽ đưa ra quyết định sau khi cân nhắc kỹ lưỡng.

## Giả lập API (API mocking)

Theo [ví dụ swagger-codegen này](https://github.com/swagger-api/swagger-codegen/blob/master/bin/python-flask-petstore.sh), việc tạo các mock Python từ tài liệu OpenAPI không quá khó. Những mock này có thể được sử dụng để kiểm thử hồi quy một cách đáng tin cậy đối với các tài liệu nhà cung cấp mới, và đặc biệt là để kiểm tra mối quan hệ giữa các endpoint và tài nguyên StackQL.  Điểm yếu đã biết: không thể phát hiện lỗi chuyển đổi sai từ nguồn (ví dụ: MS Graph, AWS) sang OpenAPI.