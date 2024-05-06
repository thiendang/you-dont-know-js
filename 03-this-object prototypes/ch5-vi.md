# You Don't Know JS: *this* & Object Prototypes
# Chapter 5: Prototypes

In Chapters 3 and 4, we mentioned the `[[Prototype]]` chain several times, but haven't said what exactly it is. We will now examine prototypes in detail.

**Note:** All of the attempts to emulate class-copy behavior, as described previously in Chapter 4, labeled as variations of "mixins", completely circumvent the `[[Prototype]]` chain mechanism we examine here in this chapter.

## `[[Prototype]]`

Các đối tượng trong JavaScript có một thuộc tính nội bộ, được ký hiệu trong đặc tả là `[[Prototype]]`, đơn giản là một tham chiếu đến một đối tượng khác. Hầu hết các đối tượng đều được gán một giá trị không phải `null` cho thuộc tính này, tại thời điểm chúng được tạo.

**Lưu ý:** Chúng ta sẽ sớm thấy rằng một đối tượng *có thể* có liên kết `[[Prototype]]` trống, mặc dù điều này ít phổ biến hơn.

Xem xét:

```js
var myObject = {
  a: 2
};

myObject.a; // 2
```

Tham chiếu `[[Prototype]]` được sử dụng để làm gì? Trong Chương 3, chúng ta đã kiểm tra thao tác `[[Get]]` được gọi khi bạn tham chiếu đến một thuộc tính trên một đối tượng, chẳng hạn như `myObject.a`. Đối với thao tác `[[Get]]` mặc định đó, bước đầu tiên là kiểm tra xem chính đối tượng có thuộc tính `a` trên nó hay không, và nếu có, nó sẽ được sử dụng.

**Lưu ý:** ES6 Proxies nằm ngoài phạm vi thảo luận của chúng tôi trong cuốn sách này (sẽ được đề cập trong một cuốn sách sau trong bộ truyện!), nhưng mọi thứ chúng tôi thảo luận ở đây về hành vi `[[Get]]` và `[[Put]]` thông thường không áp dụng nếu có `Proxy` tham gia.

Nhưng chính điều xảy ra nếu `a` **không** có trên `myObject` mới khiến chúng ta chú ý đến liên kết `[[Prototype]]` của đối tượng.

Thao tác `[[Get]]` mặc định sẽ tiếp tục theo liên kết `[[Prototype]]` của đối tượng nếu nó không thể tìm thấy trực tiếp thuộc tính được yêu cầu trên đối tượng.

```js
var anotherObject = {
  a: 2
};

// tạo một đối tượng được liên kết với `anotherObject`
var myObject = Object.create( anotherObject );

myObject.a; // 2
```

**Lưu ý:** Chúng ta sẽ giải thích `Object.create(..)` hoạt động như thế nào và cách nó hoạt động ngay sau đây. Hiện tại, chỉ cần giả sử nó tạo ra một đối tượng có liên kết `[[Prototype]]` mà chúng ta đang kiểm tra đến đối tượng được chỉ định.

Vì vậy, chúng ta có `myObject` hiện được liên kết `[[Prototype]]` với `anotherObject`. Rõ ràng `myObject.a` thực sự không tồn tại, nhưng dù vậy, việc truy cập thuộc tính vẫn thành công (được tìm thấy trên `anotherObject` thay thế) và thực sự tìm thấy giá trị `2`.

Nhưng, nếu `a` cũng không được tìm thấy trên `anotherObject`, thì chuỗi `[[Prototype]]` của nó, nếu không trống, lại được tham khảo và tuân theo.

Quá trình này tiếp tục cho đến khi tìm thấy tên thuộc tính khớp hoặc chuỗi `[[Prototype]]` kết thúc. Nếu *không bao giờ* tìm thấy thuộc tính khớp cho đến cuối chuỗi, kết quả trả về từ thao tác `[[Get]]` là `undefined`.

Tương tự như quá trình tra cứu chuỗi `[[Prototype]]` này, nếu bạn sử dụng vòng lặp `for..in` để lặp qua một đối tượng, bất kỳ thuộc tính nào có thể truy cập được thông qua chuỗi của nó (và cũng có thể `enumerable` - xem Chương 3) sẽ được liệt kê. Nếu bạn sử dụng toán tử `in` để kiểm tra sự tồn tại của một thuộc tính trên một đối tượng, `in` sẽ kiểm tra toàn bộ chuỗi của đối tượng (bất kể tính *có thể liệt kê*).

```js
var anotherObject = {
  a: 2
};

// tạo một đối tượng được liên kết với `anotherObject`
var myObject = Object.create( anotherObject );

for (var k in myObject) {
  console.log("found: " + k);
}
// found: a

("a" in myObject); // true
```

Vì vậy, chuỗi `[[Prototype]]` được tham khảo, từng liên kết một, khi bạn thực hiện tra cứu thuộc tính theo nhiều cách khác nhau. Tra cứu dừng lại khi tìm thấy thuộc tính hoặc chuỗi kết thúc.

### `Object.prototype`

Nhưng **chính xác thì** chuỗi `[[Prototype]]` "kết thúc" ở đâu?

Phần trên cùng của mọi chuỗi `[[Prototype]]` thông thường là `Object.prototype` được tích hợp sẵn. Đối tượng này bao gồm nhiều tiện ích chung được sử dụng trong toàn bộ JS, vì tất cả các đối tượng thông thường (được tích hợp sẵn, không phải tiện ích mở rộng cụ thể của host) trong JavaScript "xuất phát từ" (hay còn gọi là, có ở đầu chuỗi `[[Prototype]]` của chúng) đối tượng `Object.prototype`.

Một số tiện ích được tìm thấy ở đây mà bạn có thể quen thuộc bao gồm `.toString()` và `.valueOf()`. Trong Chương 3, chúng tôi đã giới thiệu một tiện ích khác: `.hasOwnProperty(..)`. Và một hàm khác trên `Object.prototype` mà bạn có thể không quen thuộc, nhưng chúng tôi sẽ giải quyết sau trong chương này, là `.isPrototypeOf(..)`.

### Setting & Shadowing Properties

Back in Chapter 3, we mentioned that setting properties on an object was more nuanced than just adding a new property to the object or changing an existing property's value. We will now revisit this situation more completely.

```js
myObject.foo = "bar";
```

Nếu đối tượng `myObject` đã có một thuộc tính truy cập dữ liệu thông thường có tên `foo` trực tiếp có mặt trên nó, thì việc gán chỉ đơn giản là thay đổi giá trị của thuộc tính hiện có.

