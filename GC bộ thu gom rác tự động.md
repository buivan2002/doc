GC: bộ thu gom rác tự động

Xử lý ram của go

Biên dịch thành file exe và người dùng chỉ cần tải file này về chứ không cần tải trình biên dịch cho các ứng dụng của go



Ví dụ đơn giản:



func main() {

    user := NewUser() // cấp phát vùng nhớ cho biến user

    fmt.Println(user)

}

// kết thúc hàm main → biến user không dùng nữa → GC sẽ thu gom bộ nhớ này



⚙️ GC của Go có gì hay?

Chạy song song với chương trình chính (concurrent GC)



Dừng rất ngắn (low pause time)



Không cần bạn lo về việc quên free(), không có use-after-free, double free...



| Thuật ngữ                  | Nghĩa                                    | Trong Go                                 |

| -------------------------- | ---------------------------------------- | ---------------------------------------- |

| \*\*GC (Garbage Collector)\*\* | Tự động giải phóng bộ nhớ không còn dùng | ✅ Có, chạy song song                     |

| \*\*Runtime\*\*                | Mã hệ thống hỗ trợ chạy chương trình Go  | ✅ Nhẹ, bao gồm GC, goroutine, panic,  |





%T để in ra kiểu dữ liệu



---

Trong Go, \*\*ghi đè biến\*\* (variable shadowing hoặc re-declaring) xảy ra khi bạn \*\*khai báo một biến mới có cùng tên với biến cũ\*\* nhưng \*\*trong phạm vi (scope) khác\*\*, thường là \*\*bên trong hàm hoặc block\*\*. Go cho phép điều này, \*\*nhưng có quy tắc rõ ràng\*\*.

---

\## ✅ Có 2 trường hợp ghi đè biến phổ biến:

---



\### 🔹 \*\*1. Ghi đè biến trong phạm vi nhỏ hơn (shadowing)\*\*



Bạn có thể khai báo lại biến \*\*cùng tên\*\* trong block nhỏ hơn (ví dụ: trong `if`, `for`, `func`, v.v).



\#### ✅ Ví dụ:



```go

package main



import "fmt"



func main() {

\&nbsp;   x := 10

\&nbsp;   fmt.Println("Outer x:", x)



\&nbsp;   if true {

\&nbsp;       x := 20 // 👈 biến mới, ghi đè biến x bên ngoài (shadow)

\&nbsp;       fmt.Println("Inner x:", x)

\&nbsp;   }



\&nbsp;   fmt.Println("Back to outer x:", x)

}

```



📤 Output:



```

Outer x: 10

Inner x: 20

Back to outer x: 10

```



➡️ Biến `x := 20` chỉ tồn tại \*\*trong block if\*\*, \*\*không ảnh hưởng biến `x` bên ngoài\*\*.



---



\### 🔹 \*\*2. Ghi đè trong cùng block — không được với `:=`\*\*



Trong cùng 1 scope/block, \*\*Go không cho bạn dùng `:=` để ghi đè biến đã tồn tại\*\* trừ khi \*\*ít nhất một biến là mới\*\*.



\#### ❌ Sai:



```go

x := 5

x := 10 // ❌ Error: no new variables on left side of :=

```



\#### ✅ Đúng:



```go

x := 5

x = 10 // ✅ Gán lại, không khai báo mới

```



---



\### ✅ Trường hợp đặc biệt: Ghi đè một phần



```go

a, b := 1, 2

a, c := 3, 4 // ✅ a đã tồn tại, c là biến mới → OK

```



→ Go sẽ chấp nhận \*\*ghi đè `a`\*\*, vì có biến \*\*`c` là mới\*\*.



---

\## ❗ Cẩn thận khi shadowing — dễ lỗi logic

Ví dụ dưới đây \*\*trông đúng nhưng sai ngầm\*\*:



```go

err := doSomething()

if err != nil {

\&nbsp;   err := fmt.Errorf("wrapped: %w", err) // 👈 shadowed `err`

\&nbsp;   return err // return biến `err` mới, không phải cái bạn nghĩ

}



> Ghi đè `err` trong `if` khiến bạn \\\*\\\*không trả đúng lỗi gốc\\\*\\\*, dễ gây bug khó phát hiện.



\\## 📌 Tóm gọn quy tắc



| Trường hợp       | Ghi đè được không? | Cách làm            |

| ---------------- | ------------------ | ------------------- |

| Trong block con  | ✅ Có thể shadow    | Dùng `:=`           |

| Trong cùng scope | ❌ Không dùng `:=`  | Dùng `=` để gán lại |

| Ghi đè một phần  | ✅ Nếu có biến mới  | `a, b := ...`       |



---



\\## ✅ Kết luận



\\\* Go cho phép \\\*\\\*ghi đè biến (shadowing)\\\*\\\* nhưng phải \\\*\\\*đúng phạm vi\\\*\\\*

\\\* Trong cùng block, \\\*\\\*không được dùng `:=` nếu không có biến mới\\\*\\\*

\\\* Shadowing quá mức có thể gây \\\*\\\*khó đọc và lỗi logic\\\*\\\*



------------------------------------------------------------------------

Trong Go, "biến giả" thường được hiểu là:

🔹 Biến không dùng đến, nhưng vẫn phải khai báo để giữ cú pháp hợp lệ hoặc bỏ qua giá trị nào đó.



Go không cho phép khai báo biến mà không dùng, nên nếu bạn chỉ cần 1 giá trị trong nhiều giá trị trả về, hoặc không muốn dùng một biến nào đó, thì bạn dùng:



✅ Biến giả trong Go là \\\_ (underscore)

Đây là biến đặc biệt, được gọi là blank identifier (biến trắng), đóng vai trò như một "biến giả".

\\\_ = 123       // ✅ gán được

fmt.Println(\\\_) // ❌ lỗi: không thể đọc `\\\_`





---------------------------------------------------------------------------



GO ưu tiên đọc biến cục bộ hơn. Nếu có xung đột giữa toàn cục và cục bộ nó sẽ lấy biến cụ bộ 



1\\. Biến toàn cục (Global Variables)

Đặc điểm



Được khai báo ngoài bất kỳ hàm, phương thức, hoặc khối lệnh nào.



Phạm vi sử dụng: có thể được truy cập ở bất kỳ đâu trong package nếu viết thường, hoặc từ package khác nếu viết hoa chữ cái đầu.



Thời điểm cấp phát: khi chương trình bắt đầu chạy.



Thời điểm giải phóng: khi chương trình kết thúc.



Quy ước đặt tên:



Chữ cái thường → chỉ truy cập được trong cùng package.



Chữ cái hoa → có thể truy cập từ package khác.





2\\. Biến cục bộ (Local Variables)

Đặc điểm



Được khai báo bên trong một hàm, phương thức, hoặc khối lệnh.



Phạm vi sử dụng: chỉ tồn tại và được truy cập bên trong hàm hoặc khối đó.



Thời điểm cấp phát: khi chương trình chạy đến hàm hoặc khối chứa biến.



Thời điểm giải phóng: khi thoát khỏi hàm hoặc khối chứa biến, biến sẽ bị thu hồi bộ nhớ.



Không thể truy cập biến cục bộ từ bên ngoài hàm chứa nó.



| \\\*\\\*Tiêu chí\\\*\\\*            | \\\*\\\*const\\\*\\\*                                  | \\\*\\\*var\\\*\\\*                                              |

| ----------------------- | ------------------------------------------ | ---------------------------------------------------- |

| \\\*\\\*Thời điểm gán\\\*\\\*       | Compile-time                               | Runtime                                              |

| \\\*\\\*Chiếm bộ nhớ\\\*\\\*        | \\\*\\\*Không\\\*\\\* (trình biên dịch inline giá trị) | \\\*\\\*Có\\\*\\\* (cấp phát trên stack hoặc heap)               |

| \\\*\\\*Vòng đời\\\*\\\*            | Gắn vào mã máy, không tồn tại thật sự      | Bắt đầu khi vào scope, bị GC thu hồi khi thoát scope |

| \\\*\\\*Sau khi thoát scope\\\*\\\* | Không thể truy cập, nhưng không cần xóa    | Biến bị giải phóng hoặc dọn bởi GC                   |



const đc gán lúc biên dịch rồi nên là k chạy runtime theo req đc đâu. 





-------------------------------------------------------



1\\. Khái niệm “Pass by Value” trong Go



Trong Go, mặc định khi bạn truyền một biến vào hàm, hàm sẽ nhận được một bản sao của biến đó, không phải tham chiếu đến biến gốc.



Khi bạn thay đổi giá trị bên trong hàm → chỉ thay đổi bản sao.



Biến gốc ở bên ngoài không bị ảnh hưởng. 



4\\. Làm sao để thay đổi biến gốc



Nếu bạn muốn thay đổi giá trị biến gốc trong hàm, bạn phải truyền địa chỉ (pointer):

\\\&x → lấy địa chỉ của biến x.



func increment(x \\\*int) → tham số x là con trỏ.



\\\*x++ → tăng giá trị tại địa chỉ mà x trỏ tới.



Vì cả hai cùng trỏ đến một vùng nhớ → thay đổi trong hàm cũng làm thay đổi biến gốc.





Hoặc ở trong hàm t trả về return khi đó hàm trả về giá trị mới, gán lại cho biến gốc



Tóm lại: 



Go mặc định truyền theo giá trị → hàm nhận bản sao.

Nếu không trả về giá trị → biến gốc không đổi.

Nếu trả về giá trị và gán lại → biến gốc thay đổi.

Hoặc có thể truyền con trỏ để sửa trực tiếp.



================================================================



DEFER: thực hiện sau khi hàm chứa nó kết thúc 



| \\\*\\\*Trường hợp\\\*\\\* | \\\*\\\*Hành vi\\\*\\\*                                                            |

| -------------- | ---------------------------------------------------------------------- |

| Nhiều defer    | Chạy \\\*\\\*theo thứ tự ngược lại\\\*\\\* (LIFO)                                  |

| Truyền tham số | Đánh giá ngay lập tức, không phụ thuộc vào thời điểm thực thi          |

| Closure        | Nếu dùng lambda thì lấy giá trị tại lúc thực thi                       |

| Hiệu suất      | `defer` có chi phí nhỏ, nhưng nếu gọi trong vòng lặp lớn thì nên tránh |

| Panic          | `defer` \\\*\\\*vẫn chạy\\\*\\\* trước khi chương trình kết thúc                   |





===============================================================

1\\. Closure là gì trong Go



Trong Go, closure là một hàm được định nghĩa bên trong một hàm khác và truy cập được các biến trong phạm vi của hàm cha, kể cả khi hàm cha đã kết thúc.



Nói đơn giản: Closure là một hàm có “trí nhớ”.



Là hàm ẩn nhưng có thêm khả năng giữ lại biến bên ngoài (lexical scope).



Các biến mà closure “bao” lại không mất đi sau khi scope bên ngoài kết thúc.



Ví dụ:



func concatter() func(string) string {

\&nbsp;   doc := ""

\&nbsp;   return func(word string) string {

\&nbsp;       doc += word + " "

\&nbsp;       return doc

\&nbsp;   }

}



func main() {

\&nbsp;   agg := concatter()

\&nbsp;   fmt.Println(agg("Hello")) // Hello

\&nbsp;   fmt.Println(agg("World")) // Hello World

}



👉 Ở đây agg nhớ được biến doc của lần concatter() tạo ra, dù concatter() đã return.



2\\. Hàm ẩn (anonymous function)



Là hàm không có tên, có thể gán vào biến hoặc gọi ngay.



Không nhất thiết phải dùng biến bên ngoài.



Ví dụ:



func main() {

\&nbsp;   // Hàm ẩn gán cho biến

\&nbsp;   square := func(x int) int {

\&nbsp;       return x \\\* x

\&nbsp;   }

\&nbsp;   fmt.Println(square(5)) // 25



\&nbsp;   // Hàm ẩn gọi ngay (IIFE)

\&nbsp;   result := func(a, b int) int {

\&nbsp;       return a + b

\&nbsp;   }(3, 4)

\&nbsp;   fmt.Println(result) // 7

}



👉 Đây chỉ đơn giản là “hàm không tên”.





3\\. function value

\&nbsp;func multiply(a, b int) int {

\&nbsp;   return a \\\* b

}



func main() {

\&nbsp;   // function value: gán function vào biến

\&nbsp;   var f func(int, int) int = multiply



\&nbsp;   fmt.Println(f(2, 3)) // 6

\&nbsp;   fmt.Println(multiply(2, 3)) // 6 (giống nhau về kết quả)

}

👉 f lúc này là function value, nó trỏ tới hàm multiply.





=============================================================

STRUCT Lồng nhau 



Struct lồng nhau = struct chứa struct.

Có 2 cách:



Đặt tên field (cách rõ ràng: p.Address.City).

Embed trực tiếp (cách Go style: p.City).

Lồng bao nhiêu cấp cũng được.

Thường dùng để mô hình hóa dữ liệu có quan hệ phân cấp (user → address → country).



=============================================================

METHOD: Hàm sử dụng struct cho funcation



=============================================================



INTERFACE 



Go không cho phép viết method cho interface cụ thể, chỉ cho struct hoặc type cụ thể.



type Shape interface {

\&nbsp;	Area() float64 // Chỉ cần struct hay kiểu type nào mà nó có method Area trả về float 64 thì Shape đều có thể dùng method của nó 

}



func  dothings (s Shape) float64 {

\&nbsp;	return s.Area()

}



type Rectangle struct {

\&nbsp;	Width, Height float64

}



func (r Rectangle) Area() float64 {

\&nbsp;	return r.Width \\\* r.Height

}



func main() {

\&nbsp;	var s Shape = Rectangle{3, 4}

\&nbsp;	fmt.Println("Area:", s.Area()) // Area: 12

\&nbsp;	sRect := Rectangle{4, 5}

\&nbsp;	s1 := dothings(sRect)

\&nbsp;	fmt.Println("Area:", s1) // Area: 20



}

2 CÁCH GỌI INTERFACE 



| Tiêu chí     | \\\*\\\*Cách 1: Gán vào biến interface\\\*\\\*          | \\\*\\\*Cách 2: Truyền vào hàm interface\\\*\\\*       |

| ------------ | ------------------------------------------- | ------------------------------------------ |

| Mục đích     | Lưu trữ \\\& sử dụng lại nhiều lần             | Xử lý 1 lần trong hàm xử lý chung          |

| Khi nào dùng | Khi muốn thao tác nhiều lần với biến đó     | Khi chỉ cần gọi 1 lần qua hàm              |

| Phù hợp      | Khi cần `interface` như một biến trạng thái | Khi dùng interface như 1 tham số nhất thời |

| Ví dụ        | `s.Area()` nhiều lần ở nhiều chỗ            | `doThings(x)` xong là xong                 |





3> Nếu muốn struct implement interface thì phải đáp ứng đủ các method



4> Một struct trong Go có thể implement nhiều interface cùng lúc — và đây là tính năng rất mạnh của Go.



=====================================================



ERROR 



1 Trả về error từ function nếu có lỗi:

result, err := doSomething()

if err != nil {

\&nbsp;   // xử lý lỗi

}





2 Tạo error bằng errors.New, fmt.Errorf, hoặc custom struct.



3 Check error bằng:



So sánh trực tiếp (if err == ErrX)

Type assertion (err.(MyError))

errors.Is / errors.As (từ Go 1.13)



4 Wrap error để giữ ngữ cảnh (fmt.Errorf("msg: %w", err)).



=====================================================

PANIC 



panic trong Go không dùng để thay thế error (như exception ở Java/Python), mà chỉ nên dùng trong những tình huống bất thường, không thể/không nên phục hồi.



======================================================

LOOP 

1\\. break



Dùng để thoát khỏi vòng lặp ngay lập tức (for, switch, select).

Sau khi break, chương trình sẽ nhảy ra ngoài vòng lặp/switch/select chứa nó.

break mặc định: chỉ thoát vòng lặp chứa nó.

break + label: thoát vòng lặp theo label chỉ định (thường dùng để thoát nhiều vòng lặp).



2\\. continue

Dùng để bỏ qua phần còn lại của vòng lặp hiện tại, và nhảy ngay tới lần lặp kế tiếp.

Khác với break, nó không thoát vòng lặp mà chỉ bỏ qua iteration hiện tại



=============================================

SLICE 



✅ Cách biết slice nào là bản copy, cái nào là ánh xạ tới mảng gốc

\\- Phần tử trong slice (s\\\[i]) → sửa trực tiếp array gốc.

\\- Thay đổi slice header (s = s\\\[1:], append):

Nếu còn capacity → vẫn chung array gốc.

Nếu hết capacity → cấp phát array mới → slice trong hàm “tách ra độc lập”.



\*\*variadic\*\*

Trong Go, variadic function (hàm variadic) là hàm có thể nhận số lượng tham số không cố định (0, 1, nhiều).

Khai báo bằng dấu ... trước kiểu dữ liệu tham số cuối cùng:



Variadic function = hàm nhận số lượng tham số không cố định.

Dùng ...Type ở tham số cuối cùng.

Bên trong hàm, tham số đó được xử lý như một slice.

APPEND


a := make(\[]int, 3)

fmt.Println("len of a:", len(a))

fmt.Println("cap of a:", cap(a))

// len of a: 3

// cap of a: 3



b := append(a, 4) // append mà quá cap thì nó copy sang cái mới còn không thì nó sửa vào con slice gốc luôn 

fmt.Println("appending 4 to b from a")

fmt.Println("b:", b)

fmt.Println("addr of b:", \\\&b\\\[0])

// appending 4 to b from a

// b: \\\[0 0 0 4]

// addr of b: 0x44a0c0



c := append(a, 5)

fmt.Println("appending 5 to c from a")

fmt.Println("addr of c:", \\\&c\\\[0])

fmt.Println("a:", a)

fmt.Println("b:", b)

fmt.Println("c:", c)

// appending 5 to c from a

// addr of c: 0x44a180

// a: \\\[0 0 0]

// b: \\\[0 0 0 4]

// c: \\\[0 0 0 5]

muốn k trong cap thì dùng slice thuần mySlice := \\\[]int{1, 2, 3} nó sẽ trỏ thẳng vào gốc luôn 



========================================



MAP 

Khi truyền map vào hàm, nó giống như con trỏ → thay đổi trong hàm sẽ ảnh hưởng ra ngoài.


func add(m map\\\[string]map\\\[string]int, path, country string) {

\&nbsp;   if m\\\[path] == nil {

\&nbsp;       m\\\[path] = make(map\\\[string]int)

\&nbsp;   }

\&nbsp;   m\\\[path]\\\[country]++

\&nbsp;   fmt.Println(m\\\[path]\\\[country])

}



func s() {

\&nbsp;   visits := make(map\\\[string]map\\\[string]int)



\&nbsp;   add(visits, "/home", "VN")

\&nbsp;   add(visits, "/home", "VN")

\&nbsp;   add(visits, "/home", "US")

\&nbsp;   add(visits, "/about", "US")



\&nbsp;   fmt.Println(visits)

}
🔹 1. rune là gì?



rune là alias (tên khác) của kiểu int32.



Nó được dùng để biểu diễn một Unicode code point (một ký tự trong bảng mã Unicode).



Nói cách khác: rune = một ký tự Unicode.





--------------------------------------------------

POINT 



các kiểu dữ liệu nó copy dữ liệu 

\* Kiểu cơ bản: int, float,...
\* Struct (trừ khi trỏ thẳng vào địa chỉ của nó)
\* Array 



Các kiểu dữu liệu truyền theo tham chiếu: 

Slice

Map 

Channel 

Funcation 



\* Thay đổi địa chỉ con trỏ a sang địa chỉ con trỏ x (có cái ảnh ở file ảnh nó có tổng quan luồng của POINTER) 

func changePointer(pp \\\*\\\*int) {

\&nbsp;   x := 100

\&nbsp;   \\\*pp = \\\&x // thay đổi con trỏ gốc

}

func change(p \\\*int){

\&nbsp;	\\\*p = 20

}

func main() {

\&nbsp;   a := 10

\&nbsp;   p := \\\&a

\&nbsp;   changePointer(\\\&p)

\&nbsp;   change(p)



}

===========================================================



PACKAGE



👉 Go không cho phép trong cùng một folder có file thuộc 2 package khác nhau.
Cùng 1 package (dù nhiều file) → không cần import, gọi thoải mái.

Khác package → phải import package và hàm cần gọi phải viết hoa (exported).


package main

import (

\&nbsp;	"fmt"

\&nbsp;	"Project/hello" // bat buoc phai theo ten folder 

\&nbsp;	// cac file con thi co the dat package tuy y- vi du package abc

)

=================================================================



GOROUTINES 

Goroutines là cách Go thực hiện concurrency (chạy song song nhiều tác vụ).

Nói ngắn gọn: goroutine là một luồng nhẹ (lightweight thread) được quản lý bởi Go runtime chứ không phải hệ điều hành.



1 request = 1 goroutine (ít nhất).

Nhiều request = nhiều goroutine song song.
Mỗi request đến sẽ được gắn vào 1 goroutine riêng. và trong 1 gorountine này nó lại tách ra nhiều goroutine để chạy bất đồng bộ



Request đến → goroutine handler chính.

Trong handler → bạn có thể tạo thêm n goroutine con để chạy song song với handler.

Tất cả goroutines đều bình đẳng (không có cha/con như thread), cùng được scheduler quản lý.



===============================================================

Channel



Channel giống như ống dẫn (pipe).

Một goroutine gửi dữ liệu vào channel → goroutine khác nhận dữ liệu từ channel.

Việc gửi/nhận có thể block để đảm bảo đồng bộ.



ch := make(chan int, 2) // channel chứa tối đa 2 giá trị

ch <- 1

ch <- 2

// ch <- 3 // sẽ block vì channel đã đầy

fmt.Println(<-ch) // lấy ra 1

ch <- 3           // giờ mới gửi được





Dùng close(ch) để đóng channel.



Sau khi đóng:

Không thể gửi thêm.

Nhưng vẫn có thể nhận cho đến khi hết dữ liệu.

**Goroutine sinh ra để giải quyết vấn đề đa luồng giúp code hoạt động như rails** 
---

###### **Channel sinh ra để code giống như nodejs**



1\. Với nhiều request đồng thời



Bản thân Go HTTP server đã lo rồi:

Mỗi request → 1 goroutine handler riêng.

Có 1000 req cùng lúc → có 1000 goroutines chạy song song.

Bạn không cần tự spawn thêm goroutine để “scale nhiều request”.

👉 Nghĩa là nhiều request song song không cần channel, runtime Go tự xử lý concurrency ở tầng server.



2\. Với một request đơn lẻ



Đây mới là chỗ goroutine + channel phát huy sức mạnh:

Một request có thể có nhiều tác vụ (gọi API A, API B, query DB, ghi log).

Nếu làm tuần tự → chậm.

Nếu spawn goroutines cho từng task và gom kết quả bằng channel → nhanh hơn.





ch1 := make(chan string)

ch2 := make(chan string)



go func() { time.Sleep(2 \* time.Second); ch1 <- "A done" }()

go func() { time.Sleep(5 \* time.Second); ch2 <- "B done" }()



a := <-ch1

b := <-ch2

fmt.Println(a, b)





Timeline

t=0s: goroutine A (2s) và B (5s) bắt đầu cùng lúc.



t=2s: A hoàn thành → a := <-ch1 unblock, lấy được "A done".



t=2–5s: b := <-ch2 vẫn phải chờ, vì channel ch2 chưa có dữ liệu.



t=5s: B hoàn thành → b := <-ch2 unblock.



In ra kết quả.

👉 Tổng thời gian = 5s (task lâu nhất).



--------------------------------------

Channel có capacity = 10 nghĩa là nó chứa tối đa 10 giá trị “pending”.



Khi bạn cố gửi thêm:

Không mất dữ liệu, không tự xoá.

Goroutine gửi bị block cho đến khi có goroutine nhận.

Khi đọc ra: dữ liệu lấy theo FIFO (first in, first out).





🟢

Buffered channel: goroutine gửi có thể đẩy dữ liệu vào và tiếp tục chạy, miễn là còn chỗ trống.



Unbuffered channel: goroutine gửi phải chờ cho đến khi có goroutine nhận.



---------------------------

CLOSE

Khi channel đóng thì k thể gửi vào đc nhưng vẫn lấy ra đc. Có thể check bằng biến ok



SELECT 

Dùng để biết goroutine nào trả dữ liệu về trước 



==============================================



MUTEX

Lock ghi đồng thời giống lock bản ghi nhưng ở đây là lock goroutine 

Dùng mutex khi có nhiều Goroutine tác động đồng thời vào 1 biến hoặc 1 thực thể nào đó trừ channel nó là queue routine nào vào trc lấy trước ( giống như trước đây xảy ra ACID thì phải lock lại để khỏi các transaction khác nó chỉnh sửa đồng thời vào 1 bản ghi và gây ra data rác) 



===============================================

Generics 



func SplitSlice\[T any](s \[]T) (\[]T, \[]T) {

&nbsp;	n := len(s) / 2

&nbsp;	return s\[:n], s\[n:]

}

📌 Ý nghĩa:



\[T any]: T là một type parameter, đại diện cho bất kỳ kiểu nào (any).

\[]T: Slice chứa các phần tử kiểu T.

Hàm này có thể dùng với slice kiểu int, string, float64, struct, v.v.



