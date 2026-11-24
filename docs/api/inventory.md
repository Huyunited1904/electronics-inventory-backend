# Thiết kế API tra cứu tồn kho

## 1. Mục đích

Tài liệu mô tả API tra cứu tồn kho và lịch sử nhập/xuất của hệ thống quản lý bán hàng và tồn kho đồ điện tử.

Tài liệu là căn cứ để:

- Tra cứu số lượng tồn hiện tại của sản phẩm.
- Tìm kiếm và lọc danh sách tồn kho.
- Tra cứu lịch sử nhập/xuất từ các chứng từ đã xác nhận.
- Thống nhất quyền truy cập, dữ liệu phản hồi và mã lỗi.
- Xây dựng kiểm thử về tính chính xác và nhất quán của dữ liệu.

Đường dẫn gốc:

```text
/api/inventory
```

## 2. Tài liệu liên quan

| Tài liệu | Nội dung đối chiếu |
|---|---|
| `../scope.md` | Phạm vi chức năng và phân quyền |
| `../business-rules.md` | Quy tắc tồn kho, nhập hàng và bán hàng |
| `../database.md` | Cấu trúc bảng và ràng buộc |
| `../diagrams/erd.dbml` | Quan hệ sản phẩm, tồn kho và chứng từ |
| `../architecture.md` | Kiến trúc truy vấn và quy ước chung |
| `auth.md` | Xác thực JWT |
| `catalog.md` | Thông tin sản phẩm và trạng thái kinh doanh |
| `purchasing.md` | Xác nhận phiếu nhập và tăng tồn kho |
| `sales.md` | Xác nhận đơn bán và giảm tồn kho |

Nếu có mâu thuẫn, cần thống nhất và cập nhật các tài liệu liên quan trước khi triển khai.

## 3. Phạm vi

### 3.1. Chức năng được hỗ trợ

- Xem danh sách tồn kho.
- Tìm sản phẩm theo tên hoặc SKU.
- Lọc tồn kho theo danh mục và trạng thái kinh doanh.
- Lọc sản phẩm còn hàng hoặc hết hàng.
- Xem tồn kho của một sản phẩm.
- Xem lịch sử nhập/xuất của một sản phẩm.
- Lọc lịch sử theo loại nghiệp vụ và thời điểm xác nhận.

### 3.2. Chức năng ngoài phạm vi

- Chỉnh sửa trực tiếp số lượng tồn kho.
- Tạo hoặc xóa dòng tồn kho qua API.
- Điều chỉnh tồn do kiểm kê, hỏng hóc hoặc mất hàng.
- Chuyển hàng giữa các kho.
- Giữ hàng cho đơn nháp.
- Quản lý mức tồn tối thiểu và cảnh báo sắp hết hàng.
- Tính giá vốn, giá trị tồn kho hoặc lợi nhuận.
- Tính tồn đầu kỳ, cuối kỳ hoặc số dư sau từng giao dịch.
- Xuất báo cáo Excel hoặc PDF.

Thiết kế hiện tại chỉ có một kho.

## 4. Nguồn dữ liệu

### 4.1. Tồn kho hiện tại

Tồn kho hiện tại được đọc từ bảng `inventory`.

Mỗi sản phẩm có đúng một dòng:

```text
inventory.product_id → products.id
```

Trong đó:

- `product_id` đồng thời là khóa chính và khóa ngoại.
- `quantity` là số lượng tồn hiện tại.
- `updated_at` là thời điểm cập nhật dòng tồn kho.

Dòng tồn được khởi tạo cùng sản phẩm với số lượng bằng `0`.

### 4.2. Lịch sử nhập/xuất

Thiết kế hiện tại không sử dụng bảng `stock_movements`.

Lịch sử được tổng hợp từ:

| Loại | Bảng chứng từ | Bảng chi tiết | Điều kiện |
|---|---|---|---|
| Nhập kho | `stock_receipts` | `stock_receipt_items` | Phiếu có trạng thái `CONFIRMED` |
| Xuất bán | `sales_orders` | `sales_order_items` | Đơn có trạng thái `CONFIRMED` |

Mỗi dòng chứng từ đã xác nhận tạo thành một mục lịch sử của sản phẩm tương ứng.

Chứng từ `DRAFT` hoặc `CANCELLED` không xuất hiện trong lịch sử nhập/xuất.

## 5. Xác thực và phân quyền

### 5.1. Xác thực

Tất cả endpoint yêu cầu JWT access token:

```http
Authorization: Bearer <access_token>
```

