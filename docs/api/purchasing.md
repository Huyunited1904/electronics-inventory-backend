# Thiết kế API quản lý phiếu nhập kho

## 1. Mục đích

Tài liệu mô tả API quản lý phiếu nhập kho trong hệ thống quản lý bán hàng và tồn kho đồ điện tử.

Tài liệu là căn cứ để:

- Tạo và quản lý phiếu nhập ở trạng thái nháp.
- Ghi nhận nhà cung cấp, mặt hàng, số lượng và giá nhập.
- Xác định quyền thao tác của ADMIN và STAFF.
- Xác nhận phiếu nhập và tăng tồn kho một cách nhất quán.
- Ngăn xác nhận trùng hoặc chỉnh sửa phiếu trong trạng thái không hợp lệ.
- Xây dựng kiểm thử nghiệp vụ, giao dịch và yêu cầu đồng thời.

Đường dẫn gốc:

```text
/api/stock-receipts
```

## 2. Tài liệu liên quan

| Tài liệu | Nội dung đối chiếu |
|---|---|
| `../scope.md` | Phạm vi chức năng và phân quyền |
| `../business-rules.md` | Quy tắc nhập hàng, trạng thái chứng từ và tồn kho |
| `../database.md` | Cấu trúc bảng và ràng buộc dữ liệu |
| `../diagrams/erd.dbml` | Quan hệ giữa phiếu nhập và các bảng liên quan |
| `../architecture.md` | Tổ chức xử lý nghiệp vụ và giao dịch |
| `auth.md` | Xác thực JWT |
| `users.md` | Kiểm tra tài khoản và quyền truy cập |
| `catalog.md` | Thông tin sản phẩm và trạng thái kinh doanh |
| `suppliers.md` | Thông tin nhà cung cấp |
| `inventory.md` | Tra cứu tồn kho và lịch sử nhập/xuất; bổ sung sau |

Nếu có mâu thuẫn, phải thống nhất và cập nhật các tài liệu liên quan trước khi triển khai phần bị ảnh hưởng.

## 3. Phạm vi

### 3.1. Chức năng được hỗ trợ

- Xem danh sách phiếu nhập.
- Xem chi tiết phiếu nhập.
- Tạo phiếu nhập nháp.
- Đổi nhà cung cấp của phiếu nháp.
- Thêm mặt hàng vào phiếu nháp.
- Sửa số lượng và giá nhập của mặt hàng.
- Xóa mặt hàng khỏi phiếu nháp.
- Hủy phiếu nháp.
- Xác nhận phiếu nhập.

### 3.2. Chức năng ngoài phạm vi

- Xóa phiếu nhập khỏi cơ sở dữ liệu.
- Sửa hoặc hủy phiếu đã xác nhận.
- Khôi phục phiếu đã hủy.
- Trả hàng cho nhà cung cấp.
- Quản lý công nợ và thanh toán.
- Phí vận chuyển, thuế hoặc chiết khấu nhập hàng.
- Quản lý nhiều kho, số serial hoặc IMEI.
- Nhập trực tiếp một số lượng tồn kho tùy ý.
- Quản lý bảng lịch sử tồn kho riêng.

Lịch sử nhập kho được suy ra từ phiếu nhập và các dòng mặt hàng đã xác nhận. Thiết kế hiện tại không sử dụng bảng `stock_movements`.

## 4. Mô hình dữ liệu

### 4.1. Phiếu nhập

Bảng `stock_receipts` lưu thông tin chung:

- Nhà cung cấp.
- Người tạo.
- Trạng thái.
- Thời điểm tạo.
- Người xác nhận.
- Thời điểm xác nhận.

### 4.2. Dòng mặt hàng

Bảng `stock_receipt_items` lưu:

- Phiếu nhập liên quan.
- Sản phẩm.
- Số lượng nhập.
- Giá nhập của một đơn vị sản phẩm.

Quan hệ:

```text
Một phiếu nhập → Nhiều dòng mặt hàng
Một dòng mặt hàng → Một sản phẩm
```

Ví dụ, một phiếu nhập gồm 5 laptop và 10 con chuột được lưu thành:

- Một bản ghi trong `stock_receipts`.
- Hai bản ghi trong `stock_receipt_items`.

### 4.3. Tồn kho

Bảng `inventory` lưu số lượng tồn hiện tại của từng sản phẩm.

Phiếu nháp và phiếu đã hủy không tác động đến tồn kho.

Chỉ phiếu được xác nhận thành công mới làm tăng tồn kho.

## 5. Xác thực và phân quyền

### 5.1. Xác thực

Tất cả endpoint yêu cầu JWT access token:

```http
Authorization: Bearer <access_token>
```

Backend phải:

- Xác thực token.
- Kiểm tra tài khoản tồn tại và đang hoạt động.
- Lấy vai trò hiện tại từ cơ sở dữ liệu.
- Kiểm tra quyền thao tác với phiếu nhập.

### 5.2. Bảng phân quyền

| Chức năng | ADMIN | STAFF |
|---|---|---|
| Xem danh sách và chi tiết phiếu | Được phép | Được phép |
| Tạo phiếu nháp | Được phép | Được phép |
| Đổi nhà cung cấp | Mọi phiếu nháp | Chỉ phiếu nháp do mình tạo |
| Thêm, sửa, xóa mặt hàng | Mọi phiếu nháp | Chỉ phiếu nháp do mình tạo |
| Hủy phiếu | Mọi phiếu nháp | Chỉ phiếu nháp do mình tạo |
| Xác nhận phiếu | Mọi phiếu nháp | Chỉ phiếu nháp do mình tạo |

Quyền xem không đồng nghĩa với quyền sửa hoặc xác nhận.

### 5.3. Người tạo và người xác nhận

- `createdBy` được lấy từ tài khoản đang đăng nhập khi tạo phiếu.
- `confirmedBy` được lấy từ tài khoản thực hiện xác nhận thành công.
- Client không được tự gửi các trường này.
- Người tạo có thể đồng thời là người xác nhận.
- ADMIN xác nhận phiếu do STAFF tạo không làm thay đổi `createdBy`.

## 6. Trạng thái phiếu nhập

### 6.1. Các trạng thái

