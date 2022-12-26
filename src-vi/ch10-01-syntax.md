## Generic Data Types

Chúng ta sử dụng generics để tạo định nghĩa cho các item như signature của
hàm hoặc struct, mà chúng ta có thể sử dụng với nhiều kiểu dữ liệu cụ thể
khác nhau. Hãy xem xét cách định nghĩa các hàm, struct, enum và method sử dụng
generics. Sau đó chúng ta sẽ thảo luận về cách generics ảnh hưởng đến performance của code.

### In Function Definitions

Khi định nghĩa một hàm sử dụng generics, chúng ta đặt generics trong signature
của hàm mà chúng ta thường sẽ chỉ định kiểu dữ liệu của các tham số và giá trị
trả về. Việc làm như vậy làm cho code của chúng ta linh hoạt hơn và cung cấp
nhiều tính năng hơn cho người gọi hàm của chúng ta trong khi ngăn chặn sự lặp
code.

Tiếp tục với hàm `largest` của chúng ta, Listing 10-4 cho thấy hai hàm mà cả
hai tìm giá trị lớn nhất trong một slice. Chúng ta sẽ kết hợp chúng thành một
hàm duy nhất sử dụng generics.

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-04/src/main.rs:here}}
```

<span class="caption">Listing 10-4: Hai hàm mà khác nhau chỉ ở tên và kiểu
dữ liệu trong signature</span>

Hàm `largest_i32` là hàm mà chúng ta đã tách ra trong Listing 10-3 mà tìm
giá trị lớn nhất `i32` trong một slice. Hàm `largest_char` tìm giá trị lớn
nhất `char` trong một slice. Các hàm body có cùng code, vì vậy chúng ta sẽ
loại bỏ sự lặp code bằng cách giới thiệu một tham số kiểu generic trong một
hàm duy nhất.

Để tham số hóa các kiểu trong một hàm duy nhất, chúng ta cần đặt tên cho tham
số kiểu, giống như chúng ta làm với các tham số giá trị của một hàm. Bạn có thể
sử dụng bất kỳ định danh nào làm tên cho tham số kiểu. Nhưng chúng ta sẽ sử dụng
`T` vì theo quy ước, tên tham số kiểu trong Rust là ngắn gọn, thường chỉ là
một chữ cái, và quy ước đặt tên kiểu của Rust là CamelCase. Viết tắt của
“type,” `T` là lựa chọn mặc định của hầu hết các lập trình viên Rust.

Khi chúng ta sử dụng một tham số trong body của hàm, chúng ta phải khai báo
tên tham số trong signature để trình biên dịch biết nghĩa của tên đó là gì.
Tương tự, khi chúng ta sử dụng tên tham số kiểu trong signature của một hàm,
chúng ta phải khai báo tên tham số kiểu trước khi sử dụng nó. Để định nghĩa
hàm `largest` generic, đặt các khai báo tên kiểu trong dấu ngoặc nhọn, `<>`,
giữa tên của hàm và danh sách tham số, như sau:

```rust,ignore
fn largest<T>(list: &[T]) -> &T {
```

Chúng ta đọc định nghĩa này như sau: hàm `largest` là generic với một số kiểu
`T`. Hàm này có một tham số có tên `list`, là một slice các giá trị kiểu `T`.
Hàm `largest` sẽ trả về một tham chiếu đến một giá trị cùng kiểu `T`.

Listing 10-5 hiển thị hàm `largest` được định nghĩa bằng cách sử dụng kiểu dữ
liệu generic trong signature của nó. Listing cũng hiển thị cách chúng ta có thể
gọi hàm với một slice các giá trị `i32` hoặc các giá trị `char`. Lưu ý rằng code
này sẽ không biên dịch được, nhưng chúng ta sẽ sửa nó sau trong chương này.

<span class="filename">Filename: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-05/src/main.rs}}
```

<span class="caption">Listing 10-5: Hàm `largest` sử dụng tham số kiểu generic;
code này sẽ không biên dịch được</span>

Nếu chúng ta biên dịch code này ngay bây giờ, chúng ta sẽ nhận được lỗi này:

```console
{{#include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-05/output.txt}}
```

Đoạn text help nói về `std::cmp::PartialOrd`, đó là một *trait*, và chúng ta sẽ
nói về trait trong phần tiếp theo. Hiện tại, biết rằng lỗi này nói rằng phần
thân của `largest` sẽ không hoạt động với tất cả các kiểu có thể có của `T`.
Vì chúng ta muốn so sánh các giá trị kiểu `T` trong phần thân, chúng ta chỉ có
thể sử dụng các kiểu mà các giá trị của chúng có thể được sắp xếp. Để cho phép
so sánh, thư viện chuẩn có trait `std::cmp::PartialOrd` mà bạn có thể thực thi
trên các kiểu (xem phụ lục C để biết thêm về trait này). Bằng cách tuân theo
gợi ý của đoạn text help, chúng ta giới hạn các kiểu hợp lệ cho `T` chỉ là những
kiểu mà triển khai `PartialOrd` và ví dụ này sẽ biên dịch được, vì thư viện
chuẩn triển khai `PartialOrd` trên cả `i32` và `char`.

### In Struct Definitions

Chúng ta cũng có thể định nghĩa các struct để sử dụng tham số kiểu generic trong
một hoặc nhiều trường sử dụng cú pháp `<>`. Đoạn code 10-6 định nghĩa một struct
`Point<T>` để giữ các giá trị tọa độ `x` và `y` của bất kỳ kiểu nào.

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-06/src/main.rs}}
```

<span class="caption">Listing 10-6: Một struct `Point<T>` giữ các giá trị
`tọa độ x` và `tọa độ y` của kiểu `T`</span>

Cú pháp để sử dụng generic trong định nghĩa struct giống với cú pháp được sử
dụng trong định nghĩa hàm. Đầu tiên, chúng ta khai báo tên của tham số kiểu bên
trong dấu ngoặc nhọn ngay sau tên của struct. Sau đó chúng ta sử dụng kiểu
generic trong định nghĩa struct mà không cần chỉ định kiểu dữ liệu cụ thể.

Lưu ý rằng vì chúng ta chỉ sử dụng một kiểu generic để định nghĩa `Point<T>`,
định nghĩa này nói rằng struct `Point<T>` là generic trên một kiểu `T`, và các
trường `x` và `y` *cả hai* đều là cùng một kiểu, bất kỳ kiểu nào. Nếu chúng ta
tạo một thể hiện của `Point<T>` có giá trị của các kiểu khác nhau, như trong
Đoạn code 10-7, mã nguồn của chúng ta sẽ không được biên dịch.

<span class="filename">Filename: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-07/src/main.rs}}
```

