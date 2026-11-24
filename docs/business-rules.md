# Quy tắc nghiệp vụ

## 1. Mục đích

Tài liệu xác định các quy tắc nghiệp vụ của backend quản lý bán hàng và tồn kho đồ điện tử.

Các quy tắc là căn cứ để:

- Thiết kế database và API.
- Triển khai xử lý nghiệp vụ và phân quyền.
- Xây dựng các tình huống kiểm thử.
- Đối chiếu kết quả khi nghiệm thu.

Phạm vi tổng thể được mô tả trong [Phạm vi dự án](scope.md). Tài liệu này tập trung vào điều kiện thực hiện, thay đổi dữ liệu và kết quả mong đợi của từng nghiệp vụ.

Mã quy tắc được dùng để tham chiếu trong tài liệu API, issue, pull request và test case.

## 2. Thuật ngữ

| Thuật ngữ | Ý nghĩa |
|---|---|
| ADMIN | Quản trị viên |
| STAFF | Nhân viên cửa hàng |
| SKU | Mã nhận diện duy nhất của một sản phẩm hoặc phiên bản sản phẩm |
| Chứng từ | Phiếu nhập hoặc đơn bán |
| Dòng chi tiết | Một mặt hàng trong chứng từ, gồm sản phẩm, số lượng và đơn giá |
| Xác nhận | Thao tác hoàn tất chứng từ và cập nhật tồn kho |
| Tồn kho | Số lượng hiện có của một sản phẩm tại kho |
| Giá nhập | Giá mua một đơn vị sản phẩm trong một lần nhập |
| Giá bán trên đơn | Giá một đơn vị sản phẩm được lưu riêng trong dòng đơn bán |

Hệ thống quản lý một cửa hàng và một kho. Không quản lý riêng từng thiết bị bằng serial hoặc IMEI.

## 3. Tài khoản và phân quyền

| Mã | Quy tắc |
|---|---|
| BR-AUTH-01 | Mỗi tài khoản có một email đăng nhập duy nhất. |
| BR-AUTH-02 | Mỗi tài khoản thuộc một trong hai vai trò: `ADMIN` hoặc `STAFF`. |
| BR-AUTH-03 | Tài khoản nhân viên do ADMIN cấp; không có đăng ký công khai. |
| BR-AUTH-04 | Người dùng được đổi mật khẩu của bản thân. |
| BR-AUTH-05 | Chỉ ADMIN được tạo và khóa tài khoản nhân viên. |
| BR-AUTH-06 | Tài khoản bị khóa không được đăng nhập hoặc tiếp tục thực hiện các thao tác được bảo vệ, kể cả khi đã đăng nhập trước đó. |
| BR-AUTH-07 | Tài khoản đã liên quan tới chứng từ phải được giữ lại để tra cứu lịch sử; sử dụng khóa tài khoản thay cho xóa. |
| BR-AUTH-08 | STAFF chỉ được sửa, hủy và xác nhận chứng từ do chính mình tạo. |
| BR-AUTH-09 | ADMIN được thao tác trên chứng từ của mọi người dùng nhưng vẫn phải tuân thủ các điều kiện nghiệp vụ. |
| BR-AUTH-10 | Người tạo và người xác nhận một chứng từ có thể là cùng một tài khoản. |
| BR-AUTH-11 | Người tạo và người xác nhận phải được xác định từ danh tính người đang thực hiện thao tác; không cho phép người gọi tự gán danh tính khác. |

Ma trận quyền chức năng được quy định trong `scope.md`. Mọi thao tác ngoài quyền được cấp phải bị từ chối và không thay đổi dữ liệu.

## 4. Danh mục và sản phẩm

