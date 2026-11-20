# API quản lý tài khoản

## 1. Mục đích và phạm vi

Tài liệu mô tả các API quản lý tài khoản nội bộ của backend quản lý bán hàng và tồn kho đồ điện tử.

Các chức năng thuộc phạm vi:

- ADMIN cấp tài khoản STAFF.
- ADMIN xem danh sách và chi tiết tài khoản.
- ADMIN khóa tài khoản STAFF.

Đăng nhập và đổi mật khẩu của bản thân được mô tả tại [API xác thực](auth.md), không định nghĩa lại trong tài liệu này.

Không cung cấp API công khai để:

- Tự đăng ký tài khoản.
- Tạo thêm ADMIN hoặc thay đổi vai trò.
- Xóa tài khoản.
- Mở khóa tài khoản.
- Đặt lại mật khẩu của người khác.
- Sửa hồ sơ tài khoản.

Các chức năng chưa có trong phạm vi chỉ được bổ sung sau khi cập nhật yêu cầu và tài liệu liên quan.

Tài liệu này là hợp đồng thiết kế, chưa phản ánh trạng thái triển khai.

### Tài liệu liên quan

- [Phạm vi dự án](../scope.md)
- [Quy tắc nghiệp vụ](../business-rules.md)
- [Thiết kế database](../database.md)
- [Kiến trúc backend](../architecture.md)
- [API đăng nhập và đổi mật khẩu](auth.md)

## 2. Danh sách API

| Method | Endpoint | Chức năng | Quyền |
|---|---|---|---|
| POST | `/api/users` | Cấp tài khoản STAFF | ADMIN |
| GET | `/api/users` | Xem danh sách tài khoản | ADMIN |
| GET | `/api/users/{id}` | Xem chi tiết tài khoản | ADMIN |
| PATCH | `/api/users/{id}/status` | Khóa tài khoản STAFF | ADMIN |

Mọi API trong tài liệu đều yêu cầu JWT hợp lệ và tài khoản người gọi đang hoạt động.

## 3. Quy ước chung

### 3.1. Xác thực và phân quyền

Request phải gửi:

```http
Authorization: Bearer <access_token>
```

Backend thực hiện:

1. Xác minh chữ ký và thời hạn JWT.
2. Lấy ID tài khoản từ claim `sub`.
3. Đọc tài khoản hiện tại trong MySQL.
4. Kiểm tra tài khoản tồn tại và đang hoạt động.
5. Kiểm tra vai trò hiện tại là `ADMIN`.

Không dựa vào `role`, `userId` hoặc thông tin quản trị do người gọi tự gửi để cấp quyền.

STAFF không được sử dụng các API này, kể cả khi ID tài khoản được yêu cầu là ID của chính mình.

Quyền đổi mật khẩu của bản thân được thực hiện qua API riêng trong `auth.md`.

### 3.2. Định dạng dữ liệu

- Request và response sử dụng JSON.
- Request có body phải gửi `Content-Type: application/json`.
- Tên trường JSON sử dụng `camelCase`.
- ID được biểu diễn bằng số nguyên dương trong phạm vi kiểu `INT` của database.
- Thời gian trả về theo ISO 8601, múi giờ UTC.
- Các trường request/query không được hỗ trợ phải bị từ chối.
- Không trả `password_hash`, mật khẩu hoặc access token của tài khoản khác.

Ví dụ thời gian:

```text
2026-11-20T10:00:00Z
```

### 3.3. Response thành công

Response một tài khoản:

```json
{
  "data": {
    "id": 7,
    "fullName": "Nguyễn Văn A",
    "email": "staff@example.com",
    "role": "STAFF",
    "active": true,
    "createdAt": "2026-11-20T10:00:00Z",
    "updatedAt": "2026-11-20T10:00:00Z"
  }
}
```