Backend phải:

- Xác thực token.
- Kiểm tra tài khoản tồn tại và đang hoạt động.
- Sử dụng vai trò hiện tại trong cơ sở dữ liệu.

### 5.2. Phân quyền

| Chức năng | ADMIN | STAFF |
|---|:---:|:---:|
| Xem danh sách tồn kho | ✓ | ✓ |
| Xem tồn kho một sản phẩm | ✓ | ✓ |
| Xem lịch sử nhập/xuất | ✓ | ✓ |

Lịch sử không bị giới hạn theo người tạo chứng từ vì tồn kho là dữ liệu chung của cửa hàng.

Không có quyền chỉnh sửa tồn trực tiếp cho bất kỳ vai trò nào qua nhóm API này.

## 6. Quy tắc nghiệp vụ

### 6.1. Số lượng tồn kho

- Tồn kho không được âm.
- Tạo sản phẩm mới khởi tạo tồn bằng `0`.
- Chỉ xác nhận phiếu nhập mới làm tăng tồn.
- Chỉ xác nhận đơn bán mới làm giảm tồn.
- Thao tác với chứng từ nháp không thay đổi tồn.
- Hủy chứng từ nháp không thay đổi tồn.

Trong phạm vi hiện tại:

```text
Tồn hiện tại
= Tổng số lượng nhập đã xác nhận
- Tổng số lượng bán đã xác nhận
```

Công thức này dựa trên điều kiện tồn ban đầu bằng `0` và không có nghiệp vụ điều chỉnh tồn khác.

API tra cứu tồn hiện tại vẫn đọc bảng `inventory`; không tính lại toàn bộ lịch sử cho mỗi request.

### 6.2. Không giữ hàng

Đơn bán nháp không giữ hàng.

Vì vậy:

- Không có trường số lượng đã giữ.
- Không có phép tính trừ số lượng trong đơn nháp khỏi tồn.
- Số tồn được trả về không bảo đảm một đơn bán sẽ xác nhận thành công sau đó.
- API xác nhận bán phải kiểm tra lại dưới khóa trong giao dịch.

### 6.3. Trạng thái hàng

Trạng thái tồn được suy ra từ số lượng:

| Giá trị API | Điều kiện |
|---|---|
| `IN_STOCK` | `quantity > 0` |
| `OUT_OF_STOCK` | `quantity = 0` |

Không lưu thêm cột trạng thái tồn.

Không có trạng thái `LOW_STOCK` vì thiết kế chưa có ngưỡng tồn tối thiểu.

### 6.4. Sản phẩm ngừng kinh doanh

- Sản phẩm đã ngừng kinh doanh vẫn xuất hiện nếu bộ lọc cho phép.
- Ngừng kinh doanh không đưa tồn về `0`.
- Sản phẩm ngừng kinh doanh vẫn có thể còn hàng.
- Lịch sử đã xác nhận được giữ nguyên.
- Không tự xóa dòng tồn hoặc lịch sử.

### 6.5. Thiếu hoặc sai dữ liệu tồn kho

Sản phẩm tồn tại nhưng không có dòng `inventory` là lỗi toàn vẹn dữ liệu.

Không được:

- Xem trường hợp này như tồn bằng `0`.
- Tự tạo dòng tồn trong API GET.
- Tự sửa dữ liệu khi tra cứu.

Nếu phát hiện thiếu dòng tồn hoặc số lượng âm trong dữ liệu đang xử lý:

- Trả lỗi nội bộ.
- Ghi log để điều tra.
- Không trả phản hồi thành công với số liệu đã tự điều chỉnh.

### 6.6. Thời điểm lịch sử

Lịch sử sử dụng `confirmed_at` của chứng từ.

Không sử dụng `created_at` làm thời điểm nhập/xuất vì chứng từ có thể được tạo nháp từ trước.

Người thực hiện nhập/xuất là `confirmed_by`, không mặc định là người tạo.

### 6.7. Thông tin sản phẩm

Tên, SKU, danh mục và trạng thái kinh doanh là thông tin hiện tại từ bảng `products`.

Thiết kế chưa lưu bản sao lịch sử của tên hoặc SKU.

Thay đổi tên hoặc SKU không làm thay đổi số lượng nhập/xuất đã ghi nhận.

## 7. Quy ước dữ liệu và phản hồi

### 7.1. Quy ước chung