| Mã | Quy tắc |
|---|---|
| BR-CAT-01 | Tên danh mục là bắt buộc và không được trùng. |
| BR-CAT-02 | Mỗi sản phẩm thuộc một danh mục tồn tại trong hệ thống. |
| BR-CAT-03 | Một danh mục có thể chứa nhiều sản phẩm. |
| BR-CAT-04 | Phiên bản hiện tại không cung cấp chức năng xóa danh mục. Liên kết giữa danh mục và sản phẩm phải được bảo vệ bằng khóa ngoại. |
| BR-PROD-01 | Mỗi sản phẩm có một SKU duy nhất. |
| BR-PROD-02 | Tên sản phẩm là bắt buộc. |
| BR-PROD-03 | Giá bán hiện tại không được âm. |
| BR-PROD-04 | Khi tạo sản phẩm, phải tạo dữ liệu tồn kho tương ứng với số lượng ban đầu bằng 0. Không để lại sản phẩm thiếu bản ghi tồn kho khi thao tác tạo hoàn tất. |
| BR-PROD-05 | Chỉ ADMIN được thêm, sửa thông tin, thay đổi giá bán hoặc ngừng kinh doanh sản phẩm. |
| BR-PROD-06 | Sản phẩm đã xuất hiện trong chứng từ phải được giữ lại để bảo toàn lịch sử. Khi ngừng kinh doanh, thay đổi trạng thái thay vì xóa sản phẩm. |
| BR-PROD-07 | Thay đổi giá bán hiện tại của sản phẩm không tự động thay đổi giá đã lưu trên các dòng đơn bán. |

Quy tắc xử lý sản phẩm ngừng kinh doanh được quy định tại mục 7 đối với phiếu nhập và mục 8 đối với đơn bán.

## 5. Nhà cung cấp

| Mã | Quy tắc |
|---|---|
| BR-SUP-01 | Tên nhà cung cấp là bắt buộc. |
| BR-SUP-02 | Số điện thoại, email và địa chỉ có thể chưa được cung cấp khi tạo nhà cung cấp. |
| BR-SUP-03 | Chỉ ADMIN được thêm và sửa nhà cung cấp. ADMIN và STAFF được xem thông tin nhà cung cấp. |
| BR-SUP-04 | Mỗi phiếu nhập phải gắn với một nhà cung cấp tồn tại trong hệ thống. |
| BR-SUP-05 | Một nhà cung cấp có thể liên kết với nhiều phiếu nhập. |
| BR-SUP-06 | Không được xóa nhà cung cấp đã có phiếu nhập. |
| BR-SUP-07 | Một sản phẩm có thể được nhập từ các nhà cung cấp khác nhau qua các phiếu nhập khác nhau. |

## 6. Quy tắc chung của chứng từ

### 6.1. Trạng thái

| Trạng thái | Ý nghĩa | Ảnh hưởng tới tồn kho |
|---|---|---|
| `DRAFT` | Chứng từ nháp, đang được chuẩn bị | Chưa thay đổi tồn |
| `CONFIRMED` | Chứng từ đã xác nhận thành công | Đã cộng hoặc trừ tồn |
| `CANCELLED` | Chứng từ nháp đã bị hủy | Không thay đổi tồn |

### 6.2. Chuyển trạng thái

| Trạng thái hiện tại | Thao tác | Kết quả |
|---|---|---|
| `DRAFT` | Sửa nội dung hợp lệ | Giữ `DRAFT` |
| `DRAFT` | Xác nhận hợp lệ | Chuyển thành `CONFIRMED` |
| `DRAFT` | Xác nhận không hợp lệ | Giữ `DRAFT`, không thay đổi tồn |
| `DRAFT` | Hủy | Chuyển thành `CANCELLED` |
| `CONFIRMED` | Sửa, hủy hoặc xác nhận lại | Không cho phép |
| `CANCELLED` | Sửa, xác nhận hoặc khôi phục về nháp | Không cho phép |

### 6.3. Quy tắc áp dụng chung

