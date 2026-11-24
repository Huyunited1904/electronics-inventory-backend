# Electronics Inventory Backend

Backend quản lý bán hàng và tồn kho cho một cửa hàng đồ điện tử, được phát triển trong học phần **Project I — Đại học Bách khoa Hà Nội**.

Dự án hỗ trợ quản lý tài khoản nội bộ, danh mục, sản phẩm, nhà cung cấp, phiếu nhập, đơn bán và tồn kho. Bên cạnh nghiệp vụ, dự án thực hành quy trình phát triển backend từ phân tích yêu cầu, thiết kế database/API đến kiểm thử, đóng gói và triển khai.

**CI/CD là nội dung trọng tâm**, cùng với quản lý mã nguồn, kiến trúc phần mềm, caching, embedding và RAG.

> **Trạng thái hiện tại:** Đã soạn bộ tài liệu thiết kế và hợp đồng API ban đầu; đang rà soát, đồng bộ và hoàn thiện các quyết định còn mở. Backend, migration, kiểm thử tự động và pipeline CI/CD chưa được triển khai.

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

Hệ thống phục vụ hoạt động nội bộ. Khách hàng không đăng ký tài khoản hoặc trực tiếp đặt hàng qua hệ thống trong phiên bản này.

## 2. Mục tiêu

### 2.1. Nghiệp vụ

- Quản lý danh mục, sản phẩm và nhà cung cấp.
- Ghi nhận nhập hàng và bán hàng thông qua chứng từ.
- Theo dõi số lượng tồn và lịch sử nhập/xuất.
- Phân quyền theo vai trò và người tạo chứng từ.
- Bảo đảm tồn kho nhất quán khi xử lý nhiều yêu cầu đồng thời.
- Hỗ trợ tra cứu hướng dẫn và quy trình bằng hỏi đáp có nguồn.

### 2.2. Kỹ thuật

- Tổ chức backend theo module và phân chia trách nhiệm rõ ràng.
- Quản lý mã nguồn, tài liệu và thay đổi bằng Git/GitHub.
- Xây dựng kiểm thử tự động cho các nghiệp vụ quan trọng.
- Đóng gói hệ thống bằng Docker và Docker Compose.
- Xây dựng pipeline CI/CD có kiểm thử, build và triển khai thực tế.
- Sử dụng Redis caching với TTL và cơ chế invalidation.
- Tích hợp embedding và RAG để hỏi đáp tài liệu.
- Có tài liệu giúp người khác khởi chạy, kiểm thử và đánh giá dự án.

Các mục tiêu trên là phạm vi cần thực hiện, không phải tuyên bố các chức năng đã được triển khai.

## 3. Phạm vi chức năng

### 3.1. Các nhóm chức năng

| Nhóm | Nội dung |
|---|---|
| Tài khoản | Đăng nhập, đổi mật khẩu, cấp, xem và khóa tài khoản nhân viên |
| Danh mục | Xem, thêm và sửa danh mục sản phẩm |
| Sản phẩm | Thêm, sửa, tra cứu, tìm kiếm, lọc, phân trang và ngừng kinh doanh |
| Nhà cung cấp | Xem, tìm kiếm, thêm và sửa thông tin |
| Nhập hàng | Tạo phiếu nháp, quản lý dòng hàng, hủy và xác nhận nhập |
| Bán hàng | Tạo đơn nháp, quản lý dòng hàng, hủy và xác nhận bán |
| Tồn kho | Xem, tìm kiếm và lọc số lượng tồn hiện tại |
| Lịch sử nhập/xuất | Tra cứu biến động từ chứng từ đã xác nhận |
| Tài liệu tri thức | Nạp, xem trạng thái xử lý và xóa tài liệu |
| Hỏi đáp RAG | Giải thích hướng dẫn sử dụng hệ thống và quy trình nghiệp vụ, kèm nguồn tham chiếu |

Sản phẩm mẫu gồm laptop, màn hình, chuột, bàn phím và tai nghe. Mỗi mẫu hoặc phiên bản bán riêng được quản lý bằng một SKU.

### 3.2. Vai trò người dùng

