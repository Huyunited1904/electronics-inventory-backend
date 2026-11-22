# Thiết kế API quản lý đơn bán

## 1. Mục đích

Tài liệu mô tả API quản lý đơn bán trong hệ thống quản lý bán hàng và tồn kho đồ điện tử.

Tài liệu là căn cứ để:

- Tạo và quản lý đơn bán ở trạng thái nháp.
- Ghi nhận sản phẩm, số lượng và giá bán.
- Xác định quyền thao tác của ADMIN và STAFF.
- Kiểm tra đủ tồn kho trước khi xác nhận.
- Trừ tồn kho và xác nhận đơn trong cùng một giao dịch.
- Ngăn xác nhận trùng, bán vượt tồn kho và sửa đơn không đúng trạng thái.
- Xây dựng kiểm thử chức năng, phân quyền và yêu cầu đồng thời.

Đường dẫn gốc:

```text
/api/sales-orders
```

## 2. Tài liệu liên quan

| Tài liệu | Nội dung đối chiếu |
|---|---|
| `../scope.md` | Phạm vi chức năng và phân quyền |
| `../business-rules.md` | Quy tắc bán hàng, giá bán và tồn kho |
| `../database.md` | Cấu trúc bảng và ràng buộc dữ liệu |
| `../diagrams/erd.dbml` | Quan hệ giữa đơn bán và các bảng liên quan |
| `../architecture.md` | Kiến trúc xử lý nghiệp vụ và giao dịch |
| `auth.md` | Xác thực JWT |
| `users.md` | Kiểm tra tài khoản và quyền truy cập |
| `catalog.md` | Sản phẩm, giá bán hiện tại và trạng thái kinh doanh |
| `purchasing.md` | Quy ước chứng từ và thứ tự khóa khi cập nhật tồn kho |
| `inventory.md` | Tra cứu tồn kho và lịch sử nhập/xuất; bổ sung sau |

Nếu có mâu thuẫn, phải thống nhất và cập nhật các tài liệu liên quan trước khi triển khai phần bị ảnh hưởng.

## 3. Phạm vi

### 3.1. Chức năng được hỗ trợ

- Xem danh sách đơn bán.
- Xem chi tiết đơn bán.
- Tạo đơn bán nháp.
- Thêm sản phẩm vào đơn nháp.
- Sửa số lượng của dòng sản phẩm.
- Xóa dòng sản phẩm khỏi đơn nháp.
- Hủy đơn nháp.
- Xác nhận đơn bán và trừ tồn kho.

### 3.2. Chức năng ngoài phạm vi

- Xóa đơn bán khỏi cơ sở dữ liệu.
- Sửa hoặc hủy đơn đã xác nhận.
- Khôi phục đơn đã hủy.
- Trả hàng, hoàn tiền hoặc hoàn tồn.
- Quản lý khách hàng.
- Thanh toán trực tuyến hoặc trạng thái thanh toán.
- Vận chuyển và giao hàng.
- Thuế, phí, mã giảm giá hoặc chiết khấu.
- Giữ hàng cho đơn nháp.
- Đặt trước hoặc bán vượt tồn kho.
- Sửa giá bán thủ công trên dòng đơn.
- Quản lý nhiều kho, serial hoặc IMEI.

`CONFIRMED` trong tài liệu này có nghĩa là giao dịch bán hàng đã được ghi nhận và tồn kho đã giảm thành công. Trạng thái này không đại diện cho xác nhận thanh toán từ một hệ thống thanh toán.

## 4. Mô hình dữ liệu

### 4.1. Đơn bán

Bảng `sales_orders` lưu:

- Người tạo.
- Trạng thái.
- Thời điểm tạo.
- Người xác nhận.
- Thời điểm xác nhận.

### 4.2. Dòng sản phẩm

Bảng `sales_order_items` lưu:

- Đơn bán liên quan.
- Sản phẩm.
- Số lượng bán.
- Giá bán của một đơn vị tại thời điểm thêm dòng.

Quan hệ:

```text
Một đơn bán → Nhiều dòng sản phẩm
Một dòng sản phẩm → Một sản phẩm
```

Một sản phẩm chỉ xuất hiện một lần trong cùng đơn bán.

### 4.3. Tồn kho

Bảng `inventory` lưu số lượng hiện tại của từng sản phẩm.

- Tạo đơn nháp không giữ hoặc trừ tồn kho.
- Thêm, sửa, xóa dòng trong đơn nháp không thay đổi tồn kho.
- Hủy đơn nháp không thay đổi tồn kho.
- Chỉ xác nhận thành công mới trừ tồn kho.

Thiết kế hiện tại không dùng bảng `stock_movements`. Lịch sử xuất bán được suy ra từ đơn bán và các dòng đã xác nhận.

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
- Kiểm tra quyền thao tác trên đơn bán.

### 5.2. Bảng phân quyền

| Chức năng | ADMIN | STAFF |
|---|---|---|
| Xem danh sách và chi tiết | Được phép | Được phép |
| Tạo đơn nháp | Được phép | Được phép |
| Thêm, sửa, xóa dòng | Mọi đơn nháp | Chỉ đơn nháp do mình tạo |
| Hủy đơn | Mọi đơn nháp | Chỉ đơn nháp do mình tạo |
| Xác nhận đơn | Mọi đơn nháp | Chỉ đơn nháp do mình tạo |

Quyền xem không đồng nghĩa với quyền thay đổi đơn.

### 5.3. Người tạo và người xác nhận

- `createdBy` lấy từ tài khoản tạo đơn.
- `confirmedBy` lấy từ tài khoản xác nhận thành công.
- Client không được gửi hai trường này.
- Người tạo được phép tự xác nhận đơn nếu đáp ứng các điều kiện nghiệp vụ.
- ADMIN xác nhận đơn do STAFF tạo không làm thay đổi `createdBy`.

## 6. Trạng thái đơn bán

### 6.1. Các trạng thái

| Trạng thái | Ý nghĩa | Được sửa | Tác động tồn kho |
|---|---|:---:|---|
| `DRAFT` | Đơn nháp | Có, theo phân quyền | Không |
| `CONFIRMED` | Đã xác nhận bán hàng | Không | Đã giảm tồn kho |
| `CANCELLED` | Đã hủy | Không | Không |

### 6.2. Chuyển trạng thái hợp lệ

```text
DRAFT → CONFIRMED
DRAFT → CANCELLED
```

Không hỗ trợ chuyển từ `CONFIRMED` hoặc `CANCELLED` về trạng thái khác.

