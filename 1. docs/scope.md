# Phạm vi dự án

## 1. Tổng quan

**Tên dự án:** Electronics Inventory Backend  
**Học phần:** Project I — Đại học Bách khoa Hà Nội  
**Thời gian dự kiến:** 8 tuần  
**Hình thức:** Phát triển backend, cung cấp REST API.

Dự án xây dựng backend phục vụ hoạt động quản lý bán hàng và tồn kho tại một cửa hàng đồ điện tử. Hệ thống hỗ trợ quản lý tài khoản nội bộ, danh mục, sản phẩm, nhà cung cấp, phiếu nhập, đơn bán và tra cứu tồn kho.

Bên cạnh nghiệp vụ, dự án thực hành đầy đủ quy trình phát triển phần mềm: phân tích yêu cầu, thiết kế, quản lý mã nguồn, kiểm thử, đóng gói và triển khai. CI/CD là nội dung trọng tâm. Caching, embedding và RAG là các thành phần bắt buộc trong phạm vi kỹ thuật.

Tài liệu này xác định phạm vi cần triển khai, không phản ánh trạng thái hoàn thành của các chức năng.

## 2. Mục tiêu

- Xây dựng backend thực hiện được các nghiệp vụ nhập hàng, bán hàng và quản lý tồn kho.
- Bảo đảm cập nhật dữ liệu nhất quán, không phát sinh tồn kho âm hoặc cộng/trừ tồn nhiều lần do xác nhận trùng.
- Phân quyền truy cập và thao tác phù hợp với vai trò người dùng.
- Tổ chức mã nguồn theo module và phân chia trách nhiệm rõ ràng.
- Có kiểm thử tự động cho các nghiệp vụ và tình huống lỗi quan trọng.
- Có pipeline CI/CD để kiểm tra, đóng gói và triển khai ứng dụng lên môi trường demo.
- Áp dụng caching cho dữ liệu đọc phù hợp.
- Xây dựng chức năng hỏi đáp tài liệu bằng embedding và RAG, có dẫn nguồn.
- Có tài liệu giúp người khác hiểu, khởi chạy, kiểm thử và đánh giá dự án.

## 3. Bối cảnh và giới hạn vận hành

| Nội dung | Phạm vi |
|---|---|
| Lĩnh vực | Bán lẻ đồ điện tử |
| Quy mô | Một cửa hàng, một kho |
| Người sử dụng | Quản trị viên và nhân viên cửa hàng |
| Sản phẩm mẫu | Laptop, màn hình, chuột, bàn phím, tai nghe |
| Đơn vị quản lý hàng hóa | SKU; mỗi mẫu hoặc phiên bản bán riêng được quản lý như một sản phẩm riêng |
| Quản lý tồn kho | Theo số lượng của từng sản phẩm |
| Giao diện tương tác | REST API; demo bằng Swagger hoặc Postman |

Hệ thống phục vụ hoạt động nội bộ của cửa hàng. Khách mua hàng không đăng ký tài khoản hoặc trực tiếp đặt hàng qua hệ thống trong phiên bản này.

## 4. Người dùng và phân quyền

Hệ thống có hai vai trò:

- **ADMIN:** Quản trị hệ thống, quản lý dữ liệu nền và thao tác trên các chứng từ của cửa hàng.
- **STAFF:** Tra cứu thông tin và thực hiện nghiệp vụ nhập, bán hàng theo quyền được cấp.

Tài khoản nhân viên do ADMIN cấp. Hệ thống không cung cấp chức năng tự đăng ký tài khoản công khai.

### 4.1. Ma trận phân quyền