<span class="caption">Listing 10-7: Các trường `x` và `y` phải là cùng một kiểu
vì cả hai đều có cùng kiểu dữ liệu generic `T`.</span>

Trong ví dụ này, khi chúng ta gán giá trị số nguyên 5 cho `x`, chúng ta cho
biết cho trình biên dịch biết rằng kiểu generic `T` sẽ là một số nguyên cho
thể hiện này của `Point<T>`. Sau đó khi chúng ta chỉ định 4.0 cho `y`, mà chúng
ta đã định nghĩa để có cùng kiểu dữ liệu với `x`, chúng ta sẽ nhận được một lỗi
không khớp kiểu dữ liệu như sau:

```console
{{#include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-07/output.txt}}
```

Để định nghĩa một struct `Point` mà `x` và `y` đều là generic nhưng có thể có
các kiểu dữ liệu khác nhau, chúng ta có thể sử dụng nhiều tham số kiểu generic.
Ví dụ, trong Đoạn code 10-8, chúng ta thay đổi định nghĩa của `Point` để là
generic trên các kiểu `T` và `U` mà `x` là kiểu `T` và `y` là kiểu `U`.

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-08/src/main.rs}}
```

<span class="caption">Listing 10-8: Một `Point<T, U>` generic trên hai kiểu để
`x` và `y` có thể là các giá trị của các kiểu khác nhau</span>

Bây giờ tất cả các thể hiện của `Point` được hiển thị được cho phép! Bạn có thể
sử dụng nhiều tham số kiểu generic trong một định nghĩa như bạn muốn, nhưng
sử dụng nhiều hơn một vài khiến mã của bạn khó đọc. Nếu bạn đang tìm thấy bạn
cần nhiều kiểu generic trong mã của bạn, điều đó có thể chỉ ra rằng mã của bạn
cần được cấu trúc lại thành các phần nhỏ hơn.

### In Enum Definitions

Như chúng ta đã làm với các struct, chúng ta có thể định nghĩa các enum để giữ
kiểu dữ liệu generic trong các biến thể của chúng. Hãy xem lại enum `Option<T>`
mà thư viện chuẩn cung cấp, mà chúng ta đã sử dụng trong Chương 6:

```rust
enum Option<T> {
    Some(T),
    None,
}
```

Định nghĩa này nên làm cho bạn hiểu hơn. Như bạn có thể thấy, enum `Option<T>`
generic trên kiểu `T` và có hai biến thể: `Some`, chứa một giá trị của kiểu `T`,
và một biến thể `None` không chứa bất kỳ giá trị nào. Bằng cách sử dụng enum
`Option<T>`, chúng ta có thể biểu diễn khái niệm trừu tượng của một giá trị
tùy chọn, và bởi vì `Option<T>` là generic, chúng ta có thể sử dụng trừu tượng
này không quan tâm đến kiểu của giá trị tùy chọn là gì.

Các enum cũng có thể sử dụng nhiều kiểu generic. Định nghĩa của enum `Result`
mà chúng ta đã sử dụng trong Chương 9 là một ví dụ:

```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