| Trạng thái | Ý nghĩa | Được sửa | Tác động tồn kho |
|---|---|:---:|---|
| `DRAFT` | Phiếu nháp | Có, theo phân quyền | Không |
| `CONFIRMED` | Đã xác nhận nhập kho | Không | Đã tăng tồn kho |
| `CANCELLED` | Đã hủy | Không | Không |

### 6.2. Chuyển trạng thái hợp lệ

```text
DRAFT → CONFIRMED
DRAFT → CANCELLED
```

Không hỗ trợ:

- `CONFIRMED → DRAFT`
- `CONFIRMED → CANCELLED`
- `CANCELLED → DRAFT`
- `CANCELLED → CONFIRMED`

### 6.3. Xác nhận lại

Nếu gọi xác nhận một phiếu đã `CONFIRMED`:

- Trả `409 STOCK_RECEIPT_ALREADY_CONFIRMED`.
- Không tăng tồn kho lần nữa.
- Không thay đổi người hoặc thời điểm xác nhận ban đầu.

### 6.4. Hủy lại

Nếu gọi hủy một phiếu đã `CANCELLED`:

- Trả `200 OK` với dữ liệu phiếu hiện tại.
- Không phát sinh thay đổi dữ liệu.

Người gọi vẫn phải có quyền thao tác với phiếu đó.

## 7. Quy ước dữ liệu

### 7.1. Quy ước chung

- Request và response sử dụng JSON.
- Request có body phải gửi `Content-Type: application/json`.
- Tên trường API sử dụng `camelCase`.
- Thời gian sử dụng ISO 8601 và múi giờ UTC.
- ID là số nguyên từ `1` đến `2147483647`.
- Không chấp nhận body dạng mảng, chuỗi hoặc `null`.
- Không chấp nhận trường ngoài danh sách cho phép.
- Không tự chuyển chuỗi thành số hoặc boolean trong JSON body.
- Query parameter không được lặp lại.
- Query parameter được gửi nhưng rỗng là không hợp lệ.

### 7.2. Số lượng

`quantity` phải:

- Là số nguyên JSON.
- Lớn hơn `0`.
- Không vượt quá `2147483647`.

Ví dụ hợp lệ:

```json
{
  "quantity": 5
}
```

Ví dụ không hợp lệ:

```json
{
  "quantity": "5"
}
```

Khi xác nhận, tổng tồn kho hiện tại và số lượng nhập cũng không được vượt giới hạn kiểu dữ liệu `inventory.quantity`.

### 7.3. Giá nhập

`unitCost` là giá nhập của một đơn vị sản phẩm.

API nhận và trả giá nhập dưới dạng chuỗi thập phân, tương ứng với `DECIMAL(15,2)` trong cơ sở dữ liệu.

Quy định:

- Không âm.
- Tối đa 13 chữ số phần nguyên và 2 chữ số phần thập phân.
- Không có dấu phân cách hàng nghìn.
- Không sử dụng dạng số mũ.
- Không chấp nhận khoảng trắng.
- Không tự làm tròn đầu vào có quá 2 chữ số thập phân.
- Response luôn có đúng 2 chữ số thập phân.

Ví dụ:

| Giá trị | Kết quả |
|---|---|
| `"0"` | Hợp lệ, chuẩn hóa thành `"0.00"` |
| `"12000000"` | Hợp lệ, chuẩn hóa thành `"12000000.00"` |
| `"12000000.50"` | Hợp lệ |
| `12000000` | Không hợp lệ: sai kiểu |
| `"-1.00"` | Không hợp lệ |
| `"100.123"` | Không hợp lệ |
| `"12,000,000"` | Không hợp lệ |

Giá trị lớn nhất:

```text
9999999999999.99
```

Giá nhập không tự lấy từ `products.sale_price`.

### 7.4. Thành tiền và tổng giá trị

Thành tiền một dòng:

```text
lineTotal = quantity × unitCost
```

Tổng giá trị phiếu:

```text
totalAmount = Tổng lineTotal của tất cả dòng
```

Quy định:

- Các giá trị này được tính từ dữ liệu dòng mặt hàng.
- Không lưu thêm cột tổng tiền trong thiết kế hiện tại.
- Client không được gửi `lineTotal` hoặc `totalAmount`.
- Response trả chuỗi thập phân có đúng 2 chữ số thập phân.
- Phải sử dụng phép toán thập phân chính xác; không dùng số thực JavaScript để tính tiền.
- Tổng tiền có thể lớn hơn giới hạn của một giá trị `unitCost`; không ép tổng tiền về `DECIMAL(15,2)` gây tràn hoặc cắt dữ liệu.
- Phiếu không có mặt hàng có tổng giá trị `"0.00"`.

## 8. Quy tắc nghiệp vụ

### 8.1. Tạo phiếu nháp

- Nhà cung cấp phải tồn tại.
- Phiếu mới có trạng thái `DRAFT`.
- `confirmedBy` và `confirmedAt` bằng `null`.
- Phiếu được tạo ban đầu chưa có dòng mặt hàng.
- Thêm mặt hàng qua endpoint riêng.
- Tạo phiếu không thay đổi tồn kho.

### 8.2. Mặt hàng trong phiếu

- Sản phẩm phải tồn tại.
- Sản phẩm phải đang kinh doanh khi thêm vào phiếu.
- Một sản phẩm chỉ xuất hiện một lần trong một phiếu.
- Ràng buộc UNIQUE phải áp dụng cho cặp `(receipt_id, product_id)`.
- Nếu muốn tăng số lượng của sản phẩm đã có, sửa dòng hiện tại.
- Không tự cộng dồn khi client gửi lại yêu cầu thêm sản phẩm.
- Không đổi `productId` của dòng hiện có; muốn thay sản phẩm phải xóa dòng cũ và thêm dòng mới.
- Chỉ sửa hoặc xóa dòng khi phiếu còn `DRAFT`.

### 8.3. Sản phẩm ngừng kinh doanh

Quy tắc đã thống nhất:

- Không được thêm sản phẩm đã ngừng kinh doanh vào phiếu.
- Nếu sản phẩm ngừng kinh doanh sau khi được thêm vào phiếu nháp, dòng hàng vẫn được giữ để tra cứu.
- Không được xác nhận phiếu nếu có bất kỳ dòng nào tham chiếu sản phẩm đã ngừng kinh doanh.
- Phải xóa dòng không hợp lệ trước khi xác nhận.
- Có thể hủy toàn bộ phiếu nếu không tiếp tục nhập hàng.
- Phiếu đã xác nhận trước khi sản phẩm ngừng kinh doanh được giữ nguyên.

