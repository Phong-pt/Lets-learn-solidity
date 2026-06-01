## Chapter 2

Solidity's code is encapsulated in **contracts** (được đóng gói trong cái hàm contract này), Tất cả các biến hoặc hàm đều được viết ở trong contract này nó chính xác là điểm khởi đầu của một hợp đồng.

một hợp đồng trống sẽ có dạng :

```solidity
contract HelloWorld{

}
```


Tất cả các solidity code đều bắt đầu bằng **pragma solidity**, chúng ta có thể lựa chọn version complie bằng cách :
- `^version`(ex 0.6.0) tức là sử dụng complie version lớn hơn version 0.6.0 đều được, còn bé hơn thì không
- `< version` : sử dụng compile version **nhỏ hơn** version chỉ định, lớn hơn không được
- `>= version < version` : có thể sử dụng compile ở **khoảng giữa hai version**



## Chapter 3: State Variable and Integer

**State variable** are permanently stored in contract storage. (đucợ lưu trữ vĩnh viễn trong contract). Có nghĩa là nó được ghi hẳn vào chuỗi khối Ethereum blockchain. Nghĩ tới chúng giống như viết một DB (Database)

**Unsigned Integer**: uint , Có nghĩa là value không được âm(unsigned int : số nguyên không dấu). Nó cũng có hàm **int** cho signed integers(số nguyên có dấu : âm hoặc dương đều được).

>[!NOTE] thực ra uint là bí danh(alias) của uint256, một 256-bit unsigned integer. Chúng ta có thể khai báo uints với ít bits hơn -uint8,uint128,uint64,..etc. Nhìn chung chỉ nên sử dụng kiểu uint, trừ những trường hợp cụ thể thì lúc đấy hãng dùng kiểu khác.

Ex khai báo:
```solidity
contract Example{
    uint myUnsignedInteger = 100;
}
```


## chapter 4: Math Operations

các phép toán trong solidity khá giống các ngôn ngữ khác
- phép cộng (addition): x+y
- phép trừ (subtraction): x-y
- Multiplication : x*y
- Division : x / y
- chia có dư(modulus/remainder): x % y

nó cũng hỡ trợ **exponential operator** tức là toán tử mũ: 5**2 equal 5^2

## chapter 5: Structs
thi thoảng chúng ta cần nhiều kiểu data phức tạp. Solidity cung cấp structs
ex: 

```solidity
    struct Person{
        uint age;
        string name;    // day la mot kieu du liệu string tuong tu ngon ngu khác
    }
```
>[!NOTE] HIểu đơn giản thì nó giống như oop, đây là các kiểu dữ liệu của class Person
Struct cho phép chúng ta tạo nhiều biến phức tạp với nhiều thuộc tính khác nhau.


## chapter 6: Array

Khi mà chúng ta muốn lưu trữ nhiều biến hoặc gì đó , thì chúng ta có array(mảng), Có 2 loại mảng trong solidity:
- Dynamic(động): Bạn Hiểu thì nó không bị giằng buộc bởi size, bạn có thể lưu trữ bao nhiêu biến hoặc gì đó vào mảng này cũng đc 
- Fixed (tĩnh): Còn mảng này thì nó sẽ bị **Fixed** , bị giới hạn việc lưu trữ

ex

```solidity
uint[2] fixedArray;
//mảng này chỉ chứa 2 phần tử
string[5] stringArray:
//mảng này cũng chỉ được phép chứa 5 phần tử , tuy nhiên đặc biệt ở chỗ là 5 phần tử là string
uint[] dynamicArray:
//mảng này thì thêm bao nhiêu phần tử thoải mái
```

và Bạn cũng có thể tạo array của struct. Sử dụng struct person ở chapter 5, ta có:

```solidity
Person[] people; \\đây là một mãng động 
\\cú pháp thì tên struct + kiểu mảng liền kề cách ra và tên mảng(ở đây là people) 
```

bạn cũng có thể declare (gán cho) một mảng là **public** , Và Solidity sẽ tự động create phương thức **getter** cho nó. The syntax nhìn như sau : 
```solidity
Person[] public people;
```   

các contract khác có thể đọc được dữ liệu từ mảng này, nhưng không thể ghi vào mảng này. Đây là một cách hữu ích để lưu trữ data một cách public trong hợp đồng của bạn


## chapter 7: Function declarations
Một funtion decleration nhìn như sau:
```solidity
funtion eatHamburgers(string memory _name, uint _amout) public {

}
```

Đây là một hàm tên là eatHambergers , được truyền 2 tham số (parameter) là **string** và **uint**. Lưu ý rằng phamj vi hiểm thị của hàm này chúng ta đnag để là public , chúng ta cũng cung cấp instructions (hướng dẫn) nơi mà _name sẽ được lưu trữ - trong **memory**. Điều này cần thiết cho các kiểu dữ liệu như : arrays, structs, mapping, và strings 

vậy thì loại tham chiếu là gì (reference type) ?

có 2 cách mà chúng ta có thể truyền một tham số trong solidity:
- Truyền theo value: có nghĩa là solidity sẽ tạo ra một bản copy của giá trị tham số và truyền vào hàm. Điều này cho phép hàm của bạn có thể điều chỉnh giá trị mà không cần lo lắng sẽ làm thay đổi giá trị gốc

