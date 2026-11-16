# API đăng nhập và đổi mật khẩu

## 1. Mục đích và phạm vi

Tài liệu mô tả hợp đồng API cho:

- Đăng nhập bằng email và mật khẩu.
- Đổi mật khẩu của tài khoản đang đăng nhập.

Tài liệu là căn cứ để triển khai backend, viết OpenAPI và kiểm thử.

| Method | Endpoint | Chức năng | Xác thực |
|---|---|---|---|
| POST | `/api/auth/login` | Đăng nhập | Không yêu cầu JWT |
| PATCH | `/api/users/me/password` | Đổi mật khẩu của bản thân | Yêu cầu JWT |

Không bao gồm:

- Đăng ký tài khoản công khai.
- Cấp hoặc khóa tài khoản nhân viên.
- Quên mật khẩu hoặc đặt lại mật khẩu bởi ADMIN.
- Refresh token.
- Thu hồi token hoặc đăng xuất phía server.

Tài liệu liên quan:

- [Phạm vi](../scope.md)
- [Quy tắc nghiệp vụ](../business-rules.md)
- [Database](../database.md)
- [Kiến trúc](../architecture.md)

## 2. Quy ước chung

### 2.1. Request và response

- API nhận và trả dữ liệu JSON.
- Request có body phải gửi `Content-Type: application/json`.
- Tên trường JSON sử dụng `camelCase`.
- Tên cột database tiếp tục sử dụng `snake_case`.
- Không gửi mật khẩu hoặc token trong URL.
- Không chấp nhận trường không được mô tả trong request.
- Không trả `password_hash` hoặc mật khẩu trong bất kỳ response nào.

Response thành công:

```json
{
  "data": {}
}
```

Response lỗi:

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

- `code`: mã lỗi ổn định để phía gọi API xử lý.
- `message`: thông báo dễ hiểu, không dùng làm khóa điều khiển logic.
- `details`: chỉ có khi cần mô tả lỗi đầu vào.
- Không đưa lại giá trị mật khẩu đã gửi vào thông báo lỗi.

Các ví dụ bên dưới minh họa cấu trúc; giá trị token và tài khoản không phải dữ liệu thật.

### 2.2. Chính sách JWT đã chốt

- Sử dụng access token JWT có thời hạn 1 giờ, tương đương 3.600 giây.
- Các claim gồm ID tài khoản (`sub`), thời điểm cấp (`iat`) và hết hạn (`exp`).
- `sub` biểu diễn ID tài khoản dưới dạng chuỗi.
- Không chứa mật khẩu hoặc thông tin bí mật trong payload.
- Không triển khai refresh token trong phiên bản đầu.
- Khi token hết hạn, người dùng đăng nhập lại.

API được bảo vệ nhận token qua header:

```http
Authorization: Bearer <access_token>
```

Backend phải:

1. Kiểm tra chữ ký bằng thuật toán được cấu hình.
2. Kiểm tra thời hạn và các claim bắt buộc.
3. Đọc tài khoản tương ứng trong MySQL.
4. Từ chối nếu tài khoản không tồn tại hoặc bị khóa.
5. Sử dụng vai trò hiện tại trong database để phân quyền.

Việc đổi mật khẩu không vô hiệu hóa access token đã cấp. Token cũ vẫn dùng được tới khi hết hạn nếu tài khoản còn hoạt động.

### 2.3. Chính sách mật khẩu mới đề xuất

**Trạng thái: cần xác nhận trước khi triển khai.**

Áp dụng cho mật khẩu mới và thống nhất với API cấp tài khoản sau này:

- Là chuỗi có độ dài từ 12 đến 64 ký tự.
- Không vượt quá 72 byte khi mã hóa UTF-8 để tương thích nếu lựa chọn bcrypt.
- Không bắt buộc tổ hợp chữ hoa, chữ thường, số và ký tự đặc biệt.
- Không tự động trim, chuyển chữ hoa/thường hoặc biến đổi mật khẩu.
- Mật khẩu mới không được giống mật khẩu hiện tại.
- Chỉ lưu kết quả băm bằng thư viện phù hợp.