| Mã | Quy tắc |
|---|---|
| BR-DOC-01 | Chứng từ mới được tạo ở trạng thái `DRAFT`. |
| BR-DOC-02 | Chứng từ nháp có thể chưa có dòng hàng; khi xác nhận phải có ít nhất một dòng hàng hợp lệ. |
| BR-DOC-03 | Mỗi dòng chi tiết chỉ thuộc về một chứng từ và liên kết với một sản phẩm tồn tại. |
| BR-DOC-04 | Một sản phẩm chỉ được xuất hiện một dòng trong cùng chứng từ. Muốn thay đổi số lượng phải sửa dòng hiện có. |
| BR-DOC-05 | Số lượng trên mỗi dòng phải là số nguyên dương. |
| BR-DOC-06 | Đơn giá trên mỗi dòng không được âm. |
| BR-DOC-07 | Chỉ chứng từ `DRAFT` được thêm, sửa hoặc xóa dòng hàng. |
| BR-DOC-08 | Hủy chứng từ là chuyển trạng thái sang `CANCELLED`, không xóa chứng từ và các dòng chi tiết đã lưu. |
| BR-DOC-09 | Chứng từ đã xác nhận không được sửa hoặc hủy trong phiên bản này. |
| BR-DOC-10 | Người tạo và thời điểm tạo phải được ghi nhận khi tạo chứng từ. |
| BR-DOC-11 | Người xác nhận và thời điểm xác nhận chỉ được ghi nhận khi xác nhận thành công; trước đó để trống. |
| BR-DOC-12 | Mỗi chứng từ chỉ được tác động tới tồn kho tối đa một lần. |
| BR-DOC-13 | Cập nhật tồn kho, trạng thái và thông tin xác nhận phải hoàn tất cùng nhau. Nếu bất kỳ phần nào thất bại, không được lưu kết quả dở dang. |
| BR-DOC-14 | Khi có các thao tác sửa, hủy hoặc xác nhận đồng thời, kết quả cuối cùng vẫn phải tuân thủ quy tắc trạng thái và không làm nội dung chứng từ đã xác nhận lệch với tồn kho đã cập nhật. |

## 7. Nhập hàng

| Mã | Quy tắc |
|---|---|
| BR-REC-01 | Tạo hoặc sửa phiếu nhập nháp chưa làm tăng tồn kho. |
| BR-REC-02 | Phiếu nhập có thể chứa một hoặc nhiều sản phẩm khi được xác nhận. |
| BR-REC-03 | Giá nhập được lưu riêng trên từng dòng phiếu; cùng một sản phẩm có thể có giá nhập khác nhau giữa các lần nhập. |
| BR-REC-04 | Khi xác nhận thành công, tồn của mỗi sản phẩm tăng đúng bằng số lượng trên dòng phiếu tương ứng. |
| BR-REC-05 | Nếu có dòng không hợp lệ hoặc thao tác xác nhận thất bại, toàn bộ lần xác nhận bị từ chối; phiếu vẫn là `DRAFT` và tồn kho không đổi. |
| BR-REC-06 | Xác nhận lại phiếu đã xác nhận không được cộng thêm tồn kho. |
| BR-REC-07 | Hủy phiếu nhập nháp không thay đổi tồn kho. |
| BR-REC-08 | Không được thêm sản phẩm đã ngừng kinh doanh vào phiếu nhập nháp. |
| BR-REC-09 | Nếu sản phẩm ngừng kinh doanh sau khi đã được thêm vào phiếu nháp, giữ dòng để tra cứu nhưng không cho sửa số lượng hoặc giá nhập của dòng đó. Người có quyền được xóa dòng hoặc hủy phiếu. |
| BR-REC-10 | Khi xác nhận, phải kiểm tra lại trạng thái kinh doanh của tất cả sản phẩm. Nếu có sản phẩm đã ngừng kinh doanh, từ chối toàn bộ lần xác nhận và không thay đổi tồn kho. Phải xóa dòng không hợp lệ và bảo đảm phiếu còn ít nhất một dòng hợp lệ trước khi xác nhận. Phiếu đã xác nhận trước thời điểm sản phẩm ngừng kinh doanh được giữ nguyên. |

### Ví dụ

Kho đang có 20 bàn phím. Nhân viên tạo phiếu nhập thêm 10 bàn phím.

- Khi phiếu là `DRAFT`: tồn vẫn bằng 20.
- Khi xác nhận thành công: tồn bằng 30.
- Nếu gửi lại yêu cầu xác nhận: tồn vẫn bằng 30.
- Nếu hủy phiếu khi còn nháp: tồn vẫn bằng 20.

## 8. Bán hàng