| Chức năng | ADMIN | STAFF |
|---|:---:|:---:|
| Đăng nhập, đổi mật khẩu của bản thân | Có | Có |
| Tạo và khóa tài khoản nhân viên | Có | Không |
| Xem, tìm kiếm và lọc sản phẩm | Có | Có |
| Thêm, sửa sản phẩm và giá bán | Có | Không |
| Ngừng kinh doanh sản phẩm | Có | Không |
| Xem danh mục sản phẩm | Có | Có |
| Quản lý danh mục sản phẩm | Có | Không |
| Xem nhà cung cấp | Có | Có |
| Thêm, sửa nhà cung cấp | Có | Không |
| Tạo phiếu nhập | Có | Có |
| Sửa, hủy phiếu nhập nháp | Mọi phiếu | Phiếu do mình tạo |
| Xác nhận phiếu nhập | Mọi phiếu hợp lệ | Phiếu hợp lệ do mình tạo |
| Tạo đơn bán | Có | Có |
| Sửa, hủy đơn bán nháp | Mọi đơn | Đơn do mình tạo |
| Xác nhận đơn bán | Mọi đơn hợp lệ | Đơn hợp lệ do mình tạo |
| Xem tồn kho và lịch sử nhập/xuất | Có | Có |
| Đưa tài liệu vào hệ thống RAG | Có | Không |
| Hỏi đáp dựa trên tài liệu | Có | Có |

### 4.2. Giới hạn quyền thao tác

- Quyền của ADMIN không cho phép bỏ qua các quy tắc nghiệp vụ.
- Cả ADMIN và STAFF đều không được sửa hoặc hủy chứng từ đã xác nhận.
- Không vai trò nào được sửa trực tiếp số lượng tồn kho.
- Kiểm tra quyền phải được thực hiện tại backend.

## 5. Chức năng trong phạm vi

### 5.1. Tài khoản và xác thực

- Đăng nhập bằng email và mật khẩu.
- Phân biệt quyền ADMIN và STAFF.
- Cho phép người dùng đổi mật khẩu của bản thân.
- Cho phép ADMIN cấp và khóa tài khoản nhân viên.
- Lưu mật khẩu dưới dạng đã băm.
- Ngăn tài khoản bị khóa tiếp tục thực hiện thao tác được bảo vệ.

Cơ chế xác thực và chính sách phiên được cụ thể hóa trong tài liệu thiết kế backend/API.

### 5.2. Danh mục và sản phẩm

- Quản lý danh mục sản phẩm.
- Thêm và cập nhật thông tin sản phẩm.
- Quản lý SKU duy nhất, tên, danh mục, giá bán và trạng thái kinh doanh.
- Tra cứu danh sách và chi tiết sản phẩm.
- Hỗ trợ tìm kiếm, lọc và phân trang.
- Cho phép ngừng kinh doanh sản phẩm.
- Giữ dữ liệu sản phẩm đã phát sinh giao dịch để bảo toàn lịch sử.

### 5.3. Nhà cung cấp

- Xem danh sách và thông tin nhà cung cấp.
- Thêm và sửa tên, số điện thoại, email, địa chỉ.
- Liên kết nhà cung cấp với phiếu nhập hàng.
- Không xóa nhà cung cấp đã có phiếu nhập.

### 5.4. Nhập hàng

- Tạo phiếu nhập gắn với một nhà cung cấp.
- Thêm một hoặc nhiều mặt hàng vào phiếu.
- Lưu số lượng và giá nhập riêng cho từng mặt hàng.
- Sửa hoặc hủy phiếu khi còn ở trạng thái nháp.
- Xác nhận phiếu để cộng tồn kho.
- Lưu người tạo, người xác nhận và các thời điểm liên quan.
- Tra cứu chứng từ để phục vụ kiểm tra và đối chiếu.

### 5.5. Bán hàng

- Tạo đơn bán nội bộ.
- Thêm một hoặc nhiều mặt hàng vào đơn.
- Lưu số lượng và giá bán trên từng dòng đơn.
- Sửa hoặc hủy đơn khi còn ở trạng thái nháp.
- Kiểm tra đủ tồn kho khi xác nhận.
- Xác nhận đơn để ghi nhận bán hàng và trừ tồn.
- Tra cứu chứng từ để phục vụ kiểm tra và đối chiếu.

Xác nhận đơn bán trong phiên bản này không bao gồm xử lý thanh toán trực tuyến, theo dõi công nợ hoặc vận chuyển.

### 5.6. Tồn kho và lịch sử nhập/xuất

- Xem số lượng tồn hiện tại của từng sản phẩm.
- Tạo tồn ban đầu bằng 0 khi thêm sản phẩm.
- Thay đổi tồn thông qua xác nhận phiếu nhập hoặc đơn bán.
- Tra cứu lịch sử tăng, giảm tồn theo sản phẩm và chứng từ.
- Tổng hợp lịch sử từ phiếu nhập và đơn bán đã xác nhận.