Giới hạn theo byte phải được kiểm tra rõ ràng; không được âm thầm cắt ngắn mật khẩu.

Không áp dụng chính sách độ dài mật khẩu mới để từ chối một mật khẩu hiện tại vốn đang hợp lệ khi đăng nhập hoặc xác minh đổi mật khẩu.

## 3. API đăng nhập

### 3.1. Thông tin chung

```http
POST /api/auth/login
Content-Type: application/json
```

Mục đích: xác minh thông tin đăng nhập và cấp access token.

API không yêu cầu JWT. Nếu request mang sẵn token, token đó không thay thế việc kiểm tra email và mật khẩu.

### 3.2. Request body

```json
{
  "email": "staff@example.com",
  "password": "MatKhauViDu123!"
}
```

| Trường | Kiểu | Bắt buộc | Điều kiện |
|---|---|:---:|---|
| `email` | string | Có | Không rỗng, định dạng email hợp lệ, tối đa 255 ký tự sau khi loại bỏ khoảng trắng đầu/cuối |
| `password` | string | Có | Không rỗng; không tự động trim hoặc biến đổi |

Cách phân biệt chữ hoa/thường của email phải thống nhất với API tạo tài khoản và collation database. Không tự áp dụng các quy tắc khác nhau giữa các luồng.

### 3.3. Luồng xử lý

1. Kiểm tra JSON và các trường đầu vào.
2. Tìm tài khoản theo email.
3. Xác minh mật khẩu bằng hàm kiểm tra hash của thư viện.
4. Nếu không có tài khoản hoặc mật khẩu sai, trả cùng một loại lỗi.
5. Sau khi thông tin đăng nhập hợp lệ, kiểm tra `active`.
6. Nếu tài khoản hoạt động, tạo JWT và trả thông tin cần thiết.

Nếu tài khoản bị khóa nhưng mật khẩu cũng sai, trả lỗi thông tin đăng nhập không hợp lệ; không trả trạng thái khóa trước khi xác minh mật khẩu.

### 3.4. Response thành công

**HTTP status: `200 OK`**

```json
{
  "data": {
    "accessToken": "<access_token>",
    "tokenType": "Bearer",
    "expiresIn": 3600,
    "user": {
      "id": 7,
      "fullName": "Nguyễn Văn A",
      "email": "staff@example.com",
      "role": "STAFF"
    }
  }
}
```

| Trường | Ý nghĩa |
|---|---|
| `accessToken` | JWT dùng cho các request được bảo vệ |
| `tokenType` | Luôn là `Bearer` |
| `expiresIn` | Thời hạn token tính bằng giây kể từ lúc cấp |
| `user` | Thông tin tài khoản tại thời điểm đăng nhập |

Thông tin `user.role` trong response không thay thế việc kiểm tra quyền tại backend.

Response chứa token phải có:

```http
Cache-Control: no-store
```

### 3.5. Các trường hợp lỗi

| HTTP status | Mã lỗi | Điều kiện |
|---|---|---|
| 400 | `INVALID_JSON` | Body không phải JSON hợp lệ |
| 400 | `VALIDATION_ERROR` | Thiếu trường, sai kiểu, email sai định dạng hoặc có trường không được hỗ trợ |
| 401 | `INVALID_CREDENTIALS` | Email không tồn tại hoặc mật khẩu sai |
| 403 | `ACCOUNT_DISABLED` | Thông tin đăng nhập đúng nhưng tài khoản bị khóa |
| 413 | `PAYLOAD_TOO_LARGE` | Request vượt giới hạn body được cấu hình |
| 415 | `UNSUPPORTED_MEDIA_TYPE` | Request body không sử dụng kiểu nội dung JSON được hỗ trợ |
| 500 | `INTERNAL_ERROR` | Lỗi hệ thống ngoài dự kiến |

Ví dụ sai thông tin đăng nhập:

```json
{
  "error": {
    "code": "INVALID_CREDENTIALS",
    "message": "Email hoặc mật khẩu không chính xác."
  }
}
```

Ví dụ tài khoản bị khóa:

```json
{
  "error": {
    "code": "ACCOUNT_DISABLED",
    "message": "Tài khoản đã bị khóa. Vui lòng liên hệ quản trị viên."
  }
}
```