Trong phiên bản này, dòng thuộc sản phẩm đã ngừng kinh doanh không được sửa số lượng hoặc giá nhập; người dùng có thể xóa dòng đó khỏi phiếu nháp.

### 8.4. Đổi nhà cung cấp

- Chỉ được đổi khi phiếu còn `DRAFT`.
- Nhà cung cấp mới phải tồn tại.
- Các dòng mặt hàng được giữ nguyên.
- Không tự thay đổi giá nhập đã nhập.
- Thiết kế hiện tại không có quan hệ giới hạn sản phẩm được cung cấp bởi từng nhà cung cấp.

### 8.5. Điều kiện xác nhận

Phiếu chỉ được xác nhận khi:

- Người gọi có quyền.
- Phiếu đang ở trạng thái `DRAFT`.
- Có ít nhất một dòng mặt hàng.
- Nhà cung cấp còn tồn tại.
- Tất cả sản phẩm tồn tại và đang kinh doanh.
- Các dòng có số lượng và giá nhập hợp lệ.
- Mỗi sản phẩm có dòng tồn kho tương ứng.
- Tồn kho sau khi cộng không vượt giới hạn kiểu dữ liệu.

Nếu bất kỳ điều kiện nào không thỏa mãn, từ chối toàn bộ thao tác.

### 8.6. Hủy phiếu

- Chỉ phiếu nháp được chuyển sang `CANCELLED`.
- Phiếu nháp rỗng vẫn được hủy.
- Giữ lại các dòng mặt hàng để tra cứu.
- Không tăng hoặc giảm tồn kho.
- `confirmedBy` và `confirmedAt` tiếp tục bằng `null`.

Thiết kế hiện tại không có cột người hủy, thời điểm hủy hoặc lý do hủy. API không nhận hoặc trả các trường này.

## 9. Danh sách endpoint

| Method | Endpoint | Chức năng |
|---|---|---|
| GET | `/api/stock-receipts` | Xem danh sách phiếu |
| GET | `/api/stock-receipts/{id}` | Xem chi tiết phiếu |
| POST | `/api/stock-receipts` | Tạo phiếu nháp |
| PATCH | `/api/stock-receipts/{id}` | Đổi nhà cung cấp |
| POST | `/api/stock-receipts/{id}/items` | Thêm mặt hàng |
| PATCH | `/api/stock-receipts/{id}/items/{itemId}` | Sửa dòng mặt hàng |
| DELETE | `/api/stock-receipts/{id}/items/{itemId}` | Xóa dòng mặt hàng |
| POST | `/api/stock-receipts/{id}/confirm` | Xác nhận nhập kho |
| POST | `/api/stock-receipts/{id}/cancel` | Hủy phiếu |

Không cung cấp endpoint xóa phiếu hoặc cập nhật trực tiếp trường `status`.

## 10. Cấu trúc phản hồi

### 10.1. Phiếu trong danh sách

```json
{
  "id": 1001,
  "supplierId": 1,
  "createdBy": 5,
  "status": "DRAFT",
  "createdAt": "2026-11-22T03:00:00.000Z",
  "confirmedBy": null,
  "confirmedAt": null,
  "itemCount": 2,
  "totalAmount": "61500000.00"
}
```

`itemCount` là số dòng mặt hàng, không phải tổng số lượng sản phẩm.

### 10.2. Chi tiết phiếu

```json
{
  "data": {
    "id": 1001,
    "supplierId": 1,
    "createdBy": 5,
    "status": "DRAFT",
    "createdAt": "2026-11-22T03:00:00.000Z",
    "confirmedBy": null,
    "confirmedAt": null,
    "itemCount": 2,
    "totalAmount": "61500000.00",
    "items": [
      {
        "id": 2001,
        "productId": 101,
        "quantity": 5,
        "unitCost": "12000000.00",
        "lineTotal": "60000000.00"
      },
      {
        "id": 2002,
        "productId": 102,
        "quantity": 10,
        "unitCost": "150000.00",
        "lineTotal": "1500000.00"
      }
    ]
  }
}
```

Các dòng được sắp xếp theo `id ASC`.

Response dùng ID để tham chiếu nhà cung cấp, người dùng và sản phẩm. Thông tin chi tiết của các tài nguyên này được truy vấn qua API tương ứng.

### 10.3. Dòng mặt hàng

```json
{
  "data": {
    "id": 2001,
    "productId": 101,
    "quantity": 5,
    "unitCost": "12000000.00",
    "lineTotal": "60000000.00"
  }
}
```

### 10.4. Phản hồi lỗi

```json
{
  "error": {
    "code": "PRODUCT_INACTIVE",
    "message": "Phiếu chứa sản phẩm đã ngừng kinh doanh.",
    "details": [
      {
        "field": "items",
        "message": "Sản phẩm có ID 101 đã ngừng kinh doanh."
      }
    ]
  }
}
```

Trường `details` không bắt buộc.

## 11. Xem danh sách phiếu nhập

```http
GET /api/stock-receipts
```

**Quyền:** ADMIN, STAFF.

### 11.1. Query parameters

| Trường | Kiểu | Mặc định | Quy định |
|---|---|---|---|
| `page` | Integer | `1` | Từ `1` đến `2147483647` |
| `pageSize` | Integer | `20` | Từ `1` đến `100` |
| `status` | String | Không lọc | `DRAFT`, `CONFIRMED`, `CANCELLED` |
| `supplierId` | Integer | Không lọc | ID hợp lệ |
| `createdBy` | Integer | Không lọc | ID hợp lệ |

Quy định:

- Các điều kiện lọc kết hợp bằng AND.
- Thứ tự mặc định: `id DESC`.
- Bỏ qua `status` để xem mọi trạng thái.
- ID lọc hợp lệ nhưng không tồn tại trả danh sách rỗng.
- Chưa hỗ trợ tìm kiếm tự do, lọc theo thời gian hoặc tùy chọn sắp xếp.

### 11.2. Response — `200 OK`