| Vai trò | Phạm vi trách nhiệm |
|---|---|
| **ADMIN** | Quản lý tài khoản, dữ liệu nền, tài liệu RAG và thao tác trên chứng từ của cửa hàng |
| **STAFF** | Tra cứu thông tin, hỏi đáp tài liệu, tạo và xử lý chứng từ do mình tạo |

Nguyên tắc phân quyền:

- Tài khoản nhân viên do ADMIN cấp; không có đăng ký công khai.
- ADMIN và STAFF được xem chứng từ, tồn kho và lịch sử nhập/xuất theo hợp đồng API.
- STAFF chỉ được sửa, hủy và xác nhận chứng từ do mình tạo.
- ADMIN vẫn phải tuân thủ quy tắc trạng thái, điều kiện sản phẩm và kiểm tra tồn kho.
- Chứng từ đã xác nhận không được sửa hoặc hủy.
- Không vai trò nào được sửa trực tiếp số lượng tồn kho.
- Kho tri thức dùng chung chỉ chứa nội dung được phép cung cấp cho cả ADMIN và STAFF.

### 3.3. Ngoài phạm vi

Phiên bản này không triển khai:

- Frontend.
- Tài khoản khách hàng, giỏ hàng hoặc khách tự đặt hàng.
- Nhiều cửa hàng, nhiều kho và điều chuyển kho.
- Quản lý từng thiết bị bằng serial/IMEI.
- Thanh toán trực tuyến, công nợ và kế toán.
- Vận chuyển và giao hàng.
- Đổi trả, hoàn tiền hoặc hủy chứng từ đã xác nhận.
- Tiếp nhận và xử lý hồ sơ bảo hành.
- Kiểm kê, điều chỉnh tồn thủ công hoặc xử lý hàng hỏng.
- Báo cáo kinh doanh nâng cao.
- Microservices hoặc hệ thống phân tán quy mô lớn.

Các giới hạn cụ thể của từng chức năng được mô tả trong tài liệu API.

Chi tiết tại [Phạm vi dự án](docs/scope.md).

## 4. Luồng nghiệp vụ chính

### 4.1. Nhập hàng

1. Người dùng chọn nhà cung cấp và tạo phiếu nhập ở trạng thái `DRAFT`.
2. Thêm sản phẩm, số lượng và giá nhập vào từng dòng.
3. Gửi yêu cầu xác nhận phiếu.
4. Hệ thống kiểm tra quyền, trạng thái phiếu và tính hợp lệ của các mặt hàng.
5. Nếu hợp lệ, cộng tồn, lưu người/thời điểm xác nhận và chuyển phiếu sang `CONFIRMED` trong cùng một transaction.
6. Nếu có lỗi, rollback toàn bộ lần xác nhận.

Phiếu nháp chưa làm tăng tồn kho. Một phiếu có thể chứa nhiều mặt hàng và chỉ được tác động tới tồn kho tối đa một lần.

### 4.2. Bán hàng

1. Người dùng tạo đơn bán ở trạng thái `DRAFT`.
2. Thêm sản phẩm và số lượng.
3. Backend lấy giá bán hiện tại của sản phẩm và lưu vào dòng đơn.
4. Khi xác nhận, hệ thống kiểm tra quyền, trạng thái sản phẩm và tồn kho hiện tại.
5. Nếu tất cả mặt hàng hợp lệ và đủ tồn, trừ tồn và chuyển đơn sang `CONFIRMED` trong cùng một transaction.
6. Nếu thiếu bất kỳ mặt hàng nào hoặc có lỗi, từ chối toàn bộ lần xác nhận.

Đơn nháp không trừ tồn và không giữ hàng. Số lượng lưu trên đơn nháp có thể lớn hơn tồn hiện tại; điều kiện đủ tồn được kiểm tra bắt buộc khi xác nhận.

Không cho phép client nhập hoặc chỉnh giá bán thủ công trên dòng đơn.

### 4.3. Trạng thái chứng từ