Nếu `foo` chưa có mặt trực tiếp trên `myObject`, thì chuỗi `[[Prototype]]` sẽ được duyệt, giống như đối với thao tác `[[Get]]`. Nếu `foo` không được tìm thấy ở bất kỳ đâu trong chuỗi, thì thuộc tính `foo` sẽ được thêm trực tiếp vào `myObject` với giá trị được chỉ định, như mong đợi.

Tuy nhiên, nếu `foo` đã có mặt ở đâu đó cao hơn trong chuỗi, thì hành vi tinh tế (và có lẽ đáng ngạc nhiên) có thể xảy ra với phép gán `myObject.foo = "bar"`. Chúng ta sẽ xem xét kỹ hơn điều đó ngay sau đây.

Nếu tên thuộc tính `foo` xuất hiện cả trên chính `myObject` và ở cấp cao hơn của chuỗi `[[Prototype]]` bắt đầu từ `myObject`, thì điều này được gọi là *shadowing*. Thuộc tính `foo` trực tiếp trên `myObject` *che khuất* bất kỳ thuộc tính `foo` nào xuất hiện cao hơn trong chuỗi, vì tìm kiếm `myObject.foo` luôn tìm thấy thuộc tính `foo` ở mức thấp nhất trong chuỗi.

Như chúng ta vừa gợi ý, việc shadowing `foo` trên `myObject` không đơn giản như vẻ ngoài của nó. Bây giờ chúng ta sẽ kiểm tra ba tình huống cho phép gán `myObject.foo = "bar"` khi `foo` **không** có trực tiếp trên `myObject` mà **có** ở cấp cao hơn của chuỗi `[[Prototype]]` của `myObject`:

1. Nếu một thuộc tính truy cập dữ liệu thông thường (xem Chương 3) có tên `foo` được tìm thấy ở bất kỳ đâu cao hơn trên chuỗi `[[Prototype]]`, **và nó không được đánh dấu là chỉ đọc (`writable:false`)**, thì một thuộc tính mới có tên `foo` được thêm trực tiếp vào `myObject`, dẫn đến một **thuộc tính shadowed**.
2. Nếu `foo` được tìm thấy cao hơn trên chuỗi `[[Prototype]]`, nhưng nó được đánh dấu là **chỉ đọc (`writable:false`)**, thì cả việc đặt thuộc tính hiện có đó cũng như việc tạo thuộc tính shadowed trên `myObject` đều **bị cấm**. Nếu mã đang chạy trong chế độ `strict mode`, một lỗi sẽ được ném ra. Ngược lại, việc đặt giá trị thuộc tính sẽ bị bỏ qua một cách âm thầm. Dù bằng cách nào, **không có shadowing nào xảy ra**.
3. Nếu `foo` được tìm thấy cao hơn trên chuỗi `[[Prototype]]` và nó là một setter (xem Chương 3), thì setter sẽ luôn được gọi. Không có `foo` nào sẽ được thêm vào (hay còn gọi là shadowed) trên `myObject`, cũng như setter `foo` sẽ không được định nghĩa lại.