Enum `Result` là generic trên hai kiểu, `T` và `E`, và có hai biến thể: `Ok`,
chứa một giá trị của kiểu `T`, và `Err`, chứa một giá trị của kiểu `E`. Định
nghĩa này làm cho việc sử dụng enum `Result` ở bất kỳ nơi nào chúng ta có một
thao tác có thể thành công (trả về một giá trị của một kiểu `T`) hoặc thất bại
(trả về một lỗi của một kiểu `E`). Thực ra, điều này là những gì chúng ta đã sử
dụng để mở một tệp trong Listing 9-3, trong đó `T` được điền vào với kiểu
`std::fs::File` khi tệp được mở thành công và `E` được điền vào với kiểu
`std::io::Error` khi có vấn đề khi mở tệp.

Khi bạn nhận ra các tình huống trong code của bạn với nhiều struct hoặc enum
định nghĩa khác nhau chỉ trong kiểu của các giá trị mà chúng chứa, bạn có thể
tránh sự trùng lặp bằng cách sử dụng kiểu generic thay vì vậy.

---

### In Method Definitions

Chúng ta có thể thực hiện các phương thức trên các struct và enum (như chúng
ta đã làm trong Chương 5) và sử dụng kiểu generic trong định nghĩa của chúng
cũng như vậy. Listing 10-9 hiển thị struct `Point<T>` mà chúng ta đã định nghĩa
trong Listing 10-6 với một phương thức có tên `x` được thực hiện trên nó.

<span class="filename">Tên tệp: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-09/src/main.rs}}
```

<span class="caption">Listing 10-9: Thực hiện một phương thức có tên `x` trên
struct `Point<T>` sẽ trả về một tham chiếu đến trường `x` của kiểu `T`</span>

Ở đây, chúng ta đã định nghĩa một phương thức có tên `x` trên `Point<T>` sẽ
trả về một tham chiếu đến dữ liệu trong trường `x`.

Lưu ý rằng chúng ta phải khai báo `T` ngay sau `impl` để chúng ta có thể sử dụng
`T` để chỉ định chúng ta đang thực hiện các phương thức trên kiểu `Point<T>`.
Bằng cách khai báo `T` là một kiểu generic sau `impl`, Rust có thể nhận ra kiểu
trong dấu ngoặc nhọn trong `Point` là một kiểu generic thay vì một kiểu cụ thể.
Chúng ta có thể đã chọn một tên khác cho tham số generic này so với tham số
generic được khai báo trong định nghĩa struct, nhưng việc sử dụng cùng một tên
là thông thường. Các phương thức được viết trong một `impl` khai báo kiểu
generic sẽ được định nghĩa trên bất kỳ thể hiện nào của kiểu, bất kể kiểu cụ
thể nào kết thúc thay thế cho kiểu generic.

Chúng ta cũng có thể chỉ định ràng buộc trên kiểu generic khi định nghĩa các
phương thức trên kiểu. Chúng ta có thể, ví dụ, thực hiện các phương thức chỉ
trên các thể hiện `Point<f32>` thay vì trên các thể hiện `Point<T>` với bất kỳ
kiểu generic nào. Trong Listing 10-10 chúng ta sử dụng kiểu cụ thể `f32`, có
nghĩa là chúng ta không khai báo bất kỳ kiểu nào sau `impl`.

<span class="filename">Tên tập tin: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-10/src/main.rs:here}}
```

<span class="caption">Listing 10-10: Một khối `impl` chỉ áp dụng cho một struct
với một kiểu cụ thể cụ thể cho tham số kiểu generic `T`</span>

Đoạn code này có nghĩa là kiểu `Point<f32>` sẽ có một phương thức
`distance_from_origin`; các thể hiện khác của `Point<T>` với `T` không phải
kiểu `f32` sẽ không có phương thức này được định nghĩa. Phương thức đo độ xa
của điểm của chúng ta so với điểm tại tọa độ (0.0, 0.0) và sử dụng các phép toán
toán học chỉ có sẵn cho các kiểu số thực.