| Trạng thái | Ý nghĩa | Thao tác tiếp theo |
|---|---|---|
| `DRAFT` | Chứng từ đang được chuẩn bị | Sửa, hủy hoặc xác nhận khi hợp lệ |
| `CONFIRMED` | Đã xác nhận và cập nhật tồn | Tra cứu |
| `CANCELLED` | Đã hủy khi còn nháp | Tra cứu |

Không hỗ trợ khôi phục chứng từ đã hủy hoặc đảo ngược chứng từ đã xác nhận.

### 4.4. Các yêu cầu nhất quán quan trọng

- Số lượng trên dòng chứng từ phải là số nguyên dương.
- Giá không âm; tồn kho không âm.
- Một sản phẩm chỉ xuất hiện một dòng trong cùng chứng từ.
- Cập nhật tồn và thông tin xác nhận phải nằm trong cùng transaction.
- Có lỗi thì rollback toàn bộ thay đổi của lần xác nhận.
- Không cộng hoặc trừ tồn hai lần khi gửi lại yêu cầu xác nhận.
- Không bán vượt tồn khi có nhiều yêu cầu đồng thời.
- Giá bán đã lưu không tự đổi khi sửa giá sản phẩm, sửa số lượng hoặc xác nhận đơn.
- Tồn ban đầu bằng 0; hàng có sẵn được ghi nhận qua phiếu nhập.
- Không thêm sản phẩm đã ngừng kinh doanh vào chứng từ nháp.
- Chặn xác nhận toàn bộ chứng từ còn chứa sản phẩm đã ngừng kinh doanh.
- Chứng từ đã xác nhận trước thời điểm sản phẩm ngừng kinh doanh được giữ nguyên.

Chi tiết tại [Quy tắc nghiệp vụ](docs/business-rules.md).

## 5. Công nghệ

| Thành phần | Lựa chọn cho thiết kế và triển khai |
|---|---|
| Backend | Node.js, Express |
| Database | MySQL, InnoDB |
| Cache | Redis |
| Xác thực | JWT access token |
| Băm mật khẩu | Argon2id qua thư viện `argon2` |
| Tài liệu API | Markdown, OpenAPI/Swagger |
| Kiểm thử | Jest hoặc Vitest, kết hợp Supertest; chưa chốt test runner |
| Đóng gói | Docker, Docker Compose |
| CI/CD | GitHub Actions |
| Quản lý phiên bản | Git |
| Repository hosting | GitHub |
| Embedding, LLM và vector store | Chưa chốt giải pháp cụ thể |
| Môi trường demo | Chưa chốt |

Phiên bản công cụ và các lựa chọn còn mở được cập nhật khi thiết lập môi trường.

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

Đổi mật khẩu được mô tả cùng API xác thực trong `docs/api/auth.md`; tài liệu API và module mã nguồn không bắt buộc ánh xạ một-một.

### 6.1. Phân chia trách nhiệm

- **Router:** Khai báo endpoint và gắn middleware.
- **Validator:** Kiểm tra cấu trúc, kiểu và giới hạn dữ liệu đầu vào.
- **Controller:** Nhận dữ liệu hợp lệ, gọi Service và trả response.
- **Service:** Xử lý nghiệp vụ, kiểm tra quyền theo tài nguyên và điều phối transaction.
- **Repository:** Thực hiện truy vấn database.
- **Middleware:** Xác thực, kiểm tra quyền chung, logging và xử lý lỗi.

### 6.2. Phối hợp giữa các module

- Các module gọi hàm nội bộ, không gọi HTTP lẫn nhau.
- `purchasing` và `sales` điều phối cập nhật tồn kho khi xác nhận.
- `catalog` tạo sản phẩm và dòng tồn ban đầu trong cùng transaction.
- Các Repository tham gia một transaction phải dùng cùng một connection.
- Không giữ transaction database trong lúc chờ embedding hoặc LLM.

Các module không được triển khai thành các microservice riêng trong phạm vi hiện tại.

Chi tiết tại [Kiến trúc hệ thống](docs/architecture.md).

## 7. Database và xác thực

### 7.1. Database nghiệp vụ

Thiết kế hiện tại gồm 9 bảng nghiệp vụ:

| Nhóm | Bảng |
|---|---|
| Tài khoản | `users` |
| Dữ liệu nền | `categories`, `products`, `suppliers` |
| Tồn kho | `inventory` |
| Nhập hàng | `stock_receipts`, `stock_receipt_items` |
| Bán hàng | `sales_orders`, `sales_order_items` |

Không sử dụng bảng `stock_movements`. Lịch sử nhập/xuất được tổng hợp từ chứng từ đã xác nhận và các dòng chi tiết.

Thông tin tài liệu, trạng thái xử lý, tác vụ nền và vector phục vụ RAG cần thiết kế lưu trữ bổ sung riêng. Chín bảng trên không phải toàn bộ nhu cầu lưu trữ của module knowledge.

ERD mô tả cấu trúc và quan hệ. Migration phải triển khai đầy đủ CHECK, UNIQUE, NOT NULL, FOREIGN KEY và chính sách cập nhật/xóa đã quy định; ghi chú trong DBML không thay thế ràng buộc thực thi.

Tham khảo:

- [Thiết kế database](docs/database.md)
- [Mã nguồn ERD](docs/diagrams/erd.dbml)
- [Thư mục sơ đồ](docs/diagrams/)

### 7.2. Chính sách JWT

- Access token có thời hạn 1 giờ.
- Chưa triển khai refresh token.
- Request được bảo vệ gửi `Authorization: Bearer <access_token>`.
- Backend xác minh token rồi đọc tài khoản trong MySQL để kiểm tra trạng thái và vai trò hiện tại.
- Request kiểm tra trạng thái sau khi thao tác khóa tài khoản được commit phải bị từ chối.
- Thao tác khóa không tự hủy request đã vượt qua kiểm tra trước đó.
- Đổi mật khẩu không thu hồi token cũ; token còn hiệu lực tới khi hết hạn nếu tài khoản vẫn hoạt động.

### 7.3. Chính sách mật khẩu

- Mật khẩu mới có độ dài từ 15 đến 64 Unicode code point.
- Không tự trim, biến đổi hoặc cắt ngắn mật khẩu.
- Không bắt buộc tổ hợp chữ hoa, chữ thường, số và ký tự đặc biệt.
- Mỗi nhân viên được cấp mật khẩu ban đầu ngẫu nhiên riêng.
- ADMIN chuẩn bị và bàn giao mật khẩu; API tạo tài khoản không trả mật khẩu trong response.
- Chưa bắt buộc đổi mật khẩu ở lần đăng nhập đầu tiên.
- Sử dụng Argon2id qua thư viện `argon2`.
- Chỉ lưu chuỗi hash mã hóa đầy đủ trong `users.password_hash`.

Cấu hình băm, validation và hành vi đổi mật khẩu được quy định tại [API đăng nhập và đổi mật khẩu](docs/api/auth.md).

## 8. Caching và hỏi đáp tài liệu

### 8.1. Redis caching

Thiết kế áp dụng cache-aside cho dữ liệu đọc phù hợp, bắt đầu từ danh mục sản phẩm.

- Có TTL và xóa cache liên quan sau khi cập nhật database thành công.
- MySQL là nguồn dữ liệu nghiệp vụ chính.
- Khi Redis lỗi, luồng đọc phù hợp có thể chuyển sang MySQL.
- Cần kiểm thử cache hit/miss, invalidation và dữ liệu cũ.
- Không dùng cache làm căn cứ quyết định đủ tồn hoặc trạng thái chứng từ khi xác nhận.
- API tồn kho và lịch sử ban đầu đọc trực tiếp từ database.

Chi tiết sẽ được bổ sung tại [Thiết kế caching](docs/caching.md).

### 8.2. Embedding và RAG

Kho tri thức phục vụ hướng dẫn sử dụng hệ thống và quy trình nhập hàng, bán hàng, tra cứu tồn kho.

Luồng dự kiến:

1. ADMIN nạp nội dung Markdown hoặc văn bản thuần qua JSON.
2. Hệ thống lưu nội dung và tiếp nhận tác vụ xử lý nền.
3. Chuẩn hóa văn bản, chia đoạn, tạo embedding và lập chỉ mục.
4. Chỉ tài liệu xử lý thành công và chưa bị xóa được dùng để trả lời.
5. ADMIN hoặc STAFF gửi một câu hỏi độc lập.
6. Hệ thống truy xuất các đoạn liên quan và sinh câu trả lời kèm nguồn.
7. Nếu thiếu căn cứ hoặc câu hỏi ngoài phạm vi, hệ thống nói rõ giới hạn.

ADMIN được xem trạng thái xử lý và xóa tài liệu. Kho tri thức dùng chung nên nội dung phải phù hợp để cung cấp cho cả ADMIN và STAFF.

RAG không:

- Truy vấn số liệu tức thời.
- Kiểm tra trực tiếp trạng thái của một chứng từ cụ thể.
- Tự tạo, sửa, hủy hoặc xác nhận chứng từ.
- Chạy SQL hoặc thực thi chỉ dẫn nằm trong tài liệu.

Phiên bản đầu chưa hỗ trợ PDF, DOCX, OCR, hội thoại nhiều lượt hoặc tự động đồng bộ tài liệu từ GitHub.

Tham khảo:

- [API tài liệu và hỏi đáp RAG](docs/api/knowledge.md)
- [Thiết kế embedding và RAG](docs/rag.md) — dự kiến bổ sung

Caching và RAG chưa được triển khai.

## 9. Tổ chức repository

Cấu trúc dưới đây là định hướng; không phải toàn bộ đường dẫn đã tồn tại.

| Đường dẫn | Nội dung |
|---|---|
| `README.md` | Giới thiệu và điều hướng tài liệu |
| `.gitignore` | Quy định file/thư mục không theo dõi |
| `docs/` | Phạm vi, nghiệp vụ và thiết kế |
| `docs/api/` | Hợp đồng API theo module |
| `docs/diagrams/` | Mã và ảnh sơ đồ |
| `src/app.js` | Tạo Express app và gắn middleware/router |
| `src/server.js` | Khởi động server và quản lý vòng đời |
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

**Đã soạn** thể hiện tài liệu đã được xây dựng, không đồng nghĩa chức năng đã được triển khai hoặc kiểm thử. **Dự kiến** là tài liệu sẽ bổ sung sau.

Liên kết tương đối trỏ tới tài liệu trên nhánh đang xem. Nếu README trên `main` liên kết tới tài liệu mới chỉ có ở nhánh làm việc, liên kết sẽ hoạt động sau khi tài liệu được merge vào `main`.

Các liên kết tới tài liệu dự kiến chưa tạo sẽ chưa mở được.

### 10.1. Phạm vi và thiết kế

| Tài liệu | Trạng thái |
|---|---|
| [Phạm vi dự án](docs/scope.md) | Đã soạn |
| [Quy tắc nghiệp vụ](docs/business-rules.md) | Đã soạn; còn một số quyết định mở |
| [Thiết kế database](docs/database.md) | Đã soạn; lưu trữ RAG thiết kế riêng |
| [Kiến trúc backend](docs/architecture.md) | Đã soạn |
| [Mã nguồn ERD](docs/diagrams/erd.dbml) | Đã tạo |
| [Thư mục sơ đồ](docs/diagrams/) | Chứa mã nguồn và tài liệu sơ đồ liên quan |

### 10.2. Hợp đồng API

| Tài liệu | Trạng thái |
|---|---|
| [Đăng nhập và đổi mật khẩu](docs/api/auth.md) | Đã soạn; đã đồng bộ chính sách mật khẩu |
| [Quản lý tài khoản](docs/api/users.md) | Đã soạn |
| [Danh mục và sản phẩm](docs/api/catalog.md) | Đã soạn |
| [Nhà cung cấp](docs/api/suppliers.md) | Đã soạn |
| [Nhập hàng](docs/api/purchasing.md) | Đã soạn |
| [Bán hàng](docs/api/sales.md) | Đã soạn |
| [Tồn kho và lịch sử](docs/api/inventory.md) | Đã soạn |
| [Tài liệu và hỏi đáp RAG](docs/api/knowledge.md) | Đã soạn; còn thiết kế kỹ thuật cần hoàn thiện |
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
- Các lệnh kiểm thử.