```json
{
  "data": [
    {
      "id": 1001,
      "supplierId": 1,
      "createdBy": 5,
      "status": "DRAFT",
      "createdAt": "2026-11-22T03:00:00.000Z",
      "confirmedBy": null,
      "confirmedAt": null,
      "itemCount": 2,
      "totalAmount": "61500000.00"
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

- `totalItems` là số phiếu sau khi áp dụng bộ lọc.
- Không đếm số dòng mặt hàng thay cho số phiếu khi JOIN.
- Không có kết quả: `data: []`, `totalItems: 0`, `totalPages: 0`.
- Trang vượt phạm vi: trả `200 OK` và danh sách rỗng.

## 12. Xem chi tiết phiếu nhập

```http
GET /api/stock-receipts/{id}
```

**Quyền:** ADMIN, STAFF.

Trả `200 OK` theo cấu trúc tại mục 10.2.

Có thể xem phiếu ở cả ba trạng thái.

Phiếu rỗng có:

```json
{
  "itemCount": 0,
  "totalAmount": "0.00",
  "items": []
}
```

Lỗi:

| HTTP status | Mã lỗi | Trường hợp |
|---|---|---|
| 400 | `VALIDATION_ERROR` | ID hoặc query không hợp lệ |
| 404 | `STOCK_RECEIPT_NOT_FOUND` | Phiếu không tồn tại |

## 13. Tạo phiếu nhập nháp

```http
POST /api/stock-receipts
```

**Quyền:** ADMIN, STAFF.

### 13.1. Request body

```json
{
  "supplierId": 1
}
```

Chỉ chấp nhận `supplierId`, là ID nguyên dương hợp lệ.

Không nhận `items`, `status`, `createdBy`, `confirmedBy`, thời gian hoặc tổng tiền.

### 13.2. Response — `201 Created`

Header:

```http
Location: /api/stock-receipts/1001
```

```json
{
  "data": {
    "id": 1001,
    "supplierId": 1,
    "createdBy": 5,
    "status": "DRAFT",
    "createdAt": "2026-11-22T03:00:00.000Z",
    "confirmedBy": null,
    "confirmedAt": null,
    "itemCount": 0,
    "totalAmount": "0.00",
    "items": []
  }
}
```

### 13.3. Lỗi nghiệp vụ

| HTTP status | Mã lỗi | Trường hợp |
|---|---|---|
| 400 | `VALIDATION_ERROR` | Body không hợp lệ |
| 404 | `SUPPLIER_NOT_FOUND` | Nhà cung cấp không tồn tại |

### 13.4. Gửi lại yêu cầu

Endpoint tạo phiếu chưa hỗ trợ idempotency key.

Gửi lại POST có thể tạo một phiếu nháp mới. Cơ chế chống xác nhận hai lần của cùng một phiếu không ngăn việc tạo hai phiếu khác nhau có cùng nội dung.

## 14. Đổi nhà cung cấp của phiếu nháp

```http
PATCH /api/stock-receipts/{id}
```

**Quyền:** ADMIN hoặc STAFF tạo phiếu.

### 14.1. Request body

```json
{
  "supplierId": 2
}
```

Chỉ chấp nhận trường `supplierId`.

### 14.2. Hành vi

- Phiếu phải còn `DRAFT`.
- Nhà cung cấp mới phải tồn tại.
- Giữ nguyên các dòng mặt hàng và giá nhập.
- Gửi lại nhà cung cấp hiện tại vẫn trả thành công.
- Không thay đổi người tạo hoặc thời điểm tạo.

### 14.3. Response — `200 OK`

Trả chi tiết phiếu sau cập nhật theo mục 10.2.

### 14.4. Lỗi nghiệp vụ

| HTTP status | Mã lỗi | Trường hợp |
|---|---|---|
| 403 | `FORBIDDEN` | STAFF không phải người tạo |
| 404 | `STOCK_RECEIPT_NOT_FOUND` | Phiếu không tồn tại |
| 404 | `SUPPLIER_NOT_FOUND` | Nhà cung cấp mới không tồn tại |
| 409 | `STOCK_RECEIPT_NOT_DRAFT` | Phiếu không còn là nháp |

## 15. Thêm mặt hàng vào phiếu nháp

```http
POST /api/stock-receipts/{id}/items
```

**Quyền:** ADMIN hoặc STAFF tạo phiếu.

### 15.1. Request body

```json
{
  "productId": 101,
  "quantity": 5,
  "unitCost": "12000000.00"
}
```

| Trường | Kiểu | Bắt buộc | Quy định |
|---|---|:---:|---|
| `productId` | Integer | Có | Sản phẩm tồn tại và đang kinh doanh |
| `quantity` | Integer | Có | Theo mục 7.2 |
| `unitCost` | String | Có | Theo mục 7.3 |

Không nhận ID dòng, ID phiếu trong body hoặc giá trị tính toán.

### 15.2. Hành vi

- Phiếu phải còn `DRAFT`.
- Sản phẩm chưa xuất hiện trong phiếu.
- Thêm dòng mới với số lượng và giá nhập được cung cấp.
- Không tăng tồn kho.
- Không thay đổi giá bán của sản phẩm.

### 15.3. Response — `201 Created`

Trả dòng mặt hàng vừa tạo theo mục 10.3.

### 15.4. Lỗi nghiệp vụ

| HTTP status | Mã lỗi | Trường hợp |
|---|---|---|
| 403 | `FORBIDDEN` | Không có quyền thao tác |
| 404 | `STOCK_RECEIPT_NOT_FOUND` | Phiếu không tồn tại |
| 404 | `PRODUCT_NOT_FOUND` | Sản phẩm không tồn tại |
| 409 | `STOCK_RECEIPT_NOT_DRAFT` | Phiếu không còn là nháp |
| 409 | `PRODUCT_INACTIVE` | Sản phẩm đã ngừng kinh doanh |
| 409 | `STOCK_RECEIPT_PRODUCT_ALREADY_EXISTS` | Sản phẩm đã có trong phiếu |

Gửi lại yêu cầu thêm cùng sản phẩm không được tự cộng số lượng.

## 16. Sửa dòng mặt hàng

```http
PATCH /api/stock-receipts/{id}/items/{itemId}
```

**Quyền:** ADMIN hoặc STAFF tạo phiếu.

### 16.1. Request body

Cho phép gửi một hoặc cả hai trường:

```json
{
  "quantity": 8,
  "unitCost": "11800000.00"
}
```

Quy định:

- Ít nhất một trường phải được gửi.
- Trường không gửi được giữ nguyên.
- Không chấp nhận `null`.
- Không chấp nhận `productId` hoặc các trường khác.
- `quantity` là số lượng mới của dòng, không phải số lượng cộng thêm.

### 16.2. Hành vi

- Phiếu phải còn `DRAFT`.
- Dòng phải thuộc đúng phiếu trong URL.
- Sản phẩm của dòng phải đang kinh doanh.
- Cập nhật các trường trong cùng một thao tác nguyên tử.
- Không thay đổi tồn kho.
- Gửi lại dữ liệu hiện tại vẫn trả thành công.

### 16.3. Response — `200 OK`

Trả dòng mặt hàng sau cập nhật theo mục 10.3.

### 16.4. Lỗi nghiệp vụ

| HTTP status | Mã lỗi | Trường hợp |
|---|---|---|
| 403 | `FORBIDDEN` | Không có quyền thao tác |
| 404 | `STOCK_RECEIPT_NOT_FOUND` | Phiếu không tồn tại |
| 404 | `STOCK_RECEIPT_ITEM_NOT_FOUND` | Dòng không tồn tại hoặc không thuộc phiếu |
| 409 | `STOCK_RECEIPT_NOT_DRAFT` | Phiếu không còn là nháp |
| 409 | `PRODUCT_INACTIVE` | Sản phẩm đã ngừng kinh doanh |

Không được chỉ tìm dòng theo `itemId` rồi bỏ qua kiểm tra `receipt_id`.

## 17. Xóa dòng mặt hàng

```http
DELETE /api/stock-receipts/{id}/items/{itemId}
```

**Quyền:** ADMIN hoặc STAFF tạo phiếu.

Endpoint không nhận body.

### 17.1. Hành vi

- Phiếu phải còn `DRAFT`.
- Dòng phải thuộc phiếu được chỉ định.
- Cho phép xóa dòng của sản phẩm đã ngừng kinh doanh.
- Cho phép xóa dòng cuối cùng, đưa phiếu về trạng thái nháp rỗng.
- Không thay đổi tồn kho.

### 17.2. Response — `204 No Content`

Không có response body.

Gọi lại sau khi dòng đã bị xóa trả `404 STOCK_RECEIPT_ITEM_NOT_FOUND`.

### 17.3. Lỗi nghiệp vụ

| HTTP status | Mã lỗi | Trường hợp |
|---|---|---|
| 403 | `FORBIDDEN` | Không có quyền thao tác |
| 404 | `STOCK_RECEIPT_NOT_FOUND` | Phiếu không tồn tại |
| 404 | `STOCK_RECEIPT_ITEM_NOT_FOUND` | Dòng không tồn tại hoặc không thuộc phiếu |
| 409 | `STOCK_RECEIPT_NOT_DRAFT` | Phiếu không còn là nháp |

## 18. Xác nhận phiếu nhập

```http
POST /api/stock-receipts/{id}/confirm
```

**Quyền:** ADMIN hoặc STAFF tạo phiếu.

Endpoint không nhận body. Client không được gửi người xác nhận, thời điểm xác nhận hoặc số lượng tồn kho.

### 18.1. Xử lý nghiệp vụ

Trong cùng một giao dịch:

1. Khóa bản ghi phiếu nhập.
2. Kiểm tra quyền sở hữu và trạng thái hiện tại.
3. Đọc các dòng mặt hàng.
4. Kiểm tra phiếu không rỗng.
5. Kiểm tra lại nhà cung cấp, sản phẩm và trạng thái kinh doanh.
6. Khóa các bản ghi sản phẩm và tồn kho liên quan theo thứ tự thống nhất.
7. Kiểm tra tồn kho sau nhập không bị tràn số.
8. Cộng tồn kho cho từng sản phẩm.
9. Chuyển phiếu sang `CONFIRMED`.
10. Ghi người xác nhận và thời điểm xác nhận.
11. Commit.

Chỉ trả thành công sau khi commit thành công.

### 18.2. Công thức cập nhật tồn kho

```text
quantity mới = quantity hiện tại + quantity trong dòng phiếu nhập
```

Ví dụ:

```text
Tồn kho hiện tại: 3 laptop
Số lượng trong phiếu: 5 laptop
Tồn kho sau xác nhận: 8 laptop
```

Nếu phiếu có nhiều mặt hàng, phải cập nhật thành công tất cả hoặc rollback toàn bộ.

### 18.3. Response — `200 OK`

Trả chi tiết phiếu với:

```json
{
  "status": "CONFIRMED",
  "confirmedBy": 5,
  "confirmedAt": "2026-11-22T05:00:00.000Z"
}
```

Các trường còn lại theo cấu trúc tại mục 10.2.

### 18.4. Lỗi nghiệp vụ

| HTTP status | Mã lỗi | Trường hợp |
|---|---|---|
| 403 | `FORBIDDEN` | Không có quyền xác nhận |
| 404 | `STOCK_RECEIPT_NOT_FOUND` | Phiếu không tồn tại |
| 409 | `STOCK_RECEIPT_ALREADY_CONFIRMED` | Phiếu đã xác nhận |
| 409 | `STOCK_RECEIPT_CANCELLED` | Phiếu đã hủy |
| 409 | `STOCK_RECEIPT_EMPTY` | Phiếu không có mặt hàng |
| 409 | `PRODUCT_INACTIVE` | Có sản phẩm đã ngừng kinh doanh |
| 409 | `INVENTORY_QUANTITY_OVERFLOW` | Tồn kho sau nhập vượt giới hạn |
| 500 | `INTERNAL_SERVER_ERROR` | Dữ liệu vi phạm bất biến nội bộ, ví dụ thiếu dòng tồn kho |

Theo thiết kế khóa ngoại, nhà cung cấp và sản phẩm được tham chiếu không được biến mất trong vận hành bình thường. Nếu phát hiện dữ liệu tham chiếu bị hỏng, rollback và ghi log lỗi nội bộ.

Không tự tạo bổ sung dòng tồn kho trong lúc xác nhận để che giấu lỗi dữ liệu.

### 18.5. Mất kết nối sau xác nhận

Nếu client không nhận được phản hồi do mất kết nối:

1. Gọi GET chi tiết phiếu để kiểm tra trạng thái.
2. Nếu đã `CONFIRMED`, không cần xác nhận lại.
3. Nếu gửi lại xác nhận cho phiếu đã thành công, API trả `409` và không cộng tồn lần nữa.

Lỗi mạng không đủ để kết luận giao dịch đã thất bại.

## 19. Hủy phiếu nhập

```http
POST /api/stock-receipts/{id}/cancel
```

**Quyền:** ADMIN hoặc STAFF tạo phiếu.

Endpoint không nhận body.

### 19.1. Hành vi

- Khóa phiếu và kiểm tra quyền.
- Nếu `DRAFT`, chuyển sang `CANCELLED`.
- Nếu đã `CANCELLED`, trả dữ liệu hiện tại.
- Nếu `CONFIRMED`, từ chối.
- Giữ nguyên các dòng mặt hàng.
- Không thay đổi tồn kho.

### 19.2. Response — `200 OK`

Trả chi tiết phiếu có trạng thái `CANCELLED`.

### 19.3. Lỗi nghiệp vụ

| HTTP status | Mã lỗi | Trường hợp |
|---|---|---|
| 403 | `FORBIDDEN` | Không có quyền hủy |
| 404 | `STOCK_RECEIPT_NOT_FOUND` | Phiếu không tồn tại |
| 409 | `STOCK_RECEIPT_ALREADY_CONFIRMED` | Phiếu đã xác nhận, không được hủy |

## 20. Quy tắc giao dịch và yêu cầu đồng thời

### 20.1. Khóa phiếu trước khi thay đổi

Tất cả thao tác sau phải khóa cùng bản ghi `stock_receipts` trước khi kiểm tra trạng thái và ghi dữ liệu:

- Đổi nhà cung cấp.
- Thêm, sửa hoặc xóa dòng.
- Xác nhận.
- Hủy.

Có thể sử dụng `SELECT ... FOR UPDATE` trong giao dịch.

Quy tắc này bảo đảm không có thao tác sửa dòng chen vào giữa quá trình xác nhận.

### 20.2. Hai yêu cầu xác nhận cùng phiếu

- Yêu cầu đầu tiên khóa phiếu và thực hiện xác nhận.
- Yêu cầu còn lại chờ khóa.
- Sau khi lấy được khóa, yêu cầu sau phải đọc trạng thái hiện tại.
- Nếu phiếu đã `CONFIRMED`, trả lỗi và không cập nhật tồn kho.

Không được chỉ đọc trạng thái trước giao dịch rồi sử dụng kết quả cũ.

### 20.3. Xác nhận đồng thời với hủy hoặc sửa

Các thao tác được tuần tự hóa qua khóa phiếu:

- Xác nhận thành công trước: thao tác sửa hoặc hủy sau đó bị từ chối.
- Hủy thành công trước: thao tác xác nhận bị từ chối.
- Sửa thành công trước: xác nhận phải dùng dữ liệu sau sửa.

Không cho phép phiếu ở trạng thái `CANCELLED` nhưng tồn kho đã bị cộng từ chính phiếu đó.

### 20.4. Nhiều phiếu nhập cùng một sản phẩm

Phải khóa hoặc cập nhật nguyên tử bản ghi tồn kho để tránh mất cập nhật.

Ví dụ:

```text
Tồn ban đầu: 10
Phiếu A nhập: 5
Phiếu B nhập: 7
Tồn cuối phải là: 22
```

Không được tính cả hai phép cộng từ cùng một giá trị cũ rồi ghi đè kết quả.

### 20.5. Ngừng kinh doanh đồng thời với xác nhận

Việc kiểm tra trạng thái sản phẩm phải được bảo vệ bởi khóa giữ đến cuối giao dịch xác nhận.

- Nếu ngừng kinh doanh hoàn tất trước khi xác nhận kiểm tra sản phẩm, xác nhận bị từ chối.
- Nếu xác nhận đã khóa và kiểm tra sản phẩm hợp lệ trước, thao tác ngừng kinh doanh chờ giao dịch xác nhận kết thúc.

Không sử dụng dữ liệu cache làm căn cứ cuối cùng cho kiểm tra này.

### 20.6. Thứ tự lấy khóa

Quy ước cho xác nhận phiếu:

1. Khóa phiếu.
2. Khóa sản phẩm theo `product_id` tăng dần.
3. Khóa tồn kho theo `product_id` tăng dần.
4. Thực hiện cập nhật.

Các luồng bán hàng và thao tác nhiều sản phẩm phải dùng thứ tự khóa tài nguyên tương thích.

Thứ tự khóa thống nhất giúp giảm deadlock nhưng không loại bỏ hoàn toàn khả năng deadlock.

### 20.7. Rollback và connection

- Service quản lý giao dịch.
- Các repository trong cùng giao dịch phải dùng cùng connection.
- Khi gặp lỗi, rollback trước khi trả connection về pool.
- Không giữ giao dịch mở sau khi request kết thúc.
- Nếu retry do deadlock, chỉ retry có giới hạn toàn bộ giao dịch đã được xác định rollback.
- Không tự retry mù khi chưa biết commit đã thành công hay chưa.

### 20.8. Đọc dữ liệu nhất quán

Chi tiết phiếu, danh sách dòng và tổng tiền trong một response phải phản ánh cùng một ảnh chụp dữ liệu.

Đối với danh sách, dữ liệu và tổng số phiếu phải sử dụng cùng bộ lọc và ảnh chụp nhất quán.

Các request phân trang khác nhau không bảo đảm cùng một ảnh chụp cố định.

## 21. Thứ tự kiểm tra lỗi

Sau khi kiểm tra giao thức và xác thực:

1. Validate path, query và body.
2. Tìm phiếu; trả `404` nếu không tồn tại.
3. Kiểm tra quyền thao tác với phiếu.
4. Kiểm tra trạng thái.
5. Kiểm tra dòng mặt hàng hoặc tài nguyên được tham chiếu.
6. Thực hiện nghiệp vụ.

Với thao tác ghi, các kiểm tra trạng thái và dữ liệu phụ thuộc phải được thực hiện hoặc kiểm tra lại trong giao dịch dưới khóa phù hợp.

Với endpoint dòng mặt hàng, chỉ tra cứu dòng trong phạm vi phiếu được chỉ định.

## 22. Danh sách lỗi chung

| HTTP status | Mã lỗi | Ý nghĩa |
|---|---|---|
| 400 | `INVALID_JSON` | JSON sai cú pháp |
| 400 | `VALIDATION_ERROR` | Path, query hoặc body không hợp lệ |
| 401 | `AUTH_REQUIRED` | Thiếu thông tin xác thực |
| 401 | `INVALID_TOKEN` | Token không hợp lệ hoặc tài khoản không tồn tại |
| 401 | `TOKEN_EXPIRED` | Token hết hạn |
| 403 | `ACCOUNT_DISABLED` | Tài khoản đã bị khóa |
| 403 | `FORBIDDEN` | Không có quyền thao tác |
| 404 | `STOCK_RECEIPT_NOT_FOUND` | Phiếu không tồn tại |
| 404 | `STOCK_RECEIPT_ITEM_NOT_FOUND` | Dòng không tồn tại trong phiếu |
| 404 | `SUPPLIER_NOT_FOUND` | Nhà cung cấp được gửi không tồn tại |
| 404 | `PRODUCT_NOT_FOUND` | Sản phẩm được gửi không tồn tại |
| 409 | `STOCK_RECEIPT_NOT_DRAFT` | Thao tác sửa yêu cầu phiếu nháp |
| 409 | `STOCK_RECEIPT_ALREADY_CONFIRMED` | Phiếu đã xác nhận |
| 409 | `STOCK_RECEIPT_CANCELLED` | Không thể xác nhận phiếu đã hủy |
| 409 | `STOCK_RECEIPT_EMPTY` | Không thể xác nhận phiếu rỗng |
| 409 | `STOCK_RECEIPT_PRODUCT_ALREADY_EXISTS` | Sản phẩm đã xuất hiện trong phiếu |
| 409 | `PRODUCT_INACTIVE` | Sản phẩm đã ngừng kinh doanh |
| 409 | `INVENTORY_QUANTITY_OVERFLOW` | Tồn kho sau nhập vượt giới hạn |
| 413 | `PAYLOAD_TOO_LARGE` | Body vượt giới hạn cấu hình |
| 415 | `UNSUPPORTED_MEDIA_TYPE` | Loại nội dung không được hỗ trợ |
| 500 | `INTERNAL_SERVER_ERROR` | Lỗi nội bộ |

Không trả câu SQL, stack trace hoặc thông tin kết nối cơ sở dữ liệu.

Không ánh xạ mọi lỗi database thành `409`.

## 23. Ánh xạ dữ liệu

### 23.1. Phiếu nhập

| Trường API | Cột hoặc nguồn dữ liệu |
|---|---|
| `id` | `stock_receipts.id` |
| `supplierId` | `stock_receipts.supplier_id` |
| `createdBy` | `stock_receipts.created_by` |
| `status` | `stock_receipts.status` |
| `createdAt` | `stock_receipts.created_at` |
| `confirmedBy` | `stock_receipts.confirmed_by` |
| `confirmedAt` | `stock_receipts.confirmed_at` |
| `itemCount` | Số dòng trong `stock_receipt_items` |
| `totalAmount` | Tổng thành tiền các dòng |

### 23.2. Dòng mặt hàng

| Trường API | Cột hoặc nguồn dữ liệu |
|---|---|
| `id` | `stock_receipt_items.id` |
| `productId` | `stock_receipt_items.product_id` |
| `quantity` | `stock_receipt_items.quantity` |
| `unitCost` | `stock_receipt_items.unit_cost` |
| `lineTotal` | `quantity × unit_cost` |

`receipt_id` lấy từ phiếu trong URL, không lấy từ request body.

Thiết kế hiện tại không có `updated_at` cho phiếu nhập hoặc dòng mặt hàng; API không tự bổ sung trường thời gian này.

## 24. Yêu cầu bảo mật và triển khai

- Dùng truy vấn có tham số.
- Kiểm tra quyền sở hữu phiếu tại backend.
- Không tin `createdBy`, `confirmedBy` hoặc `status` do client gửi.
- Chỉ nhận các trường được cho phép.
- Kiểm tra dòng mặt hàng thuộc đúng phiếu để tránh thao tác chéo tài nguyên.
- Không dùng cache để quyết định trạng thái phiếu, trạng thái sản phẩm hoặc tồn kho khi xác nhận.
- Không thực hiện gọi dịch vụ bên ngoài trong giao dịch xác nhận.
- Không cập nhật giá bán sản phẩm từ giá nhập.
- Không ghi JWT vào log.
- Ghi log lỗi giao dịch và lỗi toàn vẹn dữ liệu để phục vụ điều tra.
- Nếu sử dụng cache tồn kho, phải vô hiệu hóa hoặc cập nhật cache sau commit theo chính sách caching.
- Lỗi cache sau commit không được làm cộng tồn kho lại hoặc giả định giao dịch database chưa thành công.

## 25. Các tình huống kiểm thử

### 25.1. Xác thực và phân quyền

- Không có token, token sai hoặc hết hạn.
- Tài khoản bị khóa.
- ADMIN thao tác trên phiếu do STAFF tạo.
- STAFF sửa, hủy và xác nhận phiếu của mình.
- STAFF không được thay đổi phiếu của người khác.
- Người không có quyền không làm thay đổi dữ liệu.

### 25.2. Tạo và tra cứu phiếu

- Tạo phiếu với nhà cung cấp hợp lệ.
- Nhà cung cấp không tồn tại.
- Gửi trường không được phép.
- Phiếu mới có trạng thái DRAFT, chưa có dòng và chưa có người xác nhận.
- Tạo phiếu không tăng tồn kho.
- Lọc theo trạng thái, nhà cung cấp và người tạo.
- Phân trang rỗng hoặc vượt phạm vi.
- Tổng số phiếu không bị nhân lên do JOIN các dòng.
- Phiếu nháp, đã hủy và đã xác nhận đều tra cứu được.

### 25.3. Đổi nhà cung cấp

- Đổi nhà cung cấp trên phiếu nháp.
- Gửi lại nhà cung cấp hiện tại.
- Nhà cung cấp mới không tồn tại.
- Phiếu không còn là nháp.
- Các dòng, giá nhập và người tạo được giữ nguyên.

### 25.4. Thêm và sửa mặt hàng

- Thêm sản phẩm đang kinh doanh.
- Sản phẩm không tồn tại hoặc đã ngừng kinh doanh.
- Thêm trùng sản phẩm trong cùng phiếu.
- Hai yêu cầu đồng thời thêm cùng sản phẩm.
- Số lượng bằng 0, âm, thập phân, sai kiểu hoặc vượt giới hạn.
- Giá nhập bằng 0, âm, quá lớn hoặc quá 2 chữ số thập phân.
- Sửa một trường hoặc nhiều trường.
- Body rỗng hoặc chứa trường không được phép.
- Gửi `productId` khi sửa bị từ chối.
- Sửa một dòng thuộc phiếu khác bị từ chối.
- Thêm và sửa dòng không thay đổi tồn kho.

### 25.5. Xóa mặt hàng

- Xóa dòng hợp lệ.
- Xóa dòng cuối cùng.
- Xóa dòng có sản phẩm đã ngừng kinh doanh.
- Dòng không tồn tại hoặc thuộc phiếu khác.
- Xóa dòng trên phiếu đã xác nhận hoặc đã hủy bị từ chối.
- Xóa dòng không thay đổi tồn kho.

### 25.6. Xác nhận

- Xác nhận phiếu một mặt hàng và nhiều mặt hàng.
- Tồn kho tăng đúng theo từng dòng.
- Người tạo được tự xác nhận phiếu.
- ADMIN xác nhận không thay đổi người tạo.
- Phiếu rỗng bị từ chối.
- Phiếu đã hủy bị từ chối.
- Sản phẩm bị ngừng sau khi tạo nháp khiến xác nhận bị từ chối.
- Có một dòng không hợp lệ thì không cộng tồn cho bất kỳ dòng nào.
- Thiếu dòng tồn kho phải rollback và báo lỗi nội bộ.
- Tồn kho sau nhập vượt giới hạn phải rollback.
- Lỗi giữa quá trình cập nhật nhiều sản phẩm phải rollback toàn bộ.
- Người xác nhận và thời điểm xác nhận chỉ được lưu khi thành công.

### 25.7. Hủy

- Hủy phiếu nháp có mặt hàng.
- Hủy phiếu nháp rỗng.
- Gọi hủy lại trả thành công.
- Không được hủy phiếu đã xác nhận.
- Hủy không thay đổi tồn kho và không xóa dòng.

### 25.8. Yêu cầu đồng thời

- Hai yêu cầu xác nhận cùng phiếu chỉ cộng tồn một lần.
- Xác nhận đồng thời với sửa dòng.
- Xác nhận đồng thời với xóa dòng cuối cùng.
- Xác nhận đồng thời với hủy.
- Hai phiếu nhập cùng một sản phẩm không mất cập nhật.
- Xác nhận nhập đồng thời với giao dịch bán cùng sản phẩm.
- Ngừng kinh doanh đồng thời với xác nhận.
- Kiểm tra rollback và giải phóng connection khi deadlock hoặc lỗi giao dịch.

### 25.9. Số tiền và phản hồi

- Thành tiền và tổng tiền tính chính xác.
- Số tiền lớn không mất độ chính xác hoặc bị ép về kiểu quá nhỏ.
- Giá trị tiền trả dưới dạng chuỗi có 2 chữ số thập phân.
- Chi tiết phiếu không trộn dữ liệu trước và sau một cập nhật đồng thời.
- Lỗi không lộ SQL hoặc stack trace.

## 26. Các điểm cần đồng bộ và hoàn thiện

### 26.1. Quy tắc đã chốt cần đồng bộ

Cập nhật `business-rules.md` và `catalog.md`:

- Không thêm sản phẩm ngừng kinh doanh vào phiếu nhập.
- Kiểm tra lại trạng thái sản phẩm khi xác nhận.
- Phiếu nháp có sản phẩm ngừng kinh doanh phải xóa dòng đó trước khi xác nhận.
- Phiếu đã xác nhận trước thời điểm ngừng kinh doanh được giữ nguyên.

### 26.2. Các lựa chọn cụ thể trong bản thiết kế này

| Nội dung | Lựa chọn |
|---|---|
| Tạo phiếu | Tạo phiếu rỗng trước, thêm dòng qua API riêng |
| Quyền xem | ADMIN và STAFF được xem danh sách và chi tiết |
| Quyền ghi của STAFF | Chỉ phiếu do chính mình tạo |
| Sửa dòng sản phẩm ngừng kinh doanh | Không cho sửa; cho phép xóa khỏi phiếu nháp |
| Xác nhận lại | Trả 409, không tăng tồn lần nữa |
| Hủy lại | Trả 200, không thay đổi dữ liệu |
| Đổi sản phẩm của dòng | Xóa dòng cũ và thêm dòng mới |
| Giá nhập và tổng tiền | Chuỗi thập phân, tính toán chính xác |
| Lịch sử nhập | Suy ra từ phiếu và dòng đã xác nhận |
| Chống tạo phiếu trùng | Chưa có idempotency key |
| Phát hiện sửa từ dữ liệu cũ | Chưa có version hoặc If-Match |

Đối chiếu các lựa chọn này với tài liệu đã commit trước khi triển khai; nếu khác, cập nhật thống nhất thay vì duy trì hai quy tắc.

### 26.3. Nội dung cần hoàn thiện

1. **Tên giá trị enum:** đối chiếu `DRAFT`, `CONFIRMED`, `CANCELLED` với `document_status` trong database và DBML.
2. **Quy ước lỗi:** thống nhất cấu trúc và mã lỗi với các API đã có.
3. **Giới hạn vận hành:** xác định số dòng tối đa cho một phiếu trước khi triển khai để giới hạn kích thước response và thời gian giao dịch.
4. **Thứ tự khóa:** đồng bộ với `sales.md` và các thao tác cập nhật sản phẩm.
5. **Chính sách retry:** xác định số lần retry và cách phản hồi khi deadlock hoặc hết thời gian chờ khóa.
6. **Cache tồn kho:** quy định cơ chế vô hiệu hóa sau commit và xử lý lỗi cache.
7. **Giới hạn request body:** dùng cấu hình chung tại `architecture.md`.

Các mục chưa hoàn thiện phải được giải quyết trước khi triển khai phần phụ thuộc. Sau khi chốt, cập nhật vào nội dung chính và loại bỏ khỏi danh sách chờ.