- Tất cả endpoint dùng phương thức GET và không nhận body.
- Tên trường API sử dụng `camelCase`.
- ID là số nguyên từ `1` đến `2147483647`.
- Thời gian trả về theo ISO 8601, múi giờ UTC.
- Không nhận query parameter ngoài danh sách được mô tả.
- Không nhận query parameter lặp lại.
- Query parameter được gửi nhưng rỗng là không hợp lệ.
- Query số chỉ nhận dạng chữ số thập phân; không nhận số mũ hoặc số thập phân.

### 7.2. Phân trang

| Trường | Mặc định | Giới hạn |
|---|---|---|
| `page` | `1` | Từ `1` đến `2147483647` |
| `pageSize` | `20` | Từ `1` đến `100` |

Công thức:

```text
totalPages = ceil(totalItems / pageSize)
```

- Không có kết quả: `totalItems = 0`, `totalPages = 0`.
- Trang vượt phạm vi: trả `200 OK` và danh sách rỗng.
- Giữ nguyên số trang được yêu cầu trong response.

### 7.3. Phản hồi một tài nguyên

```json
{
  "data": {}
}
```

### 7.4. Phản hồi danh sách

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

### 7.5. Phản hồi lỗi

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Dữ liệu đầu vào không hợp lệ.",
    "details": [
      {
        "field": "stockStatus",
        "message": "Chỉ chấp nhận IN_STOCK hoặc OUT_OF_STOCK."
      }
    ]
  }
}
```

Trường `details` không bắt buộc.

## 8. Danh sách endpoint

| Method | Endpoint | Chức năng |
|---|---|---|
| GET | `/api/inventory` | Xem danh sách, tìm kiếm và lọc tồn kho |
| GET | `/api/inventory/{productId}` | Xem tồn kho một sản phẩm |
| GET | `/api/inventory/{productId}/history` | Xem lịch sử nhập/xuất của sản phẩm |

`productId` là ID sản phẩm, không phải ID tồn kho riêng.

Không cung cấp POST, PATCH, PUT hoặc DELETE để thay đổi tồn kho trực tiếp.

## 9. Xem danh sách tồn kho

```http
GET /api/inventory
```

**Quyền:** ADMIN, STAFF.

### 9.1. Query parameters

| Trường | Kiểu | Mặc định | Quy định |
|---|---|---|---|
| `page` | Integer | `1` | Theo mục 7.2 |
| `pageSize` | Integer | `20` | Theo mục 7.2 |
| `q` | String | Không tìm kiếm | Từ 1 đến 255 Unicode code point sau khi bỏ khoảng trắng hai đầu |
| `categoryId` | Integer | Không lọc | ID hợp lệ |
| `active` | Boolean dạng query | Không lọc | Chỉ nhận `true` hoặc `false` |
| `stockStatus` | String | Không lọc | `IN_STOCK` hoặc `OUT_OF_STOCK` |

### 9.2. Tìm kiếm và lọc

- `q` tìm theo chuỗi con của tên hoặc SKU.
- Chỉ cần tên hoặc SKU khớp.
- `%` và `_` trong từ khóa là ký tự thông thường, không phải wildcard SQL.
- Phân biệt chữ hoa/thường và dấu theo collation thống nhất của database.
- Các nhóm điều kiện kết hợp bằng AND.
- Bỏ qua `active` để lấy cả sản phẩm đang và đã ngừng kinh doanh.
- Bỏ qua `stockStatus` để lấy cả còn hàng và hết hàng.
- `categoryId` hợp lệ nhưng không tồn tại trả danh sách rỗng.
- Thứ tự mặc định: `productId DESC`.
- Chưa hỗ trợ tùy chọn sắp xếp.

### 9.3. Request mẫu

```http
GET /api/inventory?page=1&pageSize=20&categoryId=1&active=true&stockStatus=IN_STOCK
Authorization: Bearer <access_token>
```

### 9.4. Response — `200 OK`

```json
{
  "data": [
    {
      "productId": 101,
      "sku": "LAPTOP-001",
      "name": "Laptop mẫu A",
      "categoryId": 1,
      "active": true,
      "quantity": 8,
      "stockStatus": "IN_STOCK",
      "updatedAt": "2026-11-24T03:00:00.000Z"
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

`updatedAt` là thời điểm cập nhật dòng tồn kho, không phải thời điểm cập nhật thông tin sản phẩm hoặc thời điểm gửi response.

Sửa tên hoặc giá sản phẩm không tự làm thay đổi `inventory.updated_at`.

### 9.5. Lỗi

Trả `400 VALIDATION_ERROR` khi:

- Phân trang sai định dạng hoặc vượt giới hạn.
- Từ khóa rỗng sau chuẩn hóa hoặc quá dài.
- ID danh mục không hợp lệ.
- `active` hoặc `stockStatus` không được hỗ trợ.
- Query parameter bị lặp hoặc không được khai báo.

Không có kết quả không phải lỗi `404`.

## 10. Xem tồn kho của một sản phẩm

```http
GET /api/inventory/{productId}
```

**Quyền:** ADMIN, STAFF.

### 10.1. Path parameter

| Trường | Kiểu | Quy định |
|---|---|---|
| `productId` | Integer | Từ `1` đến `2147483647` |

Endpoint không nhận query parameter.

### 10.2. Response — `200 OK`

```json
{
  "data": {
    "productId": 101,
    "sku": "LAPTOP-001",
    "name": "Laptop mẫu A",
    "categoryId": 1,
    "active": true,
    "quantity": 8,
    "stockStatus": "IN_STOCK",
    "updatedAt": "2026-11-24T03:00:00.000Z"
  }
}
```

Sản phẩm hết hàng vẫn trả `200 OK`:

```json
{
  "data": {
    "productId": 102,
    "sku": "MOUSE-001",
    "name": "Chuột mẫu B",
    "categoryId": 2,
    "active": true,
    "quantity": 0,
    "stockStatus": "OUT_OF_STOCK",
    "updatedAt": "2026-11-24T04:00:00.000Z"
  }
}
```

Sản phẩm ngừng kinh doanh vẫn được tra cứu nếu tồn tại.

### 10.3. Lỗi

| HTTP status | Mã lỗi | Trường hợp |
|---|---|---|
| 400 | `VALIDATION_ERROR` | Path hoặc query không hợp lệ |
| 404 | `PRODUCT_NOT_FOUND` | Sản phẩm không tồn tại |
| 500 | `INTERNAL_SERVER_ERROR` | Sản phẩm tồn tại nhưng thiếu dòng tồn hoặc tồn không hợp lệ |

Không trả `404` chỉ vì sản phẩm có số lượng bằng `0`.

## 11. Xem lịch sử nhập/xuất

```http
GET /api/inventory/{productId}/history
```

**Quyền:** ADMIN, STAFF.

### 11.1. Query parameters

| Trường | Kiểu | Mặc định | Quy định |
|---|---|---|---|
| `page` | Integer | `1` | Theo mục 7.2 |
| `pageSize` | Integer | `20` | Theo mục 7.2 |
| `type` | String | Không lọc | `STOCK_RECEIPT` hoặc `SALES_ORDER` |
| `from` | String | Không giới hạn đầu | Thời điểm UTC hợp lệ |
| `to` | String | Không giới hạn cuối | Thời điểm UTC hợp lệ |

Định dạng thời gian được chấp nhận:

```text
YYYY-MM-DDTHH:mm:ssZ
```

hoặc:

```text
YYYY-MM-DDTHH:mm:ss.SSSZ
```

Ví dụ:

```text
2026-11-24T00:00:00Z
2026-11-24T00:00:00.000Z
```

Quy định:

- Phải là thời điểm lịch hợp lệ.
- Chỉ nhận múi giờ UTC với hậu tố `Z`.
- Không nhận ngày đơn lẻ hoặc thời gian không có múi giờ.
- Nếu có cả hai, phải bảo đảm `from < to`.

### 11.2. Khoảng thời gian

Bộ lọc áp dụng lên `confirmed_at`:

```text
confirmed_at >= from
confirmed_at < to
```

Khoảng thời gian bao gồm đầu và không bao gồm cuối.

Nếu bỏ `from`, không giới hạn đầu.

Nếu bỏ `to`, không giới hạn cuối.

Cách dùng khoảng không bao gồm cuối giúp ghép các khoảng liên tiếp mà không đếm trùng giao dịch ở ranh giới.

### 11.3. Ý nghĩa loại lịch sử

| `type` | Nguồn | `quantityChange` |
|---|---|---|
| `STOCK_RECEIPT` | Dòng phiếu nhập đã xác nhận | Số dương |
| `SALES_ORDER` | Dòng đơn bán đã xác nhận | Số âm |

Ví dụ:

```text
Nhập 5 sản phẩm → quantityChange = 5
Bán 2 sản phẩm → quantityChange = -2
```

`quantityChange` là trường tính toán trong response, không phải cột mới trong database.

### 11.4. Định danh một mục lịch sử

Mỗi mục được nhận diện bằng bộ:

```text
(type, documentId, itemId)
```

Trong đó:

- `type` phân biệt phiếu nhập và đơn bán.
- `documentId` là ID chứng từ nguồn.
- `itemId` là ID dòng chi tiết.

ID ở hai bảng khác nhau có thể trùng nhau. Không dùng riêng `documentId` hoặc `itemId` làm định danh toàn cục của lịch sử.

### 11.5. Sắp xếp

Thứ tự:

1. `confirmedAt DESC`.
2. Khi cùng thời điểm, `STOCK_RECEIPT` đứng trước `SALES_ORDER`.
3. Trong cùng loại và thời điểm, `documentId DESC`.
4. Cuối cùng, `itemId DESC`.

Các tiêu chí phụ chỉ giúp phân trang có thứ tự xác định.

Không được hiểu thứ tự này là thứ tự commit tuyệt đối giữa những giao dịch có cùng thời gian xác nhận.

### 11.6. Request mẫu

```http
GET /api/inventory/101/history?page=1&pageSize=20&from=2026-11-01T00:00:00Z&to=2026-12-01T00:00:00Z
Authorization: Bearer <access_token>
```

### 11.7. Response — `200 OK`

```json
{
  "data": [
    {
      "type": "SALES_ORDER",
      "documentId": 2001,
      "itemId": 4001,
      "productId": 101,
      "quantityChange": -2,
      "confirmedBy": 5,
      "confirmedAt": "2026-11-24T04:00:00.000Z"
    },
    {
      "type": "STOCK_RECEIPT",
      "documentId": 1001,
      "itemId": 3001,
      "productId": 101,
      "quantityChange": 10,
      "confirmedBy": 1,
      "confirmedAt": "2026-11-24T03:00:00.000Z"
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

`totalItems` là số mục lịch sử sau khi áp dụng toàn bộ bộ lọc.

### 11.8. Truy cập chứng từ nguồn

| Loại | API chi tiết |
|---|---|
| `STOCK_RECEIPT` | `GET /api/stock-receipts/{documentId}` |
| `SALES_ORDER` | `GET /api/sales-orders/{documentId}` |

API lịch sử tập trung vào thay đổi số lượng.

Giá nhập, giá bán, tổng tiền và nhà cung cấp được xem qua API chứng từ tương ứng.

### 11.9. Trường hợp không có lịch sử

Sản phẩm tồn tại nhưng chưa có chứng từ xác nhận:

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

Trả `200 OK`, không phải `404`.

### 11.10. Lỗi

| HTTP status | Mã lỗi | Trường hợp |
|---|---|---|
| 400 | `VALIDATION_ERROR` | ID, phân trang, loại hoặc thời gian không hợp lệ |
| 404 | `PRODUCT_NOT_FOUND` | Sản phẩm không tồn tại |
| 500 | `INTERNAL_SERVER_ERROR` | Lỗi truy vấn hoặc dữ liệu nội bộ không hợp lệ |

## 12. Tổng hợp lịch sử từ chứng từ

### 12.1. Nhập kho

Nguồn dữ liệu:

```text
stock_receipts
JOIN stock_receipt_items
```

Điều kiện:

- `stock_receipts.status = CONFIRMED`.
- `stock_receipt_items.product_id` bằng sản phẩm cần tra cứu.
- Áp dụng bộ lọc thời gian lên `stock_receipts.confirmed_at`.

Ánh xạ:

```text
type           = STOCK_RECEIPT
documentId     = stock_receipts.id
itemId         = stock_receipt_items.id
quantityChange = stock_receipt_items.quantity
confirmedBy    = stock_receipts.confirmed_by
confirmedAt    = stock_receipts.confirmed_at
```

### 12.2. Xuất bán

Nguồn dữ liệu:

```text
sales_orders
JOIN sales_order_items
```

Điều kiện:

- `sales_orders.status = CONFIRMED`.
- `sales_order_items.product_id` bằng sản phẩm cần tra cứu.
- Áp dụng bộ lọc thời gian lên `sales_orders.confirmed_at`.

Ánh xạ:

```text
type           = SALES_ORDER
documentId     = sales_orders.id
itemId         = sales_order_items.id
quantityChange = -sales_order_items.quantity
confirmedBy    = sales_orders.confirmed_by
confirmedAt    = sales_orders.confirmed_at
```

### 12.3. Kết hợp hai nguồn

- Sử dụng `UNION ALL` hoặc cách truy vấn tương đương giữ đầy đủ các mục lịch sử.
- Áp dụng sắp xếp và phân trang trên kết quả tổng hợp.
- Không phân trang riêng từng nguồn rồi ghép lại.
- Tổng số bản ghi phải dùng cùng bộ lọc với dữ liệu trả về.
- Không lấy chứng từ nháp hoặc đã hủy.
- Không tổng hợp lịch sử bằng thời điểm cập nhật sản phẩm hoặc tồn kho.

## 13. Tính nhất quán khi đọc

### 13.1. Dữ liệu đã commit

API chỉ trả dữ liệu đã commit.

Không sử dụng chế độ đọc cho phép đọc thay đổi chưa commit.

Việc cập nhật tồn kho và trạng thái chứng từ phải nằm trong cùng giao dịch tại module nhập hàng hoặc bán hàng.

### 13.2. Một response

Trong một response:

- Danh sách và tổng số bản ghi phải dùng cùng bộ lọc.
- Các truy vấn phụ thuộc nhau phải đọc từ cùng ảnh chụp dữ liệu.
- Không để phần đếm nhìn thấy một giao dịch mới trong khi phần dữ liệu chưa nhìn thấy giao dịch đó, hoặc ngược lại.

Nếu dùng nhiều SELECT, triển khai bằng giao dịch đọc với cơ chế snapshot phù hợp.

Không cần khóa ghi các dòng tồn kho chỉ để hiển thị dữ liệu.

### 13.3. Nhiều request khác nhau

Hai request riêng biệt có thể quan sát dữ liệu tại hai thời điểm khác nhau.

Ví dụ:

1. Client lấy tồn kho là `8`.
2. Một đơn bán được xác nhận.
3. Client lấy lịch sử và thấy thêm một lần xuất.

Đây không nhất thiết là lỗi nhất quán vì dữ liệu thay đổi giữa hai lần gọi.

Không bảo đảm phân trang offset giữa nhiều request giữ nguyên ảnh chụp dữ liệu.

### 13.4. Không suy ra tồn từ một trang lịch sử

Không được lấy tổng `quantityChange` của trang đang xem để kết luận tồn hiện tại.

Lý do:

- Có thể còn lịch sử ở các trang khác.
- Có thể đang lọc một khoảng thời gian.
- Có thể chỉ đang lọc nhập hoặc xuất.
- Có thể có giao dịch mới sau lần đọc trước.

Đối chiếu tồn bằng lịch sử phải sử dụng toàn bộ lịch sử phù hợp trong cùng ảnh chụp dữ liệu.

### 13.5. Không suy ra số dư sau từng giao dịch

API không trả `balanceAfter`.

Thiết kế chưa lưu số dư sau từng giao dịch hoặc thứ tự commit toàn cục. Không tính số dư bằng cách cộng dồn tùy ý theo thứ tự phân trang.

## 14. Cache và độ mới dữ liệu

Trong phiên bản ban đầu của nhóm API này:

- Tồn kho và lịch sử được đọc trực tiếp từ database.
- Chưa dùng Redis để trả số lượng tồn.
- Response gửi `Cache-Control: no-store` để yêu cầu client và HTTP cache không lưu phản hồi.

Khi bổ sung cache:

- Phải có chính sách hết hạn và vô hiệu hóa sau xác nhận nhập/bán.
- Phải xác định rõ mức độ chấp nhận dữ liệu cũ.
- Không dùng cache làm căn cứ cuối cùng khi xác nhận đơn bán.
- Lỗi cache không được làm tăng hoặc giảm tồn lần nữa.
- Cần cập nhật tài liệu API nếu thay đổi cam kết về độ mới dữ liệu.

Yêu cầu sử dụng caching của dự án có thể được triển khai trước ở dữ liệu danh mục hoặc sản phẩm theo thiết kế riêng.

## 15. Ánh xạ dữ liệu

### 15.1. Tồn kho

| Trường API | Cột hoặc nguồn |
|---|---|
| `productId` | `inventory.product_id` |
| `sku` | `products.sku` |
| `name` | `products.name` |
| `categoryId` | `products.category_id` |
| `active` | `products.active` |
| `quantity` | `inventory.quantity` |
| `stockStatus` | Suy ra từ `inventory.quantity` |
| `updatedAt` | `inventory.updated_at` |

### 15.2. Lịch sử

| Trường API | Nguồn |
|---|---|
| `type` | Loại chứng từ nguồn |
| `documentId` | ID phiếu nhập hoặc đơn bán |
| `itemId` | ID dòng chi tiết |
| `productId` | Sản phẩm trên dòng |
| `quantityChange` | Số lượng nhập dương hoặc số lượng bán âm |
| `confirmedBy` | Người xác nhận chứng từ |
| `confirmedAt` | Thời điểm xác nhận chứng từ |

Không cần thêm bảng hoặc cột chỉ để tạo các trường tính toán trong response.

## 16. Danh sách lỗi chung

| HTTP status | Mã lỗi | Ý nghĩa |
|---|---|---|
| 400 | `VALIDATION_ERROR` | Path hoặc query không hợp lệ |
| 401 | `AUTH_REQUIRED` | Thiếu thông tin xác thực |
| 401 | `INVALID_TOKEN` | Token không hợp lệ hoặc tài khoản không tồn tại |
| 401 | `TOKEN_EXPIRED` | Token hết hạn |
| 403 | `ACCOUNT_DISABLED` | Tài khoản bị khóa |
| 403 | `FORBIDDEN` | Vai trò không được phép truy cập |
| 404 | `PRODUCT_NOT_FOUND` | Sản phẩm không tồn tại |
| 500 | `INTERNAL_SERVER_ERROR` | Lỗi nội bộ hoặc vi phạm toàn vẹn dữ liệu |

Nhóm API đọc không phát sinh `INSUFFICIENT_STOCK`.

Thiếu hàng là trạng thái dữ liệu hợp lệ khi tra cứu; lỗi không đủ tồn chỉ phát sinh trong nghiệp vụ xác nhận bán.

## 17. Yêu cầu bảo mật và triển khai

- Sử dụng truy vấn có tham số.
- Không nối trực tiếp từ khóa hoặc bộ lọc vào SQL.
- Xử lý ký tự wildcard và escape đúng quy tắc khi dùng LIKE.
- Không cho client truyền tên bảng, tên cột hoặc đoạn SQL sắp xếp.
- Áp dụng giới hạn phân trang.
- Không thực hiện sửa hoặc khởi tạo dữ liệu trong API GET.
- Không trả câu SQL, stack trace hoặc thông tin kết nối.
- Không ghi JWT vào log.
- Không tải toàn bộ lịch sử về bộ nhớ ứng dụng rồi mới phân trang.
- Đánh giá index theo truy vấn thực tế và kế hoạch thực thi.

Các nhóm cột cần xem xét khi tối ưu:

- Sản phẩm và danh mục.
- `product_id` cùng khóa liên kết chứng từ trong bảng chi tiết.
- Trạng thái và thời điểm xác nhận trong bảng chứng từ.

Không thêm index chỉ dựa trên tên cột; cần kiểm tra dữ liệu và kế hoạch truy vấn trước.

## 18. Các tình huống kiểm thử

### 18.1. Xác thực và phân quyền

- Không có token.
- Token sai hoặc hết hạn.
- Tài khoản bị khóa hoặc không tồn tại.
- ADMIN và STAFF truy cập thành công.
- Không có endpoint sửa tồn trực tiếp.

### 18.2. Danh sách tồn kho

- Chưa có sản phẩm.
- Sản phẩm mới có tồn bằng 0.
- Tìm theo tên và SKU.
- Từ khóa chứa dấu tiếng Việt, dấu nháy, `%`, `_` hoặc ký tự escape.
- Lọc theo danh mục.
- Lọc đang kinh doanh hoặc ngừng kinh doanh.
- Lọc còn hàng hoặc hết hàng.
- Kết hợp nhiều bộ lọc.
- Danh mục lọc không tồn tại trả danh sách rỗng.
- Phân trang mặc định, giới hạn và trang vượt phạm vi.
- Query rỗng, lặp lại hoặc không được hỗ trợ.
- Thứ tự mặc định theo productId giảm dần.

### 18.3. Tồn kho chi tiết

- Sản phẩm còn hàng.
- Sản phẩm hết hàng.
- Sản phẩm ngừng kinh doanh nhưng vẫn còn tồn.
- Sản phẩm không tồn tại.
- ID sai định dạng hoặc vượt giới hạn.
- Sản phẩm tồn tại nhưng thiếu dòng tồn trả lỗi nội bộ.
- Không tự sửa tồn âm hoặc khởi tạo dòng trong request đọc.
- Sửa tên sản phẩm không làm thay đổi thời điểm cập nhật tồn.

### 18.4. Lịch sử nhập/xuất

- Sản phẩm chưa có lịch sử trả danh sách rỗng.
- Phiếu nhập đã xác nhận tạo mục nhập dương.
- Đơn bán đã xác nhận tạo mục xuất âm.
- Chứng từ nháp và đã hủy không xuất hiện.
- Lọc chỉ nhập hoặc chỉ xuất.
- Lọc theo from, to hoặc cả hai.
- Giao dịch đúng from được lấy.
- Giao dịch đúng to không được lấy.
- from bằng hoặc sau to bị từ chối.
- Thời gian sai định dạng hoặc ngày không tồn tại bị từ chối.
- Phiếu tạo trước khoảng lọc nhưng xác nhận trong khoảng vẫn xuất hiện.
- ID chứng từ nhập và bán trùng nhau không làm mất mục lịch sử.
- Nhiều giao dịch cùng thời điểm vẫn có thứ tự phân trang xác định.
- Không phân trang riêng từng nguồn trước khi ghép.
- Thay đổi tên hoặc SKU không làm mất lịch sử.

### 18.5. Nhất quán với nghiệp vụ

- Tạo hoặc sửa đơn nháp không đổi tồn và không tạo lịch sử.
- Xác nhận nhập thành công tăng tồn và tạo lịch sử nhập.
- Xác nhận bán thành công giảm tồn và tạo lịch sử xuất.
- Xác nhận thất bại không đổi tồn và không tạo lịch sử.
- Xác nhận lại không tạo thêm lịch sử hoặc thay đổi tồn.
- Hủy nháp không thay đổi tồn.
- Một chứng từ nhiều dòng cập nhật đầy đủ hoặc rollback toàn bộ.
- Lỗi giao dịch không làm xuất hiện lịch sử không tương ứng với tồn.

### 18.6. Đọc đồng thời

- Đọc danh sách trong lúc có xác nhận nhập hoặc bán.
- Danh sách và tổng số bản ghi nhất quán trong cùng response.
- Lịch sử không đọc dữ liệu chưa commit.
- Hai request riêng có thể thấy dữ liệu khác nhau theo thời điểm.
- API đọc không giữ khóa ghi không cần thiết.
- Đối chiếu toàn bộ lịch sử với tồn hiện tại trong cùng snapshot.

### 18.7. Phản hồi và lỗi

- `quantity` là số nguyên không âm.
- `quantityChange` dương khi nhập và âm khi bán.
- `stockStatus` tương ứng đúng số lượng.
- Thời gian trả theo UTC.
- Response có `Cache-Control: no-store`.
- Lỗi không lộ thông tin nội bộ.

## 19. Các điểm cần đồng bộ và hoàn thiện

### 19.1. Các lựa chọn trong bản thiết kế này

| Nội dung | Lựa chọn |
|---|---|
| Quyền đọc | ADMIN và STAFF |
| Tồn hiện tại | Đọc từ bảng inventory |
| Tồn ban đầu | 0 |
| Sản phẩm ngừng kinh doanh | Vẫn tra cứu được |
| Phân loại tồn | IN_STOCK hoặc OUT_OF_STOCK |
| Lịch sử | Tổng hợp từ chứng từ CONFIRMED |
| Dấu thay đổi tồn | Nhập dương, bán âm |
| Thời điểm lịch sử | confirmed_at |
| Người thực hiện | confirmed_by |
| Khoảng thời gian | Bao gồm from, không bao gồm to |
| Định danh lịch sử | type, documentId và itemId |
| Số dư sau giao dịch | Không cung cấp |
| Cache tồn và lịch sử | Chưa sử dụng trong phiên bản đầu |
| Chỉnh sửa tồn trực tiếp | Không hỗ trợ |

Các lựa chọn này cần được đối chiếu với tài liệu đã commit để tránh duy trì quy tắc khác nhau giữa các module.

### 19.2. Nội dung cần hoàn thiện

1. **Collation:** đồng bộ cách tìm tên và SKU với `catalog.md` và `database.md`.
2. **Enum chứng từ:** đối chiếu giá trị `CONFIRMED` với database và các API nhập/bán.
3. **Quy ước lỗi:** thống nhất mã lỗi và cấu trúc `details`.
4. **Múi giờ:** cấu hình lưu và đọc thời gian UTC thống nhất giữa ứng dụng và database.
5. **Index:** đánh giá truy vấn lịch sử và phân trang bằng dữ liệu thử nghiệm.
6. **Cache:** cập nhật thiết kế nếu bổ sung cache cho tồn kho hoặc lịch sử.

Sau khi hoàn thiện từng mục, cập nhật quyết định vào nội dung chính và loại bỏ khỏi danh sách chờ.
