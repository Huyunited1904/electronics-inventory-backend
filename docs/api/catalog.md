# Thiết kế API danh mục và sản phẩm

## 1. Mục đích

Tài liệu quy định giao tiếp API cho module quản lý danh mục và sản phẩm của hệ thống quản lý bán hàng và tồn kho đồ điện tử.

Tài liệu là căn cứ để:

- Triển khai API và xử lý nghiệp vụ.
- Kiểm tra quyền truy cập và tính hợp lệ của dữ liệu.
- Xây dựng kiểm thử cho các trường hợp thành công, thất bại và yêu cầu đồng thời.
- Thống nhất cách sử dụng dữ liệu sản phẩm trong phiếu nhập, đơn bán và quản lý tồn kho.

Đường dẫn gốc:

```text
/api
```

Các nhóm tài nguyên:

```text
/api/categories
/api/products
```

## 2. Tài liệu liên quan

| Tài liệu | Nội dung đối chiếu |
|---|---|
| `../scope.md` | Phạm vi chức năng và phân quyền |
| `../business-rules.md` | Quy tắc danh mục, sản phẩm, giá bán và tồn kho |
| `../database.md` | Cấu trúc bảng, kiểu dữ liệu và ràng buộc |
| `../diagrams/erd.dbml` | Quan hệ giữa các bảng |
| `../architecture.md` | Kiến trúc xử lý yêu cầu, xác thực và giao dịch |
| `auth.md` | Cơ chế xác thực JWT |
| `users.md` | Quy ước phản hồi, phân trang và xử lý lỗi |

Nếu phát hiện mâu thuẫn giữa các tài liệu, cần thống nhất và cập nhật các tài liệu liên quan trước khi triển khai phần bị ảnh hưởng.

## 3. Phạm vi

### 3.1. Chức năng được mô tả

**Danh mục:**

- Xem danh sách danh mục.
- Thêm danh mục.
- Sửa tên danh mục.

**Sản phẩm:**

- Xem danh sách sản phẩm.
- Tìm kiếm và lọc sản phẩm.
- Xem chi tiết sản phẩm.
- Thêm sản phẩm.
- Sửa thông tin sản phẩm.
- Ngừng kinh doanh sản phẩm.

### 3.2. Chức năng ngoài phạm vi

- Xóa danh mục hoặc sản phẩm.
- Khôi phục kinh doanh sản phẩm đã ngừng.
- Quản lý ảnh sản phẩm.
- Quản lý biến thể, số serial hoặc IMEI.
- Quản lý nhiều mức giá, khuyến mãi hoặc lịch sử giá.
- Chỉnh sửa trực tiếp số lượng tồn kho.
- Xác nhận phiếu nhập hoặc đơn bán.

API sản phẩm không thay thế API quản lý tồn kho. Số lượng tồn kho chỉ thay đổi thông qua nghiệp vụ nhập và bán hàng đã được xác nhận.

## 4. Xác thực và phân quyền

### 4.1. Xác thực

Tất cả endpoint trong tài liệu yêu cầu JWT access token:

```http
Authorization: Bearer <access_token>
```

Sau khi xác thực token, hệ thống kiểm tra tài khoản hiện tại trong cơ sở dữ liệu:

- Tài khoản phải tồn tại.
- Tài khoản phải đang hoạt động.
- Quyền truy cập được xác định theo vai trò hiện tại trong cơ sở dữ liệu.

Không chỉ dựa vào vai trò được lưu trong token để quyết định quyền truy cập.

### 4.2. Bảng phân quyền

| Chức năng | ADMIN | STAFF |
|---|:---:|:---:|
| Xem danh sách danh mục | ✓ | ✓ |
| Thêm danh mục | ✓ | — |
| Sửa tên danh mục | ✓ | — |
| Xem danh sách và chi tiết sản phẩm | ✓ | ✓ |
| Thêm sản phẩm | ✓ | — |
| Sửa thông tin sản phẩm | ✓ | — |
| Ngừng kinh doanh sản phẩm | ✓ | — |

Sản phẩm đã ngừng kinh doanh vẫn có thể được xem để phục vụ tra cứu.

## 5. Quy ước dữ liệu

### 5.1. Định dạng chung