Không commit mật khẩu database, khóa ký JWT hoặc API key thật.

## 12. Kiểm thử và CI/CD

### 12.1. Kiểm thử

Phạm vi kiểm thử mục tiêu:

- Validation, xác thực và phân quyền.
- Ràng buộc database.
- Nhập/bán thành công và các trường hợp không hợp lệ.
- Sản phẩm ngừng kinh doanh trên chứng từ nháp.
- Lưu giá bán và tính tiền chính xác.
- Rollback khi lỗi.
- Xác nhận trùng.
- Hai đơn cùng tranh lượng tồn cuối.
- Sửa/hủy đồng thời với xác nhận.
- Tính nhất quán của tồn kho và lịch sử.
- Cache hit/miss, invalidation và Redis không khả dụng.
- RAG trả lời có nguồn, thiếu căn cứ và câu hỏi ngoài phạm vi.
- Xóa tài liệu và xử lý tài liệu đồng thời.

Kiểm thử transaction và concurrency phải sử dụng MySQL thật, không chỉ mock database.

Test tự động thông thường sử dụng mock hoặc stub tại adapter AI để tránh gọi dịch vụ trả phí ngoài dự kiến. Đánh giá RAG thực tế được chạy riêng có kiểm soát.

Hiện chưa có bộ test tự động; lệnh chạy và kết quả sẽ được bổ sung khi triển khai.

### 12.2. Continuous Integration

Pipeline CI được xây dựng từ giai đoạn backend tối thiểu và mở rộng cùng tiến độ:

1. Lấy mã nguồn.
2. Thiết lập Node.js và cài dependency theo lockfile.
3. Chạy lint.
4. Chạy unit/API test.
5. Chuẩn bị database kiểm thử, chạy migration và integration test khi có.
6. Build Docker image khi đã có cấu hình đóng gói.

Các kiểm tra bắt buộc phải thành công trước khi merge khi quy trình CI đã được thiết lập.

### 12.3. Continuous Delivery/Deployment

Sau khi phiên bản đáp ứng điều kiện phát hành:

1. Sử dụng image đã kiểm tra hoặc build image theo quy trình.
2. Gắn tag xác định được phiên bản hoặc commit.
3. Đưa image tới registry.
4. Triển khai cùng migration theo thứ tự đã kiểm tra về tương thích.
5. Kiểm tra health và luồng chức năng cơ bản.
6. Ghi nhận kết quả triển khai.
7. Có phương án rollback phù hợp.

Rollback ứng dụng không mặc định hoàn tác được migration database.

**Hiện chưa có pipeline hoặc môi trường demo hoạt động.** Build image thành công chưa được coi là hoàn thành CD.

## 13. Lộ trình phát triển

| Giai đoạn | Đầu ra chính |
|---|---|
| Yêu cầu và thiết kế | Phạm vi, nghiệp vụ, ERD, kiến trúc và hợp đồng API |
| Backend tối thiểu | Express app, cấu hình, xử lý lỗi và health endpoint |
| Kiểm thử và CI ban đầu | Test runner, test đầu tiên và workflow kiểm tra |
| Database | Kết nối MySQL, migration, seed và khởi tạo ADMIN |
| Tài khoản và dữ liệu nền | Auth, users, danh mục, sản phẩm và nhà cung cấp |
| Nghiệp vụ nhập hàng | Phiếu nhập, transaction và tăng tồn |
| Nghiệp vụ bán hàng | Đơn bán, kiểm tra đủ hàng và giảm tồn an toàn |
| Tra cứu tồn kho | Danh sách tồn và lịch sử nhập/xuất |
| Caching | Cache-aside, TTL, invalidation và kiểm thử lỗi |
| Knowledge/RAG | Lưu trữ tri thức, xử lý nền, embedding và hỏi đáp có nguồn |
| Hoàn thiện và bàn giao | Kiểm thử hồi quy, báo cáo, demo và release |

