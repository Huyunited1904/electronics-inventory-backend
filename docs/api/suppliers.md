# Thiết kế API nhà cung cấp

## 1. Mục đích

Tài liệu mô tả API quản lý nhà cung cấp trong hệ thống quản lý bán hàng và tồn kho đồ điện tử.

Tài liệu là căn cứ để:

- Triển khai chức năng xem, thêm và sửa nhà cung cấp.
- Xác định quyền truy cập của ADMIN và STAFF.
- Thống nhất dữ liệu đầu vào, dữ liệu trả về và mã lỗi.
- Xây dựng kiểm thử chức năng, validation và phân quyền.
- Sử dụng nhà cung cấp làm thông tin tham chiếu khi tạo phiếu nhập kho.

Đường dẫn gốc của nhóm tài nguyên:

```text
/api/suppliers
```

## 2. Tài liệu liên quan

| Tài liệu | Nội dung đối chiếu |
|---|---|
| `../scope.md` | Phạm vi chức năng và phân quyền |
| `../business-rules.md` | Quy tắc nghiệp vụ nhà cung cấp và nhập hàng |
| `../database.md` | Cấu trúc bảng `suppliers` và các ràng buộc |
| `../diagrams/erd.dbml` | Quan hệ giữa nhà cung cấp và phiếu nhập |
| `../architecture.md` | Kiến trúc xử lý yêu cầu và quy ước chung |
| `auth.md` | Xác thực JWT và kiểm tra tài khoản |
| `users.md` | Quy ước phản hồi và xử lý lỗi |
| `catalog.md` | Quy ước phân trang, tìm kiếm và validation |
| `purchasing.md` | Sử dụng nhà cung cấp trong phiếu nhập; tài liệu được bổ sung sau |

Nếu có mâu thuẫn giữa các tài liệu, cần thống nhất và cập nhật nội dung liên quan trước khi triển khai.

## 3. Phạm vi

### 3.1. Chức năng được hỗ trợ

- Xem danh sách nhà cung cấp.
- Tìm kiếm nhà cung cấp.
- Xem chi tiết nhà cung cấp.
- Thêm nhà cung cấp.
- Sửa thông tin nhà cung cấp.

### 3.2. Chức năng ngoài phạm vi

- Xóa nhà cung cấp.
- Khóa hoặc ngừng hợp tác với nhà cung cấp.
- Quản lý tài khoản đăng nhập dành cho nhà cung cấp.
- Quản lý công nợ, thanh toán hoặc hợp đồng.
- Quản lý nhiều địa chỉ hoặc nhiều người liên hệ cho một nhà cung cấp.
- Quản lý danh sách sản phẩm riêng của từng nhà cung cấp.
- Tạo hoặc xác nhận phiếu nhập thông qua API nhà cung cấp.

Bảng `suppliers` hiện tại không có trường trạng thái hoạt động. API không nhận hoặc trả trường `active`.

## 4. Xác thực và phân quyền

### 4.1. Xác thực

Tất cả endpoint yêu cầu JWT access token:

```http
Authorization: Bearer <access_token>
```

Sau khi xác thực token, hệ thống kiểm tra tài khoản trong cơ sở dữ liệu:

- Tài khoản phải tồn tại.
- Tài khoản phải đang hoạt động.
- Quyền truy cập được xác định theo vai trò hiện tại của tài khoản.

Không chỉ dựa vào vai trò lưu trong token để quyết định quyền truy cập.

### 4.2. Bảng phân quyền

| Chức năng | ADMIN | STAFF |
|---|:---:|:---:|
| Xem danh sách nhà cung cấp | ✓ | ✓ |
| Tìm kiếm nhà cung cấp | ✓ | ✓ |
| Xem chi tiết nhà cung cấp | ✓ | ✓ |
| Thêm nhà cung cấp | ✓ | — |
| Sửa thông tin nhà cung cấp | ✓ | — |

STAFF gọi API thêm hoặc sửa nhà cung cấp phải nhận `403 FORBIDDEN`.

Kiểm tra quyền được thực hiện trước khi truy vấn nhà cung cấp đích. Người dùng không có quyền không được dùng phản hồi của API quản trị để xác định một nhà cung cấp có tồn tại hay không.