- Request body và response body sử dụng JSON.
- Request có body phải gửi `Content-Type: application/json`.
- Tên trường trong API sử dụng `camelCase`.
- Thời gian trả về theo ISO 8601, múi giờ UTC.
- ID là số nguyên dương, không vượt quá giới hạn kiểu `INT` có dấu trong cơ sở dữ liệu.
- Không chấp nhận trường body hoặc query parameter nằm ngoài danh sách được mô tả.
- Không chấp nhận query parameter lặp lại.
- Query parameter không bắt buộc có thể được bỏ qua; nếu được gửi với giá trị rỗng thì trả lỗi validation.

Ví dụ thời gian:

```json
{
  "createdAt": "2026-11-22T03:00:00.000Z"
}
```

### 5.2. Quy ước giá bán

Cột `products.sale_price` sử dụng `DECIMAL(15,2)`.

Để tránh mất độ chính xác khi chuyển đổi qua số thực, API nhận và trả giá bán dưới dạng **chuỗi thập phân**.

Ví dụ request:

```json
{
  "salePrice": "15990000.00"
}
```

Quy định:

- Giá bán không âm.
- Có tối đa 13 chữ số ở phần nguyên và 2 chữ số ở phần thập phân.
- Không sử dụng dấu phân cách hàng nghìn.
- Không sử dụng dạng số mũ.
- Không tự làm tròn dữ liệu có hơn 2 chữ số thập phân.
- Không chấp nhận khoảng trắng trong chuỗi giá.
- Response chuẩn hóa thành chuỗi có đúng 2 chữ số thập phân.

Ví dụ:

| Giá trị request | Kết quả |
|---|---|
| `"0"` | Hợp lệ, trả về `"0.00"` |
| `"15000000"` | Hợp lệ, trả về `"15000000.00"` |
| `"15000000.5"` | Hợp lệ, trả về `"15000000.50"` |
| `15000000` | Không hợp lệ: sai kiểu dữ liệu |
| `"-1.00"` | Không hợp lệ |
| `"15000000.555"` | Không hợp lệ |
| `"15,000,000"` | Không hợp lệ |
| `"1.5e7"` | Không hợp lệ |

Giới hạn lớn nhất:

```text
9999999999999.99
```

### 5.3. Cấu trúc phản hồi thành công

Đối với một tài nguyên:

```json
{
  "data": {}
}
```

Đối với danh sách phân trang:

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