### 6.3. Xác nhận lại

Gọi xác nhận đơn đã `CONFIRMED`:

- Trả `409 SALES_ORDER_ALREADY_CONFIRMED`.
- Không trừ tồn lần nữa.
- Không thay đổi người và thời điểm xác nhận ban đầu.

### 6.4. Hủy lại

Gọi hủy đơn đã `CANCELLED`:

- Kiểm tra quyền thao tác như bình thường.
- Trả `200 OK` với dữ liệu hiện tại.
- Không thay đổi dữ liệu.

## 7. Quy ước dữ liệu

### 7.1. Quy ước chung

- Request và response sử dụng JSON.
- Request có body phải gửi `Content-Type: application/json`.
- Tên trường API sử dụng `camelCase`.
- Thời gian theo ISO 8601, múi giờ UTC.
- ID là số nguyên từ `1` đến `2147483647`.
- Không chấp nhận trường ngoài danh sách cho phép.
- Không tự chuyển chuỗi thành số trong JSON body.
- Không chấp nhận body dạng mảng, chuỗi hoặc `null`.
- Query parameter không được lặp lại.
- Query parameter được gửi nhưng rỗng là không hợp lệ.
- Endpoint không mô tả query parameter thì không nhận query parameter.

### 7.2. Số lượng

`quantity` phải:

- Là số nguyên JSON.
- Lớn hơn `0`.
- Không vượt quá `2147483647`.

Hợp lệ:

```json
{
  "quantity": 2
}
```

Không hợp lệ:

```json
{
  "quantity": "2"
}
```

Không sử dụng số lượng `0` để xóa dòng; phải gọi endpoint DELETE.

### 7.3. Giá bán

`unitPrice` được lưu trong `sales_order_items.unit_price`, tương ứng với `DECIMAL(15,2)`.

Quy định:

- Lấy từ `products.sale_price` khi thêm dòng thành công.
- Không âm.
- Response trả chuỗi thập phân có đúng 2 chữ số thập phân.
- Không nhận `unitPrice` từ client.
- Không chuyển giá qua số thực JavaScript gây mất độ chính xác.

Ví dụ:

```json
{
  "unitPrice": "15990000.00"
}
```

### 7.4. Thành tiền và tổng giá trị

```text
lineTotal = quantity × unitPrice
totalAmount = Tổng lineTotal của các dòng
```

Quy định:

- Tính từ dữ liệu đã lưu trên các dòng.
- Không lấy giá hiện tại trong `products` để tính lại giá trị đơn.
- Không lưu thêm cột tổng tiền trong thiết kế hiện tại.
- Không nhận tổng tiền do client gửi.
- Trả các giá trị tiền dưới dạng chuỗi có 2 chữ số thập phân.
- Sử dụng phép toán thập phân chính xác.
- Tổng tiền có thể lớn hơn giới hạn của một giá đơn vị; không ép về kiểu `DECIMAL(15,2)` gây tràn.
- Đơn rỗng có tổng tiền `"0.00"`.

## 8. Quy tắc nghiệp vụ

### 8.1. Tạo đơn nháp

- Đơn mới có trạng thái `DRAFT`.
- Chưa có dòng sản phẩm.
- `confirmedBy` và `confirmedAt` bằng `null`.
- Không giữ hoặc trừ tồn kho.

### 8.2. Thêm sản phẩm

- Đơn phải còn `DRAFT`.
- Sản phẩm phải tồn tại và đang kinh doanh.
- Sản phẩm chưa có trong đơn.
- Số lượng phải hợp lệ.
- Giá bán được lấy từ cơ sở dữ liệu khi thêm dòng.
- Không tự cộng số lượng khi sản phẩm đã có.
- Không trừ hoặc giữ tồn kho.

Ràng buộc UNIQUE áp dụng cho:

```text
(order_id, product_id)
```

### 8.3. Chính sách tồn kho của đơn nháp

Trong bản thiết kế này, được phép lưu số lượng trên đơn nháp lớn hơn tồn kho hiện tại.

Lý do: đơn nháp không giữ hàng và tồn kho có thể thay đổi trước khi xác nhận.

Do đó:

- Thêm hoặc sửa dòng không bị từ chối chỉ vì thiếu tồn.
- Có thể thêm sản phẩm đang kinh doanh nhưng tồn bằng `0`.
- Kiểm tra đủ tồn mang tính quyết định được thực hiện khi xác nhận, trong giao dịch.
- Người dùng có thể tra cứu tồn kho qua API inventory để điều chỉnh đơn trước khi xác nhận.

Một đơn được tạo khi còn hàng vẫn có thể thiếu hàng lúc xác nhận vì đơn khác đã bán trước.

### 8.4. Lưu giá bán tại thời điểm thêm dòng

Ví dụ:

```text
Giá sản phẩm khi thêm vào đơn: 15.990.000
Giá sản phẩm được sửa sau đó: 16.490.000
Giá đã lưu trong dòng đơn: vẫn là 15.990.000
```

Quy định:

- Sửa giá sản phẩm không cập nhật giá trong dòng đơn đã có.
- Sửa số lượng không cập nhật lại giá.
- Xác nhận không lấy lại giá hiện tại để thay giá đã lưu.
- Muốn lấy giá hiện tại cho một dòng nháp, phải xóa dòng rồi thêm lại sản phẩm.
- Khi thêm lại, dòng mới có ID mới và lấy giá hiện tại tại thời điểm thêm.
- Không có API sửa giá trực tiếp hoặc tự động cập nhật lại toàn bộ giá của đơn.

### 8.5. Sản phẩm ngừng kinh doanh

Quy tắc đã thống nhất:

- Không thêm sản phẩm ngừng kinh doanh vào bất kỳ đơn nháp nào.
- Nếu sản phẩm ngừng kinh doanh sau khi đã được thêm, dòng vẫn được giữ để tra cứu.
- Không xác nhận đơn có bất kỳ dòng nào chứa sản phẩm ngừng kinh doanh.
- Phải xóa dòng đó trước khi xác nhận.
- Đơn phải còn ít nhất một dòng hợp lệ.
- Đơn đã xác nhận trước thời điểm sản phẩm ngừng kinh doanh được giữ nguyên.
- Không hoàn tồn hoặc thay đổi giá đã lưu của đơn đã xác nhận.

Trong phiên bản này, không cho sửa số lượng của dòng thuộc sản phẩm đã ngừng kinh doanh; cho phép xóa dòng khỏi đơn nháp.

### 8.6. Điều kiện xác nhận

