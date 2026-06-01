# chapter 1: Immutability of contracts 

cho đến giờ thì solidity khá giống javascript

sau khi deploy contract to Ethereum , nó immutable(có nghĩa là ko thể thay đổi), có nghĩa nó có thể sẽ không bao giờ được thay đổi hoặc updated một lần nào nữa

Đoạn code ban đầu mà bạn triển khai lên một hợp đồng sẽ tồn tại ở đó, vĩnh viễn, trên blockchain. Đây là một lý do tại sao bảo mật lại là một mối quan tâm vô cùng lớn trong Solidity. Nếu có lỗ hổng trong code hợp đồng của bạn, sẽ không có cách nào để bạn vá nó sau này. Bạn sẽ phải yêu cầu người dùng của mình bắt đầu sử dụng một địa chỉ hợp đồng thông minh khác đã được cập nhật bản sửa lỗi.

Nhưng đây cũng chính là một tính năng của hợp đồng thông minh. Code chính là luật. Nếu bạn đọc và xác minh code của một hợp đồng thông minh, bạn có thể hoàn toàn chắc chắn rằng mỗi lần gọi một hàm, nó sẽ thực hiện đúng những gì code đã quy định. Không ai có thể thay đổi hàm đó sau này và đưa ra cho bạn những kết quả bất ngờ.

**External dependencies** (các phụ thuộc bên ngoài)


rong Bài 2, chúng ta đã hard-code (gắn cứng) địa chỉ hợp đồng CryptoKitties vào DApp của mình. Nhưng điều gì sẽ xảy ra nếu hợp đồng CryptoKitties có một lỗi và ai đó tiêu diệt tất cả những chú mèo?

Điều này khó có thể xảy ra, nhưng nếu nó thực sự xảy ra thì nó sẽ khiến DApp của chúng ta trở nên hoàn toàn vô dụng — DApp của chúng ta sẽ trỏ đến một địa chỉ đã được gắn cứng không còn trả về bất kỳ chú mèo nào nữa. Zombie của chúng ta sẽ không thể ăn thịt mèo và chúng ta sẽ không thể sửa đổi hợp đồng của mình để khắc phục điều đó.

Vì lý do này, việc có các hàm cho phép bạn cập nhật các phần quan trọng của DApp thường rất hợp lý.

Ví dụ: thay vì gắn cứng địa chỉ hợp đồng CryptoKitties vào DApp của mình, có lẽ chúng ta nên có một hàm setKittyContractAddress để cho phép chúng ta thay đổi địa chỉ này trong tương lai phòng trường hợp có điều gì đó xảy ra với hợp đồng CryptoKitties.


# chapter 2 : Ownable contract

`setKittyContractAddress` là hàm `external`, do đó bất kỳ ai cũng có thể gọi nó! Điều đó có nghĩa là bất kỳ ai gọi hàm này đều có thể thay đổi địa chỉ của hợp đồng CryptoKitties, và làm hỏng ứng dụng của chúng ta đối với tất cả người dùng.

Chúng ta muốn có khả năng cập nhật địa chỉ này trong hợp đồng của mình, nhưng chúng ta không muốn tất cả mọi người đều có thể cập nhật nó.

Để xử lý những trường hợp như thế này, một phương pháp phổ biến đã xuất hiện là làm cho các hợp đồng trở thành **Ownable** (Có thể sở hữu) — nghĩa là chúng có một chủ sở hữu (là bạn) người có các đặc quyền riêng.

### Hợp đồng Ownable của OpenZeppelin
Dưới đây là hợp đồng `Ownable` được lấy từ thư viện Solidity của OpenZeppelin. OpenZeppelin là một thư viện gồm các hợp đồng thông minh an toàn và được cộng đồng kiểm duyệt mà bạn có thể sử dụng trong các DApp của riêng mình. Sau bài học này, chúng tôi thực sự khuyên bạn nên xem trang web của họ để học hỏi thêm!

Hãy đọc lướt qua hợp đồng dưới đây. Bạn sẽ thấy một vài thứ mà chúng ta chưa học, nhưng đừng lo lắng, chúng ta sẽ nói về chúng sau.

