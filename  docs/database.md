# Thiết kế cơ sở dữ liệu

## 1. Mục đích và phạm vi

Tài liệu mô tả cơ sở dữ liệu nghiệp vụ của backend quản lý bán hàng và tồn kho đồ điện tử.

Thiết kế phục vụ một cửa hàng, một kho và hai vai trò ADMIN/STAFF.

Tài liệu là căn cứ để:

- Viết script tạo bảng và migration.
- Thiết kế truy vấn trong Repository.
- Triển khai các nghiệp vụ và transaction.
- Kiểm thử ràng buộc và tính nhất quán dữ liệu.

Tài liệu liên quan:

- [Phạm vi dự án](scope.md)
- [Quy tắc nghiệp vụ](business-rules.md)
- [Mã nguồn ERD](diagrams/erd.dbml)
- [Ảnh ERD](diagrams/erd.png)

Thiết kế hiện tại gồm 9 bảng nghiệp vụ. Các bảng phục vụ tài liệu, embedding và RAG sẽ được thiết kế riêng sau.

## 2. Quy ước thiết kế

### 2.1. Công nghệ và cách đặt tên

- Hệ quản trị: MySQL.
- Storage engine: InnoDB để hỗ trợ transaction và khóa ngoại.
- Sử dụng bộ ký tự `utf8mb4`.
- Tên bảng và cột dùng tiếng Anh, định dạng `snake_case`.
- Các cột khóa ngoại có cùng kiểu dữ liệu với khóa chính được tham chiếu.

Phiên bản MySQL và collation cụ thể được chốt khi thiết lập môi trường. Cần chọn phiên bản thực thi được các ràng buộc `CHECK` trong script.

### 2.2. Kiểu dữ liệu

| Loại dữ liệu | Quy ước |
|---|---|
| ID | `INT`, khóa chính tự tăng, trừ `inventory.product_id` |
| Số lượng | `INT` |
| Giá tiền | `DECIMAL(15,2)` |
| Thời điểm | `DATETIME` |
| Trạng thái bật/tắt | `BOOLEAN` |
| Vai trò và trạng thái chứng từ | `ENUM` |
| Chuỗi | `VARCHAR` với độ dài theo từng cột |

Độ dài chuỗi và độ chính xác của giá tiền là lựa chọn thiết kế ban đầu, có thể điều chỉnh bằng migration nếu yêu cầu thay đổi.

Không dùng `FLOAT` hoặc `DOUBLE` để lưu giá tiền.

### 2.3. Giá trị rỗng và thời gian

- PK: khóa chính.
- FK: khóa ngoại.
- UNIQUE: giá trị hoặc tổ hợp giá trị không được trùng.
- NOT NULL: bắt buộc có giá trị.
- `NULL` thể hiện thông tin chưa có; không thay bằng chuỗi rỗng hoặc ID giả.
- `created_at` được gán thời điểm tạo.
- `updated_at` được cập nhật khi bản ghi thay đổi.
- Thời điểm được lưu theo quy ước UTC và chuyển đổi khi hiển thị.
- Cấu hình timezone của ứng dụng và kết nối database phải thống nhất.

## 3. Danh sách bảng

| Bảng | Chức năng |
|---|---|
| `users` | Tài khoản và vai trò người dùng |
| `categories` | Danh mục sản phẩm |
| `products` | Thông tin và giá bán hiện tại của sản phẩm |
| `suppliers` | Thông tin nhà cung cấp |
| `inventory` | Số lượng tồn hiện tại của từng sản phẩm |
| `stock_receipts` | Thông tin chung của phiếu nhập |
| `stock_receipt_items` | Các mặt hàng trong phiếu nhập |
| `sales_orders` | Thông tin chung của đơn bán |
| `sales_order_items` | Các mặt hàng trong đơn bán |

## 4. Thiết kế chi tiết

### 4.1. Bảng `users`

Lưu tài khoản nội bộ của quản trị viên và nhân viên.