Thiết kế hiện tại không sử dụng bảng `stock_movements` riêng.

### 5.7. Hỏi đáp tài liệu bằng embedding và RAG

- Chuẩn bị bộ dữ liệu mẫu gồm khoảng 5–10 tài liệu.
- Sử dụng tài liệu thông số sản phẩm, hướng dẫn sử dụng và chính sách bán hàng/bảo hành.
- Cung cấp luồng đưa tài liệu vào hệ thống, chia đoạn, tạo embedding và lưu phục vụ tìm kiếm.
- Tìm các đoạn tài liệu liên quan đến câu hỏi.
- Sử dụng nội dung tìm được để tạo câu trả lời kèm nguồn tham chiếu.
- Thông báo không đủ căn cứ khi tài liệu không cung cấp thông tin cần thiết.
- Có bộ câu hỏi đánh giá chất lượng truy xuất và câu trả lời.

RAG không được dùng để suy đoán tồn kho hoặc giá bán hiện tại. Dữ liệu giao dịch được truy xuất qua các chức năng nghiệp vụ tương ứng.

Tra cứu chính sách bảo hành không bao gồm tiếp nhận và xử lý hồ sơ bảo hành.

## 6. Nguyên tắc nghiệp vụ cốt lõi

Các nguyên tắc sau xác định giới hạn hành vi của hệ thống. Chi tiết được trình bày trong tài liệu quy tắc nghiệp vụ.

- Phiếu nhập và đơn bán có ba trạng thái: `DRAFT`, `CONFIRMED`, `CANCELLED`.
- Chỉ chứng từ `DRAFT` được sửa, hủy hoặc xác nhận.
- Chứng từ nháp chưa làm thay đổi tồn kho; đơn bán nháp không giữ hàng.
- Khi xác nhận, chứng từ phải có ít nhất một mặt hàng hợp lệ.
- Một sản phẩm chỉ xuất hiện một dòng trong cùng chứng từ.
- Số lượng trên dòng chứng từ phải là số nguyên dương; giá không âm.
- Một chứng từ chỉ được tác động tới tồn kho tối đa một lần.
- Xác nhận chứng từ phải hoàn tất toàn bộ hoặc không thay đổi dữ liệu.
- Nếu bất kỳ mặt hàng nào không đủ tồn, toàn bộ lần xác nhận đơn bán bị từ chối.
- Các yêu cầu đồng thời không được làm tồn kho âm.
- Giá bán được lưu trên dòng đơn tại thời điểm thêm sản phẩm; không tự thay đổi theo giá hiện tại trong danh mục.
- Dữ liệu lịch sử và các liên kết tới tài khoản, sản phẩm, nhà cung cấp phải được bảo toàn.

## 7. Phạm vi kỹ thuật bắt buộc

| Nội dung | Yêu cầu và bằng chứng dự kiến |
|---|---|
| Version Control System | Sử dụng Git; có lịch sử commit, nhánh làm việc và merge |
| Repository Hosting | Sử dụng GitHub để lưu mã nguồn, tài liệu, issue và pull request |
| Kiến trúc và pattern | Modular Monolith; phân lớp Router–Controller–Service–Repository; trình bày trách nhiệm và lý do áp dụng |
| Database | MySQL; có ERD, script/migration, seed, khóa và ràng buộc dữ liệu |
| API | REST API có validation, phân quyền, cấu trúc lỗi và tài liệu OpenAPI/Swagger |
| Testing | Unit test, integration/API test và kiểm thử đồng thời cho nghiệp vụ quan trọng |
| Docker | Dockerfile và Docker Compose; hướng dẫn khởi chạy và lưu dữ liệu bền vững |
| CI | Tự động kiểm tra code, chạy test và build trên các sự kiện phù hợp của repository |
| CD | Triển khai ứng dụng lên môi trường demo, kiểm tra sau triển khai và có hướng dẫn rollback |
| Caching | Redis cache-aside cho ít nhất một luồng đọc; có TTL, invalidation và minh chứng hit/miss |
| Embedding | Tạo và sử dụng vector biểu diễn các đoạn tài liệu |
| RAG | Truy xuất tài liệu và sinh câu trả lời có nguồn; có đánh giá và xử lý thiếu căn cứ |
| Vận hành cơ bản | Quản lý cấu hình/secrets, log lỗi và health endpoint |

