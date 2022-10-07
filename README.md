### 1. Seaport đẻ ra để làm gì :3

Opensea sử dụng trước đây là Wyvern Protocol. Nhưng với sự phát triển của hệ sinh thái, các vấn đề khác nhau của Giao thức Wyvern

- Được phát triển cách đây nhiều năm, mã này được duy trì lần cuối 4 năm trước.
- Không được thiết kế riêng cho Opensea, có rất nhiều tài nguyên bị lãng phí.
- Mức tiêu thụ gas cao, khoảng 280000 đơn.
- Nó không được hỗ trợ sử dụng trực tiếp mã thông báo để mua NFT và nó cần được chuyển đổi khi mua. Hơn nữa làm tăng mức tiêu thụ khí đốt và chi phí mua hàng.

Đó là để giải quyết những vấn đề này mà Seaport Protocol đã được phát triển. Nó có những đặc điểm sau:

- Mua số lượng lớn và bán số lượng lớn đều được hỗ trợ.
- Hỗ trợ các loại tiền tệ khác nhau để mua.
- Hỗ trợ thanh toán hỗn hợp. Ví dụ: đơn đặt hàng có thể được đặt để thanh toán bằng một số mã thông báo ERC20 cộng với một số mã thông báo ERC721. Mục đích chính là để loại bỏ tổn thất do việc chuyển đổi lẫn nhau của các loại tiền tệ khác nhau và giảm thêm chi phí cho người dùng.
- Sử dụng SOL để giảm tiêu thụ gas, thử nghiệm hiện tại là khoảng 134000 đơn

>     Chúng ta có thể thấy rằng mục đích quan trọng nhất của Seaport Protocol là giảm tiêu thụ khí đốt của người sử dụng và giảm độ phức tạp của quy trình mua hàng.

### 2. Cách nó hoạt động -\_-

- Hầu hết các thị trường NFT hiện chỉ cho phép một bên đồng ý cung cấp NFT và bên kia đồng ý cung cấp danh sách c ác payment token.
- Seaport có một cách tiếp cận khác: **offerer** có thể đồng ý cung cấp một số lượng nhất định các dự án ETH / ERC20 / ERC721 / ERC1155 - và thế là xong offer. Để điều này offer được chấp nhận, offerer một số vật phẩm nhất định phải được nhận, và thế là xong consideration.

Có nghĩa là, ở Seaport không có khái niệm rõ ràng về người mua và người bán.

- Khi đó offerlà ERC721 / ERC1155, đó offererlà người bán, bán ERC721 / ERC1155 để đổi lấy ETH / ERC20.
- Khi đó offerlà ETH / ERC20, đó offererlà người mua, mua ERC721 / ERC1155 và thanh toán ETH / ERC20.
- Khi offer và consideration đều là ERC721 / ERC1155, đó là hoán đổi NFT. Trong trường hợp này, không có người mua hoặc người bán.

_Cần lưu ý rằng ETH / ERC20 hoạt động như một loại tiền tệ trong Seaport và ERC721 / ERC1155 hoạt động như một loại hàng hóa. điểm này rất quan trọng._

### 3. Interface

#### Offer vs Consideration

##### 1. itemType

```solidity
// ConsiderationEnums.sol => ItemType
enum ItemType {
    // 0: ETH on mainnet, MATIC on polygon, etc.
    NATIVE,
    // 1: ERC20 items (ERC777 and ERC20 analogues could also technically work)
    ERC20,
    // 2: ERC721 items
    ERC721,
    // 3: ERC1155 items
    ERC1155,
    // 4: ERC721 items where a number of tokenIds are supported
    ERC721_WITH_CRITERIA,
    // 5: ERC1155 items where a number of ids are supported
    ERC1155_WITH_CRITERIA
}
```

##### 2. token

Contract address của token là cụ thể, một address = underfine đại diện cho token gốc.

##### 3.identifierOrCriteria

- Đối với token loại ETH / ERC20, thuộc tính này bị bỏ qua.
- Đối với các mục loại ERC721 / ERC1155, mã thông báo id.
- Nói một cách đơn giản, người bán sở hữu nhiều NFT trong một bộ sưu tập. Tại thời điểm này, anh ta có thể cung cấp offer và idFierOrCriteria trong offer này được tạo thông qua tập hợp các id token tương ứng với NFT mà anh ta sở hữu. Theo cách này, offer chứa thông tin về các id token tương ứng với tất cả các NFT mà anh ta sở hữu. Người mua có thể chọn một NFT id token nhất định để giao dịch khi họ giao dịch và họ có thể giao dịch nếu vượt qua xác min
- identifierOrCriteriaCó thể là 0, cho biết rằng người mua cung cấp tất cả các ưu đãi NFT theo bộ sưu tập. Người mua có thể chọn bất kỳ NFT nào thuộc sở hữu offerer của bộ .

##### 4. startAmount và endAmount

- Bình thường hai giá trị này bằng nhau. Cho biết số lượng token.
- Hai giá trị này khác nhau khi tiến hành đấu giá ở Hà Lan và đấu giá ở Anh. Sau đó, đơn đặt hàng xác định giá hiện tại theo hai thuộc tính hiện tại kết hợp với thời gian khối hiện tại và startAmount và endAmountc của order