Docker và CD được bổ sung sớm khi có backend chạy được, sau đó hoàn thiện cùng tiến độ.

Testing, cập nhật tài liệu và CI/CD được thực hiện xuyên suốt; không chờ viết xong toàn bộ chức năng mới bắt đầu.

## 14. Quy trình làm việc

| Nhánh | Mục đích |
|---|---|
| `main` | Phiên bản đã được tích hợp |
| `docs/*` | Thiết kế và cập nhật tài liệu |
| `feat/*` | Chức năng mới |
| `fix/*` | Sửa lỗi |
| `chore/*` | Cấu hình và bảo trì |

Quy trình thông thường:

1. Xác định nhiệm vụ và tiêu chí hoàn thành trong issue.
2. Tạo nhánh làm việc phù hợp.
3. Thay đổi code/tài liệu và bổ sung test liên quan.
4. Chạy các kiểm tra tại máy.
5. Commit, push và mở pull request.
6. Review, kiểm tra kết quả CI khi có và merge vào `main`.

Nguyên tắc:

- Nhánh mới thường bắt đầu từ `main` đã cập nhật.
- Nếu phụ thuộc nhánh chưa merge, ghi rõ trong PR.
- API hoặc schema thay đổi phải cập nhật tài liệu tương ứng.
- Không coi ví dụ trong tài liệu là bằng chứng chức năng đã chạy.
- Có thể xóa nhánh công việc sau khi merge.

Ví dụ commit:

```text
docs: đồng bộ quy tắc nghiệp vụ với thiết kế API
feat: triển khai xác nhận phiếu nhập
fix: ngăn trừ tồn khi xác nhận đơn trùng
chore: thiết lập workflow kiểm thử backend
```

## 15. Các quyết định còn mở

Các nội dung cần hoàn thiện trước khi triển khai phần liên quan:

- Chuẩn hóa email, SKU, tên danh mục và lựa chọn collation.
- Đơn vị tiền tệ và việc sử dụng phần lẻ.
- Cơ chế khởi tạo ADMIN đầu tiên.
- Thuật toán ký JWT và quản lý khóa ký.
- Giới hạn tần suất và dung lượng request.
- Số dòng tối đa của phiếu nhập và đơn bán.
- Thời gian chờ khóa và chính sách retry khi giao dịch xung đột.
- Test runner và phiên bản công cụ.
- TTL, cache key và cơ chế xử lý dữ liệu cũ.
- Lưu trữ tài liệu, tác vụ nền, embedding, LLM và vector store.
- Môi trường demo, registry và cấu hình triển khai.

Các nội dung đã chốt gồm:

- Chính sách mật khẩu và Argon2id.
- JWT access token 1 giờ và chưa có refresh token.
- Xử lý sản phẩm ngừng kinh doanh trong chứng từ.
- Lưu giá bán khi thêm dòng và không chỉnh giá thủ công.
- Đơn nháp không giữ tồn; xác nhận phải đủ tồn cho toàn bộ đơn.
- Phạm vi RAG là hướng dẫn hệ thống và quy trình nghiệp vụ.

Quyết định chi tiết được cập nhật trong tài liệu chuyên biệt và issue liên quan trước khi triển khai chức năng tương ứng.

## 16. Kết quả bàn giao dự kiến

- Source code backend và migration/seed.
- Tài liệu phạm vi, nghiệp vụ, database, kiến trúc và API.
- Đặc tả OpenAPI/Swagger.
- Dockerfile, Docker Compose và cấu hình mẫu.
- Bộ kiểm thử và bằng chứng các trường hợp nghiệp vụ quan trọng.
- Workflow CI/CD và bằng chứng triển khai demo.
- Caching có minh chứng hoạt động và xử lý invalidation.
- RAG có bộ tài liệu mẫu, câu hỏi đánh giá và nguồn trích dẫn.
- Hướng dẫn chạy, kiểm thử và triển khai.
- Báo cáo Project I, slide, kịch bản demo và release.

## 17. Giấy phép

Repository hiện chưa lựa chọn giấy phép mã nguồn mở.