| Cột | Kiểu dữ liệu | Ràng buộc / mặc định | Ý nghĩa |
|---|---|---|---|
| `id` | INT | PK, AUTO_INCREMENT | ID tài khoản |
| `full_name` | VARCHAR(100) | NOT NULL | Họ tên |
| `email` | VARCHAR(255) | NOT NULL, UNIQUE | Email đăng nhập |
| `password_hash` | VARCHAR(255) | NOT NULL | Mật khẩu đã băm |
| `role` | ENUM('ADMIN','STAFF') | NOT NULL, mặc định `STAFF` | Vai trò |
| `active` | BOOLEAN | NOT NULL, mặc định TRUE | Trạng thái hoạt động |
| `created_at` | DATETIME | NOT NULL, thời điểm tạo | Thời điểm cấp tài khoản |
| `updated_at` | DATETIME | NOT NULL, cập nhật khi sửa | Thời điểm cập nhật |

Lưu ý:

- Không lưu mật khẩu gốc.
- `active = FALSE` thể hiện tài khoản bị khóa.
- Không xóa tài khoản đã được tham chiếu bởi chứng từ.
- Trạng thái khóa phải được kiểm tra khi xử lý yêu cầu; cột `active` tự nó không thu hồi quyền truy cập.

### 4.2. Bảng `categories`

Lưu danh mục như laptop, màn hình, chuột, bàn phím và tai nghe.

| Cột | Kiểu dữ liệu | Ràng buộc | Ý nghĩa |
|---|---|---|---|
| `id` | INT | PK, AUTO_INCREMENT | ID danh mục |
| `name` | VARCHAR(100) | NOT NULL, UNIQUE | Tên danh mục |

Không xóa danh mục còn sản phẩm liên kết, kể cả sản phẩm đã ngừng kinh doanh.

### 4.3. Bảng `products`

Lưu thông tin sản phẩm. Mỗi mẫu hoặc phiên bản bán riêng được nhận diện bằng một SKU.

| Cột | Kiểu dữ liệu | Ràng buộc / mặc định | Ý nghĩa |
|---|---|---|---|
| `id` | INT | PK, AUTO_INCREMENT | ID sản phẩm |
| `sku` | VARCHAR(50) | NOT NULL, UNIQUE | Mã hàng |
| `name` | VARCHAR(255) | NOT NULL | Tên sản phẩm |
| `category_id` | INT | NOT NULL, FK → `categories.id` | Danh mục |
| `sale_price` | DECIMAL(15,2) | NOT NULL, CHECK ≥ 0 | Giá bán hiện tại |
| `active` | BOOLEAN | NOT NULL, mặc định TRUE | Còn kinh doanh |
| `created_at` | DATETIME | NOT NULL, thời điểm tạo | Thời điểm thêm sản phẩm |
| `updated_at` | DATETIME | NOT NULL, cập nhật khi sửa | Thời điểm cập nhật |

Lưu ý:

- Số lượng tồn được lưu ở `inventory`.
- Giá nhập được lưu ở từng dòng `stock_receipt_items`.
- Khi ngừng kinh doanh, cập nhật `active = FALSE`.
- Không xóa sản phẩm đã xuất hiện trong chứng từ.
- Thay đổi `sale_price` không cập nhật giá đã lưu trên đơn bán.

### 4.4. Bảng `suppliers`

Lưu thông tin nhà cung cấp.

| Cột | Kiểu dữ liệu | Ràng buộc / mặc định | Ý nghĩa |
|---|---|---|---|
| `id` | INT | PK, AUTO_INCREMENT | ID nhà cung cấp |
| `name` | VARCHAR(255) | NOT NULL | Tên nhà cung cấp |
| `phone` | VARCHAR(20) | Cho phép NULL | Số điện thoại |
| `email` | VARCHAR(255) | Cho phép NULL | Email liên hệ |
| `address` | VARCHAR(500) | Cho phép NULL | Địa chỉ |
| `created_at` | DATETIME | NOT NULL, thời điểm tạo | Thời điểm thêm |
| `updated_at` | DATETIME | NOT NULL, cập nhật khi sửa | Thời điểm cập nhật |

Lưu ý:

- Điện thoại dùng kiểu chuỗi để giữ số 0 đầu và các ký tự liên hệ.
- Không đặt UNIQUE cho tên, điện thoại hoặc email khi chưa có yêu cầu nghiệp vụ tương ứng.
- Không xóa nhà cung cấp đã có phiếu nhập.
- Không gắn cố định nhà cung cấp vào sản phẩm; nguồn nhập được xác định qua phiếu nhập.