CI/CD được xây dựng và hoàn thiện xuyên suốt quá trình phát triển. Build thành công một Docker image chưa được coi là hoàn thành phần triển khai CD.

Cache không được dùng làm nguồn quyết định tồn kho khi xác nhận bán hàng.

Các dịch vụ embedding, mô hình sinh câu trả lời, giải pháp lưu vector và môi trường triển khai sẽ được cụ thể hóa trong tài liệu kỹ thuật.

## 8. Kiến trúc dự kiến

Backend sử dụng Node.js và Express theo kiến trúc Modular Monolith. Các module nằm trong cùng một ứng dụng, được phân chia theo nghiệp vụ:

| Module | Trách nhiệm |
|---|---|
| `auth` | Đăng nhập và xác thực |
| `users` | Quản lý tài khoản và đổi mật khẩu |
| `catalog` | Quản lý sản phẩm và danh mục |
| `suppliers` | Quản lý nhà cung cấp |
| `inventory` | Tra cứu tồn kho và lịch sử nhập/xuất |
| `purchasing` | Quản lý phiếu nhập và xác nhận nhập |
| `sales` | Quản lý đơn bán và xác nhận bán |
| `knowledge` | Xử lý tài liệu, embedding và RAG |

Thiết kế chi tiết về cấu trúc mã nguồn, luồng xử lý và phạm vi transaction được trình bày trong tài liệu kiến trúc.

## 9. Ngoài phạm vi

Phiên bản này không triển khai:

- Giao diện frontend.
- Nhiều cửa hàng, nhiều kho hoặc điều chuyển kho.
- Theo dõi từng thiết bị bằng serial/IMEI.
- Tài khoản khách hàng, giỏ hàng và khách tự đặt hàng trực tuyến.
- Thanh toán trực tuyến, quản lý công nợ hoặc kế toán.
- Quản lý vận chuyển và giao hàng.
- Đổi trả hàng, hoàn tiền hoặc hủy chứng từ đã xác nhận.
- Tiếp nhận và xử lý yêu cầu bảo hành.
- Kiểm kê, điều chỉnh tồn thủ công hoặc xử lý hàng hỏng.
- Báo cáo phân tích kinh doanh nâng cao.
- Kiến trúc microservices hoặc triển khai hệ thống phân tán quy mô lớn.

## 10. Điều kiện hoàn thành phạm vi

Dự án được xem là hoàn thành phạm vi khi có các bằng chứng sau:

- Demo được luồng đăng nhập, quản lý sản phẩm, nhập hàng, bán hàng và tra cứu tồn.
- Phân quyền đúng theo ma trận đã xác định.
- Chứng minh được việc chặn xác nhận trùng, rollback khi lỗi và không bán vượt tồn trong tình huống đồng thời.
- Tra cứu được lịch sử nhập/xuất từ chứng từ đã xác nhận.
- Có cơ chế cache và minh chứng invalidation.
- Demo được luồng RAG trả lời có nguồn và tình huống không đủ căn cứ.
- Bộ test tự động chạy được; pipeline CI kiểm tra và build thành công.
- Có bằng chứng triển khai qua quy trình CD lên môi trường demo.
- Người khác có thể khởi chạy và kiểm thử dự án theo tài liệu.
- Repository có mã nguồn, tài liệu thiết kế, lịch sử phát triển và bản release phục vụ nghiệm thu.

## 11. Quản lý thay đổi phạm vi

Mọi bổ sung hoặc thay đổi phạm vi cần được ghi nhận bằng issue, nêu rõ:

- Lý do thay đổi.
- Chức năng hoặc yêu cầu bị ảnh hưởng.
- Tác động tới database, API, kiểm thử và tiến độ.
- Tiêu chí hoàn thành sau điều chỉnh.

Tài liệu liên quan phải được cập nhật cùng thay đổi thông qua pull request. Các thay đổi ảnh hưởng yêu cầu học phần hoặc nội dung nghiệm thu cần được trao đổi với giảng viên hướng dẫn.