```solidity
/**
 * @title Ownable
 * @dev Hợp đồng Ownable có một địa chỉ chủ sở hữu (owner) và cung cấp các hàm kiểm soát ủy quyền cơ bản,
 * điều này làm đơn giản hóa việc triển khai "quyền người dùng".
 */
contract Ownable {
  address private _owner;

  event OwnershipTransferred(
    address indexed previousOwner,
    address indexed newOwner
  );

  /**
   * @dev Hàm khởi tạo (constructor) của Ownable thiết lập `owner` (chủ sở hữu) ban đầu của hợp đồng là tài khoản người gửi (sender).
   */
  constructor() internal {
    _owner = msg.sender;
    emit OwnershipTransferred(address(0), _owner);
  }

  /**
   * @return địa chỉ của chủ sở hữu.
   */
  function owner() public view returns(address) {
    return _owner;
  }

  /**
   * @dev Báo lỗi (Throws) nếu được gọi bởi bất kỳ tài khoản nào khác ngoài chủ sở hữu.
   */
  modifier onlyOwner() {
    require(isOwner());
    _;
  }

  /**
   * @return true nếu `msg.sender` là chủ sở hữu của hợp đồng.
   */
  function isOwner() public view returns(bool) {
    return msg.sender == _owner;
  }

  /**
   * @dev Cho phép chủ sở hữu hiện tại từ bỏ quyền kiểm soát hợp đồng.
   * @notice Từ bỏ quyền sở hữu sẽ khiến hợp đồng không có chủ sở hữu.
   * Sẽ không thể gọi các hàm có modifier `onlyOwner` được nữa.
   */
  function renounceOwnership() public onlyOwner {
    emit OwnershipTransferred(_owner, address(0));
    _owner = address(0);
  }

  /**
   * @dev Cho phép chủ sở hữu hiện tại chuyển quyền kiểm soát hợp đồng cho một chủ sở hữu mới (newOwner).
   * @param newOwner Địa chỉ để chuyển quyền sở hữu tới.
   */
  function transferOwnership(address newOwner) public onlyOwner {
    _transferOwnership(newOwner);
  }

  /**
   * @dev Chuyển quyền kiểm soát hợp đồng cho một chủ sở hữu mới (newOwner).
   * @param newOwner Địa chỉ để chuyển quyền sở hữu tới.
   */
  function _transferOwnership(address newOwner) internal {
    require(newOwner != address(0));
    emit OwnershipTransferred(_owner, newOwner);
    _owner = newOwner;
  }
}
```

Một vài điều mới ở đây mà chúng ta chưa thấy trước đó:

- **Hàm khởi tạo (Constructors)**: `constructor()` là một hàm khởi tạo, nó là một hàm đặc biệt tùy chọn. Nó sẽ chỉ được thực thi một lần duy nhất, khi hợp đồng được tạo lần đầu tiên.
- **Hàm sửa đổi (Function Modifiers)**: `modifier onlyOwner()`. Modifiers giống như một nửa hàm, được sử dụng để sửa đổi các hàm khác, thường là để kiểm tra một số yêu cầu trước khi thực thi. Trong trường hợp này, `onlyOwner` có thể được sử dụng để giới hạn quyền truy cập sao cho chỉ chủ sở hữu của hợp đồng mới có thể chạy hàm này. Chúng ta sẽ nói thêm về function modifiers ở chương tiếp theo, và cả việc dấu `_;` kỳ lạ kia làm gì.
- **Từ khóa indexed**: đừng lo lắng về cái này, chúng ta chưa cần đến nó đâu.

Vậy về cơ bản, hợp đồng `Ownable` sẽ thực hiện những việc sau:

1. Khi một hợp đồng được tạo ra, hàm khởi tạo của nó thiết lập chủ sở hữu (owner) là `msg.sender` (người đã triển khai nó)
2. Nó thêm một modifier `onlyOwner`, giúp hạn chế quyền truy cập vào một số hàm nhất định, chỉ cho phép chủ sở hữu được gọi.
3. Nó cho phép bạn chuyển hợp đồng cho một chủ sở hữu mới.

`onlyOwner` là một yêu cầu phổ biến đối với các hợp đồng đến mức hầu hết các DApp Solidity đều bắt đầu bằng việc copy/paste hợp đồng `Ownable` này, và sau đó hợp đồng đầu tiên của họ sẽ kế thừa từ nó.

Vì chúng ta muốn giới hạn `setKittyContractAddress` thành `onlyOwner`, chúng ta cũng sẽ làm điều tương tự cho hợp đồng của mình.

# Chương 3: Function Modifier onlyOwner

Bây giờ hợp đồng cơ sở `ZombieFactory` của chúng ta đã kế thừa từ `Ownable`, chúng ta cũng có thể sử dụng function modifier `onlyOwner` trong `ZombieFeeding`.

Điều này là do cách thức hoạt động của tính kế thừa trong hợp đồng. Hãy nhớ lại:

- `ZombieFeeding` kế thừa `ZombieFactory`
- `ZombieFactory` kế thừa `Ownable`

Do đó `ZombieFeeding` cũng là `Ownable`, và có thể truy cập các hàm (functions) / sự kiện (events) / modifiers từ hợp đồng `Ownable`. Điều này cũng áp dụng cho bất kỳ hợp đồng nào kế thừa từ `ZombieFeeding` trong tương lai.

### Function Modifiers (Hàm sửa đổi)
Function modifier trông giống hệt như một hàm, nhưng sử dụng từ khóa `modifier` thay vì từ khóa `function`. Và nó không thể được gọi trực tiếp như một hàm thông thường — thay vào đó, chúng ta có thể gắn tên của modifier vào cuối định nghĩa của một hàm để thay đổi hành vi của hàm đó.

Hãy cùng xem xét kỹ hơn bằng cách phân tích `onlyOwner`:

```solidity
pragma solidity >=0.5.0 <0.6.0;

/**
 * @title Ownable
 * @dev Hợp đồng Ownable có một địa chỉ chủ sở hữu (owner) và cung cấp các hàm kiểm soát ủy quyền cơ bản,
 * điều này làm đơn giản hóa việc triển khai "quyền người dùng".
 */
contract Ownable {
  address private _owner;

  event OwnershipTransferred(
    address indexed previousOwner,
    address indexed newOwner
  );

  /**
   * @dev Hàm khởi tạo (constructor) của Ownable thiết lập `owner` (chủ sở hữu) ban đầu của hợp đồng là tài khoản người gửi (sender).
   */
  constructor() internal {
    _owner = msg.sender;
    emit OwnershipTransferred(address(0), _owner);
  }

  /**
   * @return địa chỉ của chủ sở hữu.
   */
  function owner() public view returns(address) {
    return _owner;
  }

  /**
   * @dev Báo lỗi (Throws) nếu được gọi bởi bất kỳ tài khoản nào khác ngoài chủ sở hữu.
   */
  modifier onlyOwner() {
    require(isOwner());
    _;
  }

  /**
   * @return true nếu `msg.sender` là chủ sở hữu của hợp đồng.
   */
  function isOwner() public view returns(bool) {
    return msg.sender == _owner;
  }

  /**
   * @dev Cho phép chủ sở hữu hiện tại từ bỏ quyền kiểm soát hợp đồng.
   * @notice Từ bỏ quyền sở hữu sẽ khiến hợp đồng không có chủ sở hữu.
   * Sẽ không thể gọi các hàm có modifier `onlyOwner` được nữa.
   */
  function renounceOwnership() public onlyOwner {
    emit OwnershipTransferred(_owner, address(0));
    _owner = address(0);
  }

  /**
   * @dev Cho phép chủ sở hữu hiện tại chuyển quyền kiểm soát hợp đồng cho một chủ sở hữu mới (newOwner).
   * @param newOwner Địa chỉ để chuyển quyền sở hữu tới.
   */
  function transferOwnership(address newOwner) public onlyOwner {
    _transferOwnership(newOwner);
  }

  /**
   * @dev Chuyển quyền kiểm soát hợp đồng cho một chủ sở hữu mới (newOwner).
   * @param newOwner Địa chỉ để chuyển quyền sở hữu tới.
   */
  function _transferOwnership(address newOwner) internal {
    require(newOwner != address(0));
    emit OwnershipTransferred(_owner, newOwner);
    _owner = newOwner;
  }
}
```

Hãy chú ý đến modifier `onlyOwner` trên hàm `renounceOwnership`. Khi bạn gọi `renounceOwnership`, mã bên trong `onlyOwner` sẽ được thực thi trước tiên. Sau đó, khi nó bắt gặp câu lệnh `_;` trong `onlyOwner`, nó sẽ quay lại và thực thi mã bên trong `renounceOwnership`.

Mặc dù có nhiều cách khác để bạn có thể sử dụng modifier, nhưng một trong những trường hợp sử dụng phổ biến nhất là thêm một kiểm tra `require` nhanh chóng trước khi một hàm được thực thi.

Trong trường hợp của `onlyOwner`, việc thêm modifier này vào một hàm sẽ khiến cho chỉ có chủ sở hữu của hợp đồng (là bạn, nếu bạn đã triển khai nó) mới có thể gọi hàm đó.