### 4.5. Bảng `inventory`

Lưu số lượng tồn hiện tại của mỗi sản phẩm.

| Cột | Kiểu dữ liệu | Ràng buộc / mặc định | Ý nghĩa |
|---|---|---|---|
| `product_id` | INT | PK, FK → `products.id` | Sản phẩm |
| `quantity` | INT | NOT NULL, mặc định 0, CHECK ≥ 0 | Số lượng tồn |
| `updated_at` | DATETIME | NOT NULL, cập nhật khi sửa | Lần thay đổi tồn gần nhất |

Lưu ý:

- Không cần cột `id` riêng.
- Khóa chính `product_id` bảo đảm mỗi sản phẩm có tối đa một dòng tồn.
- Backend phải tạo sản phẩm và dòng tồn bằng 0 trong cùng transaction để bảo đảm mỗi sản phẩm có đúng một dòng tồn.
- Không cung cấp chức năng sửa trực tiếp số tồn.
- Tồn chỉ thay đổi qua xác nhận phiếu nhập hoặc đơn bán.

### 4.6. Bảng `stock_receipts`

Lưu thông tin chung của phiếu nhập.

| Cột | Kiểu dữ liệu | Ràng buộc / mặc định | Ý nghĩa |
|---|---|---|---|
| `id` | INT | PK, AUTO_INCREMENT | ID phiếu nhập |
| `supplier_id` | INT | NOT NULL, FK → `suppliers.id` | Nhà cung cấp |
| `created_by` | INT | NOT NULL, FK → `users.id` | Người tạo |
| `status` | ENUM('DRAFT','CONFIRMED','CANCELLED') | NOT NULL, mặc định `DRAFT` | Trạng thái |
| `created_at` | DATETIME | NOT NULL, thời điểm tạo | Thời điểm tạo phiếu |
| `confirmed_by` | INT | Cho phép NULL, FK → `users.id` | Người xác nhận |
| `confirmed_at` | DATETIME | Cho phép NULL | Thời điểm xác nhận |

Quy tắc dữ liệu:

- `DRAFT` hoặc `CANCELLED`: `confirmed_by` và `confirmed_at` đều NULL.
- `CONFIRMED`: cả hai cột xác nhận đều phải có giá trị.
- Người tạo và người xác nhận có thể giống nhau.
- Hủy phiếu không xóa bản ghi.
- Phiếu đã xác nhận không được sửa hoặc hủy.

### 4.7. Bảng `stock_receipt_items`

Lưu từng mặt hàng trong phiếu nhập.

| Cột | Kiểu dữ liệu | Ràng buộc | Ý nghĩa |
|---|---|---|---|
| `id` | INT | PK, AUTO_INCREMENT | ID dòng chi tiết |
| `receipt_id` | INT | NOT NULL, FK → `stock_receipts.id` | Phiếu nhập |
| `product_id` | INT | NOT NULL, FK → `products.id` | Sản phẩm |
| `quantity` | INT | NOT NULL, CHECK > 0 | Số lượng nhập |
| `unit_cost` | DECIMAL(15,2) | NOT NULL, CHECK ≥ 0 | Giá nhập một đơn vị |

Ràng buộc UNIQUE:

    (receipt_id, product_id)

Ràng buộc này ngăn một sản phẩm xuất hiện thành nhiều dòng trong cùng phiếu. Sản phẩm vẫn có thể xuất hiện trong nhiều phiếu khác nhau.

Thành tiền dòng nhập được tính bằng:

    quantity × unit_cost

### 4.8. Bảng `sales_orders`

Lưu thông tin chung của đơn bán.

| Cột | Kiểu dữ liệu | Ràng buộc / mặc định | Ý nghĩa |
|---|---|---|---|
| `id` | INT | PK, AUTO_INCREMENT | ID đơn bán |
| `created_by` | INT | NOT NULL, FK → `users.id` | Người tạo |
| `status` | ENUM('DRAFT','CONFIRMED','CANCELLED') | NOT NULL, mặc định `DRAFT` | Trạng thái |
| `created_at` | DATETIME | NOT NULL, thời điểm tạo | Thời điểm tạo đơn |
| `confirmed_by` | INT | Cho phép NULL, FK → `users.id` | Người xác nhận |
| `confirmed_at` | DATETIME | Cho phép NULL | Thời điểm xác nhận |