| Mã | Quy tắc |
|---|---|
| BR-SALE-01 | Tạo hoặc sửa đơn nháp không trừ tồn và không giữ hàng. |
| BR-SALE-02 | Khi thêm sản phẩm vào đơn, lấy giá bán hiện tại và lưu riêng tại dòng đơn. |
| BR-SALE-03 | Thay đổi số lượng trên dòng đơn không tự động cập nhật đơn giá đã lưu. |
| BR-SALE-04 | Thành tiền của một dòng bằng số lượng nhân với đơn giá đã lưu. Tổng tiền đơn bằng tổng thành tiền của các dòng. |
| BR-SALE-05 | Khi xác nhận, phải kiểm tra lượng tồn có thể sử dụng tại thời điểm xử lý, không dựa vào lượng tồn lúc tạo đơn. |
| BR-SALE-06 | Nếu bất kỳ sản phẩm nào không đủ tồn, từ chối toàn bộ lần xác nhận; đơn vẫn là `DRAFT` và tồn của tất cả sản phẩm trong đơn không đổi. |
| BR-SALE-07 | Khi xác nhận thành công, tồn của mỗi sản phẩm giảm đúng bằng số lượng trên dòng đơn tương ứng. |
| BR-SALE-08 | Nhiều đơn xác nhận đồng thời không được làm tổng số lượng bán vượt quá tồn có thể sử dụng. |
| BR-SALE-09 | Xác nhận lại đơn đã xác nhận không được trừ thêm tồn kho. |
| BR-SALE-10 | Hủy đơn nháp không làm thay đổi tồn kho. |
| BR-SALE-11 | Xác nhận đơn ghi nhận hàng đã bán và xuất kho; không thể hiện trạng thái thanh toán, công nợ hoặc giao hàng. |
| BR-SALE-12 | Không được thêm sản phẩm đã ngừng kinh doanh vào đơn bán nháp. |
| BR-SALE-13 | Nếu sản phẩm ngừng kinh doanh sau khi đã được thêm vào đơn nháp, giữ dòng để tra cứu nhưng không cho sửa số lượng của dòng đó. Người có quyền được xóa dòng hoặc hủy đơn. |
| BR-SALE-14 | Khi xác nhận, phải kiểm tra lại trạng thái kinh doanh của tất cả sản phẩm. Nếu có sản phẩm đã ngừng kinh doanh, từ chối toàn bộ lần xác nhận và không thay đổi tồn kho. Phải xóa dòng không hợp lệ; các dòng còn lại phải hợp lệ và đủ tồn trước khi xác nhận. Đơn đã xác nhận trước thời điểm sản phẩm ngừng kinh doanh được giữ nguyên. |
| BR-SALE-15 | Không cho phép nhập hoặc chỉnh sửa thủ công giá bán trên dòng đơn. Xác nhận đơn sử dụng giá đã lưu, không lấy lại giá hiện tại. Muốn lấy giá hiện tại cho một sản phẩm trong đơn nháp, phải xóa dòng cũ và thêm lại sản phẩm. |
| BR-SALE-16 | Cho phép lưu số lượng trên đơn nháp lớn hơn tồn kho hiện tại. Kiểm tra đủ tồn mang tính quyết định được thực hiện khi xác nhận trong giao dịch; đơn nháp không giữ hàng. |

### Ví dụ thiếu hàng

Kho có:

- Chuột G102: 10 chiếc.
- Bàn phím K120: 1 chiếc.

Đơn bán gồm:

- Chuột G102: 3 chiếc.
- Bàn phím K120: 2 chiếc.

Kết quả xác nhận:

- Bị từ chối do thiếu bàn phím.
- Đơn giữ trạng thái `DRAFT`.
- Chuột vẫn còn 10 chiếc.
- Bàn phím vẫn còn 1 chiếc.

### Ví dụ xác nhận đồng thời

Kho còn 5 chuột. Hai đơn khác nhau cùng yêu cầu bán 4 chuột.

Nếu không có thay đổi tồn nào khác, chỉ một đơn được xác nhận thành công. Đơn còn lại phải bị từ chối do thiếu hàng. Tồn cuối cùng bằng 1.

## 9. Tồn kho và lịch sử nhập/xuất