### 3.4. Response lỗi

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Dữ liệu đầu vào không hợp lệ.",
    "details": [
      {
        "field": "email",
        "message": "Email là bắt buộc."
      }
    ]
  }
}
```

Quy ước:

- `code` là mã ổn định để phía gọi API xử lý.
- `message` là thông báo dễ hiểu.
- `details` chỉ có khi cần mô tả lỗi đầu vào.
- Không đưa lại mật khẩu, token hoặc thông tin nội bộ vào lỗi.
- Lỗi database ngoài dự kiến không được giả thành lỗi validation.

### 3.5. Quy tắc email và mật khẩu

- Email là duy nhất trong toàn bộ bảng `users`, kể cả tài khoản đã khóa.
- Loại bỏ khoảng trắng đầu/cuối email trước khi kiểm tra và lưu.
- Cách xử lý chữ hoa/thường phải thống nhất với API đăng nhập và collation database.
- Không tự động trim hoặc biến đổi mật khẩu.
- Chỉ lưu mật khẩu đã băm.

Chính sách độ dài mật khẩu, thư viện băm và cách so sánh email chưa được chốt hoàn toàn. Phải thống nhất trước khi triển khai các API liên quan.

## 4. Cấp tài khoản nhân viên

### 4.1. Thông tin chung

```http
POST /api/users
Authorization: Bearer <access_token>
Content-Type: application/json
```

Chỉ ADMIN được gọi.

API chỉ tạo tài khoản có vai trò `STAFF`, không phải API tạo người dùng với vai trò tùy ý.

### 4.2. Request body

```json
{
  "fullName": "Nguyễn Văn A",
  "email": "staff@example.com",
  "password": "<mật khẩu ban đầu đáp ứng chính sách đã chốt>"
}
```

| Trường | Kiểu | Bắt buộc | Điều kiện |
|---|---|:---:|---|
| `fullName` | string | Có | Loại bỏ khoảng trắng đầu/cuối; không rỗng; tối đa 100 ký tự |
| `email` | string | Có | Định dạng hợp lệ; tối đa 255 ký tự sau chuẩn hóa; không trùng |
| `password` | string | Có | Không rỗng; đáp ứng chính sách mật khẩu được thống nhất |

Không nhận:

- `id`
- `role`
- `active`
- `passwordHash`
- `createdAt`
- `updatedAt`

Backend tự xác định:

- `role = STAFF`
- `active = true`
- ID và các thời điểm liên quan.

### 4.3. Luồng xử lý

1. Xác thực và kiểm tra quyền ADMIN.
2. Kiểm tra, chuẩn hóa các trường được phép.
3. Kiểm tra email đã được sử dụng hay chưa.
4. Băm mật khẩu bằng thư viện đã chọn.
5. Tạo tài khoản STAFF đang hoạt động.
6. Trả thông tin tài khoản, không trả mật khẩu hoặc hash.

Kiểm tra email ở Service giúp trả lỗi rõ ràng, nhưng không thay thế UNIQUE trong database.

Nếu hai request đồng thời tạo cùng email, chỉ một tài khoản được tạo. Request còn lại trả lỗi xung đột email.

### 4.4. Response thành công

**HTTP status: `201 Created`**

Header:

```http
Location: /api/users/7
Cache-Control: no-store
```

Body:

```json
{
  "data": {
    "id": 7,
    "fullName": "Nguyễn Văn A",
    "email": "staff@example.com",
    "role": "STAFF",
    "active": true,
    "createdAt": "2026-11-20T10:00:00Z",
    "updatedAt": "2026-11-20T10:00:00Z"
  }
}
```

API không tự đăng nhập tài khoản mới và không cấp JWT trong response.

### 4.5. Các lỗi riêng

| HTTP status | Mã lỗi | Điều kiện |
|---|---|---|
| 400 | `VALIDATION_ERROR` | Thiếu trường, sai kiểu, dữ liệu không hợp lệ hoặc có trường không hỗ trợ |
| 409 | `EMAIL_ALREADY_EXISTS` | Email đã thuộc một tài khoản, kể cả tài khoản bị khóa |

Ví dụ:

```json
{
  "error": {
    "code": "EMAIL_ALREADY_EXISTS",
    "message": "Email đã được sử dụng."
  }
}
```

### 4.6. Cấp và bàn giao mật khẩu ban đầu

- Mỗi tài khoản STAFF được cấp một mật khẩu ban đầu riêng, ngẫu nhiên và đáp ứng chính sách mật khẩu đã thống nhất trước khi triển khai.
- Không dùng chung mật khẩu hoặc đặt theo quy luật dễ đoán như staff1, staff2.
- ADMIN chuẩn bị mật khẩu và nhập vào trường `password` khi tạo tài khoản. API hiện tại không tự sinh mật khẩu.
- ADMIN bàn giao thông tin đăng nhập cho đúng nhân viên qua kênh phù hợp.
- Không đưa mật khẩu thật vào issue, pull request, tài liệu hoặc log.
- Không trả lại mật khẩu trong response.
- Nhân viên được hướng dẫn đổi mật khẩu sau khi nhận tài khoản, thông qua API trong auth.md.
- Phiên bản hiện tại chưa gửi email mời hoặc bắt buộc đổi mật khẩu ở lần đăng nhập đầu tiên.

## 5. Xem danh sách tài khoản

### 5.1. Thông tin chung

```http
GET /api/users?page=1&pageSize=20&role=STAFF&active=true
Authorization: Bearer <access_token>
```

Chỉ ADMIN được gọi.

### 5.2. Query parameters

| Tham số | Kiểu logic | Bắt buộc | Mặc định | Điều kiện |
|---|---|:---:|---|---|
| `page` | integer | Không | 1 | Số nguyên ≥ 1 |
| `pageSize` | integer | Không | 20 | Số nguyên từ 1 đến 100 |
| `role` | string | Không | Không lọc | `ADMIN` hoặc `STAFF` |
| `active` | boolean | Không | Không lọc | Chuỗi query `true` hoặc `false` |

Quy ước:

- Query được gửi dưới dạng chuỗi và phải được parse rõ ràng.
- Giá trị rỗng, lặp lại hoặc không hợp lệ bị từ chối.
- Không chuyển tùy ý `1`, `0` hoặc chuỗi khác thành boolean.
- Khi dùng nhiều bộ lọc, các điều kiện được kết hợp bằng AND.
- Sắp xếp mặc định theo `id DESC`.
- Không hỗ trợ tùy ý tên cột sắp xếp từ request.

Thiết kế hiện tại không bổ sung tìm kiếm tài khoản theo tên/email.

### 5.3. Response thành công

**HTTP status: `200 OK`**

```json
{
  "data": [
    {
      "id": 7,
      "fullName": "Nguyễn Văn A",
      "email": "staff@example.com",
      "role": "STAFF",
      "active": true,
      "createdAt": "2026-11-20T10:00:00Z",
      "updatedAt": "2026-11-20T10:00:00Z"
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

Quy tắc phân trang:

- `totalItems` là tổng số tài khoản phù hợp bộ lọc.
- `totalPages` được tính từ `totalItems` và `pageSize`.
- Không có dữ liệu phù hợp: `data = []`, `totalItems = 0`, `totalPages = 0`.
- Trang vượt quá số trang hiện có trả `200` và `data = []`.
- Danh sách rỗng không phải lỗi `404`.

Header:

```http
Cache-Control: no-store
```

### 5.4. Các lỗi riêng

| HTTP status | Mã lỗi | Điều kiện |
|---|---|---|
| 400 | `VALIDATION_ERROR` | Query sai kiểu, vượt giới hạn hoặc có tham số không hỗ trợ |

Không trả toàn bộ danh sách không giới hạn khi người gọi không truyền tham số phân trang.

## 6. Xem chi tiết tài khoản

### 6.1. Thông tin chung

```http
GET /api/users/{id}
Authorization: Bearer <access_token>
```

Chỉ ADMIN được gọi.

### 6.2. Path parameter

| Tham số | Điều kiện |
|---|---|
| `id` | Số nguyên dương, nằm trong phạm vi kiểu INT đã chọn |

Ví dụ:

```http
GET /api/users/7
```

### 6.3. Response thành công

**HTTP status: `200 OK`**

```json
{
  "data": {
    "id": 7,
    "fullName": "Nguyễn Văn A",
    "email": "staff@example.com",
    "role": "STAFF",
    "active": false,
    "createdAt": "2026-11-20T10:00:00Z",
    "updatedAt": "2026-11-20T11:00:00Z"
  }
}
```

ADMIN được xem cả tài khoản hoạt động và tài khoản bị khóa.

Header:

```http
Cache-Control: no-store
```

### 6.4. Các lỗi riêng

| HTTP status | Mã lỗi | Điều kiện |
|---|---|---|
| 400 | `VALIDATION_ERROR` | ID không hợp lệ |
| 404 | `USER_NOT_FOUND` | ID hợp lệ nhưng tài khoản không tồn tại |

## 7. Khóa tài khoản nhân viên

### 7.1. Thông tin chung

```http
PATCH /api/users/{id}/status
Authorization: Bearer <access_token>
Content-Type: application/json
```

Chỉ ADMIN được gọi.

Trong phạm vi hiện tại, API chỉ hỗ trợ chuyển tài khoản STAFF sang trạng thái bị khóa.

### 7.2. Request body

```json
{
  "active": false
}
```

| Trường | Kiểu | Bắt buộc | Điều kiện |
|---|---|:---:|---|
| `active` | boolean | Có | Chỉ chấp nhận giá trị JSON `false` |

Không chấp nhận:

- Chuỗi `"false"`.
- `null`.
- Giá trị `true`, vì chức năng mở khóa chưa thuộc phạm vi.
- Các trường bổ sung như `role`, `email` hoặc `password`.

### 7.3. Điều kiện nghiệp vụ

- Tài khoản mục tiêu phải tồn tại.
- Chỉ được khóa tài khoản mang vai trò STAFF.
- Không sử dụng endpoint này để khóa ADMIN, kể cả ADMIN đang gọi.
- Không xóa tài khoản hoặc chứng từ liên quan.
- Nếu STAFF đã bị khóa, trả thành công với trạng thái hiện tại.
- Không thay đổi `updated_at` chỉ vì gửi lại yêu cầu khóa một tài khoản vốn đã bị khóa.

Việc giới hạn mục tiêu là STAFF bám theo chức năng “khóa tài khoản nhân viên”. Quản lý vòng đời tài khoản ADMIN không thuộc hợp đồng này.

### 7.4. Luồng xử lý

1. Xác thực và kiểm tra quyền ADMIN.
2. Kiểm tra ID và body.
3. Kiểm tra tài khoản mục tiêu tồn tại và có vai trò STAFF.
4. Nếu đang hoạt động, cập nhật `active = false` và `updated_at`.
5. Nếu đã bị khóa, giữ nguyên dữ liệu.
6. Trả trạng thái tài khoản sau xử lý.

### 7.5. Ảnh hưởng sau khi khóa

- Không thể đăng nhập để lấy token mới.
- JWT đã cấp không bị xóa hoặc thay đổi chữ ký.
- Những request kiểm tra trạng thái tài khoản sau khi khóa được commit phải bị từ chối.
- Request đã vượt qua kiểm tra quyền trước thời điểm khóa có thể đang xử lý; thao tác khóa không tự hủy các transaction đang chạy.
- Chứng từ do nhân viên tạo được giữ nguyên.
- ADMIN vẫn được xử lý chứng từ hợp lệ của nhân viên theo quy tắc nghiệp vụ.

### 7.6. Response thành công

**HTTP status: `200 OK`**

```json
{
  "data": {
    "id": 7,
    "fullName": "Nguyễn Văn A",
    "email": "staff@example.com",
    "role": "STAFF",
    "active": false,
    "createdAt": "2026-11-20T10:00:00Z",
    "updatedAt": "2026-11-20T11:00:00Z"
  }
}
```

### 7.7. Các lỗi riêng

| HTTP status | Mã lỗi | Điều kiện |
|---|---|---|
| 400 | `VALIDATION_ERROR` | ID/body không hợp lệ, `active` không phải `false` hoặc có trường không hỗ trợ |
| 404 | `USER_NOT_FOUND` | Tài khoản mục tiêu không tồn tại |
| 409 | `USER_NOT_LOCKABLE` | Tài khoản mục tiêu là ADMIN |

Ví dụ:

```json
{
  "error": {
    "code": "USER_NOT_LOCKABLE",
    "message": "API này chỉ hỗ trợ khóa tài khoản nhân viên."
  }
}
```

## 8. Lỗi dùng chung

Các lỗi dưới đây áp dụng cho mọi API tương ứng trong tài liệu.

| HTTP status | Mã lỗi | Điều kiện |
|---|---|---|
| 400 | `INVALID_JSON` | Body không phải JSON hợp lệ |
| 401 | `AUTH_REQUIRED` | Không gửi thông tin xác thực |
| 401 | `INVALID_TOKEN` | Token không hợp lệ hoặc tài khoản của người gọi không còn tồn tại |
| 401 | `TOKEN_EXPIRED` | Token hết hạn |
| 403 | `ACCOUNT_DISABLED` | Tài khoản người gọi bị khóa |
| 403 | `FORBIDDEN` | Tài khoản hoạt động nhưng không có quyền ADMIN |
| 413 | `PAYLOAD_TOO_LARGE` | Body vượt giới hạn cấu hình |
| 415 | `UNSUPPORTED_MEDIA_TYPE` | Body không sử dụng kiểu nội dung JSON được hỗ trợ |
| 500 | `INTERNAL_ERROR` | Lỗi ngoài dự kiến |

Lỗi liên quan body chỉ áp dụng cho request có body. GET không yêu cầu `Content-Type: application/json`.

Với lỗi `401`, backend gửi:

```http
WWW-Authenticate: Bearer
```

### Thứ tự kiểm tra

- Xác thực và phân quyền được thực hiện trước khi tra cứu tài khoản mục tiêu.
- STAFF không được phân biệt tài khoản mục tiêu tồn tại hay không thông qua API quản trị.
- Nếu request vi phạm nhiều điều kiện, lỗi trả trước phụ thuộc thứ tự kiểm tra đã triển khai.
- Không yêu cầu trả tất cả lỗi nghiệp vụ trong một response.

## 9. Ánh xạ dữ liệu

| Trường API | Cột database | Ghi chú |
|---|---|---|
| `id` | `users.id` | Backend tạo; request không tự gán |
| `fullName` | `users.full_name` | Chuẩn hóa khoảng trắng đầu/cuối |
| `email` | `users.email` | Duy nhất, dùng để đăng nhập |
| `password` | `users.password_hash` | Phải băm trước khi lưu; không trả ra |
| `role` | `users.role` | API tạo tài khoản luôn gán STAFF |
| `active` | `users.active` | Tạo mới là true; khóa là false |
| `createdAt` | `users.created_at` | Chỉ đọc qua API |
| `updatedAt` | `users.updated_at` | Backend cập nhật khi dữ liệu thay đổi |

JWT của người gọi xác định người được phép thực hiện thao tác, không được dùng làm ID tài khoản mục tiêu cho các endpoint có `{id}`.

## 10. Nhất quán dữ liệu và phạm vi cập nhật

### 10.1. Tạo tài khoản

- Sử dụng UNIQUE của database để ngăn trùng email khi có request đồng thời.
- Phân biệt lỗi trùng email với lỗi database ngoài dự kiến.
- Không lưu tài khoản nếu băm mật khẩu thất bại.
- Không báo thành công khi thao tác ghi chưa hoàn tất.
- API không bảo đảm các lần gửi lại request tạo tài khoản trả cùng response; sau lần tạo thành công, cùng email sẽ nhận lỗi xung đột.

### 10.2. Khóa tài khoản

- Thay đổi trạng thái phải áp dụng đúng tài khoản mục tiêu.
- Hai yêu cầu cùng khóa một STAFF phải cho kết quả cuối cùng là `active = false`.
- Không phát sinh thao tác xóa hoặc thay đổi vai trò.
- Không cập nhật bất kỳ phiếu nhập, đơn bán hoặc số lượng tồn nào.
- Nếu thao tác database thất bại, không trả thông báo đã khóa thành công.

### 10.3. Giữ nguyên schema

Các API trong tài liệu sử dụng bảng `users` hiện có.

Không yêu cầu thêm:

- `token_version`.
- Bảng refresh token.
- Cột bắt buộc đổi mật khẩu lần đầu.
- Cột người khóa hoặc thời điểm khóa riêng.

`updated_at` phản ánh lần thay đổi gần nhất, không phải lịch sử đầy đủ về các thao tác quản trị.

## 11. Yêu cầu bảo mật

- Dùng HTTPS trên môi trường triển khai.
- Kiểm tra quyền ADMIN tại backend cho mọi endpoint trong tài liệu.
- Chỉ nhận và cập nhật các trường được phép.
- Không truyền trực tiếp toàn bộ request body vào câu lệnh ghi dữ liệu.
- Sử dụng truy vấn có tham số hoặc cơ chế tương đương của thư viện truy cập dữ liệu.
- Không ghi mật khẩu, hash, token hoặc header Authorization vào log.
- Không cache response tài khoản trong Redis cache nghiệp vụ.
- Response thành công chứa thông tin tài khoản sử dụng `Cache-Control: no-store`.
- Không gửi mật khẩu qua URL hoặc trả lại trong response.
- Nếu bổ sung log thao tác quản trị, chỉ ghi các trường cần thiết như ID người thực hiện, ID mục tiêu và loại thao tác; không ghi dữ liệu bí mật.

## 12. Tình huống kiểm thử

### 12.1. Xác thực và phân quyền

| Mã | Tình huống | Kết quả mong đợi |
|---|---|---|
| USER-AUTH-01 | Không có JWT | 401 |
| USER-AUTH-02 | JWT sai chữ ký hoặc bị sửa | 401 |
| USER-AUTH-03 | JWT hết hạn | 401 |
| USER-AUTH-04 | Tài khoản người gọi đã bị khóa | 403 |
| USER-AUTH-05 | STAFF gọi API quản trị | 403 |
| USER-AUTH-06 | ADMIN hoạt động gọi API hợp lệ | Được xử lý theo nghiệp vụ |
| USER-AUTH-07 | Request tự gửi role ADMIN | Không được cấp quyền từ giá trị gửi lên |

### 12.2. Cấp tài khoản

| Mã | Tình huống | Kết quả mong đợi |
|---|---|---|
| USER-CREATE-01 | Dữ liệu hợp lệ | 201; STAFF hoạt động |
| USER-CREATE-02 | Thiếu hoặc sai kiểu trường bắt buộc | 400 |
| USER-CREATE-03 | Họ tên chỉ có khoảng trắng | 400 |
| USER-CREATE-04 | Email sai định dạng hoặc vượt giới hạn | 400 |
| USER-CREATE-05 | Email đã tồn tại ở tài khoản hoạt động | 409 |
| USER-CREATE-06 | Email đã tồn tại ở tài khoản bị khóa | 409 |
| USER-CREATE-07 | Hai request đồng thời tạo cùng email | Chỉ tạo một tài khoản; request còn lại 409 |
| USER-CREATE-08 | Gửi thêm role hoặc active | 400 |
| USER-CREATE-09 | Mật khẩu không đạt chính sách đã chốt | 400 |
| USER-CREATE-10 | Tạo thành công | Không có mật khẩu/hash/token trong response |
| USER-CREATE-11 | Kiểm tra dữ liệu lưu | Lưu hash, không lưu mật khẩu gốc |
| USER-CREATE-12 | Lỗi khi băm hoặc ghi dữ liệu | Không báo thành công; không tạo dữ liệu dở dang |

### 12.3. Danh sách và chi tiết

| Mã | Tình huống | Kết quả mong đợi |
|---|---|---|
| USER-READ-01 | Không truyền query | Trang 1, tối đa 20 tài khoản, id giảm dần |
| USER-READ-02 | Lọc role và active | Chỉ trả tài khoản khớp toàn bộ điều kiện |
| USER-READ-03 | Không có kết quả | 200; danh sách rỗng |
| USER-READ-04 | Trang vượt số trang | 200; danh sách rỗng |
| USER-READ-05 | page/pageSize sai hoặc vượt giới hạn | 400 |
| USER-READ-06 | active là chuỗi không hợp lệ | 400 |
| USER-READ-07 | Query không hỗ trợ hoặc lặp lại | 400 |
| USER-READ-08 | ID chi tiết không hợp lệ | 400 |
| USER-READ-09 | ID hợp lệ nhưng không tồn tại | 404 |
| USER-READ-10 | Xem tài khoản bị khóa | 200; active = false |
| USER-READ-11 | Kiểm tra response | Không lộ password_hash |

### 12.4. Khóa tài khoản

| Mã | Tình huống | Kết quả mong đợi |
|---|---|---|
| USER-LOCK-01 | ADMIN khóa STAFF hoạt động | 200; active = false |
| USER-LOCK-02 | Khóa lại STAFF đã bị khóa | 200; trạng thái và updated_at không đổi |
| USER-LOCK-03 | ID không tồn tại | 404 |
| USER-LOCK-04 | Mục tiêu là ADMIN | 409 |
| USER-LOCK-05 | active = true | 400; không mở khóa |
| USER-LOCK-06 | active là chuỗi `"false"` | 400 |
| USER-LOCK-07 | Body có trường không hỗ trợ | 400 |
| USER-LOCK-08 | Hai request cùng khóa một STAFF | Kết quả cuối là bị khóa |
| USER-LOCK-09 | STAFF đăng nhập sau khi bị khóa | Bị từ chối theo auth.md |
| USER-LOCK-10 | STAFF dùng JWT cũ sau khi khóa | Request kiểm tra sau commit bị từ chối |
| USER-LOCK-11 | STAFF đã tạo chứng từ trước khi khóa | Chứng từ vẫn tồn tại, không thay đổi |
| USER-LOCK-12 | Database lỗi khi khóa | Không báo thành công sai |

## 13. Các điểm cần chốt trước khi triển khai

| Nội dung | Trạng thái |
|---|---|
| Chỉ ADMIN được quản lý tài khoản | Đã xác định trong phạm vi |
| API tạo chỉ cấp STAFF | Cụ thể hóa chức năng cấp tài khoản nhân viên |
| Không xóa tài khoản có lịch sử | Đã xác định trong nghiệp vụ |
| JWT và kiểm tra trạng thái từ MySQL | Đã chốt trong auth.md |
| Chính sách mật khẩu mới | Chưa chốt |
| Thư viện băm và cấu hình chi phí | Chưa chốt |
| Chuẩn hóa chữ hoa/thường email và collation | Chưa chốt |
| Cách khởi tạo ADMIN đầu tiên | Chưa chốt; thực hiện qua cơ chế khởi tạo riêng, không mở API tạo ADMIN công khai |

Các lựa chọn hợp đồng được đề xuất trong tài liệu này gồm:

- Phân trang mặc định 20, tối đa 100 tài khoản.
- Sắp xếp `id DESC`.
- Khóa lại tài khoản đã khóa trả thành công và không sửa `updated_at`.
- Endpoint trạng thái chỉ khóa STAFF; chưa hỗ trợ mở khóa hoặc khóa ADMIN.

Cần rà soát các lựa chọn này cùng tài liệu phạm vi và nghiệp vụ trước khi triển khai. Nếu thay đổi, cập nhật đồng bộ hợp đồng API và các test liên quan.