Quy tắc dữ liệu:

- `DRAFT` hoặc `CANCELLED`: hai cột xác nhận đều NULL.
- `CONFIRMED`: hai cột xác nhận đều phải có giá trị.
- Không lưu trạng thái thanh toán hoặc vận chuyển.
- Thiết kế hiện tại không lưu `total_amount`; tổng tiền được tính từ các dòng chi tiết.

### 4.9. Bảng `sales_order_items`

Lưu từng mặt hàng trong đơn bán.

| Cột | Kiểu dữ liệu | Ràng buộc | Ý nghĩa |
|---|---|---|---|
| `id` | INT | PK, AUTO_INCREMENT | ID dòng chi tiết |
| `order_id` | INT | NOT NULL, FK → `sales_orders.id` | Đơn bán |
| `product_id` | INT | NOT NULL, FK → `products.id` | Sản phẩm |
| `quantity` | INT | NOT NULL, CHECK > 0 | Số lượng bán |
| `unit_price` | DECIMAL(15,2) | NOT NULL, CHECK ≥ 0 | Giá bán một đơn vị đã lưu |

Ràng buộc UNIQUE:

    (order_id, product_id)

Quy tắc giá:

- Khi thêm sản phẩm vào đơn, lấy `products.sale_price` và lưu vào `unit_price`.
- Giá đã lưu không tự thay đổi khi giá sản phẩm thay đổi.
- Sửa số lượng không tự cập nhật lại đơn giá.
- Chính sách sửa giá thủ công trên đơn nháp cần được chốt trước khi triển khai API liên quan.

Công thức tính tiền:

    Thành tiền dòng = quantity × unit_price
    Tổng tiền đơn = tổng thành tiền các dòng thuộc đơn

## 5. Quan hệ giữa các bảng

| Bảng cha | Bảng con | Quan hệ | Khóa ngoại |
|---|---|---|---|
| `categories` | `products` | 1–N | `products.category_id` |
| `products` | `inventory` | 1–1 theo nghiệp vụ | `inventory.product_id` |
| `suppliers` | `stock_receipts` | 1–N | `stock_receipts.supplier_id` |
| `users` | `stock_receipts` | 1–N theo người tạo | `stock_receipts.created_by` |
| `users` | `stock_receipts` | 1–N theo người xác nhận | `stock_receipts.confirmed_by` |
| `stock_receipts` | `stock_receipt_items` | 1–N | `stock_receipt_items.receipt_id` |
| `products` | `stock_receipt_items` | 1–N | `stock_receipt_items.product_id` |
| `users` | `sales_orders` | 1–N theo người tạo | `sales_orders.created_by` |
| `users` | `sales_orders` | 1–N theo người xác nhận | `sales_orders.confirmed_by` |
| `sales_orders` | `sales_order_items` | 1–N | `sales_order_items.order_id` |
| `products` | `sales_order_items` | 1–N | `sales_order_items.product_id` |

Lưu ý về số lượng liên kết:

- Một danh mục, nhà cung cấp hoặc người dùng có thể chưa có bản ghi con.
- Phiếu/đơn nháp có thể chưa có dòng chi tiết; khi xác nhận phải có ít nhất một dòng.
- Chứng từ chưa xác nhận chưa liên kết tới người xác nhận.
- PK/FK của `inventory` không tự bảo đảm mọi sản phẩm đều có dòng tồn; trách nhiệm này thuộc luồng tạo sản phẩm.

## 6. Ràng buộc và trách nhiệm kiểm tra