| Mã | Quy tắc |
|---|---|
| BR-INV-01 | Mỗi sản phẩm có một bản ghi tồn kho trong phạm vi một kho. |
| BR-INV-02 | Tồn kho là số nguyên không âm. |
| BR-INV-03 | Tồn ban đầu của sản phẩm bằng 0; hàng có sẵn cần được ghi nhận thông qua phiếu nhập. |
| BR-INV-04 | ADMIN và STAFF không được sửa trực tiếp số lượng tồn kho. |
| BR-INV-05 | Tồn chỉ tăng từ phiếu nhập xác nhận thành công và giảm từ đơn bán xác nhận thành công. |
| BR-INV-06 | Chứng từ nháp hoặc đã hủy không được tính vào lịch sử nhập/xuất thực tế. |
| BR-INV-07 | Lịch sử được tổng hợp từ chứng từ đã xác nhận và các dòng chi tiết; không sử dụng bảng `stock_movements` riêng. |
| BR-INV-08 | Mỗi mục lịch sử phải xác định được sản phẩm, số lượng tăng/giảm, loại chứng từ, ID chứng từ và thời điểm xác nhận. |
| BR-INV-09 | Thứ tự thời gian của lịch sử dựa trên thời điểm xác nhận, không phải thời điểm tạo chứng từ. |
| BR-INV-10 | Tồn hiện tại phải bằng tổng lượng nhập đã xác nhận trừ tổng lượng bán đã xác nhận của sản phẩm. |

Dữ liệu cache hoặc số tồn từng hiển thị cho người dùng không phải căn cứ quyết định cuối cùng khi xác nhận bán hàng.

## 10. Hỏi đáp tài liệu

| Mã | Quy tắc |
|---|---|
| BR-KNOW-01 | Chỉ ADMIN được quản lý tài liệu tri thức, bao gồm nạp, xem danh sách, xem chi tiết và trạng thái xử lý, xóa tài liệu. |
| BR-KNOW-02 | ADMIN và STAFF được hỏi đáp dựa trên tài liệu đã xử lý thành công và sẵn sàng truy xuất. |
| BR-KNOW-03 | Câu trả lời về hướng dẫn sử dụng hệ thống và quy trình nghiệp vụ phải dựa trên nội dung truy xuất được, kèm nguồn tham chiếu hỗ trợ câu trả lời. |
| BR-KNOW-04 | Khi tài liệu không đủ căn cứ hoặc có nội dung mâu thuẫn chưa giải quyết, hệ thống phải nói rõ, không tự tạo quy tắc nghiệp vụ. |
| BR-KNOW-05 | Không truy vấn hoặc suy đoán giá bán, tồn kho tức thời hay trạng thái của một chứng từ cụ thể qua RAG. Những thông tin này được tra cứu qua API nghiệp vụ tương ứng. |
| BR-KNOW-06 | Chức năng hỏi đáp không được tự tạo, sửa, hủy hoặc xác nhận chứng từ; không được thay đổi sản phẩm hoặc tồn kho. |
| BR-KNOW-07 | Câu hỏi ngoài phạm vi hướng dẫn hệ thống và quy trình nghiệp vụ phải được thông báo là ngoài phạm vi; không mở rộng chức năng thông qua câu trả lời của mô hình. |
| BR-KNOW-08 | Kho tri thức dùng chung cho ADMIN và STAFF. Chỉ nạp tài liệu phù hợp để cung cấp nội dung cho cả hai vai trò; không nạp mật khẩu, token, khóa API hoặc thông tin bị hạn chế với STAFF. |
| BR-KNOW-09 | Tài liệu đang chờ xử lý, đang xử lý hoặc xử lý thất bại không được dùng làm nguồn trả lời. |
| BR-KNOW-10 | Sau khi xóa tài liệu thành công, các request hỏi đáp bắt đầu sau đó không được sử dụng tài liệu này. Request đã lấy nội dung trước thời điểm xóa có thể hoàn tất theo giới hạn của API. |
| BR-KNOW-11 | Nguồn trích dẫn phải thuộc các đoạn thực sự được truy xuất; không tạo giả tên tài liệu, định danh, đường dẫn hoặc đoạn trích. |
| BR-KNOW-12 | Nội dung tài liệu là dữ liệu tham khảo, không phải chỉ dẫn có quyền thay đổi hành vi, phân quyền hoặc cấu hình hệ thống. |
| BR-KNOW-13 | Mỗi câu hỏi là một request độc lập; phiên bản đầu chưa sử dụng lịch sử hội thoại nhiều lượt. |
| BR-KNOW-14 | Phải phân biệt thiếu căn cứ trong tài liệu với lỗi dịch vụ. Không báo thiếu kiến thức để che giấu lỗi embedding, truy xuất hoặc sinh câu trả lời. |