Tham số kiểu generic trong định nghĩa struct không phải luôn luôn giống như
những gì bạn sử dụng trong chữ ký phương thức struct đó. Listing 10-11 sử dụng
kiểu generic `X1` và `Y1` cho struct `Point` và `X2` `Y2` cho chữ ký phương
thức `mixup` để làm cho ví dụ rõ ràng hơn. Phương thức tạo một thể hiện `Point`
mới với giá trị `x` từ `self` `Point` (của kiểu `X1`) và giá trị `y` từ
`Point` được truyền vào (của kiểu `Y2`).

<span class="filename">Tên tập tin: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-11/src/main.rs}}
```

<span class="caption">Listing 10-11: Một phương thức sử dụng kiểu generic khác
với định nghĩa struct của nó</span>

Trong `main`, chúng ta đã định nghĩa một `Point` có kiểu `i32` cho `x` (với
giá trị `5`) và kiểu `f64` cho `y` (với giá trị `10.4`). Biến `p2` là một
struct `Point` có một slice string cho `x` (với giá trị `"Hello"`) và một
`char` cho `y` (với giá trị `c`). Gọi phương thức `mixup` trên `p1` với đối
số `p2` sẽ cho chúng ta `p3`, sẽ có kiểu `i32` cho `x`, vì `x` lấy từ `p1`. Biến
`p3` sẽ có kiểu `char` cho `y`, vì `y` lấy từ `p2`. Lệnh `println!` sẽ in
`p3.x = 5, p3.y = c`.

Mục đích của ví dụ này là để minh họa một trường hợp trong đó một số tham số
generic được khai báo với `impl` và một số khác được khai báo với định nghĩa
phương thức. Ở đây, các tham số generic `X1` và `Y1` được khai báo sau `impl`
vì chúng đi kèm với định nghĩa struct. Các tham số generic `X2` và `Y2` được
khai báo sau `fn mixup`, vì chúng chỉ liên quan đến phương thức.

### Performance of Code Using Generics

Bạn có thể thắc mắc liệu có có chi phí thời gian chạy khi sử dụng tham số kiểu
generic. Tin tốt là việc sử dụng kiểu generic sẽ không làm chương trình của bạn
chạy chậm hơn nếu sử dụng kiểu cụ thể.

Rust thực hiện điều này bằng cách thực hiện monomorphization của code sử dụng
generic tại thời điểm biên dịch. *Monomorphization* là quá trình chuyển đổi code
generic thành code cụ thể bằng cách điền vào các kiểu cụ thể được sử dụng khi
biên dịch. Trong quá trình này, compiler thực hiện ngược lại các bước chúng ta
sử dụng để tạo ra hàm generic trong Listing 10-5: compiler xem xét tất cả các
nơi mà code generic được gọi và tạo ra code cho các kiểu cụ thể mà code generic
được gọi với.

Hãy xem cách làm việc này bằng cách sử dụng enum generic `Option<T>` của
thư viện chuẩn:

```rust
let integer = Some(5);
let float = Some(5.0);
```

Khi Rust biên dịch code này, nó thực hiện monomorphization. Trong quá trình
đó, compiler đọc các giá trị được sử dụng trong các instance `Option<T>` và
xác định hai loại `Option<T>`: một là `i32` và một là `f64`. Vì vậy, nó mở rộng
định nghĩa generic của `Option<T>` thành hai định nghĩa được đặc biệt hóa với
`i32` và `f64`, vì vậy thay thế định nghĩa generic với các định nghĩa cụ thể.

Phiên bản monomorphized của code trông giống như sau (compiler sử dụng các tên
khác nhau so với những gì chúng ta đang sử dụng ở đây để minh họa):

```rust
enum Option_i32 {
    Some(i32),
    None,
}

enum Option_f64 {
    Some(f64),
    None,
}

fn main() {
    let integer = Option_i32::Some(5);
    let float = Option_f64::Some(5.0);
}
```

Generic `Option<T>` được thay thế bằng các định nghĩa cụ thể được tạo ra bởi
compiler. Bởi vì Rust biên dịch code generic thành code chỉ định kiểu trong mỗi
instance, chúng ta không phải trả bất kỳ chi phí thời gian chạy nào cho việc sử
dụng generic. Khi code chạy, nó thực hiện tương tự như nếu chúng ta đã sao chép
mỗi định nghĩa bằng tay. Quá trình monomorphization làm cho generic của Rust
rất hiệu quả tại runtime.