## 5. Quy ước dữ liệu và phản hồi

### 5.1. Định dạng chung

- Request body và response body sử dụng JSON.
- Request có body phải sử dụng `Content-Type: application/json`.
- Tên trường API sử dụng `camelCase`.
- Thời gian trả về theo ISO 8601, múi giờ UTC.
- ID là số nguyên dương trong phạm vi `INT` có dấu: từ `1` đến `2147483647`.
- Không chấp nhận body dạng mảng, chuỗi hoặc `null`.
- Không chấp nhận trường body hoặc query parameter ngoài danh sách được mô tả.
- Không chấp nhận query parameter lặp lại.
- Không tự chuyển số hoặc boolean thành chuỗi để chấp nhận đầu vào sai kiểu.

Ví dụ thời gian:

```json
{
  "createdAt": "2026-11-22T03:00:00.000Z"
}
```

### 5.2. Đối tượng nhà cung cấp

```json
{
  "id": 1,
  "name": "Công ty Thiết bị Điện tử An Phát",
  "phone": "0901234567",
  "email": "contact@example.com",
  "address": "Cầu Giấy, Hà Nội",
  "createdAt": "2026-11-22T03:00:00.000Z",
  "updatedAt": "2026-11-22T03:00:00.000Z"
}
```

Các trường thông tin liên hệ chưa được cung cấp được trả về dưới dạng `null`.

Ví dụ:

```json
{
  "id": 2,
  "name": "Nhà cung cấp Minh Anh",
  "phone": null,
  "email": null,
  "address": null,
  "createdAt": "2026-11-22T04:00:00.000Z",
  "updatedAt": "2026-11-22T04:00:00.000Z"
}
```

### 5.3. Phản hồi một tài nguyên

```json
{
  "data": {
    "id": 1,
    "name": "Công ty Thiết bị Điện tử An Phát",
    "phone": "0901234567",
    "email": "contact@example.com",
    "address": "Cầu Giấy, Hà Nội",
    "createdAt": "2026-11-22T03:00:00.000Z",
    "updatedAt": "2026-11-22T03:00:00.000Z"
  }
}
```

### 5.4. Phản hồi danh sách

```json
{
  "data": [],
  "pagination": {
    "page": 1,
    "pageSize": 20,
    "totalItems": 0,
    "totalPages": 0
  }
}
```