**Đấu giá Hà lan và đấu giá Anh**

- _Đấu giá Hà Lan là đấu giá giảm dần. Kiểu lúc đầu nó sẽ đặt giá ở trên trời sau đó thì giảm dần ... em hum hiểu áp dụng trong NFT kiểu gì nhưng cái này ngoài đời thi nó áp dụng cho bán hoa ấy (nó còn có tên gọi khác là Tulip Auction) kiểu hoa thì sẽ héo nhưng lúc đầu nó còn tươi nên giá nó ở rất cao... qua vài ngày thì nó bắt đầu héo nên giá sẽ được giảm xuống đại loại thế :v_
- _Còn đấu giá kiểu Anh.. Thì là loại bt ai hô cao thì sẽ được lấy hàng_

##### 5. recipient

consideration cần chỉ rỏ recipient. Đại diện cho recipient của token.

#### Offerer

Cần lưu ý rằng nếu đơn hàng muốn được lấp đầy hoặc được offerer chuyển offer đi thì phải thuộc một trong ba trường hợp sau:

- msg.sender == offerer, tức là offerer tự mình làm.
- Ủy quyền bằng chữ ký, EDCSA 65 byte tiêu chuẩn, EIP-2098 64 byte hoặc isValidSignature() xác minh bằng EIP-1271. Trong hầu hết các trường hợp, đây là trường hợp.
- Offerer thực hiện một order đang chờ xử lý validate() trên . Trong trường hợp này, việc xác minh chữ ký có thể được bỏ qua khi order đã được điền.

#### startTime và endTime

- Thời gian bắt đầu và kết thúc của order.

#### orderType

**1) FULL và PARTIAL**

FULL Cho biết rằng order không hỗ trợ điền một phần, trong khi một số phần của order được PARTIA L phép lấp đầy. Việc điền một phần thường được sử dụng cho ERC1155 NFT. Người mua có thể mua một phần của order.

**2) OPEN và RESTRICTED**

OPEN Cho biết rằng lệnh gọi thực hiện lệnh có thể được gửi bởi bất kỳ tài khoản nào, nhưng RESTRICTED là lệnh hạn chế yêu cầu lệnh phải được thực hiện offerer theo lệnh hoặc lệnh hoặc gọi vùng hoặc phương thức để xem giá trị kỳ diệu được trả về. Giá trị ma thuật này cho biết order đã được chấp thuận hay chưa.
func: zoneisValidOrder(), isValidOrderIncludingExtraData()

```solidity
// ConsiderationEnums.sol => OrderType
enum OrderType {
    // 0: no partial fills, anyone can execute
    FULL_OPEN,
    // 1: partial fills supported, anyone can execute
    PARTIAL_OPEN,
    // 2: no partial fills, only offerer or zone can execute
    FULL_RESTRICTED,
    // 3: partial fills supported, only offerer or zone can execute
    PARTIAL_RESTRICTED
}
```

#### zone

Zone là một tài khoản phụ không bắt buộc, thường là một hợp đồng.
Có hai quyền bổ sung:

- Zone có thể hủy order tương ứng bằng cách gọi cancel(). (Lưu ý rằng offerer cũng có thể hủy order của chính mình, riêng lẻ hoặc bằng cách gọi incrementCounter () để hủy ngay lập tức tất cả các order đã ký với bộ đếm hiện tại.
- Lệnh loại RESTRICTED phải được thực thi bởi zone hoặcofferer hoặc bằng cách gọi các method isValidOrder () hoặc isValidOrderIncludingExtraData () của zone để xem magic valuet được trả về. Magic valuet này cho biết order đã được chấp thuận hay chưa.

  > Nói một cách đơn giản, zone được xác minh bổ sung trước khi đặt order và việc listing offerer có thể bị hủy..Offerer có thể sử dụng zone để thực hiện một số hoạt động liên quan đến lọc giao dịch

#### zoneHash

zoneHash đại diện cho một giá trị 32 byte tùy ý sẽ được cung cấp cho zone khi thực hiện một order hạn chế, zone có thể sử dụng khi quyết định có ủy quyền cho order hay không.

#### ConduitKey

- conduitKey là một giá trị byte32 cho biết conduit nên được sử dụng làm nguồn phê duyệt token khi thực hiện chuyển token. Theo mặc định (tức là khi conduitKey được đặt thành 0), offerer sẽ cấp quyền cho các token ERC20, ERC721 và ERC1155 trực tiếp cho Seaport để nó có thể thực hiện chuyển order chỉ định trong quá trình thực thi. Thay vào đó, offerer phê duyệt các tokens tới conduit tương ứng, Seaport sau đó sẽ hướng dẫn chuyển các tokens tương ứng.
- conduitKey có liên quan chặt chẽ với conduit. Project parties, owners hoặc platforms có thể quản lý các giao dịch NFT thông qua conduit. Cung cấp tính linh hoạt cao hơn.

#### counter

Bộ đếm, phải giống bộ đếm của offerer.
Một offerer có thể hủy ngay lập tức tất cả các order đã ký với bộ đếm hiện tại bằng cách gọi incrementCounter ().

[Bài viết tham khảo](https://github.com/cryptochou/seaport-analysis)
