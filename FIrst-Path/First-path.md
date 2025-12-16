## Chapter 2

Solidity's code is encapsulated in **contracts** (được đóng gói trong cái hàm contract này), Tất cả các biến hoặc hàm đều được viết ở trong contract này nó chính xác là điểm khởi đầu của một hợp đồng.

một hợp đồng trống sẽ có dạng :

contract HelloWorld{

}


Tất cả các solidity code đều bắt đầu bằng **pragma solidity**, chúng ta có thể lựa chọn version complie bằng cách :
- ^version(ex 0.6.0) tức là sử dụng complie version lớn hơn version 0.6.0 đều được, còn bé hơn thì không
- <version tức là sử dụng complie version nhỏ hơn, lớn hơn không được
- >=version <version thì có thể sử dụng complie ở khoảng giữa này


## Chapter 3: State Variable and Integer

**State variable** are permanently stored in contract storage. (đucợ lưu trữ vĩnh viễn trong contract). Có nghĩa là nó được ghi hẳn vào chuỗi khối Ethereum blockchain. Nghĩ tới chúng giống như viết một DB (Database)

**Unsigned Integer**: uint , Có nghĩa là value không được âm(unsigned int : số nguyên không dấu). Nó cũng có hàm **int** cho signed integers(số nguyên có dấu : âm hoặc dương đều được).

>[!NOTE] thực ra uint là bí danh(alias) của uint256, một 256-bit unsigned integer. Chúng ta có thể khai báo uints với ít bits hơn -uint8,uint128,uint64,..etc. Nhìn chung chỉ nên sử dụng kiểu uint, trừ những trường hợp cụ thể thì lúc đấy hãng dùng kiểu khác.

Ex khai báo:
contract Example{
    uint myUnsignedInteger = 100;
}
