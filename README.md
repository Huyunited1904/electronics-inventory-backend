# Electronics Inventory Backend

Backend quản lý bán hàng và tồn kho cho một cửa hàng đồ điện tử, được phát triển trong học phần **Project I — Đại học Bách khoa Hà Nội**.

Dự án hỗ trợ quản lý tài khoản nội bộ, sản phẩm, nhà cung cấp, phiếu nhập, đơn bán và tồn kho. Bên cạnh nghiệp vụ, dự án thực hành quy trình phát triển backend từ phân tích yêu cầu, thiết kế database/API đến kiểm thử, đóng gói và triển khai.

**CI/CD là nội dung trọng tâm**, cùng với quản lý mã nguồn, kiến trúc phần mềm, caching, embedding và RAG.

> **Trạng thái hiện tại:** Đang hoàn thiện tài liệu thiết kế và hợp đồng API. Backend, migration, kiểm thử tự động và pipeline CI/CD chưa được triển khai.

## 1. Thông tin dự án

| Nội dung | Thông tin |
|---|---|
| Người thực hiện | Nguyễn Quang Huy |
| Học phần | Project I |
| Trường | Đại học Bách khoa Hà Nội |
| Repository | `electronics-inventory-backend` |
| Thời gian dự kiến | 8 tuần |
| Quy mô nghiệp vụ | Một cửa hàng, một kho |
| Đối tượng sử dụng | Quản trị viên và nhân viên cửa hàng |
| Sản phẩm bàn giao | Backend REST API, tài liệu, kiểm thử và môi trường demo |
| Giao diện demo | Swagger/Postman |
| Frontend | Không thuộc phạm vi triển khai |

## 2. Mục tiêu

### Nghiệp vụ

- Quản lý sản phẩm, danh mục và nhà cung cấp.
- Ghi nhận nhập hàng và bán hàng thông qua chứng từ.
- Theo dõi số lượng tồn và lịch sử nhập/xuất.
- Phân quyền theo vai trò và người tạo chứng từ.
- Bảo đảm tồn kho nhất quán khi xử lý nhiều yêu cầu đồng thời.

### Kỹ thuật

- Tổ chức backend theo module và phân chia trách nhiệm rõ ràng.
- Quản lý mã nguồn, tài liệu và thay đổi bằng Git/GitHub.
- Xây dựng kiểm thử tự động cho nghiệp vụ quan trọng.
- Đóng gói hệ thống bằng Docker và Docker Compose.
- Xây dựng pipeline CI/CD có kiểm thử, build và triển khai thực tế.
- Sử dụng Redis caching với TTL và cơ chế invalidation.
- Tích hợp embedding/RAG để hỏi đáp tài liệu có dẫn nguồn.

Các mục tiêu trên là phạm vi cần thực hiện, không phải tuyên bố các tính năng đã hoàn thành.

## 3. Phạm vi chức năng

### 3.1. Các nhóm chức năng

| Nhóm | Nội dung |
|---|---|
| Tài khoản | Đăng nhập, đổi mật khẩu, cấp và khóa tài khoản nhân viên |
| Danh mục | Quản lý các nhóm sản phẩm |
| Sản phẩm | Thêm, sửa, tra cứu, tìm kiếm, lọc, phân trang và ngừng kinh doanh |
| Nhà cung cấp | Xem, thêm và sửa thông tin |
| Nhập hàng | Tạo phiếu nháp, quản lý dòng hàng, hủy và xác nhận nhập |
| Bán hàng | Tạo đơn nháp, quản lý dòng hàng, hủy và xác nhận bán |
| Tồn kho | Xem số lượng hiện tại của từng sản phẩm |
| Lịch sử nhập/xuất | Tra cứu biến động từ chứng từ đã xác nhận |
| Hỏi đáp tài liệu | Tra cứu thông số, hướng dẫn sử dụng và chính sách bằng RAG |

Sản phẩm mẫu gồm laptop, màn hình, chuột, bàn phím và tai nghe. Mỗi mẫu hoặc phiên bản bán riêng được quản lý bằng một SKU.

### 3.2. Vai trò người dùng

| Vai trò | Phạm vi trách nhiệm |
|---|---|
| **ADMIN** | Quản lý tài khoản, dữ liệu nền, tài liệu RAG và thao tác trên chứng từ của cửa hàng |
| **STAFF** | Tra cứu thông tin, hỏi đáp tài liệu, tạo và xử lý chứng từ do mình tạo |

Nguyên tắc phân quyền:

- Tài khoản nhân viên do ADMIN cấp, không có đăng ký công khai.
- STAFF chỉ được sửa, hủy và xác nhận chứng từ do mình tạo.
- ADMIN vẫn phải tuân thủ quy tắc trạng thái và kiểm tra tồn kho.
- Chứng từ đã xác nhận không được sửa hoặc hủy.
- Không vai trò nào được sửa trực tiếp số lượng tồn kho.

### 3.3. Ngoài phạm vi

Phiên bản này không triển khai:

- Frontend, tài khoản khách hàng hoặc khách tự đặt hàng.
- Nhiều cửa hàng, nhiều kho và điều chuyển kho.
- Quản lý từng thiết bị bằng serial/IMEI.
- Giỏ hàng, thanh toán trực tuyến, công nợ và kế toán.
- Vận chuyển và giao hàng.
- Đổi trả, hoàn tiền hoặc hủy chứng từ đã xác nhận.
- Tiếp nhận và xử lý hồ sơ bảo hành.
- Kiểm kê, điều chỉnh tồn thủ công hoặc xử lý hàng hỏng.
- Báo cáo kinh doanh nâng cao.
- Microservices và hệ thống phân tán quy mô lớn.

Chi tiết tại [Phạm vi dự án](docs/scope.md).

## 4. Luồng nghiệp vụ chính

### 4.1. Nhập hàng

1. Nhân viên tạo phiếu nhập ở trạng thái `DRAFT`.
2. Chọn nhà cung cấp và thêm các mặt hàng, số lượng, giá nhập.
3. Kiểm tra điều kiện và xác nhận phiếu.
4. Hệ thống cộng tồn, lưu người/thời điểm xác nhận và chuyển sang `CONFIRMED`.

Phiếu nháp chưa làm tăng tồn kho. Một phiếu có thể chứa nhiều mặt hàng và chỉ được tác động tới tồn kho một lần.

### 4.2. Bán hàng

1. Nhân viên tạo đơn bán ở trạng thái `DRAFT`.
2. Thêm sản phẩm, số lượng và lưu giá bán trên từng dòng.
3. Khi xác nhận, hệ thống kiểm tra tồn kho hiện tại.
4. Nếu đủ hàng, trừ tồn và chuyển đơn sang `CONFIRMED`.
5. Nếu thiếu bất kỳ mặt hàng nào, từ chối toàn bộ lần xác nhận.

Đơn nháp không trừ tồn và không giữ hàng. Không được trừ tồn dở dang hoặc bán vượt tồn khi có nhiều yêu cầu đồng thời.

### 4.3. Trạng thái chứng từ

| Trạng thái | Ý nghĩa | Thao tác tiếp theo |
|---|---|---|
| `DRAFT` | Chứng từ đang được chuẩn bị | Sửa, hủy hoặc xác nhận khi hợp lệ |
| `CONFIRMED` | Đã xác nhận và cập nhật tồn | Tra cứu |
| `CANCELLED` | Đã hủy khi còn nháp | Tra cứu |

### 4.4. Các yêu cầu nhất quán quan trọng

- Số lượng dòng chứng từ phải là số nguyên dương.
- Giá không âm; tồn kho không âm.
- Một sản phẩm chỉ xuất hiện một dòng trong cùng chứng từ.
- Cập nhật tồn và thông tin xác nhận phải nằm trong cùng transaction.
- Có lỗi thì rollback toàn bộ thay đổi của lần xác nhận.
- Không cập nhật tồn hai lần khi gửi lại yêu cầu xác nhận.
- Giá bán trên đơn không tự thay đổi theo giá hiện tại của sản phẩm.
- Tồn ban đầu bằng 0; hàng có sẵn được ghi nhận qua phiếu nhập.

Chi tiết tại [Quy tắc nghiệp vụ](docs/business-rules.md).

## 5. Công nghệ

| Thành phần | Lựa chọn dự kiến triển khai |
|---|---|
| Backend | Node.js, Express |
| Database | MySQL, InnoDB |
| Cache | Redis |
| Xác thực | JWT access token |
| Tài liệu API | Markdown, OpenAPI/Swagger |
| Kiểm thử | Jest hoặc Vitest, kết hợp Supertest; chưa chốt test runner |
| Đóng gói | Docker, Docker Compose |
| CI/CD | GitHub Actions |
| Quản lý phiên bản | Git |
| Repository hosting | GitHub |
| Embedding, LLM và vector store | Chưa chốt giải pháp cụ thể |
| Môi trường demo | Chưa chốt |

Phiên bản công cụ và các lựa chọn còn mở sẽ được cập nhật khi triển khai.

## 6. Kiến trúc backend