**Lưu ý:** Việc trao cho chủ sở hữu những quyền hạn đặc biệt đối với hợp đồng như thế này thường là cần thiết, nhưng nó cũng có thể bị sử dụng cho mục đích xấu. Ví dụ, chủ sở hữu có thể thêm một hàm cửa sau (backdoor) cho phép anh ta chuyển zombie của bất kỳ ai sang cho chính mình!

Do đó, điều quan trọng cần nhớ là: không phải vì một DApp nằm trên Ethereum thì có nghĩa là nó tự động được phi tập trung (decentralized) — bạn thực sự phải đọc toàn bộ mã nguồn để đảm bảo rằng nó không có các quyền kiểm soát đặc biệt của chủ sở hữu mà bạn có khả năng phải lo lắng. Đối với một nhà phát triển, có một sự cân bằng cẩn thận giữa việc duy trì quyền kiểm soát đối với một DApp để bạn có thể sửa các lỗi tiềm ẩn, và việc xây dựng một nền tảng không có chủ sở hữu mà người dùng có thể tin tưởng để bảo mật dữ liệu của họ.

# Chương 4: Gas (Khí gas)

Tuyệt vời! Bây giờ chúng ta đã biết cách cập nhật các phần quan trọng của DApp trong khi vẫn ngăn chặn những người dùng khác can thiệp vào hợp đồng của chúng ta.

Hãy cùng xem xét một khía cạnh khác mà Solidity khá khác biệt so với các ngôn ngữ lập trình khác:

### Gas — nhiên liệu để các DApp Ethereum hoạt động

Trong Solidity, người dùng của bạn phải trả phí mỗi khi họ thực thi một hàm trên DApp của bạn bằng một loại tiền tệ gọi là **gas**. Người dùng mua gas bằng Ether (tiền tệ trên Ethereum), do đó người dùng của bạn phải tiêu tốn ETH để có thể thực thi các hàm trên DApp của bạn.

Lượng gas cần thiết để thực thi một hàm phụ thuộc vào mức độ phức tạp của logic hàm đó. Mỗi một phép toán riêng lẻ có một mức phí gas dựa trên mức độ tài nguyên tính toán cần thiết để thực hiện phép toán đó (ví dụ: việc ghi vào bộ nhớ (storage) tốn kém hơn nhiều so với việc cộng hai số nguyên). Tổng chi phí gas của hàm của bạn là tổng chi phí gas của tất cả các phép toán riêng lẻ trong đó.

Vì việc chạy các hàm gây tốn tiền thật cho người dùng của bạn, nên việc tối ưu hóa code ở Ethereum quan trọng hơn nhiều so với các ngôn ngữ lập trình khác. Nếu mã của bạn cẩu thả, người dùng của bạn sẽ phải trả một khoản phí cao hơn để thực thi các hàm của bạn — và điều này có thể lên tới hàng triệu đô la phí không cần thiết trên hàng ngàn người dùng.

### Tại sao gas lại cần thiết?

Ethereum giống như một chiếc máy tính lớn, chậm chạp nhưng cực kỳ an toàn. Khi bạn thực thi một hàm, mọi node (nút) trên mạng đều cần chạy cùng một hàm đó để xác minh kết quả đầu ra của nó — hàng ngàn node xác minh mọi quá trình thực thi hàm là điều làm cho Ethereum trở nên phi tập trung (decentralized), và dữ liệu của nó bất biến cũng như chống được kiểm duyệt.

Những người tạo ra Ethereum muốn đảm bảo rằng ai đó không thể làm tắc nghẽn mạng bằng một vòng lặp vô hạn, hoặc chiếm đoạt toàn bộ tài nguyên mạng bằng những tính toán thực sự chuyên sâu. Vì vậy, họ đã thiết kế sao cho các giao dịch không hề miễn phí, và người dùng phải trả tiền cho cả thời gian tính toán cũng như không gian lưu trữ.

**Lưu ý:** Điều này không nhất thiết đúng với các blockchain khác, chẳng hạn như những blockchain mà các tác giả CryptoZombies đang xây dựng tại Loom Network. Có lẽ sẽ không bao giờ là hợp lý nếu chạy một trò chơi như World of Warcraft trực tiếp trên mainnet của Ethereum — chi phí gas sẽ đắt đến mức không thể chấp nhận được. Nhưng nó có thể chạy trên một blockchain với thuật toán đồng thuận khác. Chúng ta sẽ nói thêm về các loại DApp mà bạn muốn triển khai trên Loom so với trên mainnet của Ethereum trong một bài học sau.

### Gộp Struct (Struct packing) để tiết kiệm gas