- truyền tham chiếu(reference): Có nghĩa là hàm của bạn được gọi với một tham chiếu tới biến gốc. Do đó nếu hàm của bạn thay đổi biến mà nó nhận , thì biến gốc cũng sẽ bị thay đổi.

>[!NOTE] Để thuận tiện (ko yêu cầu), tên biến tham số hàm thường bắt đầu bằng  dấu gạch chân dưới (_), để phân biệt chúng với **biến toàn cục(global variables)**.

chúng ta sẽ gọi hàm trên như sau
```solidity
eatHamburgers("victor", 100);
```


## chapter 8 : Working with Structs and array

tiếp tục với ví dụ struct person ở chapter 5 và tạo mảng people ở chapter 6. Chúng ta sẽ học cách tạo một person mới và thêm chúng vào mảng people

```solidity
//create new person
Person satoshi = Person(172,"satoshi");

// add satoshi into array
people.push(satoshi);
```

chúng ta cũng có thể kết hợp 2 dòng code trên thành 1 dòng

```solidity
people.push(Person(16,"nick");)
```

array.push() thì sẽ add item vào cuối mảng


## chapter 9: Private/Public Functions

- Solidity sẽ luôn mặc định Hàm của bạn là public , dẫn đến ai cũng có thể gọi tới và thực thi code trong hàm đó
- điều đó dễ gây ra lỗ hổng và bị hacker tấn công
- vì vậy nên tập set **private** mọi lúc và set public khi bạn muốn cho mọi người thấy

cách để khai báo một hàm private:

```solidity
uint[] numbers;

function _AddToArray(uint _numbers) private {
    numbers.push(_number);
}
```

Khi set private, điều này có nghĩa là chỉ các hàm trong contract của mình mới có thể gọi và thực thi code của hàm private này. Và bạn chú ý thì ta để private sau tên hàm, tham số truyền như bthg với _ . 1 _ dưới number để phân biệt với biến toàn cục, 1 _ là để phân biệt rằng đây là hàm private


## chapter 10: More on functions

trong chapter này ta sẽ tìm hiểu về function **return values** and function modify 

### return value

để trả về giá trị từ một hàm. Thì khai báo giống như này:
```solidity
string greeting = "What's up dog";

function sayHello() public returns (string memory) {
  return greeting;
}
```

trong solidity , khai báo hàm gồm kiểu dữ liệu trả về. Như bên trên là string

### funtion modify

hàm ví dụ ở return value thực chất nó không thay đổi gì cả 

vậy nên ta có thể khai báo hàm với từ khóa **view** , có nghĩa chỉ view chứ ko modify gì cả

```solidity
function sayHello() public view returns (string memory) {}
```

nó cũng có hàm **pure**, có nghĩa rằng bạn thậm chí còn không access bất cứ dữ liệu nào trong app, ví dụ:

```solidity
function _multiply(uint a, uint b) private pure returns (uint) {
  return a * b;
}
```

hàm trên còn thậm chí chả làm thay đổi trạng thái (state) của app, nó chỉ trả về  dựa vào chính tham số của hàm , vậy nên trong trường hợp này chúng ta để nó là pure


## chapter 11 : Keccak256 and Typecasting
### keccak256
trong bài này họ đề cập tới việc muốn hàm generateRandomDna trả về một giá trị (Gần như:semi) ngẫu nhiên. Nên họ sử dụng **keccak256**

- keccak256 là một version của SHA3, 1 hash function cơ bản là ánh xạ 1 input thành 1 số 256-bit hexa ngẫu nhiên . 1 thay đổi nhỏ trong data sẽ dẫn tới mã hash thay đổi rất nhiều

- keccak256 yêu cầu dữ liệu đầu vào là `bytes`, có nghĩa là tất cả giá trị trước khi đưa vào sử dụng thì phải chuyển qua byte và chuyển bằng cách `keccak256(abi.encodePacked("aaaaa"))`

- tạo ra một số ngẫu nhiên là một vấn đề **bảo mật** , tuy nhiên thì cryptoZOmbie chưa cần điều đó lắm. 

### Typecasting

```solidity
uint8 a = 5;
uint b = 6;
// throws an error because a * b returns a uint, not uint8:
uint8 c = a * b;
// we have to typecast b as a uint8 to make it work:
uint8 c = a * uint8(b);
```

## chapter12
## chapter13: Events
events là cách hợp đồng của bạn truyền đạt những gì diễn ra trên blockchain lên giao diện người dùng của bạn. Nó có thể Lắng cho vài event nhất định và hành động khi nó xảy ra. Tức là nó ghi lại log lên blockchain

ví dụ :
```solidity
// declare the event
event IntegersAdded(uint x, uint y, uint result);

function add(uint _x, uint _y) public returns (uint) {
  uint result = _x + _y;
  // fire an event to let the app know the function was called:
  emit IntegersAdded(_x, _y, result);
  return result;
}
```

nhưu code ở trên thì app front-end sẽ listen được event. Và đoạn java ở dưới đây là để nghe event. Tức là ko có nó thì ko biết event nói gì dù nó đã dudocj gọi rồi(event được bắn lên blockchain log)