Backend sử dụng **Modular Monolith**: một ứng dụng được phân chia thành các module theo nghiệp vụ.

| Module | Trách nhiệm |
|---|---|
| `auth` | Đăng nhập và xác thực |
| `users` | Quản lý tài khoản và đổi mật khẩu |
| `catalog` | Sản phẩm và danh mục |
| `suppliers` | Nhà cung cấp |
| `inventory` | Tồn kho và lịch sử nhập/xuất |
| `purchasing` | Phiếu nhập và xác nhận nhập |
| `sales` | Đơn bán và xác nhận bán |
| `knowledge` | Tài liệu, embedding và RAG |

### Phân chia trách nhiệm

- **Router:** Khai báo endpoint và gắn middleware.
- **Validator:** Kiểm tra dữ liệu đầu vào.
- **Controller:** Nhận request, gọi Service và trả response.
- **Service:** Xử lý nghiệp vụ và điều phối transaction.
- **Repository:** Thực hiện truy vấn database.
- **Middleware:** Xác thực, kiểm tra quyền chung, logging và xử lý lỗi.

Các Repository tham gia một transaction phải dùng cùng một connection.

Các module cùng chạy trong một backend; không triển khai mỗi module thành một microservice riêng.

Chi tiết tại [Kiến trúc hệ thống](docs/architecture.md).

## 7. Database và xác thực

### 7.1. Database nghiệp vụ

Thiết kế hiện tại gồm 9 bảng:

| Nhóm | Bảng |
|---|---|
| Tài khoản | `users` |
| Dữ liệu nền | `categories`, `products`, `suppliers` |
| Tồn kho | `inventory` |
| Nhập hàng | `stock_receipts`, `stock_receipt_items` |
| Bán hàng | `sales_orders`, `sales_order_items` |

Không sử dụng bảng `stock_movements`. Lịch sử nhập/xuất được tổng hợp từ chứng từ đã xác nhận và các dòng chi tiết.

Các bảng phục vụ RAG sẽ được thiết kế riêng.

Tham khảo [Thiết kế database](docs/database.md) và [Mã nguồn ERD](docs/diagrams/erd.dbml).

### 7.2. Chính sách JWT

- Access token có thời hạn 1 giờ.
- Chưa triển khai refresh token.
- Request được bảo vệ gửi `Authorization: Bearer <access_token>`.
- Backend xác minh token rồi đọc tài khoản trong MySQL để kiểm tra trạng thái và vai trò hiện tại.
- Tài khoản bị khóa không được tiếp tục thực hiện request được bảo vệ.
- Đổi mật khẩu không thu hồi token cũ; token còn hiệu lực tới khi hết hạn nếu tài khoản vẫn hoạt động.

Chính sách độ dài mật khẩu và thư viện băm **chưa được chốt**.

Chi tiết tại [API đăng nhập và đổi mật khẩu](docs/api/auth.md).

## 8. Caching và hỏi đáp tài liệu

### Redis caching

Thiết kế áp dụng cache-aside cho dữ liệu đọc phù hợp, dự kiến bắt đầu từ danh mục sản phẩm.

- Có TTL và xóa cache liên quan sau khi cập nhật database thành công.
- MySQL là nguồn dữ liệu nghiệp vụ chính.
- Không dùng cache làm căn cứ quyết định tồn khi xác nhận bán.
- Cần xử lý Redis lỗi và kiểm thử dữ liệu cũ.

### Embedding và RAG

Luồng dự kiến:

1. ADMIN đưa tài liệu vào hệ thống.
2. Trích xuất nội dung, chia đoạn và tạo embedding.
3. Lưu vector cùng thông tin nguồn.
4. Truy xuất các đoạn liên quan khi có câu hỏi.
5. Sinh câu trả lời dựa trên ngữ cảnh và kèm nguồn.

RAG phải nêu thiếu căn cứ khi tài liệu không đủ thông tin. Không suy đoán giá/tồn kho hiện tại và không tự cập nhật dữ liệu giao dịch.

Hai phần trên chưa được triển khai.

## 9. Tổ chức repository

Cấu trúc dưới đây là định hướng, không phải toàn bộ đường dẫn đã tồn tại.