Trong Bài 1, chúng ta đã đề cập rằng có các kiểu `uint` khác: `uint8`, `uint16`, `uint32`, v.v.

Thông thường, không có lợi ích gì khi sử dụng các kiểu con này vì Solidity dành sẵn 256 bit bộ nhớ bất kể kích thước của `uint`. Ví dụ, sử dụng `uint8` thay vì `uint` (`uint256`) sẽ không giúp bạn tiết kiệm được chút gas nào.

Nhưng có một ngoại lệ cho điều này: bên trong các `struct` (cấu trúc).

Nếu bạn có nhiều `uint` bên trong một `struct`, việc sử dụng `uint` có kích thước nhỏ hơn nếu có thể sẽ cho phép Solidity gộp các biến này lại với nhau để chiếm ít không gian lưu trữ hơn. Ví dụ:

```solidity
struct NormalStruct {
  uint a;
  uint b;
  uint c;
}

struct MiniMe {
  uint32 a;
  uint32 b;
  uint c;
}

// `mini` sẽ tốn ít gas hơn `normal` nhờ vào việc struct packing (gộp struct)
NormalStruct normal = NormalStruct(10, 20, 30);
MiniMe mini = MiniMe(10, 20, 30); 
```

Vì lý do này, bên trong một `struct`, bạn sẽ muốn sử dụng các kiểu số nguyên con nhỏ nhất có thể đáp ứng được nhu cầu của mình.

Bạn cũng sẽ muốn nhóm các kiểu dữ liệu giống nhau lại với nhau (nghĩa là đặt chúng cạnh nhau trong `struct`) để Solidity có thể giảm thiểu không gian lưu trữ cần thiết. Ví dụ, một struct với các trường `uint c; uint32 a; uint32 b;` sẽ tốn ít gas hơn một struct với các trường `uint32 a; uint c; uint32 b;` bởi vì các trường `uint32` đã được nhóm lại cùng nhau.

# Chương 5: Đơn vị thời gian (Time Units)

Thuộc tính `level` (cấp độ) khá dễ hiểu. Sau này, khi chúng ta tạo ra một hệ thống chiến đấu, những zombie thắng nhiều trận chiến hơn sẽ tăng cấp dần theo thời gian và có quyền truy cập vào nhiều khả năng hơn.

Thuộc tính `readyTime` (thời gian sẵn sàng) cần được giải thích thêm một chút. Mục tiêu là thêm vào một "thời gian hồi chiêu" (cooldown period), tức là khoảng thời gian mà một zombie phải chờ sau khi ăn hoặc tấn công trước khi nó được phép ăn / tấn công lại. Nếu không có điều này, zombie có thể tấn công và nhân bản 1.000 lần mỗi ngày, điều này sẽ làm cho trò chơi trở nên quá dễ dàng.

Để theo dõi xem một zombie phải chờ bao lâu cho đến khi nó có thể tấn công lại, chúng ta có thể sử dụng các đơn vị thời gian của Solidity.

### Các đơn vị thời gian (Time units)

Solidity cung cấp một số đơn vị tích hợp sẵn để xử lý thời gian.

Biến `now` sẽ trả về *unix timestamp* (dấu thời gian unix) hiện tại của block mới nhất (là số giây đã trôi qua kể từ ngày 1 tháng 1 năm 1970). Dấu thời gian unix tại thời điểm tôi viết bài này là `1515527488`.

**Lưu ý:** Unix time theo truyền thống được lưu trữ trong một số 32-bit. Điều này sẽ dẫn đến vấn đề "Năm 2038", khi các dấu thời gian unix 32-bit sẽ bị tràn bộ nhớ (overflow) và làm hỏng rất nhiều hệ thống cũ. Vì vậy, nếu chúng ta muốn DApp của mình tiếp tục chạy trong 20 năm nữa, chúng ta có thể sử dụng số 64-bit để thay thế — nhưng đổi lại người dùng của chúng ta sẽ phải tốn nhiều gas hơn khi sử dụng DApp. Quyết định thiết kế là ở bạn!

Solidity cũng chứa các đơn vị thời gian như `seconds` (giây), `minutes` (phút), `hours` (giờ), `days` (ngày), `weeks` (tuần) và `years` (năm). Chúng sẽ chuyển đổi thành một `uint` tương ứng với số giây trong khoảng thời gian đó. Ví dụ `1 minutes` là 60, `1 hours` là 3600 (60 giây x 60 phút), `1 days` là 86400 (24 giờ x 60 phút x 60 giây), v.v.

Dưới đây là một ví dụ về cách các đơn vị thời gian này có thể trở nên hữu ích:

```solidity
uint lastUpdated;

// Gán `lastUpdated` bằng `now` (thời điểm hiện tại)
function updateTimestamp() public {
  lastUpdated = now;
}

// Sẽ trả về `true` nếu 5 phút đã trôi qua kể từ khi hàm `updateTimestamp` 
// được gọi, và trả về `false` nếu 5 phút chưa trôi qua
function fiveMinutesHavePassed() public view returns (bool) {
  return (now >= (lastUpdated + 5 minutes));
}
```

Chúng ta có thể sử dụng các đơn vị thời gian này cho tính năng hồi chiêu của Zombie.

**Lưu ý:** Việc sử dụng `uint32(...)` là cần thiết bởi vì `now` mặc định sẽ trả về một `uint256`. Vì vậy, chúng ta cần phải ép kiểu (chuyển đổi rõ ràng) nó sang `uint32`.

`now + cooldownTime` sẽ bằng với dấu thời gian unix hiện tại (tính bằng giây) cộng với số giây của 1 ngày — kết quả này sẽ bằng với dấu thời gian unix của đúng 1 ngày sau tính từ bây giờ. Sau này, chúng ta có thể so sánh xem `readyTime` của zombie này có lớn hơn `now` hay không để biết liệu đã trôi qua đủ thời gian để có thể sử dụng lại zombie đó chưa.

Chúng ta sẽ triển khai tính năng giới hạn các hành động dựa trên `readyTime` trong chương tiếp theo.

# Chương 6: Thời gian hồi chiêu của Zombie (Zombie Cooldowns)

Bây giờ chúng ta đã có thuộc tính `readyTime` trong struct `Zombie`, hãy chuyển sang file `zombiefeeding.sol` và triển khai bộ đếm thời gian hồi chiêu.

Chúng ta sẽ sửa đổi hàm `feedAndMultiply` sao cho:

- Việc ăn sẽ kích hoạt thời gian hồi chiêu của zombie, và
- Các zombie không thể ăn thịt mèo cho đến khi thời gian hồi chiêu của chúng kết thúc.

Điều này sẽ giúp cho các zombie không thể cứ ăn thịt mèo không giới hạn và nhân bản cả ngày được. Trong tương lai, khi chúng ta thêm chức năng chiến đấu, chúng ta sẽ thiết lập sao cho việc tấn công các zombie khác cũng phụ thuộc vào thời gian hồi chiêu này.

Đầu tiên, chúng ta sẽ định nghĩa một số hàm trợ giúp (helper functions) cho phép chúng ta thiết lập và kiểm tra `readyTime` của một zombie.

### Truyền các struct dưới dạng tham số (arguments)

Bạn có thể truyền một con trỏ `storage` trỏ tới một struct như là một tham số vào trong một hàm `private` hoặc `internal`. Điều này rất hữu ích, ví dụ, để truyền các struct `Zombie` của chúng ta qua lại giữa các hàm với nhau.

Cú pháp trông như thế này:

```solidity
function _doStuff(Zombie storage _zombie) internal {
  // thực hiện các thao tác với _zombie
}
```

Bằng cách này, chúng ta có thể truyền một tham chiếu (reference) tới zombie của mình vào trong một hàm, thay vì phải truyền ID của zombie rồi mới tìm kiếm nó.

# Chương 7: Các hàm Public & Bảo mật (Public Functions & Security)

Bây giờ chúng ta hãy sửa đổi `feedAndMultiply` để tính đến bộ đếm thời gian hồi chiêu của chúng ta.

Nhìn lại hàm này, bạn có thể thấy chúng ta đã đặt nó là `public` trong bài học trước. Một phương pháp bảo mật quan trọng là kiểm tra tất cả các hàm `public` và `external` của bạn, và thử nghĩ ra những cách mà người dùng có thể lạm dụng chúng. Hãy nhớ rằng — trừ khi các hàm này có một modifier như `onlyOwner`, nếu không thì bất kỳ người dùng nào cũng có thể gọi chúng và truyền cho chúng bất kỳ dữ liệu nào mà họ muốn.

Xem xét lại hàm cụ thể này, người dùng có thể gọi trực tiếp hàm và truyền vào bất kỳ `_targetDna` hoặc `_species` nào họ muốn. Điều này nghe không giống một trò chơi cho lắm — chúng ta muốn họ tuân theo các quy tắc của mình!

Khi kiểm tra kỹ hơn, hàm này chỉ cần được gọi bởi `feedOnKitty()`, vì vậy cách dễ nhất để ngăn chặn các kiểu khai thác này là đặt nó thành `internal`.