Chất lượng câu trả lời và nguồn trích dẫn phải được đánh giá bằng bộ câu hỏi kiểm thử. Không mặc định mọi câu trả lời do mô hình sinh ra đều chính xác.

## 11. Các tình huống kiểm thử trọng tâm

| Tình huống | Kết quả mong đợi | Quy tắc liên quan |
|---|---|---|
| Tạo sản phẩm có SKU đã tồn tại | Từ chối, không tạo sản phẩm trùng | BR-PROD-01 |
| Kiểm tra phạm vi quản lý danh mục | Không cung cấp API xóa danh mục; khóa ngoại bảo vệ liên kết sản phẩm | BR-CAT-04 |
| STAFF xác nhận phiếu của người khác | Từ chối, không đổi dữ liệu | BR-AUTH-08 |
| Xác nhận chứng từ không có dòng hàng | Từ chối, chứng từ giữ `DRAFT` | BR-DOC-02 |
| Thêm cùng sản phẩm thành hai dòng trong một phiếu | Không cho lưu hai dòng trùng sản phẩm | BR-DOC-04 |
| Nhập số lượng bằng 0 hoặc âm | Từ chối | BR-DOC-05 |
| Xác nhận nhập thành công | Cộng đủ tồn và ghi nhận xác nhận | BR-REC-04 |
| Xác nhận lại cùng phiếu nhập | Không cộng thêm tồn | BR-REC-06 |
| Một mặt hàng trong đơn không đủ tồn | Từ chối toàn bộ, không trừ dở dang | BR-SALE-06 |
| Hai đơn cùng tranh lượng hàng cuối | Không bán vượt tồn, không tồn âm | BR-SALE-08 |
| Xác nhận lại cùng đơn bán | Không trừ thêm tồn | BR-SALE-09 |
| Lỗi giữa quá trình xác nhận | Không lưu cập nhật dở dang | BR-DOC-13 |
| Sửa hoặc hủy đồng thời với xác nhận | Trạng thái, nội dung và tồn kho vẫn nhất quán | BR-DOC-14 |
| Đổi giá sản phẩm sau khi thêm vào đơn | Giá đã lưu trên dòng đơn không tự đổi | BR-PROD-07 |
| Tra cứu lịch sử | Chỉ có chứng từ đã xác nhận | BR-INV-06 |
| Hỏi nội dung không có trong tài liệu | Nêu thiếu căn cứ | BR-KNOW-04 |
| Thêm sản phẩm đã ngừng kinh doanh vào phiếu nhập hoặc đơn bán | Từ chối, không tạo dòng | BR-REC-08, BR-SALE-12 |
| Sản phẩm ngừng kinh doanh sau khi được thêm vào chứng từ nháp | Không cho xác nhận; không thay đổi tồn kho | BR-REC-10, BR-SALE-14 |
| Sản phẩm ngừng kinh doanh sau khi chứng từ đã xác nhận | Chứng từ và tác động tồn kho trước đó được giữ nguyên | BR-REC-10, BR-SALE-14 |
| Client gửi giá bán khi thêm hoặc sửa dòng đơn | Từ chối trường không được phép | BR-SALE-15 |
| Số lượng đơn nháp vượt tồn hiện tại | Cho lưu nháp; khi xác nhận phải kiểm tra đủ tồn | BR-SALE-16 |
| Hỏi tồn kho tức thời qua RAG | Thông báo ngoài phạm vi, hướng tới chức năng tra cứu tồn | BR-KNOW-05 |
| Tài liệu chưa xử lý xong hoặc xử lý lỗi | Không được dùng làm nguồn trả lời | BR-KNOW-09 |
| Hỏi đáp mới sau khi xóa tài liệu | Không sử dụng tài liệu đã xóa | BR-KNOW-10 |
| Tài liệu chứa chỉ dẫn yêu cầu bỏ qua phân quyền | Không thực hiện theo chỉ dẫn trong tài liệu | BR-KNOW-12 |