### 5.5. Phản hồi lỗi

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Dữ liệu đầu vào không hợp lệ.",
    "details": [
      {
        "field": "name",
        "message": "Tên nhà cung cấp không được để trống."
      }
    ]
  }
}
```

Trường `details` không bắt buộc.

Không trả câu SQL nội bộ, stack trace hoặc thông tin kết nối cơ sở dữ liệu.

## 6. Quy tắc nghiệp vụ và validation

### 6.1. Tên nhà cung cấp

- Trường `name` bắt buộc khi tạo mới.
- Phải là chuỗi.
- Loại bỏ khoảng trắng ở hai đầu trước khi kiểm tra độ dài và lưu.
- Không được rỗng sau khi chuẩn hóa.
- Có tối đa 255 ký tự.
- Không tự thay đổi chữ hoa, chữ thường hoặc khoảng trắng bên trong tên.

### 6.2. Thông tin liên hệ

Các trường `phone`, `email` và `address` không bắt buộc.

Quy tắc chuẩn hóa:

- Chấp nhận chuỗi hoặc `null`.
- Nếu là chuỗi, loại bỏ khoảng trắng ở hai đầu.
- Chuỗi rỗng sau khi chuẩn hóa được chuyển thành `null`.
- Giá trị không rỗng phải thỏa mãn validation của từng trường.

| Trường | Giới hạn | Quy định |
|---|---|---|
| `phone` | Tối đa 20 ký tự | Là chuỗi; có ít nhất một chữ số; chỉ chứa chữ số ASCII, khoảng trắng, `+`, `-`, `(`, `)`; dấu `+` nếu có chỉ xuất hiện một lần ở đầu |
| `email` | Tối đa 255 ký tự | Đúng định dạng email theo bộ validator dùng chung của hệ thống |
| `address` | Tối đa 500 ký tự | Chuỗi văn bản không rỗng sau chuẩn hóa, hoặc `null` |

Giới hạn độ dài chuỗi được tính theo Unicode code point, không tính theo số byte UTF-8.

Số điện thoại phải được lưu dưới dạng chuỗi để giữ số `0` ở đầu và ký hiệu quốc tế.

Validation số điện thoại chỉ kiểm tra hình thức đầu vào; không xác nhận số đó tồn tại hoặc thuộc nhà cung cấp.

Validation email không bao gồm xác minh quyền sở hữu hoặc kiểm tra hộp thư có tồn tại.

### 6.3. Phân biệt bỏ qua trường và gửi `null`

| Tình huống | Khi tạo mới | Khi cập nhật |
|---|---|---|
| Không gửi `phone`, `email` hoặc `address` | Lưu `null` | Giữ nguyên giá trị hiện tại |
| Gửi `null` | Lưu `null` | Xóa giá trị hiện tại, lưu `null` |
| Gửi chuỗi rỗng hoặc chỉ có khoảng trắng | Chuẩn hóa thành `null` | Chuẩn hóa thành `null` |
| Gửi chuỗi hợp lệ | Lưu giá trị đã chuẩn hóa | Thay thế bằng giá trị đã chuẩn hóa |

Trường `name` không được gửi `null` hoặc chuỗi rỗng.

### 6.4. Tính duy nhất

Theo cấu trúc bảng hiện tại:

- `suppliers.id` là định danh duy nhất.
- `name`, `phone` và `email` không có ràng buộc UNIQUE.
- Hai nhà cung cấp có thể có cùng tên hoặc thông tin liên hệ.
- Không tự động gộp bản ghi theo tên, số điện thoại hoặc email.
- Không trả lỗi `409` chỉ vì những thông tin này trùng nhau.

ADMIN cần tra cứu trước khi thêm để hạn chế tạo bản ghi trùng do thao tác nhầm.

Nếu bổ sung yêu cầu chống trùng trong tương lai, phải xác định rõ tiêu chí và cập nhật đồng thời tài liệu nghiệp vụ, database và API.

### 6.5. Quan hệ với phiếu nhập

- Mỗi phiếu nhập tham chiếu nhà cung cấp qua `stock_receipts.supplier_id`.
- Sửa thông tin nhà cung cấp không thay đổi ID nhà cung cấp.
- Sửa thông tin không thay đổi `supplier_id` của các phiếu nhập liên quan.
- Không thay đổi trạng thái, mặt hàng, số lượng hoặc giá nhập của phiếu nhập.
- Không làm thay đổi tồn kho.

Thiết kế hiện tại không lưu bản sao tên và thông tin liên hệ của nhà cung cấp trên phiếu nhập.

Vì vậy, khi tra cứu phiếu nhập bằng cách liên kết với bảng `suppliers`, thông tin nhà cung cấp được hiển thị là thông tin hiện tại.

### 6.6. Thời gian cập nhật

- `createdAt` được tạo bởi hệ thống và không được chỉnh sửa qua API.
- `updatedAt` thay đổi khi dữ liệu thực tế thay đổi.
- Nếu dữ liệu sau chuẩn hóa giống hoàn toàn dữ liệu hiện tại, trả thành công và giữ nguyên `updatedAt`.

## 7. Danh sách endpoint

| Method | Endpoint | Chức năng | Quyền |
|---|---|---|---|
| GET | `/api/suppliers` | Xem danh sách và tìm kiếm | ADMIN, STAFF |
| GET | `/api/suppliers/{id}` | Xem chi tiết | ADMIN, STAFF |
| POST | `/api/suppliers` | Thêm nhà cung cấp | ADMIN |
| PATCH | `/api/suppliers/{id}` | Sửa thông tin nhà cung cấp | ADMIN |

Không cung cấp endpoint DELETE hoặc endpoint thay đổi trạng thái nhà cung cấp.

## 8. Xem danh sách và tìm kiếm nhà cung cấp

```http
GET /api/suppliers
```

**Quyền:** ADMIN, STAFF.

### 8.1. Query parameters

| Trường | Kiểu | Bắt buộc | Mặc định | Quy định |
|---|---|:---:|---|---|
| `page` | Integer | Không | `1` | Số nguyên từ `1` đến `2147483647` |
| `pageSize` | Integer | Không | `20` | Số nguyên từ `1` đến `100` |
| `q` | String | Không | Không tìm kiếm | Từ 1 đến 255 ký tự sau khi loại bỏ khoảng trắng hai đầu |

Query parameter số phải có dạng chữ số thập phân; không chấp nhận số thập phân, số mũ hoặc dấu âm.

Nếu một query parameter được gửi với giá trị rỗng, trả `400 VALIDATION_ERROR`.

### 8.2. Quy tắc tìm kiếm

- `q` tìm theo tên, số điện thoại hoặc email.
- Chỉ cần một trong ba trường khớp là bản ghi được chọn.
- Tìm kiếm theo chuỗi con.
- Không tìm kiếm theo địa chỉ trong phiên bản này.
- `%` và `_` trong từ khóa được xử lý như ký tự thông thường, không phải ký tự đại diện SQL.
- Quy tắc phân biệt chữ hoa/thường và dấu phụ thuộc collation được thống nhất trong thiết kế cơ sở dữ liệu.
- Số điện thoại được tìm theo chuỗi đã lưu; chưa hỗ trợ chuẩn hóa để bỏ dấu cách, dấu gạch nối hoặc dấu ngoặc khi tìm kiếm.

### 8.3. Phân trang và sắp xếp

- Thứ tự mặc định: `id DESC`.
- Chưa hỗ trợ tùy chọn thứ tự sắp xếp.
- `totalItems` là số bản ghi sau khi áp dụng điều kiện tìm kiếm.
- `totalPages = ceil(totalItems / pageSize)`.
- Khi không có bản ghi, `totalPages = 0`.
- Trang vượt phạm vi trả `200 OK` với `data: []`, giữ nguyên `page` được yêu cầu và trả tổng số bản ghi thực tế.

### 8.4. Request mẫu

```http
GET /api/suppliers?page=1&pageSize=20&q=An%20Ph%C3%A1t
Authorization: Bearer <access_token>
```

### 8.5. Response — `200 OK`

```json
{
  "data": [
    {
      "id": 1,
      "name": "Công ty Thiết bị Điện tử An Phát",
      "phone": "0901234567",
      "email": "contact@example.com",
      "address": "Cầu Giấy, Hà Nội",
      "createdAt": "2026-11-22T03:00:00.000Z",
      "updatedAt": "2026-11-22T03:00:00.000Z"
    }
  ],
  "pagination": {
    "page": 1,
    "pageSize": 20,
    "totalItems": 1,
    "totalPages": 1
  }
}
```

### 8.6. Lỗi validation

Trả `400 VALIDATION_ERROR` khi:

- `page` hoặc `pageSize` sai định dạng hoặc ngoài phạm vi.
- `q` rỗng sau chuẩn hóa hoặc vượt giới hạn.
- Query parameter bị lặp.
- Có query parameter không được hỗ trợ.

Không tìm thấy kết quả tìm kiếm không phải lỗi `404`.

## 9. Xem chi tiết nhà cung cấp

```http
GET /api/suppliers/{id}
```

**Quyền:** ADMIN, STAFF.

### 9.1. Path parameter

| Trường | Kiểu | Bắt buộc | Quy định |
|---|---|:---:|---|
| `id` | Integer | Có | Từ `1` đến `2147483647` |

### 9.2. Request mẫu

```http
GET /api/suppliers/1
Authorization: Bearer <access_token>
```

### 9.3. Response — `200 OK`

Trả đối tượng nhà cung cấp trong trường `data`, theo cấu trúc tại mục 5.3.

Response không bao gồm danh sách phiếu nhập, công nợ hoặc thống kê mua hàng.

### 9.4. Lỗi nghiệp vụ

| HTTP status | Mã lỗi | Trường hợp |
|---|---|---|
| 400 | `VALIDATION_ERROR` | ID không hợp lệ hoặc có query parameter không được hỗ trợ |
| 404 | `SUPPLIER_NOT_FOUND` | Nhà cung cấp không tồn tại |

## 10. Thêm nhà cung cấp

```http
POST /api/suppliers
```

**Quyền:** ADMIN.

### 10.1. Request body

| Trường | Kiểu | Bắt buộc | Quy định |
|---|---|:---:|---|
| `name` | String | Có | Từ 1 đến 255 ký tự sau chuẩn hóa |
| `phone` | String hoặc null | Không | Theo mục 6.2 |
| `email` | String hoặc null | Không | Theo mục 6.2 |
| `address` | String hoặc null | Không | Theo mục 6.2 |

Request đầy đủ:

```json
{
  "name": "Công ty Thiết bị Điện tử An Phát",
  "phone": "0901234567",
  "email": "contact@example.com",
  "address": "Cầu Giấy, Hà Nội"
}
```

Request tối thiểu:

```json
{
  "name": "Nhà cung cấp Minh Anh"
}
```

Chỉ chấp nhận bốn trường được mô tả.

Không chấp nhận `id`, `active`, `createdAt`, `updatedAt` hoặc các trường khác.

### 10.2. Xử lý nghiệp vụ

1. Xác thực và kiểm tra quyền ADMIN.
2. Kiểm tra cấu trúc body và danh sách trường cho phép.
3. Chuẩn hóa và validate dữ liệu.
4. Tạo bản ghi `suppliers`.
5. Trả dữ liệu đã lưu.

API không tự tạo tài khoản, phiếu nhập hoặc sản phẩm.

### 10.3. Response — `201 Created`

Header:

```http
Location: /api/suppliers/1
```

Body:

```json
{
  "data": {
    "id": 1,
    "name": "Công ty Thiết bị Điện tử An Phát",
    "phone": "0901234567",
    "email": "contact@example.com",
    "address": "Cầu Giấy, Hà Nội",
    "createdAt": "2026-11-22T03:00:00.000Z",
    "updatedAt": "2026-11-22T03:00:00.000Z"
  }
}
```

### 10.4. Lỗi validation

Trả `400 VALIDATION_ERROR` khi:

- Thiếu `name`.
- `name` là `null`, sai kiểu, rỗng hoặc quá dài.
- Thông tin liên hệ sai kiểu, sai định dạng hoặc quá dài.
- Body chứa trường không được phép.
- Có query parameter không được hỗ trợ.

Tên, số điện thoại hoặc email trùng không gây lỗi theo thiết kế hiện tại.

### 10.5. Gửi lại yêu cầu tạo

API chưa hỗ trợ idempotency key.

Nếu client gửi lại một yêu cầu tạo đã thành công, hệ thống có thể tạo thêm một nhà cung cấp với ID mới, kể cả khi nội dung giống nhau.

Khi mất kết nối hoặc không rõ yêu cầu trước đã thành công hay chưa, client cần kiểm tra lại danh sách trước khi gửi lại thao tác tạo.

## 11. Sửa thông tin nhà cung cấp

```http
PATCH /api/suppliers/{id}
```

**Quyền:** ADMIN.

### 11.1. Request body

Cho phép một hoặc nhiều trường:

- `name`
- `phone`
- `email`
- `address`

Áp dụng validation và chuẩn hóa tại mục 6.

Ví dụ cập nhật số điện thoại và địa chỉ:

```json
{
  "phone": "0987654321",
  "address": "Nam Từ Liêm, Hà Nội"
}
```

Ví dụ xóa email hiện tại:

```json
{
  "email": null
}
```

### 11.2. Quy định cập nhật

- Body phải chứa ít nhất một trường được phép.
- Trường không được gửi phải được giữ nguyên.
- Không chấp nhận body rỗng.
- Không chấp nhận trường ngoài danh sách cho phép.
- Không cho phép thay đổi ID hoặc thời điểm tạo.
- Các trường trong một request được cập nhật nguyên tử: thành công toàn bộ hoặc không thay đổi trường nào.
- Nếu một trường không hợp lệ, không cập nhật các trường còn lại.
- Sửa nhà cung cấp không cập nhật dữ liệu nghiệp vụ của phiếu nhập hoặc tồn kho.
- Dữ liệu không thay đổi sau chuẩn hóa vẫn trả thành công và giữ nguyên `updatedAt`.

### 11.3. Response — `200 OK`

```json
{
  "data": {
    "id": 1,
    "name": "Công ty Thiết bị Điện tử An Phát",
    "phone": "0987654321",
    "email": "contact@example.com",
    "address": "Nam Từ Liêm, Hà Nội",
    "createdAt": "2026-11-22T03:00:00.000Z",
    "updatedAt": "2026-11-22T05:00:00.000Z"
  }
}
```

### 11.4. Lỗi nghiệp vụ

| HTTP status | Mã lỗi | Trường hợp |
|---|---|---|
| 400 | `VALIDATION_ERROR` | ID, query hoặc body không hợp lệ |
| 404 | `SUPPLIER_NOT_FOUND` | Nhà cung cấp không tồn tại |

Sau bước xác thực và phân quyền, kiểm tra định dạng đầu vào trước khi tra cứu nhà cung cấp.

Không được suy luận rằng nhà cung cấp không tồn tại chỉ vì câu UPDATE không làm thay đổi giá trị nào.

## 12. Danh sách lỗi chung

| HTTP status | Mã lỗi | Ý nghĩa |
|---|---|---|
| 400 | `INVALID_JSON` | Body không phải JSON hợp lệ |
| 400 | `VALIDATION_ERROR` | Path, query hoặc body không hợp lệ |
| 401 | `AUTH_REQUIRED` | Thiếu thông tin xác thực |
| 401 | `INVALID_TOKEN` | Token không hợp lệ hoặc tài khoản tương ứng không tồn tại |
| 401 | `TOKEN_EXPIRED` | Token hết hạn |
| 403 | `ACCOUNT_DISABLED` | Tài khoản đã bị khóa |
| 403 | `FORBIDDEN` | Vai trò không được phép thực hiện thao tác |
| 404 | `SUPPLIER_NOT_FOUND` | Không tìm thấy nhà cung cấp |
| 413 | `PAYLOAD_TOO_LARGE` | Request body vượt giới hạn cấu hình |
| 415 | `UNSUPPORTED_MEDIA_TYPE` | Request có body nhưng loại nội dung không được hỗ trợ |
| 500 | `INTERNAL_SERVER_ERROR` | Lỗi nội bộ chưa được ánh xạ thành lỗi cụ thể |

Nhóm API này hiện không định nghĩa lỗi nghiệp vụ `409` vì chưa có quy tắc duy nhất đối với tên hoặc thông tin liên hệ của nhà cung cấp.

Không chuyển mọi lỗi cơ sở dữ liệu thành lỗi trùng dữ liệu.

## 13. Ánh xạ dữ liệu

| Trường API | Cột cơ sở dữ liệu | Ghi chú |
|---|---|---|
| `id` | `suppliers.id` | Khóa chính, hệ thống tạo |
| `name` | `suppliers.name` | Bắt buộc |
| `phone` | `suppliers.phone` | Có thể null |
| `email` | `suppliers.email` | Có thể null |
| `address` | `suppliers.address` | Có thể null |
| `createdAt` | `suppliers.created_at` | Hệ thống quản lý |
| `updatedAt` | `suppliers.updated_at` | Hệ thống quản lý |

Quan hệ với phiếu nhập:

```text
stock_receipts.supplier_id → suppliers.id
```

Một nhà cung cấp có thể được tham chiếu bởi nhiều phiếu nhập. Nhà cung cấp mới có thể chưa có phiếu nhập nào.

Không cần bổ sung bảng hoặc cột để triển khai các endpoint trong tài liệu này.

## 14. Tính nhất quán và xử lý đồng thời

### 14.1. Tạo và cập nhật

Mỗi thao tác ghi phải bảo đảm tính nguyên tử.

Nếu chỉ thực hiện một câu INSERT hoặc UPDATE trên bảng `suppliers`, tính nguyên tử của câu lệnh được bảo đảm bởi InnoDB.

Nếu cách triển khai sử dụng nhiều câu lệnh phụ thuộc nhau để đọc, kiểm tra và ghi dữ liệu, cần bảo đảm chúng không tạo ra kết quả sai khi có yêu cầu đồng thời; sử dụng giao dịch và khóa phù hợp khi cần.

### 14.2. Cập nhật đồng thời

Phiên bản hiện tại chưa sử dụng trường phiên bản hoặc header `If-Match`.

Quy định:

- Chỉ cập nhật các trường có trong request.
- Không ghi lại toàn bộ bản ghi dựa trên dữ liệu đã đọc trước đó.
- Hai request sửa các trường khác nhau không được ghi đè những trường không được gửi.
- Khi hai request cùng sửa một trường, giá trị cuối cùng phụ thuộc thứ tự cập nhật thực tế trong cơ sở dữ liệu.
- Response phải phản ánh kết quả thao tác của request, không được vô tình trả dữ liệu của một cập nhật khác xen giữa quá trình ghi và đọc kết quả.

### 14.3. Phân trang

Trong cùng một response, danh sách và tổng số bản ghi phải:

- Dùng cùng điều kiện tìm kiếm.
- Được đọc từ một ảnh chụp dữ liệu nhất quán.

Phân trang giữa nhiều request không bảo đảm ảnh chụp cố định. Nếu có dữ liệu được thêm hoặc sửa giữa các lần gọi, kết quả ở các trang có thể thay đổi.

## 15. Yêu cầu bảo mật và triển khai

- Sử dụng truy vấn có tham số.
- Không nối trực tiếp từ khóa tìm kiếm vào SQL.
- Khi dùng LIKE, xử lý ký tự đại diện và ký tự escape theo quy tắc của cơ sở dữ liệu.
- Áp dụng danh sách trường cho phép đối với request body.
- Không ánh xạ nguyên body vào câu lệnh INSERT hoặc UPDATE.
- Kiểm tra validation tại backend.
- Không ghi JWT hoặc thông tin xác thực vào log.
- Hạn chế ghi toàn bộ số điện thoại, email và địa chỉ vào log khi không cần thiết.
- Không trả thông tin lỗi nội bộ cho client.
- API GET không được làm thay đổi dữ liệu.
- Chưa triển khai cache trong hợp đồng API này. Khi bổ sung cache, cần xác định cách vô hiệu hóa dữ liệu sau thao tác thêm hoặc sửa.
- Không sử dụng cache để bỏ qua kiểm tra khóa ngoại khi tạo phiếu nhập.

## 16. Các tình huống kiểm thử

### 16.1. Xác thực và phân quyền

- Không gửi token.
- Token không hợp lệ hoặc hết hạn.
- Token của tài khoản không còn tồn tại.
- Token của tài khoản bị khóa.
- ADMIN sử dụng đầy đủ các endpoint.
- STAFF xem danh sách và chi tiết thành công.
- STAFF thêm hoặc sửa nhận `403 FORBIDDEN`.
- Request không có quyền không làm thay đổi dữ liệu.

### 16.2. Danh sách và tìm kiếm

- Danh sách chưa có dữ liệu.
- Phân trang mặc định.
- Phân trang ở giới hạn hợp lệ.
- Trang vượt phạm vi.
- `page` hoặc `pageSize` bằng 0, âm, số thập phân hoặc vượt giới hạn.
- Query parameter rỗng, lặp lại hoặc không được hỗ trợ.
- Tìm theo tên, số điện thoại và email.
- Tìm kiếm không có kết quả trả `200` và danh sách rỗng.
- Từ khóa chứa dấu tiếng Việt, dấu nháy, `%`, `_` hoặc ký tự escape.
- Bản ghi có `phone` hoặc `email` bằng null.
- Thứ tự kết quả là `id DESC`.
- `totalItems` được tính theo đúng điều kiện tìm kiếm.

### 16.3. Xem chi tiết

- ID tồn tại.
- ID không tồn tại.
- ID bằng 0, âm, sai định dạng hoặc vượt phạm vi.
- Các trường liên hệ chưa có thông tin được trả về null.
- Response không chứa dữ liệu nội bộ hoặc thông tin ngoài hợp đồng.

### 16.4. Thêm nhà cung cấp

- Tạo với đầy đủ thông tin.
- Tạo chỉ có tên.
- Thiếu tên, tên null hoặc sai kiểu.
- Tên chỉ có khoảng trắng.
- Tên đúng giới hạn và vượt giới hạn 255 ký tự.
- Số điện thoại giữ nguyên số 0 ở đầu.
- Số điện thoại sai kiểu, sai ký tự hoặc vượt 20 ký tự.
- Email sai định dạng hoặc vượt 255 ký tự.
- Địa chỉ vượt 500 ký tự.
- Thông tin liên hệ null, rỗng hoặc chỉ có khoảng trắng.
- Gửi trường không được phép.
- Body dạng mảng hoặc null.
- Tên hoặc thông tin liên hệ trùng vẫn được tạo theo quy tắc hiện tại.
- Gửi lại POST có thể tạo bản ghi mới, đúng với giới hạn chưa hỗ trợ idempotency.

### 16.5. Sửa nhà cung cấp

- Sửa một trường và nhiều trường.
- Body rỗng.
- Body chỉ chứa trường không được phép.
- Xóa thông tin liên hệ bằng null.
- Chuỗi liên hệ rỗng được chuẩn hóa thành null.
- Không gửi trường thì giữ nguyên.
- Không cho phép xóa tên bằng null hoặc chuỗi rỗng.
- Một trường không hợp lệ khiến toàn bộ cập nhật bị từ chối.
- Gửi lại dữ liệu hiện tại không thay đổi `updatedAt`.
- Nhà cung cấp không tồn tại trả `404`.
- Sửa nhà cung cấp đang được tham chiếu bởi phiếu nhập.
- Không thay đổi `supplier_id`, trạng thái phiếu nhập hoặc tồn kho.
- Hai yêu cầu sửa các trường khác nhau không làm mất dữ liệu của nhau.
- Hai yêu cầu sửa cùng trường tuân theo giới hạn cập nhật đồng thời đã mô tả.

### 16.6. Lỗi giao thức và hệ thống

- JSON sai cú pháp.
- Content-Type không được hỗ trợ.
- Body vượt giới hạn cấu hình.
- Lỗi cơ sở dữ liệu không để lộ câu SQL hoặc stack trace.
- Thao tác ghi thất bại không để lại dữ liệu cập nhật một phần.

## 17. Các điểm cần đối chiếu trước khi triển khai

Các lựa chọn được xác định trong bản thiết kế này cần được đối chiếu và đồng bộ với tài liệu chung:

| Nội dung | Quy định trong tài liệu |
|---|---|
| Tính duy nhất | Chỉ ID là duy nhất; không cấm trùng tên, điện thoại hoặc email |
| Thông tin liên hệ | Không bắt buộc; cho phép null |
| Chuỗi liên hệ rỗng | Chuẩn hóa thành null |
| Cập nhật một phần | Bỏ qua trường để giữ nguyên; gửi null để xóa thông tin liên hệ |
| Tìm kiếm | Theo tên, điện thoại hoặc email |
| Phân trang | Mặc định 20 bản ghi, tối đa 100 |
| Sắp xếp | ID giảm dần |
| Xóa và thay đổi trạng thái | Không hỗ trợ |
| Thông tin nhà cung cấp trên phiếu nhập | Dùng dữ liệu hiện tại, chưa lưu bản sao lịch sử |
| Chống gửi trùng yêu cầu tạo | Chưa hỗ trợ idempotency key |

Các nội dung cần hoàn thiện tại tài liệu liên quan:

1. **Collation:** chốt tại `database.md` để xác định cách tìm kiếm phân biệt chữ hoa/thường và dấu.
2. **Validator email:** thống nhất thư viện và cấu hình dùng chung trước khi triển khai, tránh mỗi module kiểm tra theo một cách.
3. **Quy ước lỗi:** đối chiếu mã lỗi và cấu trúc `details` với `auth.md`, `users.md` và `catalog.md`.
4. **Giới hạn request body:** cấu hình tập trung và mô tả tại `architecture.md`.
5. **API phiếu nhập:** mô tả việc kiểm tra `supplierId` và điều kiện được đổi nhà cung cấp của phiếu nháp trong `purchasing.md`.
6. **Cache:** nếu triển khai Redis cho nhà cung cấp, bổ sung chính sách hết hạn và vô hiệu hóa cache trong tài liệu caching.

Sau khi giải quyết từng nội dung, cập nhật quyết định vào tài liệu tương ứng và loại bỏ khỏi danh sách cần hoàn thiện.