# chương 8: More on Function Modifiers

Function modifiers có tham số (Arguments)
Trước đây chúng ta đã xem một ví dụ đơn giản về onlyOwner. Nhưng các function modifier cũng có thể nhận các tham số truyền vào y như một hàm bình thường. Ví dụ:

```solidity
// Một mapping để lưu trữ tuổi của người dùng:
mapping (uint => uint) public age;

// Modifier yêu cầu người dùng này phải lớn hơn một độ tuổi nhất định:
modifier olderThan(uint _age, uint _userId) {
  require(age[_userId] >= _age);
  _;
}

// Phải trên 16 tuổi mới được lái xe ô tô (ít nhất là ở Mỹ).
// Chúng ta có thể gọi modifier `olderThan` với các tham số truyền vào như sau:
function driveCar(uint _userId) public olderThan(16, _userId) {
  // Một số logic của hàm
}
```

Bạn có thể thấy ở đây modifier olderThan nhận các tham số giống hệt như một hàm. Và hàm driveCar đã truyền các tham số của nó vào cho modifier đó.


# chương 9 : Zombie Modifiers

chương này chỉ cần lưu ý cách một hàm chứ modifier 
ví dụ function .... external modifier {}

và thêm calldata giống như memory nhưng chỉ dùng được với hàm đc khai báo là external

# chương 10: Saving gas with "view" Functions

Hàm "view" thì sẽ không tốn gas
Lý do :
1. hàm view chỉ đọc dữ liệu từ blockchain chứ không thay đổi trạng thái (state) của mạng lười (không ghi, không sửa , không xóa)
2. Cơ chế hoạt động : Thay vì tạo ra một giao dịch (transaction) rồi phát tán lên toàn mạng lưới để hàng ngàn nút (node) cùng xử lý, Web3.js chỉ cần truy vấn dữ liệu trực tiếp từ node Ethereum cục bộ (hoặc node của bên dịch vụ như Infura) đang kết nối với trình duyệt của bạn. Việc này giống như bạn đọc một cuốn sách có sẵn trên kệ vậy – nhanh chóng và miễn phí.

**Lưu ý** Hàm view chỉ MIỄN PHÍ khi nó được gọi từ bên ngoài (External call). Nếu một hàm view được gọi nội bộ (internally) bởi một hàm khác không phải là hàm view trong cùng một contract, nó vẫn sẽ tính phí gas bình thường. Lý do là vì hàm "cha" kia đã tạo ra một giao dịch trên blockchain, nên toàn bộ các bước tính toán bên trong (bao gồm cả hàm view được gọi ké) đều phải được mọi node trên mạng lưới xác thực.


# chương 11: Storage

GHi dữ liệu bằng Storage thì sẽ rất đắt đỏ vì nó được lưu trữ vĩnh viễn trên blockchain. Hàng ngàn nodes sẽ phải tải dữ liệu đó về và lưu trữ trên ổ nhớ của họ mãi mãi

Để tiết kiệm nhất : Hãy tránh ghi storage vào nếu thực sự cần thiết, tuyệt đối bắt bắt buộc


Điều này dẫn đến một tư duy lập trình khá "ngược đời" so với các ngôn ngữ truyền thống:

Trong các ngôn ngữ như JavaScript hay Python, việc duyệt vòng lặp qua một mảng dữ liệu lớn rất tốn tài nguyên, nên ta thường lưu sẵn kết quả vào một biến để lần sau tìm kiếm cho nhanh.

Nhưng trong Solidity, việc chạy vòng lặp để khởi tạo lại một mảng mới trong memory mỗi khi gọi hàm lại RẺ HƠN RẤT NHIỀU so với việc lưu mảng đó trong storage. Đặc biệt, nếu vòng lặp đó nằm trong một hàm external view, người dùng của bạn sẽ không tốn một đồng xu nào cả!

Cách khai báo mảng trong Memory
Từ khóa memory giúp bạn tạo ra một mảng tạm thời ngay bên trong hàm. Mảng này sẽ "tan biến" ngay sau khi hàm kết thúc và không để lại dấu vết gì trên blockchain.

Cú pháp khai báo như sau:

```solidity
function getArray() external pure returns(uint[] memory) {
  // Khởi tạo một mảng mới trong memory với độ dài cố định là 3
  uint[] memory values = new uint[](3);

  // Gán giá trị cho mảng
  values[0] = 1;
  values[1] = 2;
  values[2] = 3;

  return values;
}
```
Lưu ý cực kỳ quan trọng về Mảng trong Memory:
Mảng memory bắt buộc phải khai báo kèm theo độ dài cố định (như số 3 trong ví dụ trên). Tại thời điểm này trong Solidity, mảng memory không thể thay đổi kích thước (không thể dùng hàm .push() để nhét thêm phần tử như mảng storage được).

# for loop
Chương này mang đến một trong những bài học đắt giá nhất về tư duy lập trình Smart Contract: Tại sao cấu trúc dữ liệu đơn giản nhất đôi khi lại là phương án tồi tệ nhất về mặt chi phí (Gas)?

Hãy cùng phân tích tại sao việc dùng vòng lặp for để quét toàn bộ mảng dữ liệu lại tối ưu hơn việc dùng mapping trực tiếp trong trường hợp này.

Vấn đề của cách làm ngây thơ (Naive Approach)
Nếu dùng cách tiếp cận thông thường giống như các ngôn ngữ Web2, bạn sẽ tạo một mapping để lưu danh sách zombie của chủ sở hữu:

```solidity
mapping (address => uint[]) public ownerToZombies;
```

Mọi thứ trông có vẻ rất mượt mà cho đến khi game của chúng ta có chức năng Chuyển nhượng Zombie (Transfer) từ người này sang người khác. Để chuyển một Zombie nằm ở giữa mảng của người cũ sang người mới, Solidity sẽ phải:

Xóa Zombie đó khỏi mảng của người cũ.

Dịch chuyển (Shift) tất cả các Zombie đứng sau nó lên một vị trí để lấp đầy khoảng trống.

Giảm độ dài của mảng đi 1.

💸 Cơn ác mộng về Gas: > Việc dịch chuyển này yêu cầu bạn phải ghi đè dữ liệu (Write to Storage) lên từng phần tử bị dịch chuyển. Nếu một người chơi có 50 con zombie và họ bán con đầu tiên, hợp đồng sẽ phải thực hiện 49 phép ghi vào storage. Chi phí giao dịch lúc này sẽ cực kỳ đắt đỏ và biến động liên tục, khiến người dùng không thể biết trước mình sẽ tốn bao nhiêu tiền phí.
(Lưu ý: Bạn có thể đổi vị trí phần tử cuối cùng lên thay thế để không phải dịch mảng, nhưng điều đó sẽ làm đảo lộn hoàn toàn thứ tự hiển thị quân đội zombie của người chơi).

Giải pháp: Chuyển độ phức tạp từ hàm "Ghi" sang hàm "Đọc"
Vì hàm external view hoàn toàn miễn phí khi gọi từ bên ngoài, chúng ta sẽ bỏ luôn cái mapping (address => uint[]) tốn kém kia đi. Thay vào đó:

Khi Chuyển nhượng (Transfer): Chúng ta chỉ cần thay đổi địa chỉ của chủ sở hữu trong một biến duy nhất (ví dụ: zombieToOwner[zombieId] = newOwner). Thao tác này cực rẻ vì chỉ tốn 1 lần ghi storage.

Khi Hiển thị danh sách (getZombiesByOwner): Chúng ta sẽ dùng vòng lặp for chạy xuyên suốt toàn bộ mảng zombies tổng của trò chơi, nhặt ra những con zombie nào thuộc về _owner và nhét vào một mảng tạm thời trong memory.

Cú pháp vòng lặp for trong Solidity
Cú pháp for trong Solidity gần như tương đồng hoàn toàn với JavaScript hoặc C++. Dưới đây là ví dụ tìm các số chẵn từ 1 đến 10 để bạn hình dung:

```solidity
function getEvens() pure external returns(uint[] memory) {
  // Khởi tạo mảng trong memory với độ dài cố định là 5
  uint[] memory values = new uint[](5);
  
  // Biến đếm index cho mảng 'values'
  uint counter = 0;

  // Vòng lặp for từ 1 đến 10
  for (uint i = 1; i <= 10; i++) {
    // Nếu i là số chẵn
    if (i % 2 == 0) {
      values[counter] = i; // Gán vào mảng memory
      counter++;           // Tăng index của mảng lên 1
    }
  }
  return values;
}
```
Tư duy này có vẻ hơi "ngược đời" so với lập trình truyền thống (chấp nhận chạy vòng lặp duyệt qua một danh sách lớn thay vì truy vấn index trực tiếp), nhưng trên Blockchain, nó giúp người chơi của bạn tiết kiệm được rất nhiều tiền thật!