### 5.4. Cấu trúc phản hồi lỗi

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Dữ liệu đầu vào không hợp lệ.",
    "details": [
      {
        "field": "salePrice",
        "message": "Giá bán phải là chuỗi thập phân không âm."
      }
    ]
  }
}
```

Trường `details` không bắt buộc.

Response không được chứa thông tin kết nối cơ sở dữ liệu, câu SQL nội bộ hoặc stack trace.

## 6. Quy tắc nghiệp vụ

### 6.1. Danh mục

- Tên danh mục bắt buộc và không được trống sau khi loại bỏ khoảng trắng ở hai đầu.
- Tên danh mục có tối đa 100 ký tự.
- Tên danh mục phải duy nhất theo quy tắc so sánh của cơ sở dữ liệu.
- Sửa tên danh mục không thay đổi ID danh mục.
- Sửa tên danh mục không làm thay đổi liên kết của các sản phẩm đang thuộc danh mục đó.
- Không có thao tác xóa danh mục trong tài liệu này.

### 6.2. Sản phẩm

- Mỗi sản phẩm có một SKU duy nhất.
- SKU vẫn được giữ khi sản phẩm ngừng kinh doanh và không được cấp lại cho sản phẩm khác.
- Mỗi sản phẩm phải thuộc một danh mục tồn tại.
- Sản phẩm mới mặc định có `active = true`.
- Sản phẩm mới phải có đúng một dòng tồn kho với số lượng ban đầu bằng `0`.
- Việc tạo sản phẩm và dòng tồn kho phải thực hiện trong cùng một giao dịch.
- Không cho phép truyền số lượng tồn kho qua API thêm hoặc sửa sản phẩm.
- Giá bán hiện tại được lưu tại `products.sale_price`.
- Ngừng kinh doanh không xóa sản phẩm và không xóa dòng tồn kho.
- Sản phẩm đã ngừng kinh doanh vẫn xuất hiện trong kết quả tra cứu nếu bộ lọc cho phép.

### 6.3. Quan hệ với phiếu nhập và đơn bán

- Các dòng phiếu nhập và đơn bán liên kết sản phẩm bằng `product_id`.
- Thay đổi giá bán sản phẩm không cập nhật `sales_order_items.unit_price` đã lưu.
- Ngừng kinh doanh không tự động sửa trạng thái phiếu nhập, đơn bán hoặc số lượng tồn kho.
- Việc một sản phẩm đã ngừng kinh doanh có được thêm vào hoặc xác nhận trong chứng từ hay không phải tuân theo quy tắc của API nhập hàng và bán hàng.
- API danh mục và sản phẩm không tự thực hiện các thao tác xác nhận chứng từ.

Thiết kế hiện tại lưu giá trên dòng chứng từ nhưng không lưu bản sao tên sản phẩm hoặc SKU. Vì vậy, dữ liệu tra cứu chứng từ thông qua bảng `products` có thể hiển thị tên hoặc SKU hiện tại sau khi chúng được chỉnh sửa.

## 7. Danh sách endpoint

| Method | Endpoint | Chức năng | Quyền |
|---|---|---|---|
| GET | `/api/categories` | Xem danh sách danh mục | ADMIN, STAFF |
| POST | `/api/categories` | Thêm danh mục | ADMIN |
| PATCH | `/api/categories/{id}` | Sửa tên danh mục | ADMIN |
| GET | `/api/products` | Xem, tìm kiếm và lọc sản phẩm | ADMIN, STAFF |
| GET | `/api/products/{id}` | Xem chi tiết sản phẩm | ADMIN, STAFF |
| POST | `/api/products` | Thêm sản phẩm | ADMIN |
| PATCH | `/api/products/{id}` | Sửa thông tin sản phẩm | ADMIN |
| PATCH | `/api/products/{id}/status` | Ngừng kinh doanh sản phẩm | ADMIN |

## 8. API danh mục

### 8.1. Xem danh sách danh mục

```http
GET /api/categories
```

**Quyền:** ADMIN, STAFF.

#### Query parameters

| Trường | Kiểu | Bắt buộc | Mặc định | Quy định |
|---|---|:---:|---|---|
| `page` | Integer | Không | `1` | Từ `1` trở lên |
| `pageSize` | Integer | Không | `20` | Từ `1` đến `100` |

Thứ tự mặc định: `id ASC`.

#### Request mẫu

```http
GET /api/categories?page=1&pageSize=20
Authorization: Bearer <access_token>
```

#### Response — `200 OK`

```json
{
  "data": [
    {
      "id": 1,
      "name": "Laptop"
    },
    {
      "id": 2,
      "name": "Điện thoại"
    }
  ],
  "pagination": {
    "page": 1,
    "pageSize": 20,
    "totalItems": 2,
    "totalPages": 1
  }
}
```

Nếu không có danh mục hoặc trang yêu cầu vượt quá số trang hiện có, trả `200 OK` với `data: []`.

### 8.2. Thêm danh mục

```http
POST /api/categories
```

**Quyền:** ADMIN.

#### Request body

| Trường | Kiểu | Bắt buộc | Quy định |
|---|---|:---:|---|
| `name` | String | Có | Loại bỏ khoảng trắng hai đầu; từ 1 đến 100 ký tự |

```json
{
  "name": "Laptop"
}
```

Không chấp nhận các trường khác, bao gồm `id`.

#### Response — `201 Created`

Header:

```http
Location: /api/categories/1
```

Header xác định URI tài nguyên vừa tạo; tài liệu hiện tại không cung cấp endpoint GET chi tiết danh mục.

```json
{
  "data": {
    "id": 1,
    "name": "Laptop"
  }
}
```

#### Lỗi nghiệp vụ

| HTTP status | Mã lỗi | Trường hợp |
|---|---|---|
| 400 | `VALIDATION_ERROR` | Thiếu tên, sai kiểu, tên rỗng hoặc quá dài |
| 409 | `CATEGORY_NAME_ALREADY_EXISTS` | Tên danh mục đã tồn tại |

Ràng buộc UNIQUE trong cơ sở dữ liệu phải bảo vệ tính duy nhất kể cả khi nhiều yêu cầu tạo danh mục diễn ra đồng thời.

### 8.3. Sửa tên danh mục

```http
PATCH /api/categories/{id}
```

**Quyền:** ADMIN.

#### Request body

```json
{
  "name": "Máy tính xách tay"
}
```

Áp dụng validation giống API thêm danh mục.

Chỉ chấp nhận trường `name`. Body rỗng không hợp lệ.

#### Response — `200 OK`

```json
{
  "data": {
    "id": 1,
    "name": "Máy tính xách tay"
  }
}
```

#### Hành vi

- Không thay đổi ID danh mục.
- Không thay đổi `products.category_id` của các sản phẩm liên quan.
- Gửi lại đúng tên hiện tại được xem là thành công.
- Kiểm tra tên trùng phải loại trừ chính danh mục đang sửa.

#### Lỗi nghiệp vụ

| HTTP status | Mã lỗi | Trường hợp |
|---|---|---|
| 400 | `VALIDATION_ERROR` | ID hoặc body không hợp lệ |
| 404 | `CATEGORY_NOT_FOUND` | Danh mục không tồn tại |
| 409 | `CATEGORY_NAME_ALREADY_EXISTS` | Tên đã được danh mục khác sử dụng |

## 9. API sản phẩm

### 9.1. Đối tượng sản phẩm trả về

```json
{
  "id": 101,
  "sku": "LAPTOP-001",
  "name": "Laptop mẫu A",
  "categoryId": 1,
  "salePrice": "15990000.00",
  "active": true,
  "createdAt": "2026-11-22T03:00:00.000Z",
  "updatedAt": "2026-11-22T03:00:00.000Z"
}
```

Đối tượng sản phẩm không bao gồm số lượng tồn kho. Thông tin tồn kho được truy vấn qua API quy định trong `inventory.md`.

### 9.2. Xem danh sách, tìm kiếm và lọc sản phẩm

```http
GET /api/products
```

**Quyền:** ADMIN, STAFF.

#### Query parameters

| Trường | Kiểu | Bắt buộc | Mặc định | Quy định |
|---|---|:---:|---|---|
| `page` | Integer | Không | `1` | Từ `1` trở lên |
| `pageSize` | Integer | Không | `20` | Từ `1` đến `100` |
| `q` | String | Không | Không lọc | Từ 1 đến 255 ký tự sau khi loại bỏ khoảng trắng hai đầu |
| `categoryId` | Integer | Không | Không lọc | ID nguyên dương hợp lệ |
| `active` | Boolean dạng query | Không | Không lọc | Chỉ nhận `true` hoặc `false` |

Quy định:

- `q` tìm theo tên sản phẩm hoặc SKU.
- Tìm kiếm theo chuỗi con.
- Ký tự `%` và `_` trong `q` được xem là ký tự tìm kiếm thông thường, không phải ký tự đại diện SQL.
- Các nhóm điều kiện lọc kết hợp bằng AND.
- Trong điều kiện tìm kiếm, tên hoặc SKU khớp là đủ.
- Thứ tự mặc định: `id DESC`.
- Khi bỏ qua `active`, kết quả bao gồm cả sản phẩm đang và đã ngừng kinh doanh.
- Nếu `categoryId` hợp lệ về định dạng nhưng không tồn tại, trả danh sách rỗng.

#### Request mẫu

```http
GET /api/products?page=1&pageSize=20&categoryId=1&active=true&q=Laptop
Authorization: Bearer <access_token>
```

#### Response — `200 OK`

```json
{
  "data": [
    {
      "id": 101,
      "sku": "LAPTOP-001",
      "name": "Laptop mẫu A",
      "categoryId": 1,
      "salePrice": "15990000.00",
      "active": true,
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

`totalItems` được tính sau khi áp dụng toàn bộ điều kiện lọc.

Nếu không có kết quả:

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

Trang vượt phạm vi vẫn trả `200 OK` với danh sách rỗng và thông tin phân trang tương ứng.

### 9.3. Xem chi tiết sản phẩm

```http
GET /api/products/{id}
```

**Quyền:** ADMIN, STAFF.

#### Response — `200 OK`

Trả đối tượng sản phẩm theo cấu trúc tại mục 9.1 trong trường `data`.

Sản phẩm có `active = false` vẫn trả `200 OK` nếu tồn tại.

#### Lỗi nghiệp vụ

| HTTP status | Mã lỗi | Trường hợp |
|---|---|---|
| 400 | `VALIDATION_ERROR` | ID không hợp lệ |
| 404 | `PRODUCT_NOT_FOUND` | Sản phẩm không tồn tại |

### 9.4. Thêm sản phẩm

```http
POST /api/products
```

**Quyền:** ADMIN.

#### Request body

| Trường | Kiểu | Bắt buộc | Quy định |
|---|---|:---:|---|
| `sku` | String | Có | Từ 1 đến 50 ký tự sau khi loại bỏ khoảng trắng hai đầu; duy nhất |
| `name` | String | Có | Từ 1 đến 255 ký tự sau khi loại bỏ khoảng trắng hai đầu |
| `categoryId` | Integer | Có | ID của danh mục tồn tại |
| `salePrice` | String | Có | Chuỗi thập phân theo mục 5.2 |

```json
{
  "sku": "LAPTOP-001",
  "name": "Laptop mẫu A",
  "categoryId": 1,
  "salePrice": "15990000.00"
}
```

Không chấp nhận:

- `id`
- `active`
- `quantity`
- `createdAt`
- `updatedAt`
- Bất kỳ trường nào khác ngoài bốn trường được mô tả.

#### Xử lý nghiệp vụ

Trong cùng một giao dịch:

1. Tạo bản ghi `products` với `active = true`.
2. Tạo bản ghi `inventory` có `product_id` tương ứng và `quantity = 0`.
3. Commit khi cả hai thao tác thành công.

Nếu bất kỳ thao tác nào thất bại, rollback toàn bộ.

Không được để lại:

- Sản phẩm không có dòng tồn kho.
- Dòng tồn kho không có sản phẩm tương ứng.
- Nhiều dòng tồn kho cho cùng một sản phẩm.

#### Response — `201 Created`

Header:

```http
Location: /api/products/101
```

Body trả đối tượng sản phẩm vừa tạo trong trường `data`.

#### Lỗi nghiệp vụ

| HTTP status | Mã lỗi | Trường hợp |
|---|---|---|
| 400 | `VALIDATION_ERROR` | Thiếu trường, sai kiểu hoặc giá trị không hợp lệ |
| 404 | `CATEGORY_NOT_FOUND` | Danh mục được tham chiếu không tồn tại |
| 409 | `PRODUCT_SKU_ALREADY_EXISTS` | SKU đã được sử dụng |

SKU của sản phẩm đã ngừng kinh doanh vẫn được xem là đã sử dụng.

### 9.5. Sửa thông tin sản phẩm

```http
PATCH /api/products/{id}
```

**Quyền:** ADMIN.

#### Request body

Cho phép gửi một hoặc nhiều trường:

| Trường | Kiểu | Quy định |
|---|---|---|
| `sku` | String | Áp dụng validation như khi thêm |
| `name` | String | Áp dụng validation như khi thêm |
| `categoryId` | Integer | Danh mục phải tồn tại |
| `salePrice` | String | Áp dụng quy ước tại mục 5.2 |

Ví dụ:

```json
{
  "name": "Laptop mẫu A phiên bản cập nhật",
  "salePrice": "15490000.00"
}
```

Quy định:

- Phải có ít nhất một trường hợp lệ.
- Trường không gửi được giữ nguyên.
- Không chấp nhận `null` cho các trường được phép sửa.
- Không chấp nhận `active`, `quantity`, `id` hoặc các trường thời gian.
- Không chấp nhận trường ngoài danh sách cho phép.

#### Hành vi

- Có thể sửa thông tin của sản phẩm đang hoặc đã ngừng kinh doanh.
- Sửa sản phẩm đã ngừng kinh doanh không làm sản phẩm hoạt động trở lại.
- Không thay đổi ID sản phẩm.
- Không thay đổi số lượng tồn kho.
- Không thay đổi giá đã lưu trong các dòng đơn bán.
- Kiểm tra SKU trùng phải loại trừ chính sản phẩm đang sửa.
- Các trường trong cùng một request phải được cập nhật nguyên tử: thành công toàn bộ hoặc không cập nhật trường nào.
- `updatedAt` thay đổi khi dữ liệu thực tế thay đổi.
- Nếu toàn bộ giá trị sau chuẩn hóa giống dữ liệu hiện tại, trả `200 OK` và giữ nguyên `updatedAt`.

#### Response — `200 OK`

Trả đối tượng sản phẩm sau cập nhật trong trường `data`.

#### Lỗi nghiệp vụ

| HTTP status | Mã lỗi | Trường hợp |
|---|---|---|
| 400 | `VALIDATION_ERROR` | ID hoặc body không hợp lệ |
| 404 | `PRODUCT_NOT_FOUND` | Sản phẩm cần sửa không tồn tại |
| 404 | `CATEGORY_NOT_FOUND` | Danh mục mới không tồn tại |
| 409 | `PRODUCT_SKU_ALREADY_EXISTS` | SKU thuộc về sản phẩm khác |

Nếu sản phẩm đích không tồn tại, trả `PRODUCT_NOT_FOUND` trước khi kiểm tra sự tồn tại của danh mục được gửi trong body.

### 9.6. Ngừng kinh doanh sản phẩm

```http
PATCH /api/products/{id}/status
```

**Quyền:** ADMIN.

#### Request body

```json
{
  "active": false
}
```

Chỉ chấp nhận trường `active` với giá trị boolean `false`.

Các giá trị sau không hợp lệ:

```json
{
  "active": "false"
}
```

```json
{
  "active": true
}
```

API hiện tại không hỗ trợ khôi phục kinh doanh.

#### Hành vi

- Chuyển `products.active` thành `false`.
- Không xóa sản phẩm.
- Không xóa hoặc thay đổi số lượng tồn kho.
- Không sửa các dòng chứng từ liên quan.
- Không tự hủy phiếu nhập hoặc đơn bán đang ở trạng thái nháp.
- Cho phép ngừng kinh doanh khi tồn kho vẫn lớn hơn `0`.
- Nếu sản phẩm đã ngừng kinh doanh, trả `200 OK` và giữ nguyên `updatedAt`.

#### Response — `200 OK`

```json
{
  "data": {
    "id": 101,
    "sku": "LAPTOP-001",
    "name": "Laptop mẫu A",
    "categoryId": 1,
    "salePrice": "15990000.00",
    "active": false,
    "createdAt": "2026-11-22T03:00:00.000Z",
    "updatedAt": "2026-11-22T04:00:00.000Z"
  }
}
```

#### Lỗi nghiệp vụ

| HTTP status | Mã lỗi | Trường hợp |
|---|---|---|
| 400 | `VALIDATION_ERROR` | ID, body hoặc giá trị `active` không hợp lệ |
| 404 | `PRODUCT_NOT_FOUND` | Sản phẩm không tồn tại |

## 10. Danh sách lỗi chung

| HTTP status | Mã lỗi | Ý nghĩa |
|---|---|---|
| 400 | `INVALID_JSON` | Body không phải JSON hợp lệ |
| 400 | `VALIDATION_ERROR` | Path, query hoặc body không hợp lệ |
| 401 | `AUTH_REQUIRED` | Không cung cấp thông tin xác thực |
| 401 | `INVALID_TOKEN` | Token không hợp lệ hoặc không xác định được tài khoản hợp lệ |
| 401 | `TOKEN_EXPIRED` | Token hết hạn |
| 403 | `ACCOUNT_DISABLED` | Tài khoản đã bị khóa |
| 403 | `FORBIDDEN` | Vai trò không được phép thực hiện thao tác |
| 404 | `CATEGORY_NOT_FOUND` | Không tìm thấy danh mục được yêu cầu |
| 404 | `PRODUCT_NOT_FOUND` | Không tìm thấy sản phẩm được yêu cầu |
| 409 | `CATEGORY_NAME_ALREADY_EXISTS` | Tên danh mục bị trùng |
| 409 | `PRODUCT_SKU_ALREADY_EXISTS` | SKU bị trùng |
| 413 | `PAYLOAD_TOO_LARGE` | Request body vượt giới hạn cấu hình |
| 415 | `UNSUPPORTED_MEDIA_TYPE` | Request có body nhưng không sử dụng loại nội dung được hỗ trợ |
| 500 | `INTERNAL_SERVER_ERROR` | Lỗi nội bộ chưa được xử lý thành lỗi nghiệp vụ |

Không chuyển mọi lỗi cơ sở dữ liệu thành lỗi `409`. Chỉ những vi phạm ràng buộc đã được nhận diện chính xác mới được ánh xạ sang mã lỗi nghiệp vụ tương ứng.

Kiểm tra quyền phải được thực hiện trước khi tra cứu tài nguyên nghiệp vụ. STAFF gọi API quản trị nhận `403 FORBIDDEN`, không phụ thuộc ID sản phẩm hoặc danh mục có tồn tại hay không.

## 11. Ánh xạ dữ liệu

### 11.1. Danh mục

| Trường API | Cột cơ sở dữ liệu |
|---|---|
| `id` | `categories.id` |
| `name` | `categories.name` |

### 11.2. Sản phẩm

| Trường API | Cột cơ sở dữ liệu |
|---|---|
| `id` | `products.id` |
| `sku` | `products.sku` |
| `name` | `products.name` |
| `categoryId` | `products.category_id` |
| `salePrice` | `products.sale_price` |
| `active` | `products.active` |
| `createdAt` | `products.created_at` |
| `updatedAt` | `products.updated_at` |

### 11.3. Tồn kho khi tạo sản phẩm

| Cột cơ sở dữ liệu | Giá trị |
|---|---|
| `inventory.product_id` | ID sản phẩm vừa tạo |
| `inventory.quantity` | `0` |
| `inventory.updated_at` | Thời điểm khởi tạo |

## 12. Tính nhất quán và yêu cầu đồng thời

### 12.1. Bảo vệ tính duy nhất

Các ràng buộc UNIQUE trong cơ sở dữ liệu là lớp bảo vệ cuối cùng cho:

- `categories.name`
- `products.sku`

Kiểm tra trước bằng truy vấn không thay thế được ràng buộc UNIQUE.

Nếu hai yêu cầu cùng tạo một SKU chưa tồn tại:

- Một yêu cầu có thể tạo thành công.
- Yêu cầu còn lại phải nhận `409 PRODUCT_SKU_ALREADY_EXISTS`.
- Không được tạo hai sản phẩm trùng SKU hoặc để lại dữ liệu tồn kho không nhất quán.

### 12.2. Giao dịch tạo sản phẩm

Service quản lý giao dịch tạo sản phẩm và tồn kho.

Các repository tham gia giao dịch phải dùng cùng một database connection.

Chỉ gửi response thành công sau khi commit thành công.

### 12.3. Cập nhật đồng thời

Trong phiên bản này:

- Chưa sử dụng trường phiên bản hoặc `If-Match` để phát hiện chỉnh sửa dựa trên dữ liệu cũ.
- Nếu hai yêu cầu cùng sửa một trường, giá trị cuối cùng phụ thuộc vào thứ tự cập nhật được cơ sở dữ liệu thực hiện.
- Chỉ cập nhật những trường có trong request, tránh ghi đè các trường không liên quan.
- API sửa thông tin sản phẩm không được ghi lại trường `active`.
- API ngừng kinh doanh không được ghi lại tên, SKU, danh mục hoặc giá bán.

Giới hạn này cần được hiểu khi kiểm thử nhiều ADMIN chỉnh sửa cùng một sản phẩm.

### 12.4. Phân trang

Dữ liệu và tổng số bản ghi trong một response cần được truy vấn với cùng điều kiện lọc và một ảnh chụp dữ liệu nhất quán.

Phân trang giữa nhiều request không bảo đảm ảnh chụp cố định: khi có thêm hoặc sửa dữ liệu giữa các lần gọi, nội dung các trang có thể thay đổi.

## 13. Yêu cầu bảo mật và triển khai

- Sử dụng truy vấn có tham số; không nối trực tiếp dữ liệu người dùng vào SQL.
- Áp dụng danh sách trường cho phép khi tạo và cập nhật.
- Không ánh xạ nguyên request body vào câu lệnh cập nhật cơ sở dữ liệu.
- Kiểm tra validation ở backend.
- Giữ nguyên độ chính xác thập phân khi đọc, ghi và trả giá bán.
- Không ghi JWT hoặc thông tin xác thực vào log.
- Các API đọc không được làm thay đổi dữ liệu.
- Không triển khai cache chỉ dựa trên tài liệu này. Khi bổ sung cache, phải có quy tắc hết hạn hoặc vô hiệu hóa cache sau khi thay đổi danh mục, giá bán hoặc trạng thái sản phẩm.
- Kiểm tra nghiệp vụ quan trọng khi nhập/bán hàng không được dựa hoàn toàn vào dữ liệu sản phẩm có thể đã cũ trong cache.

## 14. Các tình huống kiểm thử

### 14.1. Xác thực và phân quyền

- Không có token.
- Token sai hoặc hết hạn.
- Token thuộc tài khoản đã bị khóa.
- STAFF xem danh mục và sản phẩm thành công.
- STAFF gọi API thêm, sửa hoặc ngừng kinh doanh nhận `403`.
- Yêu cầu không có quyền không làm thay đổi dữ liệu.

### 14.2. Danh mục

- Tạo danh mục hợp lệ.
- Tên thiếu, sai kiểu, rỗng hoặc chỉ có khoảng trắng.
- Tên đúng giới hạn và vượt giới hạn 100 ký tự.
- Tên trùng sau khi chuẩn hóa khoảng trắng hai đầu.
- Sửa danh mục không tồn tại.
- Sửa thành tên của danh mục khác.
- Gửi lại tên hiện tại.
- Hai yêu cầu đồng thời tạo hoặc đổi sang cùng một tên.
- Sửa tên không làm mất liên kết với sản phẩm.

### 14.3. Tạo sản phẩm

- Tạo sản phẩm và dòng tồn kho bằng `0` thành công.
- Thiếu một trong các trường bắt buộc.
- SKU hoặc tên rỗng sau khi chuẩn hóa.
- SKU hoặc tên vượt giới hạn.
- SKU trùng sản phẩm đang kinh doanh.
- SKU trùng sản phẩm đã ngừng kinh doanh.
- Danh mục không tồn tại.
- Giá bằng `0`, giá lớn nhất hợp lệ.
- Giá âm, vượt giới hạn, sai kiểu hoặc quá 2 chữ số thập phân.
- Gửi trường không được phép như `quantity` hoặc `active`.
- Tạo dòng tồn kho thất bại phải rollback sản phẩm.
- Hai yêu cầu đồng thời tạo cùng SKU chỉ tạo được một sản phẩm.

### 14.4. Tra cứu sản phẩm

- Danh sách chưa có dữ liệu.
- Phân trang hợp lệ và trang vượt phạm vi.
- `page`, `pageSize` không hợp lệ.
- Lọc theo danh mục và trạng thái.
- Kết hợp tìm kiếm với nhiều điều kiện lọc.
- Tìm kiếm chứa `%`, `_`, dấu nháy hoặc ký tự Unicode.
- Query parameter lặp lại hoặc không được hỗ trợ.
- Xem chi tiết sản phẩm không tồn tại.
- Xem chi tiết sản phẩm đã ngừng kinh doanh.

### 14.5. Sửa sản phẩm

- Sửa một trường và nhiều trường.
- Body rỗng hoặc chứa trường không được phép.
- Giá trị `null` hoặc sai kiểu.
- Đổi SKU thành SKU đã được sử dụng.
- Đổi sang danh mục không tồn tại.
- Một trường không hợp lệ khiến toàn bộ cập nhật bị từ chối.
- Gửi lại dữ liệu hiện tại không thay đổi `updatedAt`.
- Sửa giá không thay đổi giá trong dòng đơn bán.
- Sửa thông tin không thay đổi tồn kho hoặc trạng thái kinh doanh.
- Sửa thông tin đồng thời với ngừng kinh doanh không làm sản phẩm hoạt động trở lại.

### 14.6. Ngừng kinh doanh

- Ngừng sản phẩm đang kinh doanh.
- Ngừng sản phẩm vẫn còn tồn kho.
- Gửi lại yêu cầu cho sản phẩm đã ngừng.
- ID không tồn tại.
- Gửi `true`, `"false"`, `null` hoặc thiếu `active`.
- Không thay đổi tồn kho và các chứng từ liên quan.

## 15. Các điểm cần đối chiếu trước khi triển khai

Các lựa chọn dưới đây được mô tả trong bản thiết kế này nhưng cần đối chiếu với `scope.md`, `business-rules.md` và các API đã commit:

| Nội dung | Lựa chọn trong tài liệu |
|---|---|
| Quản lý danh mục | ADMIN được thêm và sửa tên; không xóa |
| Sửa SKU | ADMIN được sửa, vẫn phải bảo đảm duy nhất |
| Sản phẩm đã ngừng | Được xem và sửa thông tin; chưa hỗ trợ khôi phục kinh doanh |
| Ngừng khi còn tồn | Cho phép; không thay đổi tồn kho |
| Giá bán qua API | Chuỗi thập phân; response có 2 chữ số thập phân |
| Danh sách mặc định | Bao gồm cả sản phẩm đang và đã ngừng kinh doanh |
| Tìm kiếm | Theo chuỗi con của tên hoặc SKU |
| Phân trang | `page = 1`, `pageSize = 20`, tối đa `100` |

Các nội dung cần xác định thêm:

1. **Collation cụ thể của cơ sở dữ liệu:** quyết định cách phân biệt chữ hoa/thường và dấu khi kiểm tra tên danh mục, SKU và tìm kiếm. Không được giả định quy tắc so sánh khi chưa chốt collation.
2. **Sản phẩm ngừng kinh doanh trong chứng từ:** quy định cụ thể tại `purchasing.md` và `sales.md`, bao gồm chứng từ nháp đã chứa sản phẩm trước khi ngừng kinh doanh.
3. **Quy ước lỗi dùng chung:** đối chiếu tên mã lỗi và cấu trúc `details` với `auth.md`, `users.md` trước khi triển khai.
4. **Giới hạn request body:** cấu hình tập trung và ghi tại tài liệu kiến trúc.
5. **Cache:** thiết kế riêng cơ chế vô hiệu hóa và cách xử lý dữ liệu cũ khi triển khai Redis.

Những điểm chưa thống nhất phải được cập nhật trước khi viết code phụ thuộc vào chúng.