Chỉ xác nhận khi:

- Người gọi có quyền.
- Đơn đang `DRAFT`.
- Có ít nhất một dòng.
- Tất cả sản phẩm tồn tại và đang kinh doanh.
- Các dòng có số lượng và giá đã lưu hợp lệ.
- Mỗi sản phẩm có dòng tồn kho tương ứng.
- Tất cả sản phẩm đều đủ tồn.

Nếu một điều kiện không thỏa mãn:

- Không xác nhận đơn.
- Không trừ tồn kho của bất kỳ dòng nào.
- Không ghi người hoặc thời điểm xác nhận.
- Giữ nguyên đơn ở trạng thái nháp.

### 8.7. Hủy đơn

- Chỉ chuyển đơn nháp sang `CANCELLED`.
- Đơn rỗng được phép hủy.
- Giữ lại các dòng để tra cứu.
- Không hoàn tồn vì đơn nháp chưa trừ tồn.
- Không xóa đơn khỏi cơ sở dữ liệu.

Thiết kế hiện tại không có người hủy, thời điểm hủy hoặc lý do hủy. API không nhận hoặc trả các trường này.

## 9. Danh sách endpoint

| Method | Endpoint | Chức năng |
|---|---|---|
| GET | `/api/sales-orders` | Xem danh sách đơn |
| GET | `/api/sales-orders/{id}` | Xem chi tiết đơn |
| POST | `/api/sales-orders` | Tạo đơn nháp |
| POST | `/api/sales-orders/{id}/items` | Thêm sản phẩm |
| PATCH | `/api/sales-orders/{id}/items/{itemId}` | Sửa số lượng |
| DELETE | `/api/sales-orders/{id}/items/{itemId}` | Xóa dòng |
| POST | `/api/sales-orders/{id}/confirm` | Xác nhận bán hàng |
| POST | `/api/sales-orders/{id}/cancel` | Hủy đơn |

Không có endpoint sửa phần thông tin chung của đơn vì thiết kế hiện tại chưa có trường nghiệp vụ cho người dùng chỉnh sửa ở phần này.

Không có endpoint cập nhật trực tiếp `status`, `createdBy`, `confirmedBy` hoặc thời gian.

## 10. Cấu trúc phản hồi

### 10.1. Đơn trong danh sách

```json
{
  "id": 1001,
  "createdBy": 5,
  "status": "DRAFT",
  "createdAt": "2026-11-22T03:00:00.000Z",
  "confirmedBy": null,
  "confirmedAt": null,
  "itemCount": 2,
  "totalAmount": "32480000.00"
}
```

`itemCount` là số dòng, không phải tổng số lượng bán.

### 10.2. Chi tiết đơn

```json
{
  "data": {
    "id": 1001,
    "createdBy": 5,
    "status": "DRAFT",
    "createdAt": "2026-11-22T03:00:00.000Z",
    "confirmedBy": null,
    "confirmedAt": null,
    "itemCount": 2,
    "totalAmount": "32480000.00",
    "items": [
      {
        "id": 2001,
        "productId": 101,
        "quantity": 2,
        "unitPrice": "15990000.00",
        "lineTotal": "31980000.00"
      },
      {
        "id": 2002,
        "productId": 102,
        "quantity": 2,
        "unitPrice": "250000.00",
        "lineTotal": "500000.00"
      }
    ]
  }
}
```

Các dòng sắp xếp theo `id ASC`.

Response dùng ID tham chiếu sản phẩm và người dùng. Thông tin chi tiết của các tài nguyên này được lấy qua API tương ứng.

### 10.3. Dòng sản phẩm

```json
{
  "data": {
    "id": 2001,
    "productId": 101,
    "quantity": 2,
    "unitPrice": "15990000.00",
    "lineTotal": "31980000.00"
  }
}
```

### 10.4. Phản hồi lỗi

```json
{
  "error": {
    "code": "INSUFFICIENT_STOCK",
    "message": "Không đủ tồn kho để xác nhận đơn bán.",
    "details": [
      {
        "field": "items",
        "message": "Sản phẩm có ID 101 yêu cầu 2, tồn kho hiện tại là 1."
      }
    ]
  }
}
```

Trường `details` không bắt buộc.

Thông tin tồn trong lỗi phản ánh thời điểm kiểm tra; tồn kho có thể thay đổi sau khi giao dịch kết thúc.

## 11. Xem danh sách đơn bán

```http
GET /api/sales-orders
```

**Quyền:** ADMIN, STAFF.

### 11.1. Query parameters

| Trường | Kiểu | Mặc định | Quy định |
|---|---|---|---|
| `page` | Integer | `1` | Từ `1` đến `2147483647` |
| `pageSize` | Integer | `20` | Từ `1` đến `100` |
| `status` | String | Không lọc | `DRAFT`, `CONFIRMED`, `CANCELLED` |
| `createdBy` | Integer | Không lọc | ID hợp lệ |

Quy định:

- Query số phải có dạng chữ số thập phân; không nhận số mũ hoặc số thập phân.
- Các điều kiện lọc kết hợp bằng AND.
- Thứ tự mặc định: `id DESC`.
- Bỏ qua trạng thái để xem mọi trạng thái.
- `createdBy` hợp lệ nhưng không tồn tại trả danh sách rỗng.
- Chưa hỗ trợ tìm kiếm tự do, lọc theo ngày hoặc tùy chọn sắp xếp.

### 11.2. Response — `200 OK`