## 12. Nguyên tắc triển khai và cập nhật

- Các điều kiện nghiệp vụ phải được kiểm tra tại backend.
- Kết hợp ràng buộc database và xử lý nghiệp vụ để bảo vệ dữ liệu.
- Các thay đổi liên quan khi xác nhận chứng từ phải nằm trong cùng transaction.
- Thiết kế phải xử lý được yêu cầu đồng thời, không chỉ trường hợp thao tác lần lượt.
- HTTP status, mã lỗi và cấu trúc response được quy định trong tài liệu API.
- Khi thay đổi quy tắc, phải cập nhật tài liệu, phần triển khai và test liên quan trong cùng phạm vi công việc.
- Không tái sử dụng mã quy tắc đã có cho một ý nghĩa khác.

## 13. Các quyết định đã thống nhất và nội dung còn cần chốt

### 13.1. Các quyết định đã thống nhất

| Nội dung | Quyết định |
|---|---|
| Sản phẩm ngừng kinh doanh | Không thêm vào phiếu nhập hoặc đơn bán; không xác nhận chứng từ nháp còn chứa sản phẩm này. Chứng từ đã xác nhận được giữ nguyên. |
| Giá bán trên đơn nháp | Lấy giá khi thêm dòng; không chỉnh giá thủ công; sửa số lượng hoặc xác nhận không lấy lại giá. |
| Số lượng trên đơn nháp | Có thể vượt tồn hiện tại; không giữ hàng; bắt buộc kiểm tra đủ tồn khi xác nhận. |
| Mật khẩu mới | Từ 15 đến 64 Unicode code point; không tự trim hoặc biến đổi; không bắt buộc tổ hợp chữ hoa, chữ thường, số và ký tự đặc biệt. |
| Mật khẩu ban đầu | Mỗi nhân viên nhận mật khẩu ngẫu nhiên riêng do ADMIN chuẩn bị; không dùng chung hoặc đặt theo quy luật dễ đoán. |
| Đổi mật khẩu | Phải xác minh mật khẩu hiện tại; mật khẩu mới phải khác mật khẩu hiện tại. |
| Băm mật khẩu | Sử dụng Argon2id; cấu hình kỹ thuật được đồng bộ tại tài liệu xác thực. |
| JWT | Access token có thời hạn 1 giờ; chưa có refresh token. |
| Sau đổi mật khẩu | Mật khẩu cũ không đăng nhập được; token đã cấp vẫn có hiệu lực tới khi hết hạn nếu tài khoản còn hoạt động. |
| Sau khóa tài khoản | Request kiểm tra trạng thái sau khi khóa được commit phải bị từ chối; không tự hủy request đã vượt qua kiểm tra trước đó. |
| Phạm vi RAG | Hướng dẫn hệ thống và quy trình nghiệp vụ; không truy vấn số liệu tức thời hoặc thực hiện giao dịch. |

Chính sách mật khẩu mới áp dụng khi cấp tài khoản và đổi mật khẩu. Không dùng chính sách độ dài mới để tự động từ chối việc xác minh mật khẩu hiện tại của tài khoản đã tồn tại.

### 13.2. Các nội dung còn cần chốt

| Nội dung | Quyết định cần làm rõ |
|---|---|
| Chuẩn hóa dữ liệu duy nhất | Quy tắc chữ hoa/thường, dấu và khoảng trắng đối với email, SKU, tên danh mục; đồng bộ với collation. |
| Đơn vị tiền tệ | Đơn vị tiền sử dụng và việc có cho phép phần lẻ hay không. |
| Giới hạn chứng từ | Số dòng tối đa trong một phiếu nhập hoặc đơn bán. |
| Khởi tạo ADMIN đầu tiên | Cơ chế khởi tạo và cấp thông tin đăng nhập an toàn, không mở API tạo ADMIN công khai. |

Quy ước hiện tại của API là truyền giá dưới dạng chuỗi thập phân và từ chối đầu vào có quá 2 chữ số thập phân, không tự làm tròn.

Các quyết định bổ sung phải được cập nhật đồng bộ vào tài liệu database, kiến trúc, API và kiểm thử liên quan.