| Đường dẫn | Nội dung |
|---|---|
| `README.md` | Giới thiệu và điều hướng tài liệu |
| `.gitignore` | Quy định file/thư mục không theo dõi |
| `docs/` | Phạm vi, nghiệp vụ và thiết kế |
| `docs/api/` | Hợp đồng API theo module |
| `docs/diagrams/` | Mã và ảnh sơ đồ |
| `src/app.js` | Tạo Express app và gắn middleware/router |
| `src/server.js` | Khởi động server |
| `src/config/` | Cấu hình môi trường |
| `src/modules/` | Mã nguồn các module nghiệp vụ |
| `src/infrastructure/` | Database, cache và dịch vụ AI |
| `src/shared/` | Middleware, lỗi và tiện ích dùng chung |
| `database/migrations/` | Thay đổi schema có phiên bản |
| `database/seeds/` | Dữ liệu khởi tạo và dữ liệu mẫu |
| `tests/unit/` | Kiểm thử logic riêng lẻ |
| `tests/integration/` | Kiểm thử API, database và transaction |
| `.github/workflows/` | Workflow CI/CD |
| `Dockerfile` | Đóng gói backend |
| `compose.yaml` | Khởi chạy các service |
| `.env.example` | Cấu hình mẫu, không chứa secret thật |

## 10. Danh mục tài liệu

**Đã soạn** thể hiện tài liệu đã được xây dựng, không đồng nghĩa chức năng đã được triển khai. **Dự kiến** là tài liệu sẽ bổ sung sau.

Tài liệu hiện được phát triển trên nhánh `doc`. Liên kết từ `main` chỉ hoạt động sau khi file tương ứng được đưa vào `main`. Các liên kết tới file dự kiến chưa tạo sẽ chưa mở được.

### 10.1. Phạm vi và thiết kế

| Tài liệu | Trạng thái |
|---|---|
| [Phạm vi dự án](docs/scope.md) | Đã soạn |
| [Quy tắc nghiệp vụ](docs/business-rules.md) | Đã soạn; còn một số quyết định mở |
| [Thiết kế database](docs/database.md) | Đã soạn |
| [Kiến trúc backend](docs/architecture.md) | Đã soạn |
| [Mã nguồn ERD](docs/diagrams/erd.dbml) | Đã tạo |
| [Ảnh ERD](docs/diagrams/erd.png) | Cần đối chiếu tên file đã upload |

### 10.2. Hợp đồng API

| Tài liệu | Trạng thái |
|---|---|
| [Đăng nhập và đổi mật khẩu](docs/api/auth.md) | Đã soạn; chính sách mật khẩu chưa chốt |
| [Quản lý tài khoản](docs/api/users.md) | Dự kiến |
| [Danh mục và sản phẩm](docs/api/catalog.md) | Dự kiến |
| [Nhà cung cấp](docs/api/suppliers.md) | Dự kiến |
| [Nhập hàng](docs/api/purchasing.md) | Dự kiến |
| [Bán hàng](docs/api/sales.md) | Dự kiến |
| [Tồn kho và lịch sử](docs/api/inventory.md) | Dự kiến |
| [Tài liệu và hỏi đáp RAG](docs/api/knowledge.md) | Dự kiến |
| [Đặc tả OpenAPI](docs/openapi.yaml) | Dự kiến |

### 10.3. Kiểm thử và vận hành

| Tài liệu | Trạng thái |
|---|---|
| [Kiểm thử](docs/testing.md) | Dự kiến |
| [Docker và hướng dẫn chạy](docs/docker.md) | Dự kiến |
| [CI/CD và triển khai](docs/ci-cd.md) | Dự kiến |
| [Caching](docs/caching.md) | Dự kiến |
| [Embedding và RAG](docs/rag.md) | Dự kiến |

## 11. Khởi chạy và cấu hình

Hiện chưa có phiên bản backend chạy được. Hướng dẫn thực thi sẽ được bổ sung khi có mã nguồn và cấu hình đã kiểm chứng.

Nội dung cần bổ sung:

- Yêu cầu phiên bản Node.js và các công cụ liên quan.
- Cài đặt dependency theo lockfile.
- Cấu hình biến môi trường.
- Khởi tạo database, chạy migration và seed.
- Khởi tạo ADMIN đầu tiên.
- Chạy backend bằng Node.js hoặc Docker Compose.
- Địa chỉ Swagger và health endpoint.

Không commit mật khẩu database, secret JWT hoặc API key thật.

## 12. Kiểm thử và CI/CD

### 12.1. Kiểm thử

Phạm vi kiểm thử mục tiêu:

- Validation, xác thực và phân quyền.
- Ràng buộc database.
- Nhập/bán thành công và các trường hợp không hợp lệ.
- Rollback khi lỗi.
- Xác nhận trùng.
- Hai đơn cùng tranh lượng tồn cuối.
- Sửa/hủy đồng thời với xác nhận.
- Cache hit/miss, invalidation và Redis không khả dụng.
- RAG trả lời có nguồn và xử lý thiếu căn cứ.