| Điều kiện | Cơ chế bảo đảm |
|---|---|
| Email, SKU, tên danh mục không trùng | UNIQUE và chuẩn hóa đầu vào |
| Bản ghi được tham chiếu phải tồn tại | FOREIGN KEY |
| Một sản phẩm chỉ có một dòng trong chứng từ | UNIQUE theo cặp |
| Số lượng dòng chứng từ dương | CHECK và validation |
| Giá không âm | CHECK và validation |
| Tồn không âm | CHECK và xử lý cập nhật có kiểm soát |
| Thông tin xác nhận phù hợp trạng thái | CHECK kết hợp xử lý nghiệp vụ |
| Chỉ sửa/hủy/xác nhận chứng từ nháp | Service và cơ chế kiểm soát đồng thời |
| Chứng từ xác nhận có ít nhất một dòng | Service |
| STAFF chỉ thao tác chứng từ mình tạo | Kiểm tra danh tính và quyền tại backend |
| Không cập nhật dở dang | Transaction |
| Không xác nhận trùng hoặc bán vượt tồn | Transaction và khóa/cập nhật có điều kiện |
| Bản ghi tồn luôn đi cùng sản phẩm | Transaction khi tạo sản phẩm |
| Tổng nhập trừ tổng bán bằng tồn hiện tại | Xử lý nghiệp vụ và kiểm thử đối chiếu |

Không coi validation ở ứng dụng là thay thế cho ràng buộc database, hoặc ngược lại.

Chi tiết chuẩn hóa email, SKU và tên danh mục cần thống nhất với collation trước khi viết migration chính thức.

## 7. Chính sách xóa và cập nhật khóa

Định hướng triển khai khóa ngoại:

- Dùng `ON DELETE RESTRICT` để ngăn xóa bản ghi đang được tham chiếu.
- Dùng `ON UPDATE RESTRICT`; ID không được thay đổi trong nghiệp vụ thông thường.
- Không dùng xóa dây chuyền cho chứng từ hoặc dữ liệu lịch sử.

Các thao tác xóa dòng chi tiết chỉ được thực hiện khi chứng từ còn nháp và người dùng có quyền.

Khóa ngoại không tự ngăn việc sửa hoặc xóa dòng chi tiết của chứng từ đã xác nhận. Backend phải kiểm tra trạng thái và kiểm soát thao tác đồng thời.

Hệ thống sử dụng:

- Khóa tài khoản thay cho xóa tài khoản có lịch sử.
- Ngừng kinh doanh thay cho xóa sản phẩm có lịch sử.
- Hủy chứng từ nháp thay cho xóa chứng từ.

## 8. Phạm vi transaction

### 8.1. Tạo sản phẩm

Một transaction bao gồm:

1. Tạo bản ghi `products`.
2. Tạo bản ghi `inventory` tương ứng với `quantity = 0`.
3. Commit khi cả hai bước thành công; rollback nếu có lỗi.

### 8.2. Xác nhận phiếu nhập

Trong cùng transaction:

1. Đọc và kiểm soát truy cập đồng thời tới phiếu.
2. Kiểm tra quyền, trạng thái và các dòng chi tiết.
3. Cộng tồn cho tất cả sản phẩm trong phiếu.
4. Cập nhật trạng thái, người xác nhận và thời điểm xác nhận.
5. Commit; rollback toàn bộ nếu bất kỳ bước nào thất bại.

### 8.3. Xác nhận đơn bán

Trong cùng transaction:

1. Đọc và kiểm soát truy cập đồng thời tới đơn.
2. Kiểm tra quyền, trạng thái và các dòng chi tiết.
3. Kiểm tra và trừ tồn bằng cơ chế an toàn khi có nhiều yêu cầu đồng thời.
4. Cập nhật trạng thái, người xác nhận và thời điểm xác nhận.
5. Commit; rollback toàn bộ nếu thiếu hàng hoặc có lỗi.

Yêu cầu chung:

- Các Repository tham gia phải dùng cùng một connection của transaction.
- Không chỉ đọc tồn rồi cập nhật mà thiếu cơ chế bảo vệ trước yêu cầu đồng thời.
- Luồng sửa/hủy chứng từ phải phối hợp cùng cơ chế khóa của luồng xác nhận.
- Khi xử lý nhiều sản phẩm, sử dụng thứ tự nhất quán theo `product_id` để giảm nguy cơ deadlock.

Lựa chọn câu lệnh khóa và cập nhật cụ thể được hoàn thiện khi triển khai Service/Repository.

## 9. Lịch sử nhập/xuất không dùng bảng riêng

Thiết kế không có bảng `stock_movements`.

Lịch sử của một sản phẩm được tổng hợp từ hai nguồn:

| Nguồn | Điều kiện | Số lượng biến động |
|---|---|---|
| `stock_receipts` kết hợp `stock_receipt_items` | Phiếu `CONFIRMED` | `+quantity` |
| `sales_orders` kết hợp `sales_order_items` | Đơn `CONFIRMED` | `-quantity` |