Không sử dụng hai thông báo khác nhau để phân biệt email không tồn tại và mật khẩu sai.

## 4. API đổi mật khẩu

### 4.1. Thông tin chung

```http
PATCH /api/users/me/password
Authorization: Bearer <access_token>
Content-Type: application/json
```

Mục đích: người dùng đổi mật khẩu của chính tài khoản đang đăng nhập.

Áp dụng cho cả ADMIN và STAFF đang hoạt động.

API không nhận ID tài khoản mục tiêu. Danh tính được xác định từ token đã xác minh và tài khoản trong database.

ADMIN cũng không dùng endpoint này để đổi mật khẩu của người khác.

### 4.2. Request body

```json
{
  "currentPassword": "MatKhauCu123!",
  "newPassword": "MatKhauMoi456!"
}
```

| Trường | Kiểu | Bắt buộc | Điều kiện |
|---|---|:---:|---|
| `currentPassword` | string | Có | Không rỗng; phải khớp mật khẩu hiện tại |
| `newPassword` | string | Có | Đáp ứng chính sách mật khẩu mới và khác mật khẩu hiện tại |

Không nhận các trường như `userId`, `role`, `active` hoặc `passwordHash`.

Không yêu cầu `confirmPassword` ở API. Nếu sau này có giao diện, giao diện có thể yêu cầu nhập lại để tránh gõ nhầm.

### 4.3. Luồng xử lý

1. Xác minh JWT.
2. Đọc tài khoản hiện tại và kiểm tra trạng thái hoạt động.
3. Kiểm tra dữ liệu request.
4. Xác minh `currentPassword`.
5. Kiểm tra chính sách và việc không tái sử dụng mật khẩu hiện tại.
6. Tạo hash cho mật khẩu mới.
7. Cập nhật `password_hash` và `updated_at`.
8. Trả kết quả khi cập nhật thành công.

Không so sánh trực tiếp hai chuỗi hash để kết luận hai mật khẩu có giống nhau. Sử dụng hàm kiểm tra mật khẩu của thư viện.

### 4.4. Xử lý đồng thời

Không để hai request cùng xác minh một mật khẩu cũ rồi ghi đè mật khẩu mới của nhau mà không phát hiện.

Cách triển khai cần bảo đảm dữ liệu tài khoản, trạng thái hoạt động và hash hiện tại vẫn phù hợp khi ghi thay đổi, thông qua khóa hoặc cập nhật có điều kiện.

Nếu mật khẩu hoặc trạng thái tài khoản đã thay đổi trước khi ghi, request không được ghi đè một cách mù quáng. Trả lỗi phù hợp với trạng thái phát hiện được.

Không cần bổ sung cột `token_version` để triển khai API này.

### 4.5. Response thành công

**HTTP status: `200 OK`**

```json
{
  "data": {
    "message": "Đổi mật khẩu thành công."
  }
}
```

Sau khi thành công:

- Mật khẩu cũ không còn đăng nhập được.
- Mật khẩu mới dùng được cho các lần đăng nhập tiếp theo.
- Access token hiện tại và các token đã cấp trước đó vẫn có hiệu lực tới khi hết hạn.
- Không cấp access token mới trong response đổi mật khẩu.
- Không trả mật khẩu hoặc hash mới.

### 4.6. Các trường hợp lỗi

| HTTP status | Mã lỗi | Điều kiện |
|---|---|---|
| 400 | `INVALID_JSON` | Body không phải JSON hợp lệ |
| 400 | `VALIDATION_ERROR` | Thiếu trường, sai kiểu, mật khẩu mới không đạt chính sách hoặc có trường không được hỗ trợ |
| 400 | `CURRENT_PASSWORD_INCORRECT` | Mật khẩu hiện tại không chính xác |
| 400 | `PASSWORD_REUSE_NOT_ALLOWED` | Mật khẩu mới giống mật khẩu hiện tại |
| 401 | `AUTH_REQUIRED` | Không gửi thông tin xác thực |
| 401 | `INVALID_TOKEN` | Token sai định dạng, sai chữ ký, thiếu claim hợp lệ hoặc tài khoản tương ứng không còn tồn tại |
| 401 | `TOKEN_EXPIRED` | Token hết hạn |
| 403 | `ACCOUNT_DISABLED` | Tài khoản bị khóa |
| 409 | `ACCOUNT_STATE_CHANGED` | Cập nhật bị từ chối vì dữ liệu tài khoản thay đổi đồng thời |
| 413 | `PAYLOAD_TOO_LARGE` | Request vượt giới hạn body |
| 415 | `UNSUPPORTED_MEDIA_TYPE` | Kiểu nội dung request không được hỗ trợ |
| 500 | `INTERNAL_ERROR` | Lỗi hệ thống ngoài dự kiến |

