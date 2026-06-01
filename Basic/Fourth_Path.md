# Fourth_Path

## chapter 1 
payable : Hiểu nôm na đơn giản là cho phép gọi đến một hàm để có thể nhận tiền và chuyển tiền

Hãy dành một chút thời gian để ngẫm về điều này. Khi bạn gọi một hàm API trên một máy chủ web thông thường, bạn không thể gửi kèm đô la Mỹ cùng với lượt gọi hàm đó — và bạn cũng chẳng thể gửi Bitcoin.

Nhưng với Ethereum, bởi vì tiền mặt (Ether), dữ liệu (nội dung giao dịch), và bản thân mã nguồn của hợp đồng thông minh đều cùng tồn tại trên mạng lưới Ethereum, bạn hoàn toàn có thể vừa gọi một hàm, vừa gửi tiền vào hợp đồng đó cùng một lúc.

Điều này mở ra những logic cực kỳ thú vị, ví dụ như bắt buộc phải trả một khoản phí nhất định cho hợp đồng thì mới được phép thực thi một hàm nào đó.


## chapter 2
Withdraw : Hiểm nôm na đơn giản là sau khi contract nhận tiền thì nó sẽ không biết làm gì nữa (ý là tiền sẽ bị mắc kẹt trong tài khoản), thì tk withdraw này giúp rút tiền ra khỏi hợp đồng



Bạn có thể viết một hàm để rút Ether từ hợp đồng như sau:

```Solidity
contract GetPaid is Ownable {
  function withdraw() external onlyOwner {
    address payable _owner = address(uint160(owner()));
    _owner.transfer(address(this).balance);
  }
}

```
Lưu ý rằng chúng ta đang sử dụng hàm owner() và bộ bổ trợ onlyOwner từ hợp đồng Ownable (giả định là hợp đồng này đã được import trước đó).

Một điều cực kỳ quan trọng cần lưu ý là: bạn không thể chuyển Ether đến một địa chỉ trừ khi địa chỉ đó thuộc kiểu dữ liệu address payable (địa chỉ có thể thanh toán). Tuy nhiên, biến _owner lại đang có kiểu dữ liệu là uint160, nghĩa là chúng ta phải ép kiểu nó một cách tường minh sang dạng address payable.

Một khi bạn đã ép kiểu địa chỉ từ uint160 sang address payable, bạn có thể chuyển Ether đến địa chỉ đó bằng cách sử dụng hàm .transfer(). Lúc này, cụm address(this).balance sẽ trả về tổng số dư đang được lưu trữ trên hợp đồng. Ví dụ: nếu có 100 người dùng, mỗi người trả 1 Ether vào hợp đồng của chúng ta, thì address(this).balance sẽ bằng 100 Ether.

Bạn có thể sử dụng hàm transfer để gửi tiền đến bất kỳ địa chỉ Ethereum nào. Chẳng hạn, bạn có thể viết một hàm để hoàn trả lại Ether cho msg.sender (người gọi hàm) nếu họ trả thừa tiền cho một món đồ:

Solidity
uint itemFee = 0.001 ether;
msg.sender.transfer(msg.value - itemFee);
Hoặc trong một hợp đồng có bên mua và bên bán, bạn có thể lưu địa chỉ của người bán vào bộ nhớ lưu trữ (storage). Sau đó, khi có ai đó mua hàng của họ, bạn sẽ chuyển thẳng số tiền mà người mua đã trả cho người bán: seller.transfer(msg.value).

💡 Giải thích thêm một chút về mặt kỹ thuật:
address(this).balance: this đại diện cho chính hợp đồng hiện tại. Câu lệnh này lấy ra toàn bộ số dư (tính bằng Wei) mà hợp đồng đang nắm giữ.

msg.value: Là lượng Ether (tính bằng Wei) mà người dùng thực sự gửi kèm theo khi họ gọi hàm.

Ép kiểu (Explicit Casting): Đoạn code trên thuộc phiên bản Solidity cũ (thường là CryptoZombies đời đầu). Ở các phiên bản Solidity mới hơn (v0.8.0 trở lên), việc ép kiểu đã được đơn giản hóa thẳng thành payable(owner()) thay vì phải đi đường vòng qua uint160.