Mỗi mục lịch sử cung cấp:

- Sản phẩm.
- Loại chứng từ.
- ID chứng từ.
- Số lượng tăng hoặc giảm.
- Người xác nhận.
- Thời điểm xác nhận.

Hai nguồn được ghép bằng `UNION ALL`. Sắp xếp theo thời điểm xác nhận, bổ sung loại chứng từ và ID làm tiêu chí phụ khi trùng thời điểm.

Tồn được đối chiếu theo công thức:

    Tồn hiện tại = Tổng nhập đã xác nhận − Tổng bán đã xác nhận

Công thức áp dụng vì tồn ban đầu bằng 0 và hệ thống không cho điều chỉnh tồn trực tiếp.

## 10. Chỉ mục

### 10.1. Chỉ mục bắt buộc

- PRIMARY KEY của các bảng.
- UNIQUE trên `users.email`.
- UNIQUE trên `categories.name`.
- UNIQUE trên `products.sku`.
- UNIQUE trên `stock_receipt_items(receipt_id, product_id)`.
- UNIQUE trên `sales_order_items(order_id, product_id)`.
- Chỉ mục đáp ứng yêu cầu khóa ngoại.

Không tạo thêm chỉ mục trùng với chỉ mục đã có. Ví dụ UNIQUE bắt đầu bằng `receipt_id` có thể phục vụ truy vấn các dòng của một phiếu.

### 10.2. Chỉ mục cần đánh giá theo truy vấn

Các nhu cầu có thể cần chỉ mục bổ sung:

- Liệt kê chứng từ theo người tạo, trạng thái và thời gian.
- Tra cứu lịch sử theo sản phẩm.
- Lọc sản phẩm theo danh mục và trạng thái kinh doanh.
- Sắp xếp, phân trang danh sách chứng từ.

Chỉ mục bổ sung được quyết định dựa trên truy vấn thực tế và kế hoạch thực thi. Không mặc định thêm chỉ mục cho mọi cột.

## 11. Giới hạn thiết kế hiện tại

- Không quản lý nhiều kho.
- Không theo dõi từng thiết bị bằng serial/IMEI.
- Không có bảng khách hàng, thanh toán, vận chuyển hoặc công nợ.
- Không hỗ trợ đổi trả, điều chỉnh kiểm kê hoặc chứng từ đảo.
- Không lưu snapshot tên và thông tin liên hệ của sản phẩm, người dùng hoặc nhà cung cấp trên chứng từ.
- Vì vậy, khi các thông tin mô tả được sửa, việc tra cứu chứng từ qua liên kết sẽ thấy thông tin hiện tại; số lượng và đơn giá đã lưu trên dòng chứng từ vẫn được giữ nguyên.
- Chưa lưu người hủy, thời điểm hủy hoặc lịch sử mọi lần chỉnh sửa chứng từ.
- Chưa thiết kế các bảng phục vụ RAG.

Nếu bổ sung các yêu cầu trên, phải cập nhật thiết kế, migration, API và test liên quan.

## 12. Thứ tự triển khai và kiểm tra

Thứ tự tạo bảng đề xuất:

1. `users`
2. `categories`
3. `suppliers`
4. `products`
5. `inventory`
6. `stock_receipts`
7. `stock_receipt_items`
8. `sales_orders`
9. `sales_order_items`

Các kiểm tra tối thiểu:

- Tạo bảng thành công trên database trống.
- Không cho trùng email, SKU hoặc tên danh mục.
- Không cho khóa ngoại tham chiếu bản ghi không tồn tại.
- Không cho số lượng dòng bằng 0/âm hoặc giá âm.
- Không cho tồn kho âm.
- Không cho trùng sản phẩm trong cùng chứng từ.
- Không cho trạng thái xác nhận mâu thuẫn với thông tin xác nhận.
- Không xóa dữ liệu cha đang được tham chiếu.
- Kiểm tra transaction rollback và các tình huống xác nhận đồng thời bằng test tích hợp.

Mọi thay đổi schema sau khi triển khai phải được quản lý qua migration và cập nhật đồng bộ tài liệu cùng ERD.