Ví dụ mật khẩu hiện tại sai:

```json
{
  "error": {
    "code": "CURRENT_PASSWORD_INCORRECT",
    "message": "Mật khẩu hiện tại không chính xác."
  }
}
```

Ví dụ token hết hạn:

```json
{
  "error": {
    "code": "TOKEN_EXPIRED",
    "message": "Phiên đăng nhập đã hết hạn. Vui lòng đăng nhập lại."
  }
}
```

Khi response là `401` trên API được bảo vệ, backend gửi header `WWW-Authenticate: Bearer`.

## 5. Ánh xạ dữ liệu

| Trường API / JWT | Cột database | Cách sử dụng |
|---|---|---|
| `email` | `users.email` | Tìm tài khoản đăng nhập |
| `password` | `users.password_hash` | Xác minh bằng thư viện hash |
| `currentPassword` | `users.password_hash` | Xác minh trước khi đổi mật khẩu |
| `newPassword` | `users.password_hash` | Băm trước khi lưu |
| JWT `sub` | `users.id` | Xác định tài khoản đang gọi |
| `user.fullName` | `users.full_name` | Thông tin trả về |
| `user.role` | `users.role` | Vai trò hiện tại |
| Không nhận từ request | `users.active` | Kiểm tra trạng thái tài khoản |
| Không nhận từ request | `users.updated_at` | Cập nhật khi đổi mật khẩu |

Hai API không thay đổi role, email, tên hoặc trạng thái hoạt động của tài khoản.

## 6. Yêu cầu bảo mật và vận hành

- Môi trường triển khai phải dùng HTTPS để bảo vệ mật khẩu và token trên đường truyền.
- Không ghi request body chứa mật khẩu hoặc header Authorization vào log.
- Không commit secret ký JWT vào repository.
- Thuật toán xác minh token phải do backend cấu hình, không lựa chọn tùy ý theo token gửi lên.
- Không trả chi tiết SQL, stack trace hoặc cấu hình nội bộ trong lỗi.
- Lỗi kết nối database không được chuyển thành lỗi “sai mật khẩu”.
- Không cấp token nếu bước xác minh tài khoản chưa hoàn tất.
- Đề xuất bổ sung giới hạn tần suất cho đăng nhập và đổi mật khẩu trước khi công khai môi trường demo.

Khi triển khai giới hạn tần suất, cần chốt ngưỡng và cập nhật hợp đồng lỗi `429 RATE_LIMIT_EXCEEDED`, bao gồm thời gian chờ phù hợp. Không coi tính năng này đã hoàn thành chỉ vì được đề cập trong tài liệu.

## 7. Tình huống kiểm thử

### 7.1. Đăng nhập

| Mã | Tình huống | Kết quả mong đợi |
|---|---|---|
| AUTH-LOGIN-01 | STAFF hoạt động, thông tin đúng | 200; token và thông tin STAFF |
| AUTH-LOGIN-02 | ADMIN hoạt động, thông tin đúng | 200; token và thông tin ADMIN |
| AUTH-LOGIN-03 | Email không tồn tại | 401 `INVALID_CREDENTIALS` |
| AUTH-LOGIN-04 | Sai mật khẩu | Cùng status, mã lỗi và thông báo như email không tồn tại |
| AUTH-LOGIN-05 | Tài khoản khóa, mật khẩu đúng | 403 `ACCOUNT_DISABLED` |
| AUTH-LOGIN-06 | Tài khoản khóa, mật khẩu sai | 401 `INVALID_CREDENTIALS` |
| AUTH-LOGIN-07 | Thiếu hoặc sai kiểu dữ liệu | 400 |
| AUTH-LOGIN-08 | JSON không hợp lệ | 400 `INVALID_JSON` |
| AUTH-LOGIN-09 | Đăng nhập thành công | Không có mật khẩu/hash trong response |
| AUTH-LOGIN-10 | Kiểm tra token được cấp | Có claim hợp lệ và thời hạn 3.600 giây |
| AUTH-LOGIN-11 | Database lỗi | Không cấp token; không báo sai mật khẩu |