Hầu hết các nhà phát triển cho rằng việc gán một thuộc tính (`[[Put]]`) sẽ luôn dẫn đến shadowing nếu thuộc tính đó đã tồn tại ở cấp cao hơn trên chuỗi `[[Prototype]]`, nhưng như bạn có thể thấy, điều đó chỉ đúng trong một (# 1) trong ba tình huống vừa được mô tả.

Nếu bạn muốn shadow `foo` trong trường hợp # 2 và # 3, bạn không thể sử dụng phép gán `=`, mà thay vào đó phải sử dụng `Object.defineProperty(..)` (xem Chương 3) để thêm `foo` vào `myObject`.

**Lưu ý:** Trường hợp # 2 có thể là đáng ngạc nhiên nhất trong ba trường hợp. Sự hiện diện của một thuộc tính *chỉ đọc* ngăn cản một thuộc tính có cùng tên được tạo ngầm định (shadowed) ở cấp thấp hơn của chuỗi `[[Prototype]]`. Lý do cho hạn chế này chủ yếu là để củng cố ảo tưởng về các thuộc tính được thừa hưởng từ lớp. Nếu bạn nghĩ về `foo` ở cấp cao hơn của chuỗi như đã được thừa hưởng (sao chép xuống) cho `myObject`, thì việc thực thi bản chất không thể ghi của thuộc tính `foo` trên `myObject` là hợp lý. Tuy nhiên, nếu bạn tách biệt ảo tưởng khỏi thực tế và nhận ra rằng không có việc sao chép thừa kế thực sự nào xảy ra (xem Chương 4 và 5), thì việc `myObject` bị ngăn không có thuộc tính `foo` chỉ vì một số đối tượng khác có `foo` không thể ghi trên nó là hơi bất thường. Thậm chí còn kỳ lạ hơn khi hạn chế này chỉ áp dụng cho phép gán `=`, nhưng lại không được thực thi khi sử dụng `Object.defineProperty(..)` .

Shadowing với **các phương thức** dẫn đến *đa hình giả rõ ràng* xấu xí (xem Chương 4) nếu bạn cần ủy quyền giữa chúng. Thông thường, shadowing phức tạp và tinh tế hơn mức cần thiết, **vì vậy bạn nên tránh nó nếu có thể**. Xem Chương 6 để biết một mẫu thiết kế thay thế, trong số những thứ khác, không khuyến khích shadowing để ủng hộ các lựa chọn thay thế cleaner.

Shadowing thậm chí có thể xảy ra ngầm định theo những cách tinh tế, vì vậy cần phải cẩn thận nếu cố gắng tránh nó. Xem xét:

```js
var anotherObject = {
  a: 2
};

var myObject = Object.create( anotherObject );

anotherObject.a; // 2
myObject.a; // 2

anotherObject.hasOwnProperty( "a" ); // true
myObject.hasOwnProperty( "a" ); // false

myObject.a++; // oops, shadowing ngầm định!

anotherObject.a; // 2
myObject.a; // 3

myObject.hasOwnProperty( "a" ); // true
```

Mặc dù có vẻ như `myObject.a++` nên (thông qua ủy quyền) tìm kiếm và chỉ tăng giá trị của chính thuộc tính `anotherObject.a` *tại chỗ*, thay vào đó, hoạt động `++` tương ứng với `myObject.a = myObject.a + 1`. Kết quả là `[[Get]]` tìm kiếm thuộc tính `a` thông qua `[[Prototype]]` để lấy giá trị hiện tại `2` từ `anotherObject.a`, tăng giá trị lên một, sau đó `[[Put]]` gán giá trị `3` cho một thuộc tính shadowed mới `a` trên `myObject`. Ối!

Hãy rất cẩn thận khi xử lý các thuộc tính được ủy quyền mà bạn sửa đổi. Nếu bạn muốn tăng `anotherObject.a`, thì cách duy nhất đúng là `anotherObject.a++`.

## "Class"

Đến thời điểm này, bạn có thể đang tự hỏi: "*Tại sao* một đối tượng cần liên kết với một đối tượng khác?" Lợi ích thực sự là gì? Đó là một câu hỏi rất phù hợp để hỏi, nhưng trước tiên chúng ta phải hiểu `[[Prototype]]` **không phải là** gì trước khi có thể hiểu và đánh giá đầy đủ `[[Prototype]]` **là gì** và cách sử dụng nó.

Như đã giải thích trong Chương 4, trong JavaScript, không có các mẫu/bản thiết kế trừu tượng cho các đối tượng được gọi là "lớp" như trong các ngôn ngữ hướng đối tượng. JavaScript **chỉ** có các đối tượng.

Trên thực tế, JavaScript **gần như là duy nhất** trong số các ngôn ngữ có lẽ là ngôn ngữ duy nhất có quyền sử dụng nhãn "hướng đối tượng", bởi vì nó là một trong số rất ít ngôn ngữ mà một đối tượng có thể được tạo trực tiếp, hoàn toàn không cần đến lớp.

Trong JavaScript, các lớp không thể (vì chúng không tồn tại!) mô tả những gì một đối tượng có thể làm. Đối tượng tự xác định hành vi của riêng mình. **Chỉ có** đối tượng.

### "Class" Functions

Có một kiểu hành vi kỳ lạ trong JavaScript đã bị lạm dụng trong nhiều năm để *hack* thứ gì đó *trông giống như* "lớp". Chúng ta sẽ xem xét kỹ lưỡng cách tiếp cận này.

Hành vi "giống-như-lớp" kỳ lạ này dựa trên một đặc điểm kỳ lạ của các hàm: tất cả các hàm theo mặc định đều có một thuộc tính công khai, không thể liệt kê (xem Chương 3) được gọi là `prototype`, nó trỏ đến một đối tượng tùy ý khác.

```js
function Foo() {
  // ...
}

Foo.prototype; // { }
```

Đối tượng này thường được gọi là "prototype của Foo", bởi vì chúng ta truy cập nó thông qua tham chiếu thuộc tính `Foo.prototype` được đặt tên không may. Tuy nhiên, thuật ngữ đó hoàn toàn có thể dẫn chúng ta đến nhầm lẫn, như chúng ta sẽ thấy ngay sau đây. Thay vào đó, tôi sẽ gọi nó là "đối tượng trước đây được gọi là prototype của Foo". Chỉ là đùa thôi. Làm sao về: "đối tượng được dán nhãn tùy ý 'Foo dot prototype'"?

Bất kể chúng ta gọi nó là gì, chính xác thì đối tượng này là gì?

Cách giải thích trực tiếp nhất là mỗi đối tượng được tạo từ việc gọi `new Foo()` (xem Chương 2) sẽ (hơi tùy ý) được liên kết `[[Prototype]]` với đối tượng "Foo dot prototype" này.

Hãy minh họa:

```js
function Foo() {
  // ...
}

var a = new Foo();

Object.getPrototypeOf( a ) === Foo.prototype; // true
```

Khi `a` được tạo bằng cách gọi `new Foo()`, một trong những điều xảy ra (xem Chương 2 cho tất cả *bốn* bước) là `a` có được một liên kết `[[Prototype]]` nội bộ với đối tượng mà `Foo.prototype` đang trỏ đến.

Dừng lại một chút và suy ngẫm về ý nghĩa của tuyên bố đó.

Trong các ngôn ngữ hướng đối tượng, nhiều **bản sao** (hay còn gọi là "thể hiện") của một lớp có thể được tạo, giống như đóng dấu thứ gì đó từ một khuôn mẫu. Như chúng ta đã thấy trong Chương 4, điều này xảy ra bởi vì quá trình khởi tạo (hoặc thừa kế từ) một lớp có nghĩa là, "sao chép kế hoạch hành vi từ lớp đó vào một đối tượng vật lý", và điều này được thực hiện lại cho mỗi thể hiện mới.

Nhưng trong JavaScript, không có hành động sao chép nào được thực hiện. Bạn không tạo nhiều thể hiện của một lớp. Bạn có thể tạo nhiều đối tượng `[[Prototype]]` *liên kết* với một đối tượng chung. Nhưng theo mặc định, không có sao chép nào xảy ra, và do đó, các đối tượng này không hoàn toàn tách biệt và ngắt kết nối với nhau, mà thay vào đó, khá ***liên kết***.

`new Foo()` tạo ra một đối tượng mới (chúng ta gọi nó là `a`), và **đối tượng** mới `a` **đó** được liên kết `[[Prototype]]` nội bộ với đối tượng `Foo.prototype`.

**Chúng ta kết thúc với hai đối tượng, được liên kết với nhau.** Chỉ vậy thôi. Chúng ta không khởi tạo một lớp. Chúng tôi chắc chắn không thực hiện bất kỳ việc sao chép hành vi nào từ "lớp" vào một đối tượng cụ thể. Chúng tôi chỉ khiến hai đối tượng được liên kết với nhau.

Trên thực tế, bí mật mà hầu hết các nhà phát triển JS không biết, là việc gọi hàm `new Foo()` thực sự hầu như không liên quan trực tiếp đến quá trình tạo liên kết. **Đó chỉ là một tác dụng phụ tình cờ.** `new Foo()` là một cách gián tiếp, vòng vo để đạt được điều chúng ta muốn: **một đối tượng mới được liên kết với một đối tượng khác**.

Chúng ta có thể đạt được điều mình muốn theo cách *trực tiếp* hơn không? **Có!** Người hùng là `Object.create(..)`. Nhưng chúng ta sẽ tìm hiểu về điều đó sau một chút.

#### What's in a name?

Trong JavaScript, chúng ta không tạo *bản sao* từ một đối tượng ("lớp") sang một đối tượng khác ("thể hiện"). Thay vào đó, chúng ta tạo *liên kết* giữa các đối tượng. Đối với cơ chế `[[Prototype]]`, về mặt trực quan, các mũi tên di chuyển từ phải sang trái và từ dưới lên trên.

<img src="fig3.png">

Cơ chế này thường được gọi là "thừa kế nguyên mẫu" (chúng ta sẽ khám phá chi tiết về mã trong thời gian ngắn), thường được cho là phiên bản ngôn ngữ động của "thừa kế cổ điển". Nó là một nỗ lực để dựa vào sự hiểu biết chung về "thừa kế" có nghĩa gì trong thế giới hướng đối tượng, nhưng *tinh chỉnh* (đọc: che đậy) ngữ nghĩa đã hiểu, để phù hợp với lập trình theo kịch bản động.

Từ "thừa kế" có một ý nghĩa rất mạnh mẽ (xem Chương 4), với nhiều tiền lệ về mặt tư duy. Chỉ đơn giản thêm "nguyên mẫu" phía trước để phân biệt hành vi thực tế gần như *hoàn toàn ngược lại* trong JavaScript đã khiến cho gần hai thập kỷ qua trở nên hỗn loạn.

Tôi muốn nói rằng việc gắn "nguyên mẫu" trước "thừa kế" để đảo ngược hoàn toàn ý nghĩa thực sự của nó giống như cầm một quả cam trong một tay, một quả táo trong tay kia và khăng khăng gọi quả táo là "cam đỏ". Bất kể nhãn má gây nhầm lẫn nào mà tôi đặt trước nó, điều đó cũng không thay đổi được *sự thật* rằng một loại trái cây là táo và loại còn lại là cam.

Cách tiếp cận tốt hơn là gọi táo một cách đơn giản là táo - sử dụng thuật ngữ chính xác và trực tiếp nhất. Điều đó giúp dễ dàng hiểu hơn cả điểm giống và **nhiều điểm khác biệt** của chúng, vì tất cả chúng ta đều có một sự hiểu biết đơn giản, chung về "táo" có nghĩa là gì.

Do sự nhầm lẫn và trộn lẫn các thuật ngữ, tôi tin rằng chính nhãn "thừa kế nguyên mẫu" (và cố gắng áp dụng sai tất cả thuật ngữ định hướng lớp liên quan của nó, như "lớp", "trình xây dựng", "thể hiện", "đa hình", v.v.) đã gây ra **hại nhiều hơn lợi** trong việc giải thích cơ chế của JavaScript thực sự hoạt động như thế nào.

"Thừa kế" ngầm định một hoạt động *sao chép*, và JavaScript không sao chép các thuộc tính của đối tượng (theo mặc định). Thay vào đó, JS tạo ra một liên kết giữa hai đối tượng, nơi một đối tượng về cơ bản có thể *ủy quyền* quyền truy cập thuộc tính/chức năng cho một đối tượng khác. "Ủy quyền" (xem Chương 6) là một thuật ngữ chính xác hơn nhiều cho cơ chế liên kết đối tượng của JavaScript.

Một thuật ngữ khác đôi khi được sử dụng trong JavaScript là "thừa kế khác biệt". Ý tưởng ở đây là chúng ta mô tả hành vi của một đối tượng theo những gì *khác biệt* so với một mô tả tổng quát hơn. Ví dụ, bạn giải thích rằng một chiếc xe là một loại phương tiện, nhưng có chính xác 4 bánh, thay vì mô tả lại tất cả các chi tiết cụ thể về những gì tạo nên một phương tiện nói chung (động cơ, v.v.).

Nếu bạn cố gắng nghĩ về bất kỳ đối tượng nào trong JS như tổng hợp tất cả các hành vi có sẵn thông qua ủy quyền, và **trong suy nghĩ của bạn, bạn làm phẳng** tất cả các hành vi đó thành một *thứ* cụ thể, thì bạn có thể (hơi) thấy "thừa kế khác biệt" có thể phù hợp như thế nào.

Nhưng giống như với "thừa kế nguyên mẫu", "thừa kế khác biệt" giả vờ rằng mô hình tinh thần của bạn quan trọng hơn những gì đang xảy ra thực tế trong ngôn ngữ. Nó bỏ qua thực tế rằng đối tượng `B` không thực sự được xây dựng khác biệt, mà thay vào đó được xây dựng với các đặc điểm cụ thể được xác định, cùng với "lỗ hổng" không có gì được xác định. Chính trong những "lỗ hổng" này (khoảng trống trong hoặc thiếu định nghĩa) mà ủy quyền *có thể* tiếp quản và "điền vào" chúng bằng hành vi được ủy quyền một cách linh hoạt.

Đối tượng, theo mặc định, không được làm phẳng thành một đối tượng khác biệt duy nhất, **thông qua sao chép**, như mô hình tinh thần của "thừa kế khác biệt" ngầm định. Do đó, "thừa kế khác biệt" không thực sự phù hợp để mô tả cách cơ chế `[[Prototype]]` của JavaScript thực sự hoạt động.

Bạn *có thể chọn* ưu tiên thuật ngữ và mô hình tinh thần "thừa kế khác biệt" tùy theo sở thích, nhưng không thể phủ nhận thực tế rằng nó *chỉ* phù hợp với những nhào lộn trong suy nghĩ của bạn, chứ không phải hành vi vật lý trong engine.

### "Constructors"

Let's go back to some earlier code:

```js
function Foo() {
	// ...
}

var a = new Foo();
```

Điều gì chính xác khiến chúng ta nghĩ `Foo` là một "lớp"?

Thứ nhất, chúng ta thấy việc sử dụng từ khóa `new`, giống như các ngôn ngữ hướng đối tượng sử dụng khi xây dựng các thể hiện của lớp. Thứ hai, dường như chúng ta thực sự đang thực thi phương thức *trình xây dựng* của một lớp, bởi vì `Foo()` thực sự là một phương thức được gọi, giống như cách trình xây dựng của một lớp thực sự được gọi khi bạn khởi tạo lớp đó.

Để làm tăng thêm sự nhầm lẫn về ngữ nghĩa của "trình xây dựng", đối tượng `Foo.prototype` được gắn nhãn tùy ý còn có một thủ thuật khác. Xem xét đoạn mã này:

```js
function Foo() {
  // ...
}

Foo.prototype.constructor === Foo; // true

var a = new Foo();
a.constructor === Foo; // true
```

Theo mặc định (vào thời điểm khai báo trên dòng 1 của đoạn trích!), đối tượng `Foo.prototype` có một thuộc tính công khai, không thể liệt kê (xem Chương 3) được gọi là `.constructor`, và thuộc tính này là một tham chiếu ngược lại với hàm (`Foo` trong trường hợp này) mà đối tượng được liên kết với. Hơn nữa, chúng ta thấy rằng đối tượng `a` được tạo ra bởi cuộc gọi "trình xây dựng" `new Foo()` dường như cũng có một thuộc tính được gọi là `.constructor` tương tự, trỏ đến "hàm đã tạo ra nó".

**Lưu ý:** Điều này thực ra không đúng. `a` không có thuộc tính `.constructor` nào trên nó, và mặc dù `a.constructor` thực sự giải quyết thành hàm `Foo`, "trình xây dựng" **không thực sự có nghĩa là** "được xây dựng bởi", như vẻ bề ngoài của nó. Chúng tôi sẽ giải thích sự kỳ lạ này ngay sau đây.

Ồ, vâng, cũng vậy... theo quy ước trong thế giới JavaScript, "lớp" được đặt tên bằng chữ cái hoa, vì vậy thực tế là `Foo` thay vì `foo` là một manh mối rõ ràng cho thấy chúng ta muốn nó là một "lớp". Điều đó hoàn toàn hiển nhiên với bạn, phải không!?

**Lưu ý:** Quy ước này mạnh mẽ đến mức nhiều trình kiểm tra cú pháp JS thực sự *phàn nàn* nếu bạn gọi `new` trên một phương thức có tên viết thường, hoặc nếu chúng ta không gọi `new` trên một hàm bắt đầu bằng chữ cái hoa. Loại điều đó khiến người ta bối rối rằng chúng ta phải vật lộn rất nhiều để có được "định hướng lớp" (giả) *đúng* trong JavaScript đến nỗi chúng ta tạo ra các quy tắc kiểm tra cú pháp để đảm bảo sử dụng chữ hoa, ngay cả khi chữ hoa không có nghĩa gì ***đối với engine JS.

#### Constructor Or Call?

In the above snippet, it's tempting to think that `Foo` is a "constructor", because we call it with `new` and we observe that it "constructs" an object.

In reality, `Foo` is no more a "constructor" than any other function in your program. Functions themselves are **not** constructors. However, when you put the `new` keyword in front of a normal function call, that makes that function call a "constructor call". In fact, `new` sort of hijacks any normal function and calls it in a fashion that constructs an object, **in addition to whatever else it was going to do**.

For example:

```js
function NothingSpecial() {
	console.log( "Don't mind me!" );
}

var a = new NothingSpecial();
// "Don't mind me!"

a; // {}
```

`NothingSpecial` is just a plain old normal function, but when called with `new`, it *constructs* an object, almost as a side-effect, which we happen to assign to `a`. The **call** was a *constructor call*, but `NothingSpecial` is not, in and of itself, a *constructor*.

In other words, in JavaScript, it's most appropriate to say that a "constructor" is **any function called with the `new` keyword** in front of it.

Functions aren't constructors, but function calls are "constructor calls" if and only if `new` is used.

### Mechanics

Are *those* the only common triggers for ill-fated "class" discussions in JavaScript?

**Not quite.** JS developers have strived to simulate as much as they can of class-orientation:

```js
function Foo(name) {
	this.name = name;
}

Foo.prototype.myName = function() {
	return this.name;
};

var a = new Foo( "a" );
var b = new Foo( "b" );

a.myName(); // "a"
b.myName(); // "b"
```

This snippet shows two additional "class-orientation" tricks in play:

1. `this.name = name`: adds the `.name` property onto each object (`a` and `b`, respectively; see Chapter 2 about `this` binding), similar to how class instances encapsulate data values.

2. `Foo.prototype.myName = ...`: perhaps the more interesting technique, this adds a property (function) to the `Foo.prototype` object. Now, `a.myName()` works, but perhaps surprisingly. How?

In the above snippet, it's strongly tempting to think that when `a` and `b` are created, the properties/functions on the `Foo.prototype` object are *copied* over to each of `a` and `b` objects. **However, that's not what happens.**

At the beginning of this chapter, we explained the `[[Prototype]]` link, and how it provides the fall-back look-up steps if a property reference isn't found directly on an object, as part of the default `[[Get]]` algorithm.

So, by virtue of how they are created, `a` and `b` each end up with an internal `[[Prototype]]` linkage to `Foo.prototype`. When `myName` is not found on `a` or `b`, respectively, it's instead found (through delegation, see Chapter 6) on `Foo.prototype`.

#### "Constructor" Redux

Recall the discussion from earlier about the `.constructor` property, and how it *seems* like `a.constructor === Foo` being true means that `a` has an actual `.constructor` property on it, pointing at `Foo`? **Not correct.**

This is just unfortunate confusion. In actuality, the `.constructor` reference is also *delegated* up to `Foo.prototype`, which **happens to**, by default, have a `.constructor` that points at `Foo`.

It *seems* awfully convenient that an object `a` "constructed by" `Foo` would have access to a `.constructor` property that points to `Foo`. But that's nothing more than a false sense of security. It's a happy accident, almost tangentially, that `a.constructor` *happens* to point at `Foo` via this default `[[Prototype]]` delegation. There's actually several ways that the ill-fated assumption of `.constructor` meaning "was constructed by" can come back to bite you.

For one, the `.constructor` property on `Foo.prototype` is only there by default on the object created when `Foo` the function is declared. If you create a new object, and replace a function's default `.prototype` object reference, the new object will not by default magically get a `.constructor` on it.

Consider:

```js
function Foo() { /* .. */ }

Foo.prototype = { /* .. */ }; // create a new prototype object

var a1 = new Foo();
a1.constructor === Foo; // false!
a1.constructor === Object; // true!
```

`Object(..)` didn't "construct" `a1` did it? It sure seems like `Foo()` "constructed" it. Many developers think of `Foo()` as doing the construction, but where everything falls apart is when you think "constructor" means "was constructed by", because by that reasoning, `a1.constructor` should be `Foo`, but it isn't!

What's happening? `a1` has no `.constructor` property, so it delegates up the `[[Prototype]]` chain to `Foo.prototype`. But that object doesn't have a `.constructor` either (like the default `Foo.prototype` object would have had!), so it keeps delegating, this time up to `Object.prototype`, the top of the delegation chain. *That* object indeed has a `.constructor` on it, which points to the built-in `Object(..)` function.

**Misconception, busted.**

Of course, you can add `.constructor` back to the `Foo.prototype` object, but this takes manual work, especially if you want to match native behavior and have it be non-enumerable (see Chapter 3).

For example:

```js
function Foo() { /* .. */ }

Foo.prototype = { /* .. */ }; // create a new prototype object

// Need to properly "fix" the missing `.constructor`
// property on the new object serving as `Foo.prototype`.
// See Chapter 3 for `defineProperty(..)`.
Object.defineProperty( Foo.prototype, "constructor" , {
	enumerable: false,
	writable: true,
	configurable: true,
	value: Foo    // point `.constructor` at `Foo`
} );
```

That's a lot of manual work to fix `.constructor`. Moreover, all we're really doing is perpetuating the misconception that "constructor" means "was constructed by". That's an *expensive* illusion.

The fact is, `.constructor` on an object arbitrarily points, by default, at a function who, reciprocally, has a reference back to the object -- a reference which it calls `.prototype`. The words "constructor" and "prototype" only have a loose default meaning that might or might not hold true later. The best thing to do is remind yourself, "constructor does not mean constructed by".

`.constructor` is not a magic immutable property. It *is* non-enumerable (see snippet above), but its value is writable (can be changed), and moreover, you can add or overwrite (intentionally or accidentally) a property of the name `constructor` on any object in any `[[Prototype]]` chain, with any value you see fit.

By virtue of how the `[[Get]]` algorithm traverses the `[[Prototype]]` chain, a `.constructor` property reference found anywhere may resolve quite differently than you'd expect.

See how arbitrary its meaning actually is?

The result? Some arbitrary object-property reference like `a1.constructor` cannot actually be *trusted* to be the assumed default function reference. Moreover, as we'll see shortly, just by simple omission, `a1.constructor` can even end up pointing somewhere quite surprising and insensible.

`a1.constructor` is extremely unreliable, and an unsafe reference to rely upon in your code. **Generally, such references should be avoided where possible.**

## "(Prototypal) Inheritance"

We've seen some approximations of "class" mechanics as typically hacked into JavaScript programs. But JavaScript "class"es would be rather hollow if we didn't have an approximation of "inheritance".

Actually, we've already seen the mechanism which is commonly called "prototypal inheritance" at work when `a` was able to "inherit from" `Foo.prototype`, and thus get access to the `myName()` function. But we traditionally think of "inheritance" as being a relationship between two "classes", rather than between "class" and "instance".

<img src="fig3.png">

Recall this figure from earlier, which shows not only delegation from an object (aka, "instance") `a1` to object `Foo.prototype`, but from `Bar.prototype` to `Foo.prototype`, which somewhat resembles the concept of Parent-Child class inheritance. *Resembles*, except of course for the direction of the arrows, which show these are delegation links rather than copy operations.

And, here's the typical "prototype style" code that creates such links:

```js
function Foo(name) {
	this.name = name;
}

Foo.prototype.myName = function() {
	return this.name;
};

function Bar(name,label) {
	Foo.call( this, name );
	this.label = label;
}

// here, we make a new `Bar.prototype`
// linked to `Foo.prototype`
Bar.prototype = Object.create( Foo.prototype );

// Beware! Now `Bar.prototype.constructor` is gone,
// and might need to be manually "fixed" if you're
// in the habit of relying on such properties!

Bar.prototype.myLabel = function() {
	return this.label;
};

var a = new Bar( "a", "obj a" );

a.myName(); // "a"
a.myLabel(); // "obj a"
```

**Note:** To understand why `this` points to `a` in the above code snippet, see Chapter 2.

The important part is `Bar.prototype = Object.create( Foo.prototype )`. `Object.create(..)` *creates* a "new" object out of thin air, and links that new object's internal `[[Prototype]]` to the object you specify (`Foo.prototype` in this case).

In other words, that line says: "make a *new* 'Bar dot prototype' object that's linked to 'Foo dot prototype'."

When `function Bar() { .. }` is declared, `Bar`, like any other function, has a `.prototype` link to its default object. But *that* object is not linked to `Foo.prototype` like we want. So, we create a *new* object that *is* linked as we want, effectively throwing away the original incorrectly-linked object.

**Note:** A common mis-conception/confusion here is that either of the following approaches would *also* work, but they do not work as you'd expect:

```js
// doesn't work like you want!
Bar.prototype = Foo.prototype;

// works kinda like you want, but with
// side-effects you probably don't want :(
Bar.prototype = new Foo();
```

`Bar.prototype = Foo.prototype` doesn't create a new object for `Bar.prototype` to be linked to. It just makes `Bar.prototype` be another reference to `Foo.prototype`, which effectively links `Bar` directly to **the same object as** `Foo` links to: `Foo.prototype`. This means when you start assigning, like `Bar.prototype.myLabel = ...`, you're modifying **not a separate object** but *the* shared `Foo.prototype` object itself, which would affect any objects linked to `Foo.prototype`. This is almost certainly not what you want. If it *is* what you want, then you likely don't need `Bar` at all, and should just use only `Foo` and make your code simpler.

`Bar.prototype = new Foo()` **does in fact** create a new object which is duly linked to `Foo.prototype` as we'd want. But, it uses the `Foo(..)` "constructor call" to do it. If that function has any side-effects (such as logging, changing state, registering against other objects, **adding data properties to `this`**, etc.), those side-effects happen at the time of this linking (and likely against the wrong object!), rather than only when the eventual `Bar()` "descendants" are created, as would likely be expected.

So, we're left with using `Object.create(..)` to make a new object that's properly linked, but without having the side-effects of calling `Foo(..)`. The slight downside is that we have to create a new object, throwing the old one away, instead of modifying the existing default object we're provided.

It would be *nice* if there was a standard and reliable way to modify the linkage of an existing object. Prior to ES6, there's a non-standard and not fully-cross-browser way, via the `.__proto__` property, which is settable. ES6 adds a `Object.setPrototypeOf(..)` helper utility, which does the trick in a standard and predictable way.

Compare the pre-ES6 and ES6-standardized techniques for linking `Bar.prototype` to `Foo.prototype`, side-by-side:

```js
// pre-ES6
// throws away default existing `Bar.prototype`
Bar.prototype = Object.create( Foo.prototype );

// ES6+
// modifies existing `Bar.prototype`
Object.setPrototypeOf( Bar.prototype, Foo.prototype );
```

Ignoring the slight performance disadvantage (throwing away an object that's later garbage collected) of the `Object.create(..)` approach, it's a little bit shorter and may be perhaps a little easier to read than the ES6+ approach. But it's probably a syntactic wash either way.

### Inspecting "Class" Relationships

What if you have an object like `a` and want to find out what object (if any) it delegates to? Inspecting an instance (just an object in JS) for its inheritance ancestry (delegation linkage in JS) is often called *introspection* (or *reflection*) in traditional class-oriented environments.

Consider:

```js
function Foo() {
	// ...
}

Foo.prototype.blah = ...;

var a = new Foo();
```

How do we then introspect `a` to find out its "ancestry" (delegation linkage)? The first approach embraces the "class" confusion:

```js
a instanceof Foo; // true
```

The `instanceof` operator takes a plain object as its left-hand operand and a **function** as its right-hand operand. The question `instanceof` answers is: **in the entire `[[Prototype]]` chain of `a`, does the object arbitrarily pointed to by `Foo.prototype` ever appear?**

Unfortunately, this means that you can only inquire about the "ancestry" of some object (`a`) if you have some **function** (`Foo`, with its attached `.prototype` reference) to test with. If you have two arbitrary objects, say `a` and `b`, and want to find out if *the objects* are related to each other through a `[[Prototype]]` chain, `instanceof` alone can't help.

**Note:** If you use the built-in `.bind(..)` utility to make a hard-bound function (see Chapter 2), the function created will not have a `.prototype` property. Using `instanceof` with such a function transparently substitutes the `.prototype` of the *target function* that the hard-bound function was created from.

It's fairly uncommon to use hard-bound functions as "constructor calls", but if you do, it will behave as if the original *target function* was invoked instead, which means that using `instanceof` with a hard-bound function also behaves according to the original function.

This snippet illustrates the ridiculousness of trying to reason about relationships between **two objects** using "class" semantics and `instanceof`:

```js
// helper utility to see if `o1` is
// related to (delegates to) `o2`
function isRelatedTo(o1, o2) {
	function F(){}
	F.prototype = o2;
	return o1 instanceof F;
}

var a = {};
var b = Object.create( a );

isRelatedTo( b, a ); // true
```

Inside `isRelatedTo(..)`, we borrow a throw-away function `F`, reassign its `.prototype` to arbitrarily point to some object `o2`, then ask if `o1` is an "instance of" `F`. Obviously `o1` isn't *actually* inherited or descended or even constructed from `F`, so it should be clear why this kind of exercise is silly and confusing. **The problem comes down to the awkwardness of class semantics forced upon JavaScript**, in this case as revealed by the indirect semantics of `instanceof`.

The second, and much cleaner, approach to `[[Prototype]]` reflection is:

```js
Foo.prototype.isPrototypeOf( a ); // true
```

Notice that in this case, we don't really care about (or even *need*) `Foo`, we just need an **object** (in our case, arbitrarily labeled `Foo.prototype`) to test against another **object**. The question `isPrototypeOf(..)` answers is: **in the entire `[[Prototype]]` chain of `a`, does `Foo.prototype` ever appear?**

Same question, and exact same answer. But in this second approach, we don't actually need the indirection of referencing a **function** (`Foo`) whose `.prototype` property will automatically be consulted.

We *just need* two **objects** to inspect a relationship between them. For example:

```js
// Simply: does `b` appear anywhere in
// `c`s [[Prototype]] chain?
b.isPrototypeOf( c );
```

Notice, this approach doesn't require a function ("class") at all. It just uses object references directly to `b` and `c`, and inquires about their relationship. In other words, our `isRelatedTo(..)` utility above is built-in to the language, and it's called `isPrototypeOf(..)`.

We can also directly retrieve the `[[Prototype]]` of an object. As of ES5, the standard way to do this is:

```js
Object.getPrototypeOf( a );
```

And you'll notice that object reference is what we'd expect:

```js
Object.getPrototypeOf( a ) === Foo.prototype; // true
```

Most browsers (not all!) have also long supported a non-standard alternate way of accessing the internal `[[Prototype]]`:

```js
a.__proto__ === Foo.prototype; // true
```

The strange `.__proto__` (not standardized until ES6!) property "magically" retrieves the internal `[[Prototype]]` of an object as a reference, which is quite helpful if you want to directly inspect (or even traverse: `.__proto__.__proto__...`) the chain.

Just as we saw earlier with `.constructor`, `.__proto__` doesn't actually exist on the object you're inspecting (`a` in our running example). In fact, it exists (non-enumerable; see Chapter 2) on the built-in `Object.prototype`, along with the other common utilities (`.toString()`, `.isPrototypeOf(..)`, etc).

Moreover, `.__proto__` looks like a property, but it's actually more appropriate to think of it as a getter/setter (see Chapter 3).

Roughly, we could envision `.__proto__` implemented (see Chapter 3 for object property definitions) like this:

```js
Object.defineProperty( Object.prototype, "__proto__", {
	get: function() {
		return Object.getPrototypeOf( this );
	},
	set: function(o) {
		// setPrototypeOf(..) as of ES6
		Object.setPrototypeOf( this, o );
		return o;
	}
} );
```

So, when we access (retrieve the value of) `a.__proto__`, it's like calling `a.__proto__()` (calling the getter function). *That* function call has `a` as its `this` even though the getter function exists on the `Object.prototype` object (see Chapter 2 for `this` binding rules), so it's just like saying `Object.getPrototypeOf( a )`.

`.__proto__` is also a settable property, just like using ES6's `Object.setPrototypeOf(..)` shown earlier. However, generally you **should not change the `[[Prototype]]` of an existing object**.

There are some very complex, advanced techniques used deep in some frameworks that allow tricks like "subclassing" an `Array`, but this is commonly frowned on in general programming practice, as it usually leads to *much* harder to understand/maintain code.

**Note:** As of ES6, the `class` keyword will allow something that approximates "subclassing" of built-in's like `Array`. See Appendix A for discussion of the `class` syntax added in ES6.

The only other narrow exception (as mentioned earlier) would be setting the `[[Prototype]]` of a default function's `.prototype` object to reference some other object (besides `Object.prototype`). That would avoid replacing that default object entirely with a new linked object. Otherwise, **it's best to treat object `[[Prototype]]` linkage as a read-only characteristic** for ease of reading your code later.

**Note:** The JavaScript community unofficially coined a term for the double-underscore, specifically the leading one in properties like `__proto__`: "dunder". So, the "cool kids" in JavaScript would generally pronounce `__proto__` as "dunder proto".

## Object Links

As we've now seen, the `[[Prototype]]` mechanism is an internal link that exists on one object which references some other object.

This linkage is (primarily) exercised when a property/method reference is made against the first object, and no such property/method exists. In that case, the `[[Prototype]]` linkage tells the engine to look for the property/method on the linked-to object. In turn, if that object cannot fulfill the look-up, its `[[Prototype]]` is followed, and so on. This series of links between objects forms what is called the "prototype chain".

### `Create()`ing Links

We've thoroughly debunked why JavaScript's `[[Prototype]]` mechanism is **not** like *classes*, and we've seen how it instead creates **links** between proper objects.

What's the point of the `[[Prototype]]` mechanism? Why is it so common for JS developers to go to so much effort (emulating classes) in their code to wire up these linkages?

Remember we said much earlier in this chapter that `Object.create(..)` would be a hero? Now, we're ready to see how.

```js
var foo = {
	something: function() {
		console.log( "Tell me something good..." );
	}
};

var bar = Object.create( foo );

bar.something(); // Tell me something good...
```

`Object.create(..)` creates a new object (`bar`) linked to the object we specified (`foo`), which gives us all the power (delegation) of the `[[Prototype]]` mechanism, but without any of the unnecessary complication of `new` functions acting as classes and constructor calls, confusing `.prototype` and `.constructor` references, or any of that extra stuff.

**Note:** `Object.create(null)` creates an object that has an empty (aka, `null`) `[[Prototype]]` linkage, and thus the object can't delegate anywhere. Since such an object has no prototype chain, the `instanceof` operator (explained earlier) has nothing to check, so it will always return `false`. These special empty-`[[Prototype]]` objects are often called "dictionaries" as they are typically used purely for storing data in properties, mostly because they have no possible surprise effects from any delegated properties/functions on the `[[Prototype]]` chain, and are thus purely flat data storage.

We don't *need* classes to create meaningful relationships between two objects. The only thing we should **really care about** is objects linked together for delegation, and `Object.create(..)` gives us that linkage without all the class cruft.

#### `Object.create()` Polyfilled

`Object.create(..)` was added in ES5. You may need to support pre-ES5 environments (like older IE's), so let's take a look at a simple **partial** polyfill for `Object.create(..)` that gives us the capability that we need even in those older JS environments:

```js
if (!Object.create) {
	Object.create = function(o) {
		function F(){}
		F.prototype = o;
		return new F();
	};
}
```

This polyfill works by using a throw-away `F` function and overriding its `.prototype` property to point to the object we want to link to. Then we use `new F()` construction to make a new object that will be linked as we specified.

This usage of `Object.create(..)` is by far the most common usage, because it's the part that *can be* polyfilled. There's an additional set of functionality that the standard ES5 built-in `Object.create(..)` provides, which is **not polyfillable** for pre-ES5. As such, this capability is far-less commonly used. For completeness sake, let's look at that additional functionality:

```js
var anotherObject = {
	a: 2
};

var myObject = Object.create( anotherObject, {
	b: {
		enumerable: false,
		writable: true,
		configurable: false,
		value: 3
	},
	c: {
		enumerable: true,
		writable: false,
		configurable: false,
		value: 4
	}
} );

myObject.hasOwnProperty( "a" ); // false
myObject.hasOwnProperty( "b" ); // true
myObject.hasOwnProperty( "c" ); // true

myObject.a; // 2
myObject.b; // 3
myObject.c; // 4
```

The second argument to `Object.create(..)` specifies property names to add to the newly created object, via declaring each new property's *property descriptor* (see Chapter 3). Because polyfilling property descriptors into pre-ES5 is not possible, this additional functionality on `Object.create(..)` also cannot be polyfilled.

The vast majority of usage of `Object.create(..)` uses the polyfill-safe subset of functionality, so most developers are fine with using the **partial polyfill** in pre-ES5 environments.

Some developers take a much stricter view, which is that no function should be polyfilled unless it can be *fully* polyfilled. Since `Object.create(..)` is one of those partial-polyfill'able utilities, this narrower perspective says that if you need to use any of the functionality of `Object.create(..)` in a pre-ES5 environment, instead of polyfilling, you should use a custom utility, and stay away from using the name `Object.create` entirely. You could instead define your own utility, like:

```js
function createAndLinkObject(o) {
	function F(){}
	F.prototype = o;
	return new F();
}

var anotherObject = {
	a: 2
};

var myObject = createAndLinkObject( anotherObject );

myObject.a; // 2
```

I do not share this strict opinion. I fully endorse the common partial-polyfill of `Object.create(..)` as shown above, and using it in your code even in pre-ES5. I'll leave it to you to make your own decision.

### Links As Fallbacks?

It may be tempting to think that these links between objects *primarily* provide a sort of fallback for "missing" properties or methods. While that may be an observed outcome, I don't think it represents the right way of thinking about `[[Prototype]]`.

Consider:

```js
var anotherObject = {
	cool: function() {
		console.log( "cool!" );
	}
};

var myObject = Object.create( anotherObject );

myObject.cool(); // "cool!"
```

That code will work by virtue of `[[Prototype]]`, but if you wrote it that way so that `anotherObject` was acting as a fallback **just in case** `myObject` couldn't handle some property/method that some developer may try to call, odds are that your software is going to be a bit more "magical" and harder to understand and maintain.

That's not to say there aren't cases where fallbacks are an appropriate design pattern, but it's not very common or idiomatic in JS, so if you find yourself doing so, you might want to take a step back and reconsider if that's really appropriate and sensible design.

**Note:** In ES6, an advanced functionality called `Proxy` is introduced which can provide something of a "method not found" type of behavior. `Proxy` is beyond the scope of this book, but will be covered in detail in a later book in the *"You Don't Know JS"* series.

**Don't miss an important but nuanced point here.**

Designing software where you intend for a developer to, for instance, call `myObject.cool()` and have that work even though there is no `cool()` method on `myObject` introduces some "magic" into your API design that can be surprising for future developers who maintain your software.

You can however design your API with less "magic" to it, but still take advantage of the power of `[[Prototype]]` linkage.

```js
var anotherObject = {
	cool: function() {
		console.log( "cool!" );
	}
};

var myObject = Object.create( anotherObject );

myObject.doCool = function() {
	this.cool(); // internal delegation!
};

myObject.doCool(); // "cool!"
```

Here, we call `myObject.doCool()`, which is a method that *actually exists* on `myObject`, making our API design more explicit (less "magical"). *Internally*, our implementation follows the **delegation design pattern** (see Chapter 6), taking advantage of `[[Prototype]]` delegation to `anotherObject.cool()`.

In other words, delegation will tend to be less surprising/confusing if it's an internal implementation detail rather than plainly exposed in your API interface design. We will expound on **delegation** in great detail in the next chapter.

## Review (TL;DR)

When attempting a property access on an object that doesn't have that property, the object's internal `[[Prototype]]` linkage defines where the `[[Get]]` operation (see Chapter 3) should look next. This cascading linkage from object to object essentially defines a "prototype chain" (somewhat similar to a nested scope chain) of objects to traverse for property resolution.

All normal objects have the built-in `Object.prototype` as the top of the prototype chain (like the global scope in scope look-up), where property resolution will stop if not found anywhere prior in the chain. `toString()`, `valueOf()`, and several other common utilities exist on this `Object.prototype` object, explaining how all objects in the language are able to access them.

The most common way to get two objects linked to each other is using the `new` keyword with a function call, which among its four steps (see Chapter 2), it creates a new object linked to another object.

The "another object" that the new object is linked to happens to be the object referenced by the arbitrarily named `.prototype` property of the function called with `new`. Functions called with `new` are often called "constructors", despite the fact that they are not actually instantiating a class as *constructors* do in traditional class-oriented languages.

While these JavaScript mechanisms can seem to resemble "class instantiation" and "class inheritance" from traditional class-oriented languages, the key distinction is that in JavaScript, no copies are made. Rather, objects end up linked to each other via an internal `[[Prototype]]` chain.

For a variety of reasons, not the least of which is terminology precedent, "inheritance" (and "prototypal inheritance") and all the other OO terms just do not make sense when considering how JavaScript *actually* works (not just applied to our forced mental models).

Instead, "delegation" is a more appropriate term, because these relationships are not *copies* but delegation **links**.