```json
{
  "data": [
    {
      "id": 1001,
      "createdBy": 5,
      "status": "DRAFT",
      "createdAt": "2026-11-22T03:00:00.000Z",
      "confirmedBy": null,
      "confirmedAt": null,
      "itemCount": 2,
      "totalAmount": "32480000.00"
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

- Đếm số đơn, không đếm số dòng sau khi JOIN.
- Không có kết quả: trả `data: []`, `totalItems: 0`, `totalPages: 0`.
- Trang vượt phạm vi: trả `200 OK` và danh sách rỗng.
- Tổng tiền dùng giá đã lưu trên dòng đơn.

## 12. Xem chi tiết đơn bán

```http
GET /api/sales-orders/{id}
```

**Quyền:** ADMIN, STAFF.

Trả `200 OK` theo mục 10.2.

Cho phép xem cả đơn nháp, đã xác nhận và đã hủy.

Đơn rỗng có:

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
| 404 | `SALES_ORDER_NOT_FOUND` | Đơn không tồn tại |

## 13. Tạo đơn bán nháp

```http
POST /api/sales-orders
```

**Quyền:** ADMIN, STAFF.

### 13.1. Request body

Gửi đối tượng JSON rỗng:

```json
{}
```

Không chấp nhận bất kỳ trường nào, bao gồm:

- `items`
- `status`
- `createdBy`
- `confirmedBy`
- `createdAt`
- `confirmedAt`
- `totalAmount`

Việc thêm sản phẩm thực hiện qua endpoint riêng.

### 13.2. Response — `201 Created`

Header:

```http
Location: /api/sales-orders/1001
```

```json
{
  "data": {
    "id": 1001,
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

### 13.3. Gửi lại yêu cầu tạo

Endpoint chưa hỗ trợ idempotency key.

Gửi lại POST có thể tạo đơn nháp khác với ID mới.

Cơ chế chống xác nhận hai lần cho cùng một đơn không ngăn việc tạo và xác nhận hai đơn khác nhau có cùng nội dung.

## 14. Thêm sản phẩm vào đơn nháp

```http
POST /api/sales-orders/{id}/items
```

**Quyền:** ADMIN hoặc STAFF tạo đơn.

### 14.1. Request body

```json
{
  "productId": 101,
  "quantity": 2
}
```

| Trường | Kiểu | Bắt buộc | Quy định |
|---|---|:---:|---|
| `productId` | Integer | Có | Sản phẩm tồn tại và đang kinh doanh |
| `quantity` | Integer | Có | Theo mục 7.2 |

Không nhận `unitPrice`, `lineTotal`, `orderId` hoặc ID dòng.

### 14.2. Xử lý nghiệp vụ

Trong một giao dịch:

1. Khóa đơn và kiểm tra quyền.
2. Kiểm tra đơn còn `DRAFT`.
3. Kiểm tra sản phẩm chưa có trong đơn.
4. Đọc và khóa sản phẩm để kiểm tra trạng thái, lấy giá bán nhất quán.
5. Tạo dòng với giá vừa đọc.
6. Commit.

Không cập nhật tồn kho.

### 14.3. Response — `201 Created`

Trả dòng được tạo theo mục 10.3.

### 14.4. Lỗi nghiệp vụ

| HTTP status | Mã lỗi | Trường hợp |
|---|---|---|
| 403 | `FORBIDDEN` | Không có quyền thao tác |
| 404 | `SALES_ORDER_NOT_FOUND` | Đơn không tồn tại |
| 404 | `PRODUCT_NOT_FOUND` | Sản phẩm không tồn tại |
| 409 | `SALES_ORDER_NOT_DRAFT` | Đơn không còn là nháp |
| 409 | `SALES_ORDER_PRODUCT_ALREADY_EXISTS` | Sản phẩm đã có trong đơn |
| 409 | `PRODUCT_INACTIVE` | Sản phẩm đã ngừng kinh doanh |

Không trả `INSUFFICIENT_STOCK` ở bước thêm dòng vì đơn nháp chưa giữ hàng.

## 15. Sửa số lượng dòng sản phẩm

```http
PATCH /api/sales-orders/{id}/items/{itemId}
```

**Quyền:** ADMIN hoặc STAFF tạo đơn.

### 15.1. Request body

```json
{
  "quantity": 3
}
```

Chỉ chấp nhận trường `quantity`.

Đây là số lượng mới của dòng, không phải số lượng cộng thêm.

### 15.2. Hành vi

- Đơn phải còn `DRAFT`.
- Dòng phải thuộc đúng đơn trong URL.
- Sản phẩm của dòng phải đang kinh doanh.
- Không thay đổi `productId`.
- Không lấy lại giá hiện tại của sản phẩm.
- Giữ nguyên `unitPrice` đã lưu.
- Tính lại thành tiền theo số lượng mới.
- Không kiểm tra đủ tồn như điều kiện bắt buộc ở bước này.
- Không giữ hoặc trừ tồn.
- Gửi lại số lượng hiện tại vẫn trả thành công.

### 15.3. Response — `200 OK`

```json
{
  "data": {
    "id": 2001,
    "productId": 101,
    "quantity": 3,
    "unitPrice": "15990000.00",
    "lineTotal": "47970000.00"
  }
}
```

### 15.4. Lỗi nghiệp vụ

| HTTP status | Mã lỗi | Trường hợp |
|---|---|---|
| 400 | `VALIDATION_ERROR` | Body, ID hoặc số lượng không hợp lệ |
| 403 | `FORBIDDEN` | Không có quyền thao tác |
| 404 | `SALES_ORDER_NOT_FOUND` | Đơn không tồn tại |
| 404 | `SALES_ORDER_ITEM_NOT_FOUND` | Dòng không tồn tại hoặc không thuộc đơn |
| 409 | `SALES_ORDER_NOT_DRAFT` | Đơn không còn là nháp |
| 409 | `PRODUCT_INACTIVE` | Sản phẩm đã ngừng kinh doanh |

Không được chỉ tìm dòng theo `itemId` rồi bỏ qua kiểm tra `order_id`.

## 16. Xóa dòng sản phẩm

```http
DELETE /api/sales-orders/{id}/items/{itemId}
```

**Quyền:** ADMIN hoặc STAFF tạo đơn.

Endpoint không nhận body.

### 16.1. Hành vi

- Đơn phải còn `DRAFT`.
- Dòng phải thuộc đơn được chỉ định.
- Cho phép xóa dòng có sản phẩm ngừng kinh doanh.
- Cho phép xóa dòng cuối cùng.
- Không thay đổi tồn kho.
- Không thay đổi giá trên các dòng còn lại.

### 16.2. Response — `204 No Content`

Không có response body.

Gọi lại sau khi xóa trả `404 SALES_ORDER_ITEM_NOT_FOUND`.

### 16.3. Lỗi nghiệp vụ

| HTTP status | Mã lỗi | Trường hợp |
|---|---|---|
| 403 | `FORBIDDEN` | Không có quyền thao tác |
| 404 | `SALES_ORDER_NOT_FOUND` | Đơn không tồn tại |
| 404 | `SALES_ORDER_ITEM_NOT_FOUND` | Dòng không tồn tại hoặc không thuộc đơn |
| 409 | `SALES_ORDER_NOT_DRAFT` | Đơn không còn là nháp |

## 17. Xác nhận đơn bán

```http
POST /api/sales-orders/{id}/confirm
```

**Quyền:** ADMIN hoặc STAFF tạo đơn.

Endpoint không nhận body.

Client không được gửi giá bán, tổng tiền, người xác nhận, thời điểm xác nhận hoặc số lượng tồn kho.

### 17.1. Quy trình xác nhận

Trong cùng một giao dịch:

1. Khóa bản ghi đơn bán.
2. Kiểm tra quyền và trạng thái hiện tại.
3. Đọc các dòng sản phẩm.
4. Kiểm tra đơn không rỗng.
5. Khóa các sản phẩm theo ID tăng dần.
6. Kiểm tra tất cả sản phẩm đang kinh doanh.
7. Khóa các dòng tồn kho theo ID sản phẩm tăng dần.
8. Kiểm tra toàn bộ sản phẩm đủ tồn.
9. Trừ tồn kho cho từng sản phẩm.
10. Chuyển đơn sang `CONFIRMED`.
11. Ghi người xác nhận và thời điểm xác nhận.
12. Commit.

Chỉ trả thành công sau khi commit thành công.

Giá bán sử dụng là giá đã lưu trong `sales_order_items.unit_price`.

### 17.2. Điều kiện và công thức tồn kho

Với mỗi dòng:

```text
quantity tồn hiện tại >= quantity cần bán
```

Sau xác nhận:

```text
quantity tồn mới = quantity tồn hiện tại - quantity cần bán
```

Ví dụ:

```text
Tồn hiện tại: 5 laptop
Số lượng bán: 2 laptop
Tồn sau xác nhận: 3 laptop
```

Bán đúng bằng tồn hiện tại được phép và đưa tồn về `0`.

### 17.3. Thiếu tồn kho

Nếu có bất kỳ mặt hàng nào thiếu:

- Trả `409 INSUFFICIENT_STOCK`.
- Không trừ tồn các mặt hàng còn đủ.
- Không xác nhận đơn một phần.
- Giữ nguyên trạng thái `DRAFT`.
- Người dùng có thể giảm số lượng, xóa dòng, chờ nhập thêm hoặc hủy đơn.

Ví dụ:

```json
{
  "error": {
    "code": "INSUFFICIENT_STOCK",
    "message": "Không đủ tồn kho để xác nhận đơn bán.",
    "details": [
      {
        "field": "items",
        "message": "Sản phẩm có ID 101 yêu cầu 2, tồn kho hiện tại là 1."
      },
      {
        "field": "items",
        "message": "Sản phẩm có ID 102 yêu cầu 3, tồn kho hiện tại là 0."
      }
    ]
  }
}
```

Khi phát hiện thiếu tồn, trả thông tin các dòng thiếu đã được kiểm tra trong cùng giao dịch, sắp xếp theo `productId` tăng dần.

### 17.4. Response — `200 OK`

Trả chi tiết đơn theo mục 10.2, với:

```json
{
  "status": "CONFIRMED",
  "confirmedBy": 5,
  "confirmedAt": "2026-11-22T05:00:00.000Z"
}
```

Các giá trị số lượng và giá trên dòng được giữ nguyên.

### 17.5. Lỗi nghiệp vụ

| HTTP status | Mã lỗi | Trường hợp |
|---|---|---|
| 403 | `FORBIDDEN` | Không có quyền xác nhận |
| 404 | `SALES_ORDER_NOT_FOUND` | Đơn không tồn tại |
| 409 | `SALES_ORDER_ALREADY_CONFIRMED` | Đơn đã xác nhận |
| 409 | `SALES_ORDER_CANCELLED` | Đơn đã hủy |
| 409 | `SALES_ORDER_EMPTY` | Đơn không có dòng |
| 409 | `PRODUCT_INACTIVE` | Có sản phẩm ngừng kinh doanh |
| 409 | `INSUFFICIENT_STOCK` | Có sản phẩm không đủ tồn |
| 500 | `INTERNAL_SERVER_ERROR` | Vi phạm bất biến dữ liệu nội bộ |

Ví dụ lỗi dữ liệu nội bộ:

- Sản phẩm được tham chiếu không tồn tại dù có khóa ngoại.
- Thiếu dòng tồn kho của sản phẩm.
- Số lượng hoặc giá đã lưu vi phạm ràng buộc.
- Tồn kho đang âm.

Những trường hợp này phải rollback và ghi log. Không tự tạo dòng tồn hoặc tự sửa số lượng để tiếp tục bán.

### 17.6. Mất kết nối hoặc không nhận được phản hồi

Client cần gọi GET chi tiết đơn để kiểm tra trạng thái.

- Nếu đã `CONFIRMED`, giao dịch đã được ghi nhận.
- Gửi lại xác nhận cho cùng đơn không được trừ tồn lần nữa.
- Không kết luận giao dịch thất bại chỉ dựa vào lỗi mạng.
- Không tự tạo một đơn mới để thay thế khi chưa xác định kết quả đơn cũ.

## 18. Hủy đơn bán

```http
POST /api/sales-orders/{id}/cancel
```

**Quyền:** ADMIN hoặc STAFF tạo đơn.

Endpoint không nhận body.

### 18.1. Hành vi

- Khóa đơn và kiểm tra quyền.
- Nếu `DRAFT`, chuyển sang `CANCELLED`.
- Nếu đã `CANCELLED`, trả dữ liệu hiện tại.
- Nếu `CONFIRMED`, từ chối.
- Giữ các dòng sản phẩm và giá đã lưu.
- Không tăng hoặc giảm tồn kho.
- `confirmedBy` và `confirmedAt` tiếp tục bằng `null`.

### 18.2. Response — `200 OK`

Trả chi tiết đơn có trạng thái `CANCELLED`.

### 18.3. Lỗi nghiệp vụ

| HTTP status | Mã lỗi | Trường hợp |
|---|---|---|
| 403 | `FORBIDDEN` | Không có quyền hủy |
| 404 | `SALES_ORDER_NOT_FOUND` | Đơn không tồn tại |
| 409 | `SALES_ORDER_ALREADY_CONFIRMED` | Đơn đã xác nhận, không được hủy |

## 19. Giao dịch và yêu cầu đồng thời

### 19.1. Khóa đơn trước khi thay đổi

Các thao tác sau phải khóa bản ghi `sales_orders` trước khi kiểm tra trạng thái và ghi dữ liệu:

- Thêm dòng.
- Sửa số lượng.
- Xóa dòng.
- Xác nhận.
- Hủy.

Có thể sử dụng `SELECT ... FOR UPDATE` trong giao dịch.

Không cho phép thao tác sửa dòng chen vào quá trình xác nhận.

### 19.2. Hai yêu cầu xác nhận cùng đơn

- Yêu cầu đầu tiên khóa đơn và xử lý.
- Yêu cầu còn lại chờ khóa.
- Sau khi nhận khóa, phải đọc trạng thái hiện tại.
- Nếu đơn đã xác nhận, trả `409` và không trừ tồn.

Không dựa vào trạng thái đã đọc trước khi lấy khóa.

### 19.3. Hai đơn bán cùng sản phẩm

Ví dụ:

```text
Tồn ban đầu: 5
Đơn A muốn bán: 4
Đơn B muốn bán: 3
```

Không được xác nhận thành công cả hai nếu không có nhập hàng xen giữa.

Yêu cầu lấy được khóa tồn kho trước có thể thành công. Yêu cầu sau phải kiểm tra số tồn mới sau khi lấy khóa và bị từ chối nếu không đủ.

Tồn kho không được âm.

### 19.4. Xác nhận đồng thời với nhập hàng

Luồng nhập và bán phải sử dụng khóa tồn kho và thứ tự khóa tương thích.

- Nếu nhập hoàn tất trước, bán kiểm tra trên tồn đã tăng.
- Nếu bán kiểm tra trước lúc nhập hoàn tất, bán có thể bị từ chối vì thiếu tồn.
- Không được ghi đè làm mất số lượng từ giao dịch khác.

Một lần xác nhận bị từ chối vì thiếu tồn có thể được thực hiện lại sau khi có hàng nhập.

### 19.5. Xác nhận đồng thời với sửa hoặc hủy

Các thao tác được tuần tự hóa qua khóa đơn:

- Sửa hoàn tất trước: xác nhận sử dụng dữ liệu đã sửa.
- Xác nhận hoàn tất trước: sửa hoặc hủy bị từ chối.
- Hủy hoàn tất trước: xác nhận bị từ chối.

Không cho phép đơn đã hủy nhưng tồn kho vẫn bị trừ từ chính đơn đó.

### 19.6. Ngừng kinh doanh đồng thời

Việc kiểm tra trạng thái sản phẩm phải được bảo vệ bởi khóa giữ đến hết giao dịch.

- Ngừng kinh doanh hoàn tất trước khi xác nhận kiểm tra: xác nhận bị từ chối.
- Xác nhận đã khóa và kiểm tra sản phẩm hợp lệ trước: thao tác ngừng kinh doanh chờ giao dịch kết thúc.

Quy tắc tương tự áp dụng khi thêm dòng hoặc sửa số lượng để kiểm tra trạng thái nhất quán.

### 19.7. Cập nhật giá đồng thời với thêm dòng

Khi thêm dòng, đọc giá và trạng thái sản phẩm từ cùng một bản ghi được khóa.

Giá lưu vào dòng phải tương ứng với giá sản phẩm tại lần đọc được bảo vệ đó.

Sau khi dòng được tạo:

- Cập nhật giá sản phẩm không sửa dòng cũ.
- Xác nhận đơn không thay giá đã lưu.

### 19.8. Thứ tự khóa

Quy ước:

1. Khóa đơn bán.
2. Khóa các sản phẩm theo `product_id` tăng dần.
3. Khóa tồn kho theo `product_id` tăng dần.
4. Thực hiện cập nhật.

Đồng bộ quy ước này với luồng xác nhận phiếu nhập.

Thứ tự khóa giúp giảm deadlock, không bảo đảm loại bỏ hoàn toàn deadlock.

### 19.9. Cập nhật tồn kho

Sau khi đã khóa và kiểm tra, có thể dùng cập nhật có điều kiện để bảo vệ thêm:

```sql
UPDATE inventory
SET quantity = quantity - ?,
    updated_at = ?
WHERE product_id = ?
  AND quantity >= ?;
```

Phải kiểm tra kết quả cập nhật. Nếu bất kỳ dòng nào không cập nhật được như dự kiến, rollback toàn bộ.

Câu lệnh cập nhật có điều kiện không thay thế giao dịch bao trùm toàn bộ đơn.

### 19.10. Rollback và connection

- Service quản lý giao dịch.
- Các repository tham gia phải dùng cùng connection.
- Rollback khi có lỗi.
- Trả connection về pool sau khi kết thúc giao dịch.
- Không gọi dịch vụ bên ngoài khi đang giữ giao dịch.
- Chỉ retry có giới hạn khi đã xác định giao dịch trước rollback.
- Không retry mù khi chưa biết kết quả commit.

### 19.11. Cập nhật đơn nháp từ dữ liệu cũ

Phiên bản hiện tại chưa dùng version hoặc `If-Match`.

Nếu hai request cùng sửa số lượng của một dòng:

- Các thao tác được tuần tự hóa bằng khóa.
- Request thực hiện sau có thể thay số lượng bằng giá trị của nó.
- Backend chưa phát hiện người dùng đang sửa dựa trên dữ liệu cũ.

Khóa giao dịch bảo vệ tính nhất quán nhưng không thay thế cơ chế phát hiện xung đột chỉnh sửa của người dùng.

### 19.12. Đọc nhất quán

Chi tiết đơn, các dòng và tổng tiền trong một response phải lấy từ cùng ảnh chụp dữ liệu.

Danh sách và tổng số đơn cũng phải sử dụng cùng bộ lọc và ảnh chụp nhất quán.

Các request phân trang khác nhau không bảo đảm ảnh chụp cố định.

## 20. Thứ tự kiểm tra lỗi

Sau kiểm tra giao thức và xác thực:

1. Validate path, query và body.
2. Tìm đơn.
3. Kiểm tra quyền thao tác.
4. Kiểm tra trạng thái.
5. Kiểm tra dòng hoặc sản phẩm tham chiếu.
6. Kiểm tra điều kiện nghiệp vụ.
7. Ghi dữ liệu.

Với thao tác ghi, các kiểm tra phụ thuộc dữ liệu phải được thực hiện hoặc kiểm tra lại trong giao dịch dưới khóa phù hợp.

Khi xác nhận, ưu tiên:

1. Trạng thái đơn.
2. Đơn rỗng.
3. Tính hợp lệ của dữ liệu và trạng thái kinh doanh sản phẩm.
4. Đủ tồn kho.

Nếu đơn vừa có sản phẩm ngừng kinh doanh vừa có sản phẩm thiếu tồn, trả lỗi sản phẩm ngừng kinh doanh trước.

Dòng không thuộc đơn trong URL được xử lý như không tìm thấy dòng.

## 21. Danh sách lỗi chung

| HTTP status | Mã lỗi | Ý nghĩa |
|---|---|---|
| 400 | `INVALID_JSON` | JSON sai cú pháp |
| 400 | `VALIDATION_ERROR` | Path, query hoặc body không hợp lệ |
| 401 | `AUTH_REQUIRED` | Thiếu thông tin xác thực |
| 401 | `INVALID_TOKEN` | Token không hợp lệ hoặc tài khoản không tồn tại |
| 401 | `TOKEN_EXPIRED` | Token hết hạn |
| 403 | `ACCOUNT_DISABLED` | Tài khoản đã bị khóa |
| 403 | `FORBIDDEN` | Không có quyền thao tác |
| 404 | `SALES_ORDER_NOT_FOUND` | Đơn không tồn tại |
| 404 | `SALES_ORDER_ITEM_NOT_FOUND` | Dòng không tồn tại trong đơn |
| 404 | `PRODUCT_NOT_FOUND` | Sản phẩm được gửi không tồn tại |
| 409 | `SALES_ORDER_NOT_DRAFT` | Thao tác sửa yêu cầu đơn nháp |
| 409 | `SALES_ORDER_ALREADY_CONFIRMED` | Đơn đã xác nhận |
| 409 | `SALES_ORDER_CANCELLED` | Không thể xác nhận đơn đã hủy |
| 409 | `SALES_ORDER_EMPTY` | Không thể xác nhận đơn rỗng |
| 409 | `SALES_ORDER_PRODUCT_ALREADY_EXISTS` | Sản phẩm đã xuất hiện trong đơn |
| 409 | `PRODUCT_INACTIVE` | Sản phẩm đã ngừng kinh doanh |
| 409 | `INSUFFICIENT_STOCK` | Không đủ tồn để bán |
| 413 | `PAYLOAD_TOO_LARGE` | Body vượt giới hạn cấu hình |
| 415 | `UNSUPPORTED_MEDIA_TYPE` | Loại nội dung không được hỗ trợ |
| 500 | `INTERNAL_SERVER_ERROR` | Lỗi nội bộ |

Không ánh xạ mọi lỗi database thành `409`.

Không trả câu SQL, stack trace hoặc thông tin kết nối cơ sở dữ liệu.

## 22. Ánh xạ dữ liệu

### 22.1. Đơn bán

| Trường API | Cột hoặc nguồn |
|---|---|
| `id` | `sales_orders.id` |
| `createdBy` | `sales_orders.created_by` |
| `status` | `sales_orders.status` |
| `createdAt` | `sales_orders.created_at` |
| `confirmedBy` | `sales_orders.confirmed_by` |
| `confirmedAt` | `sales_orders.confirmed_at` |
| `itemCount` | Số dòng trong `sales_order_items` |
| `totalAmount` | Tổng thành tiền các dòng |

### 22.2. Dòng sản phẩm

| Trường API | Cột hoặc nguồn |
|---|---|
| `id` | `sales_order_items.id` |
| `productId` | `sales_order_items.product_id` |
| `quantity` | `sales_order_items.quantity` |
| `unitPrice` | `sales_order_items.unit_price` |
| `lineTotal` | `quantity × unit_price` |

`order_id` lấy từ đơn trong URL.

Thiết kế hiện tại không có `updated_at` cho đơn và dòng sản phẩm; API không tự bổ sung trường này.

Tên và SKU không được lưu bản sao trên dòng đơn. Khi tra cứu qua bảng sản phẩm, hệ thống nhận thông tin tên và SKU hiện tại; giá bán vẫn dùng giá đã lưu trên dòng.

## 23. Yêu cầu bảo mật và triển khai

- Sử dụng truy vấn có tham số.
- Kiểm tra quyền sở hữu đơn ở backend.
- Không tin các trường định danh người dùng hoặc trạng thái do client gửi.
- Không nhận giá bán do client chỉ định.
- Áp dụng danh sách trường cho phép.
- Kiểm tra dòng thuộc đúng đơn.
- Không dùng cache làm căn cứ cuối cùng khi kiểm tra trạng thái hoặc đủ tồn.
- Không gửi thành công trước khi commit.
- Không ghi JWT vào log.
- Ghi log lỗi giao dịch và lỗi toàn vẹn dữ liệu.
- Không tích hợp thanh toán hoặc gửi thông báo bên ngoài trong giao dịch xác nhận.
- Khi có cache tồn kho, vô hiệu hóa hoặc cập nhật sau commit theo tài liệu caching.
- Lỗi cache sau commit không được dẫn đến trừ tồn lần nữa.
- Sử dụng phép toán thập phân chính xác để tính tiền.

## 24. Các tình huống kiểm thử

### 24.1. Xác thực và phân quyền

- Không có token, token sai hoặc hết hạn.
- Tài khoản không tồn tại hoặc bị khóa.
- ADMIN thao tác đơn do STAFF tạo.
- STAFF thao tác đơn của mình.
- STAFF không được sửa, hủy hoặc xác nhận đơn của người khác.
- Request bị từ chối không làm thay đổi dữ liệu.

### 24.2. Tạo và tra cứu

- Tạo bằng JSON rỗng.
- Body thiếu, sai kiểu hoặc chứa trường không được phép.
- Đơn mới DRAFT, rỗng, chưa có người xác nhận.
- Tạo đơn không thay đổi tồn kho.
- Lọc theo trạng thái và người tạo.
- Phân trang rỗng hoặc vượt phạm vi.
- Tổng số đơn không bị nhân lên do JOIN.
- Xem được cả ba trạng thái.
- ID không tồn tại hoặc không hợp lệ.

### 24.3. Thêm dòng

- Thêm sản phẩm đang kinh doanh.
- Sản phẩm không tồn tại hoặc ngừng kinh doanh.
- Thêm sản phẩm tồn bằng 0 vẫn tạo được dòng nháp.
- Số lượng lớn hơn tồn vẫn tạo được dòng nháp.
- Thêm trùng sản phẩm bị từ chối.
- Hai request đồng thời thêm cùng sản phẩm chỉ tạo một dòng.
- Số lượng bằng 0, âm, thập phân, sai kiểu hoặc vượt giới hạn.
- Gửi `unitPrice` bị từ chối.
- Giá được lấy từ sản phẩm và lưu chính xác.
- Giá bằng 0 được xử lý theo dữ liệu sản phẩm hợp lệ.

### 24.4. Lưu giá

- Sửa giá sản phẩm không thay đổi giá đã lưu.
- Sửa số lượng không lấy lại giá.
- Xác nhận không lấy lại giá.
- Xóa rồi thêm lại lấy giá hiện tại.
- Cập nhật giá đồng thời với thêm dòng không tạo giá sai hoặc dữ liệu không nhất quán.
- Tổng tiền dùng giá dòng, không JOIN giá hiện tại để tính lại.

### 24.5. Sửa và xóa dòng

- Sửa số lượng hợp lệ.
- Gửi lại số lượng hiện tại.
- Không cho sửa `productId` hoặc `unitPrice`.
- Không cho sửa số lượng dòng có sản phẩm đã ngừng kinh doanh.
- Cho phép xóa dòng có sản phẩm ngừng kinh doanh.
- Xóa dòng cuối cùng.
- Dòng thuộc đơn khác bị từ chối.
- Sửa hoặc xóa trên đơn đã xác nhận hoặc đã hủy bị từ chối.
- Không làm thay đổi tồn kho.

### 24.6. Xác nhận

- Đơn một dòng và nhiều dòng.
- Đủ tồn cho toàn bộ đơn.
- Bán đúng bằng tồn, kết quả tồn bằng 0.
- Thiếu một sản phẩm thì toàn bộ đơn thất bại.
- Không trừ các dòng còn đủ nếu có dòng thiếu.
- Đơn rỗng bị từ chối.
- Đơn đã hủy bị từ chối.
- Sản phẩm bị ngừng sau khi thêm khiến xác nhận bị từ chối.
- Thiếu dòng tồn kho trả lỗi nội bộ và rollback.
- Giá và số lượng đã lưu không bị thay đổi.
- Người xác nhận và thời điểm xác nhận chỉ được lưu khi thành công.
- Lỗi giữa quá trình trừ nhiều sản phẩm phải rollback toàn bộ.

### 24.7. Hủy

- Hủy đơn nháp rỗng hoặc có dòng.
- Gọi hủy lại trả thành công.
- Không hủy được đơn đã xác nhận.
- Hủy không hoàn tồn và không xóa dòng.
- Không ghi người hoặc thời điểm xác nhận.

### 24.8. Yêu cầu đồng thời

- Hai request xác nhận cùng đơn chỉ trừ tồn một lần.
- Hai đơn bán cùng sản phẩm không bán vượt tồn.
- Xác nhận bán đồng thời với xác nhận nhập không mất cập nhật.
- Xác nhận đồng thời với sửa số lượng.
- Xác nhận đồng thời với xóa dòng cuối.
- Xác nhận đồng thời với hủy.
- Ngừng kinh doanh đồng thời với xác nhận.
- Hai request sửa số lượng được tuần tự hóa theo thiết kế.
- Deadlock hoặc lỗi giao dịch không làm rò rỉ connection.
- Retry sau rollback không trừ tồn hai lần.

### 24.9. Phản hồi và độ chính xác

- Thành tiền và tổng tiền chính xác với số lớn.
- Tiền trả dưới dạng chuỗi có 2 chữ số thập phân.
- Chi tiết đơn không trộn dữ liệu trước và sau cập nhật đồng thời.
- Tổng tiền đơn rỗng bằng `"0.00"`.
- JSON lỗi không lộ SQL hoặc stack trace.
- Lỗi mạng sau commit không khiến hệ thống tự trừ tồn lại.

## 25. Các điểm cần đồng bộ và hoàn thiện

### 25.1. Quy tắc đã chốt cần đồng bộ

Cập nhật `business-rules.md` và `catalog.md`:

- Không thêm sản phẩm ngừng kinh doanh vào đơn bán.
- Kiểm tra lại trạng thái sản phẩm khi xác nhận.
- Đơn nháp chứa sản phẩm ngừng kinh doanh phải xóa dòng đó trước khi xác nhận.
- Đơn đã xác nhận trước thời điểm ngừng kinh doanh được giữ nguyên.
- Giá bán được lưu khi thêm dòng.
- Thay đổi giá sản phẩm hoặc số lượng dòng không tự cập nhật giá đã lưu.
- Đơn nháp không giữ hoặc trừ tồn.
- Xác nhận phải đủ tồn cho toàn bộ đơn.

### 25.2. Các lựa chọn cụ thể trong bản thiết kế này

| Nội dung | Lựa chọn |
|---|---|
| Tạo đơn | Tạo đơn rỗng, thêm dòng qua endpoint riêng |
| Quyền xem | ADMIN và STAFF xem danh sách và chi tiết |
| Quyền ghi của STAFF | Chỉ đơn do mình tạo |
| Tồn kho khi lưu nháp | Cho phép số lượng lớn hơn tồn; kiểm tra bắt buộc khi xác nhận |
| Giá bán | Lấy từ sản phẩm lúc thêm; không nhận giá thủ công |
| Lấy lại giá hiện tại | Xóa dòng nháp và thêm lại |
| Sản phẩm ngừng kinh doanh | Không thêm, sửa số lượng hoặc xác nhận; cho phép xóa dòng nháp |
| Xác nhận lại | Trả 409, không trừ tồn thêm |
| Hủy lại | Trả 200, không thay đổi dữ liệu |
| Chống tạo đơn trùng | Chưa hỗ trợ idempotency key |
| Phát hiện chỉnh sửa từ dữ liệu cũ | Chưa có version hoặc If-Match |
| Lịch sử xuất bán | Suy ra từ đơn và dòng đã xác nhận |

Các lựa chọn được cụ thể hóa trong tài liệu này cần được đối chiếu với tài liệu đã commit trước khi triển khai.

### 25.3. Nội dung cần hoàn thiện

1. **Enum trạng thái:** đối chiếu `DRAFT`, `CONFIRMED`, `CANCELLED` với database và DBML.
2. **Quy ước lỗi:** đồng bộ mã lỗi và cấu trúc `details` giữa các module.
3. **Giới hạn số dòng:** xác định số dòng tối đa của đơn trước khi triển khai, thống nhất với giới hạn vận hành của phiếu nhập.
4. **Thứ tự khóa:** đồng bộ với `purchasing.md` và cập nhật sản phẩm.
5. **Retry và thời gian chờ khóa:** quy định số lần retry, thời gian chờ và phản hồi khi không thể hoàn thành giao dịch.
6. **Cache:** xác định cách vô hiệu hóa tồn kho sau commit và xử lý lỗi cache.
7. **Giới hạn body:** dùng cấu hình chung trong `architecture.md`.

Sau khi chốt từng nội dung, cập nhật vào phần chính và loại bỏ khỏi danh sách cần hoàn thiện.