### 7.2. Đổi mật khẩu và xác thực

| Mã | Tình huống | Kết quả mong đợi |
|---|---|---|
| AUTH-PASS-01 | Token hợp lệ, mật khẩu hiện tại đúng, mật khẩu mới hợp lệ | 200; lưu hash mới |
| AUTH-PASS-02 | Thiếu token | 401 `AUTH_REQUIRED` |
| AUTH-PASS-03 | Token bị sửa hoặc sai chữ ký | 401 `INVALID_TOKEN` |
| AUTH-PASS-04 | Token hết hạn | 401 `TOKEN_EXPIRED` |
| AUTH-PASS-05 | Tài khoản bị khóa sau khi đã cấp token | 403 |
| AUTH-PASS-06 | Tài khoản không còn tồn tại | 401 |
| AUTH-PASS-07 | Mật khẩu hiện tại sai | 400; hash không đổi |
| AUTH-PASS-08 | Mật khẩu mới vi phạm chính sách | 400; hash không đổi |
| AUTH-PASS-09 | Mật khẩu mới giống hiện tại | 400; hash không đổi |
| AUTH-PASS-10 | Request có `userId` hoặc `role` | 400; không sửa tài khoản khác |
| AUTH-PASS-11 | Sau khi đổi mật khẩu | Mật khẩu cũ đăng nhập thất bại, mật khẩu mới đăng nhập thành công |
| AUTH-PASS-12 | Dùng token đã cấp trước khi đổi mật khẩu | Vẫn dùng được nếu chưa hết hạn và tài khoản hoạt động |
| AUTH-PASS-13 | Hai request đồng thời dùng cùng mật khẩu cũ | Không ghi đè mất cập nhật; request xung đột thất bại |
| AUTH-PASS-14 | Lỗi khi cập nhật database | Không báo thành công khi mật khẩu chưa được lưu |
| AUTH-PASS-15 | Mật khẩu chứa khoảng trắng | Không bị tự động trim hoặc biến đổi |
| AUTH-PASS-16 | Đổi mật khẩu thành công | Không trả token mới hoặc password hash |

Nếu request vi phạm nhiều điều kiện, thứ tự kiểm tra có thể quyết định lỗi được trả trước. Mỗi test tập trung vào một điều kiện và không yêu cầu backend trả toàn bộ lỗi nghiệp vụ cùng lúc.

## 8. Các điểm cần xác nhận trước khi triển khai

| Nội dung | Trạng thái |
|---|---|
| JWT access token 1 giờ | Đã chốt |
| Không refresh token | Đã chốt |
| Kiểm tra trạng thái và vai trò trong MySQL | Đã chốt |
| Token cũ còn hiệu lực sau đổi mật khẩu | Đã chốt |
| Không bổ sung cột database | Giữ nguyên thiết kế |
| Mật khẩu mới 12–64 ký tự, tối đa 72 byte UTF-8 | Đề xuất trong tài liệu này, cần xác nhận |
| Chuẩn hóa và so sánh email | Cần thống nhất với API cấp tài khoản và database |
| Thư viện băm mật khẩu, thuật toán JWT và cấu hình tương ứng | Chốt khi triển khai |
| Giới hạn tần suất và dung lượng request | Chốt khi cấu hình middleware |

Khi các chính sách được xác nhận, cập nhật đồng bộ tài liệu quy tắc nghiệp vụ và OpenAPI. Không âm thầm triển khai khác với hợp đồng đã công bố.