Kiểm thử transaction và concurrency phải sử dụng MySQL thật.

Hiện chưa có bộ test tự động; lệnh chạy và kết quả sẽ được bổ sung khi triển khai.

### 12.2. CI/CD

Pipeline mục tiêu:

1. Kiểm tra mã nguồn và cài dependency.
2. Chạy lint, unit test và integration test.
3. Build Docker image.
4. Gắn tag xác định được phiên bản.
5. Triển khai lên môi trường demo.
6. Kiểm tra sau triển khai.
7. Có phương án rollback phù hợp.

CI được dựng từ giai đoạn backend tối thiểu. Docker và CD được bổ sung sớm rồi hoàn thiện cùng tiến độ.

**Hiện chưa có pipeline hoặc môi trường demo hoạt động.** Build image thành công chưa được coi là hoàn thành CD.

## 13. Lộ trình phát triển

| Giai đoạn | Đầu ra chính |
|---|---|
| Yêu cầu và thiết kế | Phạm vi, nghiệp vụ, ERD, kiến trúc và hợp đồng API |
| Nền tảng | Express, kết nối MySQL, migration, seed và health endpoint |
| Kiểm thử và CI ban đầu | Test runner, test đầu tiên và workflow kiểm tra |
| Tài khoản và dữ liệu nền | Auth/users, danh mục, sản phẩm, nhà cung cấp |
| Nghiệp vụ giao dịch | Nhập/bán, transaction, tồn kho và lịch sử |
| Caching và RAG | Cache có invalidation; hỏi đáp có nguồn |
| Hoàn thiện và bàn giao | Kiểm thử hồi quy, triển khai, báo cáo, demo và release |

Testing, cập nhật tài liệu và CI/CD được thực hiện xuyên suốt, không chờ viết xong toàn bộ chức năng mới bắt đầu.

## 14. Quy trình làm việc

| Nhánh | Mục đích |
|---|---|
| `main` | Phiên bản đã được tích hợp |
| `doc` | Nhánh tài liệu ban đầu |
| `docs/*` | Các nhiệm vụ tài liệu tiếp theo |
| `feat/*` | Chức năng mới |
| `fix/*` | Sửa lỗi |
| `chore/*` | Cấu hình và bảo trì |

Quy trình thông thường:

1. Xác định nhiệm vụ và tiêu chí hoàn thành trong issue.
2. Tạo nhánh làm việc phù hợp.
3. Thay đổi code/tài liệu và bổ sung test liên quan.
4. Commit, push và mở pull request.
5. Review, kiểm tra và merge vào `main`.

Nguyên tắc:

- Nhánh mới thường bắt đầu từ `main` đã cập nhật.
- Nếu phụ thuộc nhánh chưa merge, cần ghi rõ trong PR.
- API hoặc schema thay đổi phải cập nhật tài liệu tương ứng.
- Khi CI được thiết lập, các kiểm tra bắt buộc phải thành công trước khi merge.
- Có thể xóa nhánh công việc sau khi merge.

Ví dụ commit:

```text
docs: bổ sung thiết kế API quản lý tài khoản
feat: triển khai xác nhận phiếu nhập
fix: ngăn trừ tồn khi xác nhận đơn trùng
```

## 15. Các quyết định còn mở

- Chính sách mật khẩu và thư viện băm.
- Xử lý sản phẩm ngừng kinh doanh trên chứng từ nháp.
- Quyền sửa giá thủ công trên đơn bán nháp.
- Chuẩn hóa email, SKU và tên danh mục.
- Đơn vị tiền tệ và quy tắc làm tròn.
- Lựa chọn embedding, LLM, vector store và lưu tài liệu.
- Môi trường demo và cấu hình triển khai.

Các quyết định được cập nhật trong tài liệu chuyên biệt và issue liên quan trước khi triển khai chức năng tương ứng.

## 16. Kết quả bàn giao dự kiến

- Source code backend và script/migration/seed.
- Tài liệu phạm vi, nghiệp vụ, database, kiến trúc và API.
- Dockerfile, Docker Compose và cấu hình mẫu.
- Bộ kiểm thử và bằng chứng các trường hợp nghiệp vụ quan trọng.
- Workflow CI/CD và bằng chứng triển khai demo.
- Caching và RAG được kiểm chứng.
- Hướng dẫn chạy, kiểm thử và triển khai.
- Báo cáo Project I, slide, kịch bản demo và release.

## 17. Giấy phép

Repository hiện chưa lựa chọn giấy phép mã nguồn mở.
