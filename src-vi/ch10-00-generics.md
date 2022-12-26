# Generic Types, Traits, and Lifetimes

Mỗi ngôn ngữ lập trình đều có các công cụ để xử lý hiệu quả việc trùng lặp các
khái niệm. Trong Rust, một trong những công cụ đó là *generics*: là các đại diện
trừu tượng cho các kiểu cụ thể hoặc các tính chất khác. Chúng ta có thể biểu
thị hành vi của generics hoặc cách chúng liên quan đến các generics khác mà không cần biết những gì sẽ được đặt vào chỗ đó khi biên dịch và chạy code.

Các hàm có thể nhận các tham số của một kiểu generic, thay vì một kiểu cụ thể
như `i32` hoặc `String`, một cách tương tự như một hàm nhận các tham số với các
giá trị không xác định để chạy cùng mã trên nhiều giá trị cụ thể khác nhau. Thực
ra, chúng ta đã sử dụng generics trong chương 6 với `Option<T>`, chương 8 với
`Vec<T>` và `HashMap<K, V>`, và chương 9 với `Result<T, E>`. Trong chương này,
bạn sẽ khám phá cách định nghĩa các kiểu, các hàm và các phương thức của riêng
bạn với generics!

Đầu tiên, chúng ta sẽ xem lại cách trích xuất một hàm để giảm bớt sự trùng lặp
mã. Chúng ta sẽ sử dụng cùng một kỹ thuật để tạo một hàm generic từ hai hàm khác
chỉ khác nhau ở các kiểu của các tham số của chúng. Chúng ta cũng sẽ giải thích
cách sử dụng các kiểu generic trong các định nghĩa struct và enum.

Sau đó, bạn sẽ học cách sử dụng *traits* để định nghĩa hành vi một cách generic.
Bạn có thể kết hợp traits với các kiểu generic để hạn chế một kiểu generic để
chấp nhận chỉ những kiểu có hành vi nhất định, thay vì chỉ bất kỳ kiểu nào.

Cuối cùng, chúng ta sẽ thảo luận về *lifetimes*: một loại generics cung cấp
thông tin cho trình biên dịch về cách tham chiếu liên quan đến nhau. Lifetimes
cho phép chúng ta cung cấp cho trình biên dịch đủ thông tin về các giá trị
borrowed để nó có thể đảm bảo các tham chiếu sẽ hợp lệ trong nhiều tình huống
hơn mà nó không thể mà không có sự giúp đỡ của chúng ta.

## Removing Duplication by Extracting a Function

Generics cho phép chúng ta thay thế các kiểu cụ thể bằng một "giữ chỗ"
(placeholder) để biểu thị nhiều kiểu khác nhau để loại bỏ sự trùng lặp code.
Trước khi đi sâu vào cú pháp generics, hãy xem xét cách để loại bỏ sự trùng lặp
code bằng cách không sử dụng kiểu generic bằng cách trích xuất một hàm để thay
thế các giá trị cụ thể bằng một "giữ chỗ" để biểu thị cho nhiều kiểu khác
nhau. Sau đó, chúng ta sẽ áp dụng cùng một kỹ thuật để trích xuất một hàm
generic!

Chúng ta bắt đầu với chương trình ngắn gọn trong Listing 10-1 tìm số lớn nhất
trong một danh sách.

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-01/src/main.rs:here}}
```

<span class="caption">Listing 10-1: Tìm số lớn nhất trong một dãy số</span>

Chúng ta lưu trữ một danh sách các số nguyên trong biến `number_list` và đặt
một tham chiếu đến số đầu tiên trong dãy số trong một biến có tên `largest`.
Sau đó, chúng ta duyệt qua tất cả các số trong dãy số, và nếu số hiện tại lớn
hơn số được lưu trong `largest`, thì thay thế tham chiếu trong biến đó. Tuy
nhiên, nếu số hiện tại nhỏ hơn hoặc bằng với số lớn nhất đã được xem, thì
biến đó không thay đổi, và code sẽ chuyển sang số tiếp theo trong dãy số. Sau
khi xem qua tất cả các số trong dãy số, `largest` sẽ tham chiếu đến số lớn
nhất, trong trường hợp này là 100.

Bây giờ chúng ta được giao nhiệm vụ tìm số lớn nhất trong hai dãy số khác nhau.
Để làm điều đó, chúng ta có thể chọn để sao chép code trong Listing 10-1 và
sử dụng cùng một logic tại hai nơi khác nhau trong chương trình, như trong
Listing 10-2.

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-02/src/main.rs}}
```

<span class="caption">Listing 10-2: Code để tìm số lớn nhất trong *hai* dãy
số</span>

Mặc dù code này chạy được, việc sao chép code là một việc mệt mỏi và dễ gây
lỗi. Chúng ta cũng phải nhớ cập nhật code ở nhiều nơi khi chúng ta muốn thay
đổi nó.

Để loại bỏ sự trùng lặp này, chúng ta sẽ tạo một trừu tượng bằng cách định
nghĩa một hàm hoạt động trên bất kỳ dãy số nguyên nào được truyền vào một
tham số. Giải pháp này làm cho code của chúng ta rõ ràng hơn và cho phép chúng
ta biểu thị khái niệm tìm số lớn nhất trong một dãy số trừu tượng.

Trong Listing 10-3, chúng ta trích xuất code tìm số lớn nhất thành một hàm có
tên `largest`. Sau đó chúng ta gọi hàm để tìm số lớn nhất trong hai dãy số từ
Listing 10-2. Chúng ta cũng có thể sử dụng hàm trên bất kỳ dãy số `i32` khác
nào chúng ta có trong tương lai.

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-03/src/main.rs:here}}
```

<span class="caption">Listing 10-3: Code trừu tượng để tìm số lớn nhất trong
hai dãy số</span>

Hàm `largest` có một tham số gọi là `list`, nó biểu thị bất kỳ dãy số `i32`
cụ thể nào chúng ta có thể truyền vào hàm. Do đó, khi chúng ta gọi hàm, code
chạy trên các giá trị cụ thể mà chúng ta truyền vào.

Tóm lại, đây là các bước chúng ta đã thực hiện để thay đổi code từ Listing
10-2 thành Listing 10-3:

1. Xác định code trùng lặp.
2. Tách code trùng lặp thành phần thân của hàm và chỉ định các đầu vào và giá
   trị trả về của code đó trong chữ ký của hàm.
3. Cập nhật hai trường hợp code trùng lặp để gọi hàm thay vì thế.

Tiếp theo, chúng ta sẽ sử dụng các bước này với generics để giảm bớt sự trùng
lặp code. Giống như phần thân hàm có thể hoạt động trên một `list` trừu tượng
thay vì các giá trị cụ thể, generics cho phép code hoạt động trên các kiểu
trừu tượng.

Ví dụ, giả sử chúng ta có hai hàm: một hàm tìm phần tử lớn nhất trong một dãy
số `i32` và một hàm tìm phần tử lớn nhất trong một dãy số `char`. Làm thế nào
chúng ta có thể loại bỏ sự trùng lặp này? Cùng nhau tìm hiểu nào!
