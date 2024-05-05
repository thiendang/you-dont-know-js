# You Don't Know JS: *this* & Object Prototypes
# Chapter 2: `this` All Makes Sense Now!

Trong Chương 1, chúng ta đã loại bỏ rất nhiều những hiểu nhầm về `this`, đồng thời học rằng `this` là thứ (binding) được tạo ra trong mỗi lần gọi hàm, thứ đó phụ thuộc hoàn toàn vào **call-site** (cách mà hàm được gọi).

## Call-site

Để hiểu về binding của `this` , ta buộc phải hiểu về call-site: vị trí trong đoạn code nơi mà một function được gọi (**không phải nơi nó được khai báo**). Chúng ta phải tìm hiểu call-site để trả lời cho câu hỏi: `this` *này* là một tham chiếu đến cái gì?

Việc đi tìm call-site nói một cách tổng quan chỉ là: "đi tìm vị trí mà một function được gọi từ đó", nhưng việc này không phải lúc nào cũng dễ dàng, bởi có một vài coding patterns che đi vị trí thực sự của call-site.

What's important is to think about the **call-stack** (the stack of functions that have been called to get us to the current moment in execution). The call-site we care about is *in* the invocation *before* the currently executing function.

Ví dụ dưới đây sẽ minh họa về call-stack và call-site:

```js
function baz() {
    // call-stack là: `baz`
    // vì vậy, call-site của `baz` là ở global scope

    console.log( "baz" );
    bar(); // <-- call-site cho `bar`
}

function bar() {
    // call-stack là: `baz` -> `bar`
    // vậy, call-site của `bar` là ở trong `baz`

    console.log( "bar" );
    foo(); // <-- call-site cho `foo`
}

function foo() {
    // call-stack là: `baz` -> `bar` -> `foo`
    // vậy, call-site của `foo` là trong `bar`

    console.log( "foo" );
}

baz(); // <-- call-site cho `baz`
```

Hãy cẩn thận khi phân tích code để tìm call-site thực (từ call-stack), bởi đây là thứ duy nhất ảnh quyết định đến `this` ràng buộc cái gì.

**Note:** You can visualize a call-stack in your mind by looking at the chain of function calls in order, as we did with the comments in the above snippet. But this is painstaking and error-prone. Another way of seeing the call-stack is using a debugger tool in your browser. Most modern desktop browsers have built-in developer tools, which includes a JS debugger. In the above snippet, you could have set a breakpoint in the tools for the first line of the `foo()` function, or simply inserted the `debugger;` statement on that first line. When you run the page, the debugger will pause at this location, and will show you a list of the functions that have been called to get to that line, which will be your call stack. So, if you're trying to diagnose `this` binding, use the developer tools to get the call-stack, then find the second item from the top, and that will show you the real call-site.

## Chỉ có quy tắc

Bây giờ chúng ta sẽ chuyển sang tìm hiểu *cách* call-site xác định `this` sẽ trỏ đến đâu trong quá trình thực thi của một hàm.

Bạn cần kiểm tra call-site và xác định xem 4 quy tắc nào được áp dụng. Đầu tiên, chúng ta sẽ giải thích từng quy tắc này một cách độc lập, sau đó sẽ minh họa thứ tự ưu tiên của chúng, trong trường hợp nhiều quy tắc *có thể* áp dụng cho call-site.

### Quy tắc ràng buộc mặc định

Quy tắc đầu tiên chúng ta sẽ xem xét đến từ trường hợp gọi hàm phổ biến nhất: gọi hàm độc lập. Hãy coi quy tắc `this` này như quy tắc bắt lỗi mặc định khi không có quy tắc nào khác được áp dụng.

Xem xét đoạn code này:

```js
function foo() {
	var a = 3; // Phần này thêm vào của người dịch.
	console.log( this.a );
}

var a = 2;

foo(); // 2
```

Điều đầu tiên cần lưu ý, nếu bạn chưa biết, là các biến được khai báo trong phạm vi toàn cục, như `var a = 2`, đồng nghĩa với các thuộc tính của đối tượng toàn cục có cùng tên. Chúng không phải là bản sao của nhau, mà *chính là* nhau. Hãy coi chúng như hai mặt của một đồng xu.

Thứ hai, chúng ta thấy rằng khi `foo()` được gọi, `this.a` sẽ trỏ đến biến toàn cục `a` của chúng ta. Tại sao? Bởi vì trong trường hợp này, *quy tắc ràng buộc mặc định* được áp dụng cho cuộc gọi hàm, do đó `this` trỏ đến đối tượng toàn cục.

Làm thế nào chúng ta biết rằng quy tắc *ràng buộc mặc định* được áp dụng ở đây? Chúng ta kiểm tra call-site để xem cách `foo()` được gọi. Trong đoạn code của chúng ta, `foo()` được gọi với một tham chiếu hàm đơn giản, không có bất kỳ "trang trí" nào. Không có quy tắc nào khác mà chúng ta sẽ trình bày sẽ áp dụng ở đây, do đó, *quy tắc ràng buộc mặc định* được áp dụng thay thế.

Nếu `strict mode` được bật, thì đối tượng toàn cục không đủ điều kiện cho *quy tắc ràng buộc mặc định*, do đó `this` được đặt thành `undefined`.

```js
function foo() {
	"use strict";

	console.log( this.a );
}

var a = 2;

foo(); // TypeError: `this` is `undefined`
```

Một chi tiết tinh tế nhưng quan trọng là: mặc dù các quy tắc ràng buộc `this` tổng thể hoàn toàn dựa trên call-site, đối tượng toàn cục **chỉ** đủ điều kiện cho *quy tắc ràng buộc mặc định* nếu nội dung của `foo()` **không** chạy trong `strict mode`; trạng thái `strict mode` của call-site của `foo()` không liên quan.

```js
function foo() {
	console.log( this.a );
}

var a = 2;

(function(){
	"use strict";

	foo(); // 2
})();
```

**Lưu ý:** Việc cố ý trộn lẫn `strict mode` và non-`strict mode` trong code của bạn thường không được khuyến khích. Toàn bộ chương trình của bạn có thể nên là **Strict** hoặc **non-Strict**. Tuy nhiên, đôi khi bạn bao gồm một thư viện của bên thứ ba có độ **Strict** khác với code của bạn, vì vậy cần lưu ý đến những chi tiết tương thích tinh tế này.

### Ràng buộc ngầm định

Một quy tắc khác cần xem xét là: liệu call-site có một đối tượng ngữ cảnh, còn được gọi là đối tượng sở hữu hoặc chứa, mặc dù các thuật ngữ thay thế này có thể hơi gây hiểu nhầm.

Xem xét:

```js
function foo() {
    console.log( this.a );
}

var obj = {
    a: 2,
    foo: foo
};

obj.foo(); // 2
```

Trước tiên, hãy chú ý đến cách `foo()` được khai báo và sau đó được thêm vào `obj` như một thuộc tính tham chiếu. Bất kể `foo()` được khai báo ban đầu *trên* `obj` hay được thêm vào như một tham chiếu sau đó (như đoạn code này hiển thị), trong cả hai trường hợp, hàm **thực sự không được "sở hữu" hay "chứa"** bởi đối tượng `obj`.

Tuy nhiên, call-site *sử dụng* ngữ cảnh `obj` để **tham chiếu** đến hàm, vì vậy bạn *có thể* nói rằng đối tượng `obj` "sở hữu" hoặc "chứa" **tham chiếu hàm** tại thời điểm hàm được gọi.

Bất kể bạn chọn gọi mẫu này như thế nào, tại thời điểm `foo()` được gọi, nó được dẫn trước bởi một tham chiếu đối tượng đến `obj`. Khi có một đối tượng ngữ cảnh cho tham chiếu hàm, quy tắc *ràng buộc ngầm định* nói rằng chính đối tượng *ấy* nên được sử dụng cho ràng buộc `this` của cuộc gọi hàm.

Vì `obj` là `this` cho cuộc gọi `foo()`, nên `this.a` đồng nghĩa với `obj.a`.

Chỉ có cấp/mức cuối cùng của chuỗi tham chiếu thuộc tính đối tượng mới quan trọng đối với call-site. Ví dụ:

```js
function foo() {
    console.log( this.a );
}

var obj2 = {
    a: 42,
    foo: foo
};

var obj1 = {
    a: 2,
    obj2: obj2
};

obj1.obj2.foo(); // 42
```

#### Mất ràng buộc ngầm định

Một trong những điều gây bực bội nhất mà ràng buộc `this` tạo ra là khi một hàm được ràng buộc ngầm định bị mất ràng buộc đó, thường có nghĩa là nó quay trở lại *ràng buộc mặc định*, có thể là đối tượng toàn cục hoặc `undefined`, tùy thuộc vào `strict mode`.

Xem xét:

```js
function foo() {
    console.log( this.a );
}

var obj = {
    a: 2,
    foo: foo
};

var bar = obj.foo; // tham chiếu/biệt danh hàm!

var a = "oops, global"; // `a` cũng là thuộc tính trên đối tượng toàn cục

bar(); // "oops, global"
```

Mặc dù `bar` dường như là một tham chiếu đến `obj.foo`, trên thực tế, nó chỉ là một tham chiếu khác đến chính `foo`. Hơn nữa, call-site mới là điều quan trọng, và call-site là `bar()`, đây là một cuộc gọi đơn giản, không được trang trí, do đó *ràng buộc mặc định* được áp dụng.

Cách tinh vi hơn, phổ biến hơn và bất ngờ hơn mà điều này xảy ra là khi chúng ta xem xét việc truyền một hàm callback:

```js
function foo() {
    console.log( this.a );
}

function doFoo(fn) {
    // `fn` chỉ là một tham chiếu khác đến `foo`

    fn(); // <-- call-site!
}

var obj = {
    a: 2,
    foo: foo
};

var a = "oops, global"; // `a` cũng là thuộc tính trên đối tượng toàn cục

doFoo( obj.foo ); // "oops, global"
```

Truyền tham số chỉ là một sự gán ngầm định, và vì chúng ta đang truyền một hàm, nên đó là một sự gán tham chiếu ngầm định, vì vậy kết quả cuối cùng giống như đoạn code trước.

Điều gì xảy ra nếu hàm bạn đang truyền callback của mình không phải của riêng bạn, mà được tích hợp sẵn trong ngôn ngữ? Không có gì khác biệt, kết quả vẫn như vậy.

```js
function foo() {
    console.log( this.a );
}

var obj = {
    a: 2,
    foo: foo
};

var a = "oops, global"; // `a` cũng là thuộc tính trên đối tượng toàn cục

setTimeout( obj.foo, 100 ); // "oops, global"
```

Hãy nghĩ về giả code lý thuyết thô sơ này của `setTimeout()` được cung cấp như một hàm tích hợp sẵn từ môi trường JavaScript:

```js
function setTimeout(fn,delay) {
    // chờ (bằng cách nào đó) trong `delay` mili giây
    fn(); // <-- call-site!
}
```

Rất phổ biến là các callback hàm của chúng ta *mất* ràng buộc `this` của chúng, như chúng ta vừa thấy. Nhưng một cách khác mà `this` có thể khiến chúng ta ngạc nhiên là khi hàm mà chúng ta đã truyền callback của mình cố tình thay đổi `this` cho cuộc gọi. Trình xử lý sự kiện trong các thư viện JavaScript phổ biến rất thích buộc callback của bạn phải có `this` trỏ đến, ví dụ như, phần tử DOM kích hoạt sự kiện. Mặc dù điều đó đôi khi có thể hữu ích, nhưng đôi khi nó lại gây khó chịu. Thật không may, các công cụ này hiếm khi cho bạn lựa chọn.

Bằng cách nào `this` bị thay đổi bất ngờ, bạn không thực sự kiểm soát được cách tham chiếu hàm callback của mình sẽ được thực thi, vì vậy bạn (chưa) có cách nào để kiểm soát call-site để cung cấp ràng buộc dự định. Chúng ta sẽ sớm thấy một cách để "fixing" vấn đề đó bằng cách "fixing" `this`.

### Ràng buộc tường minh

Như chúng ta vừa thấy với *ràng buộc ngầm định*, chúng ta phải thay đổi đối tượng đang xét để bao gồm một tham chiếu đến chính hàm đó, và sử dụng tham chiếu hàm thuộc tính này để gián tiếp (ngầm định) ràng buộc `this` với đối tượng.

Nhưng, nếu bạn muốn buộc một cuộc gọi hàm sử dụng một đối tượng cụ thể cho ràng buộc `this`, mà không cần đặt tham chiếu hàm thuộc tính trên đối tượng thì sao?

"Tất cả" các hàm trong ngôn ngữ đều có một số tiện ích có sẵn cho chúng (thông qua `[[Prototype]]` của chúng - chúng ta sẽ thảo luận thêm về điều này sau), có thể hữu ích cho tác vụ này. Cụ thể, các hàm có các phương thức `call(..)` và `apply(..)` . Về mặt kỹ thuật, môi trường host của JavaScript đôi khi cung cấp các hàm đặc biệt (một cách nói nhẹ nhàng!) mà chúng không có chức năng như vậy. Nhưng những trường hợp đó rất ít. Phần lớn các hàm được cung cấp, và chắc chắn tất cả các hàm bạn sẽ tạo ra, đều có quyền truy cập vào `call(..)` và `apply(..)`.

Làm thế nào các tiện ích này hoạt động? Cả hai đều lấy, làm tham số đầu tiên của chúng, một đối tượng để sử dụng cho `this`, và sau đó gọi hàm với `this` được chỉ định. Vì bạn đang trực tiếp nêu rõ what bạn muốn `this` là gì, chúng ta gọi nó là *ràng buộc tường minh*.

Xem xét:

```js
function foo() {
    console.log( this.a );
}

var obj = {
    a: 2
};

foo.call( obj ); // 2
```

Gọi `foo` với *ràng buộc tường minh* bằng `foo.call(..)` cho phép chúng ta buộc `this` của nó thành `obj`.

Nếu bạn truyền một giá trị nguyên thủy đơn giản (kiểu `string`, `boolean` hoặc `number`) làm ràng buộc `this`, thì giá trị nguyên thủy đó sẽ được bọc trong dạng đối tượng của nó (`new String(..)`, `new Boolean(..)`, hoặc `new Number(..)`, tương ứng). Điều này thường được gọi là "boxing".

**Lưu ý:** Về ràng buộc `this`, `call(..)` và `apply(..)` là giống nhau. Chúng *có* hoạt động khác nhau với các tham số bổ sung của chúng, nhưng đó không phải là điều chúng ta quan tâm hiện tại.

Thật không may, *ràng buộc tường minh* riêng lẻ vẫn không cung cấp bất kỳ giải pháp nào cho vấn đề được đề cập trước đó, về việc một hàm "mất" ràng buộc `this` dự định của nó, hoặc chỉ bị ghi đè bởi một framework, v.v.

#### Ràng buộc cứng

Nhưng một biến thể xung quanh *ràng buộc tường minh* thực sự hiệu quả. Xem xét:

```js
function foo() {
    console.log( this.a );
}

var obj = {
    a: 2
};

var bar = function() {
    foo.call( obj );
};

bar(); // 2
setTimeout( bar, 100 ); // 2

// `bar` ràng buộc cứng `this` của `foo` với `obj`
// để nó không thể bị ghi đè
bar.call( window ); // 2
```

Hãy xem cách biến thể này hoạt động. Chúng ta tạo một hàm `bar()` mà bên trong, nó gọi thủ công `foo.call(obj)`, do đó buộc `foo` với ràng buộc `obj` cho `this`. Bất kể cách bạn gọi hàm `bar` sau đó, nó sẽ luôn luôn gọi thủ công `foo` với `obj`. Ràng buộc này vừa rõ ràng vừa mạnh, nên chúng ta gọi nó là *ràng buộc cứng*.

Cách điển hình nhất để bọc một hàm với *ràng buộc cứng* là tạo một pass-thru của bất kỳ đối số nào được truyền và bất kỳ giá trị trả về nào được nhận:

```js
function foo(something) {
    console.log( this.a, something );
    return this.a + something;
}

var obj = {
    a: 2
};

var bar = function() {
    return foo.apply( obj, arguments );
};

var b = bar( 3 ); // 2 3
console.log( b ); // 5
```

Một cách khác để thể hiện mẫu này là tạo một trình trợ giúp có thể tái sử dụng:

```js
function foo(something) {
    console.log( this.a, something );
    return this.a + something;
}

// trình trợ giúp `bind` đơn giản
function bind(fn, obj) {
    return function() {
        return fn.apply( obj, arguments );
    };
}

var obj = {
    a: 2
};

var bar = bind( foo, obj );

var b = bar( 3 ); // 2 3
console.log( b ); // 5
```

Vì *ràng buộc cứng* là một mẫu rất phổ biến, nên nó được cung cấp với một tiện ích tích hợp từ ES5: `Function.prototype.bind`, và nó được sử dụng như thế này:

```js
function foo(something) {
    console.log( this.a, something );
    return this.a + something;
}

var obj = {
    a: 2
};

var bar = foo.bind( obj );

var b = bar( 3 ); // 2 3
console.log( b ); // 5
```

`bind(..)` trả về một hàm mới được mã hóa cứng để gọi hàm gốc với ngữ cảnh `this` được đặt theo cách bạn đã chỉ định.

**Lưu ý:** Bắt đầu từ ES6, hàm được ràng buộc cứng được tạo ra bởi `bind(..)` có thuộc tính `.name` bắt nguồn từ *hàm target* gốc. Ví dụ: `bar = foo.bind(..)` nên có giá trị `bar.name` là `"bound foo"`, đây là tên gọi hàm sẽ hiển thị trong stack trace.

#### API Call "Contexts"

Nhiều hàm trong thư viện, và thực tế là nhiều hàm tích hợp mới trong ngôn ngữ JavaScript và môi trường host, cung cấp một tham số tùy chọn, thường được gọi là "context" (ngữ cảnh), được thiết kế như một giải pháp thay thế để bạn không cần sử dụng `bind(..)` để đảm bảo hàm callback của bạn sử dụng một `this` cụ thể.

Ví dụ:

```js
function foo(el) {
    console.log( el, this.id );
}

var obj = {
    id: "tuyệt vời"
};

// sử dụng `obj` làm `this` cho các cuộc gọi `foo(..)`
[1, 2, 3].forEach( foo, obj ); // 1 tuyệt vời  2 tuyệt vời  3 tuyệt vời
```

Bên trong, các hàm khác nhau này gần như chắc chắn sử dụng *ràng buộc tường minh* thông qua `call(..)` hoặc `apply(..)`, giúp bạn tiết kiệm công sức.

### `new` Binding

Quy tắc thứ tư và cuối cùng cho ràng buộc `this` đòi hỏi chúng ta phải suy nghĩ lại về một quan niệm sai lầm rất phổ biến về hàm và đối tượng trong JavaScript.

Trong các ngôn ngữ hướng đối tượng truyền thống, "constructor" là các phương thức đặc biệt gắn với các lớp, khi lớp được khởi tạo với toán tử `new`, thì constructor của lớp đó được gọi. Điều này thường trông giống như:

```js
something = new MyClass(..);
```

JavaScript có toán tử `new`, và mẫu code để sử dụng nó về cơ bản giống với những gì chúng ta thấy trong các ngôn ngữ hướng đối tượng đó; hầu hết các developer cho rằng cơ chế của JavaScript đang thực hiện một điều tương tự. Tuy nhiên, thực sự không có *bất kỳ kết nối nào* với chức năng hướng đối tượng được ngầm định bởi việc sử dụng `new` trong JS.

Trước tiên, hãy định nghĩa lại "constructor" trong JavaScript là gì. Trong JS, constructor **chỉ là các hàm** tình cờ được gọi với toán tử `new` phía trước. Chúng không được gắn với các lớp, cũng không khởi tạo một lớp. Chúng thậm chí không phải là các loại hàm đặc biệt. Chúng chỉ là các hàm thông thường, về bản chất, bị "bắt cóc" bởi việc sử dụng `new` trong việc gọi chúng.

Ví dụ, hàm `Number(..)` hoạt động như một constructor, trích dẫn từ đặc điểm kỹ thuật ES5.1:

> 15.7.2 Constructor Number
>
> Khi Number được gọi như một phần của biểu thức new, nó là một constructor: nó khởi tạo đối tượng mới được tạo.

Vì vậy, khá nhiều hàm thông thường, bao gồm cả các hàm đối tượng tích hợp như `Number(..)` (xem Chương 3) có thể được gọi với `new` phía trước, và điều đó làm cho cuộc gọi hàm đó trở thành một *cuộc gọi constructor*. Đây là một sự phân biệt quan trọng nhưng tinh tế: thực sự không có thứ gì gọi là "constructor functions", mà là các cuộc gọi construction *của* các hàm.

Khi một hàm được gọi với `new` phía trước, còn được gọi là constructor call, thì những điều sau đây sẽ được thực hiện tự động:

1. một đối tượng hoàn toàn mới được tạo ra (hay được xây dựng) từ hư không
2. *đối tượng mới được xây dựng được liên kết `[[Prototype]]`*
3. đối tượng mới được xây dựng được đặt làm ràng buộc `this` cho cuộc gọi hàm đó
4. trừ khi hàm trả về đối tượng thay thế riêng của nó, thì cuộc gọi hàm được gọi với `new` sẽ *tự động* trả về đối tượng mới được xây dựng.

Các bước 1, 3 và 4 áp dụng cho thảo luận hiện tại của chúng tôi. Chúng tôi sẽ bỏ qua bước 2 hiện tại và quay lại trong Chương 5.

Xem xét đoạn code này:

```js
function foo(a) {
    this.a = a;
}

var bar = new foo( 2 );
console.log( bar.a ); // 2
```

Bằng cách gọi `foo(..)` với `new` phía trước, chúng ta đã xây dựng một đối tượng mới và đặt đối tượng mới đó làm `this` cho cuộc gọi của `foo(..)`. **Vì vậy, `new` là cách cuối cùng mà `this` của một cuộc gọi hàm có thể được ràng buộc.** Chúng tôi sẽ gọi đây là *ràng buộc mới*.

## Everything In Order

Bây giờ chúng ta đã khám phá ra 4 quy tắc ràng buộc `this` trong các cuộc gọi hàm. Tất cả những gì bạn cần làm là xác định vị trí cuộc gọi và kiểm tra xem quy tắc nào áp dụng. Nhưng, nếu vị trí cuộc gọi có nhiều quy tắc đủ điều kiện thì sao? Phải có một thứ tự ưu tiên cho các quy tắc này, và vì vậy, chúng tôi sẽ tiếp theo trình bày thứ tự áp dụng các quy tắc.

Nên rõ ràng rằng *ràng buộc mặc định* là quy tắc có mức độ ưu tiên thấp nhất trong 4 quy tắc. Vì vậy, chúng ta sẽ tạm gác nó sang một bên.

Quy tắc nào có mức độ ưu tiên cao hơn, *ràng buộc ngầm định* hay *ràng buộc tường minh*? Hãy thử nghiệm:

```js
function foo() {
	console.log( this.a );
}

var obj1 = {
	a: 2,
	foo: foo
};

var obj2 = {
	a: 3,
	foo: foo
};

obj1.foo(); // 2
obj2.foo(); // 3

obj1.foo.call( obj2 ); // 3
obj2.foo.call( obj1 ); // 2
```

Vì vậy, *ràng buộc tường minh* có mức độ ưu tiên cao hơn *ràng buộc ngầm định*, nghĩa là bạn nên hỏi **trước** xem *ràng buộc tường minh* có áp dụng trước khi kiểm tra *ràng buộc ngầm định*.

Bây giờ, chúng ta chỉ cần tìm ra vị trí của *ràng buộc mới* trong thứ tự ưu tiên.

```js
function foo(something) {
	this.a = something;
}

var obj1 = {
	foo: foo
};

var obj2 = {};

obj1.foo( 2 );
console.log( obj1.a ); // 2

obj1.foo.call( obj2, 3 );
console.log( obj2.a ); // 3

var bar = new obj1.foo( 4 );
console.log( obj1.a ); // 2
console.log( bar.a ); // 4
```

Được rồi, *ràng buộc mới* có mức độ ưu tiên cao hơn *ràng buộc ngầm định*. Nhưng bạn có nghĩ *ràng buộc mới* có mức độ ưu tiên cao hơn hay thấp hơn *ràng buộc tường minh*?

**Lưu ý:** `new` và `call`/`apply` không thể được sử dụng cùng nhau, vì vậy `new foo.call(obj1)` không được phép, để kiểm tra *ràng buộc mới* trực tiếp với *ràng buộc tường minh*. Nhưng chúng ta vẫn có thể sử dụng *ràng buộc cứng* để kiểm tra thứ tự ưu tiên của hai quy tắc.

Trước khi khám phá điều đó trong một danh sách code, hãy nhớ lại cách *ràng buộc cứng* hoạt động, đó là `Function.prototype.bind(..)` tạo ra một hàm wrapper mới được mã hóa cứng để bỏ qua ràng buộc `this` của riêng nó (bất kể nó là gì), và sử dụng ràng buộc thủ công mà chúng tôi cung cấp.

Theo lý luận đó, có vẻ như hiển nhiên để cho rằng *ràng buộc cứng* (là một dạng của *ràng buộc tường minh*) có mức độ ưu tiên cao hơn *ràng buộc mới*, và do đó không thể bị ghi đè bởi `new`.

Hãy kiểm tra:

```js
function foo(something) {
	this.a = something;
}

var obj1 = {};

var bar = foo.bind( obj1 );
bar( 2 );
console.log( obj1.a ); // 2

var baz = new bar( 3 );
console.log( obj1.a ); // 2
console.log( baz.a ); // 3
```

Whoa! `bar` được ràng buộc cứng với `obj1`, nhưng `new bar(3)` **không** thay đổi `obj1.a` thành `3` như chúng ta mong đợi. Thay vào đó, cuộc gọi *ràng buộc cứng* (với `obj1`) tới `bar(..)` ***có thể*** bị ghi đè bởi `new`. Vì `new` đã được áp dụng, chúng tôi đã lấy lại đối tượng mới được tạo, được đặt tên là `baz`, và thực tế chúng ta thấy rằng `baz.a` có giá trị là `3`.

Điều này sẽ gây ngạc nhiên nếu bạn quay lại hàm trợ giúp "giả" của chúng tôi:

```js
function bind(fn, obj) {
    return function() {
        fn.apply( obj, arguments );
    };
}
```

Nếu bạn suy luận về cách thức hoạt động của code trợ giúp, nó không có cách nào để cuộc gọi toán tử `new` ghi đè lên ràng buộc cứng với `obj` như chúng ta vừa quan sát.

Nhưng `Function.prototype.bind(..)` tích hợp từ ES5 trở lên phức tạp hơn, thực tế là khá nhiều. Đây là polyfill (được định dạng lại một chút) được cung cấp bởi trang MDN cho `bind(..)`:

```js
if (!Function.prototype.bind) {
	Function.prototype.bind = function(oThis) {
		if (typeof this !== "function") {
			// closest thing possible to the ECMAScript 5
			// internal IsCallable function
			throw new TypeError( "Function.prototype.bind - what " +
				"is trying to be bound is not callable"
			);
		}

		var aArgs = Array.prototype.slice.call( arguments, 1 ),
			fToBind = this,
			fNOP = function(){},
			fBound = function(){
				return fToBind.apply(
					(
						this instanceof fNOP &&
						oThis ? this : oThis
					),
					aArgs.concat( Array.prototype.slice.call( arguments ) )
				);
			}
		;

		fNOP.prototype = this.prototype;
		fBound.prototype = new fNOP();

		return fBound;
	};
}
```

**Lưu ý:** Polyfill `bind(..)` hiển thị ở trên khác với `bind(..)` tích hợp trong ES5 liên quan đến các hàm được ràng buộc cứng sẽ được sử dụng với `new` (xem bên dưới lý do tại sao điều đó hữu ích). Do polyfill không thể tạo ra một hàm mà không có `.prototype` như tiện ích tích hợp thực hiện, nên có một số gián tiếp tinh tế để gần đúng cùng một hành vi. Hãy cẩn thận nếu bạn định sử dụng `new` với một hàm được ràng buộc cứng và bạn dựa vào polyfill này.

Phần cho phép `new` ghi đè *ràng buộc cứng* là:

```js
this instanceof fNOP &&
oThis ? this : oThis

// ... and:

fNOP.prototype = this.prototype;
fBound.prototype = new fNOP();
```

Chúng tôi sẽ không thực sự đi sâu vào việc giải thích cách thức hoạt động của trò lừa này (nó phức tạp và vượt quá phạm vi của chúng tôi ở đây), nhưng về cơ bản, tiện ích xác định xem hàm được ràng buộc cứng có được gọi bằng `new` hay không (kết quả là một đối tượng được xây dựng mới là `this` của nó), và nếu có, nó sử dụng *đối tượng mới được tạo* đó thay vì *ràng buộc cứng* được chỉ định trước đó cho `this`.

Tại sao `new` có thể ghi đè *ràng buộc cứng* lại hữu ích?

Lý do chính cho hành vi này là để tạo ra một hàm (có thể được sử dụng với `new` để xây dựng các đối tượng) về cơ bản bỏ qua ràng buộc `this` *cứng* nhưng đặt trước một số hoặc tất cả các đối số của hàm. Một trong những khả năng của `bind(..)` là bất kỳ đối số nào được truyền sau đối số ràng buộc `this` đầu tiên sẽ được mặc định thành các đối số chuẩn cho hàm cơ bản (về mặt kỹ thuật được gọi là "ứng dụng một phần", là một phần của "currying").

Ví dụ:

```js
function foo(p1,p2) {
	this.val = p1 + p2;
}

// using `null` here because we don't care about
// the `this` hard-binding in this scenario, and
// it will be overridden by the `new` call anyway!
var bar = foo.bind( null, "p1" );

var baz = new bar( "p2" );

baz.val; // p1p2
```

### Determining `this`

Bây giờ, chúng ta có thể tóm tắt các quy tắc xác định `this` từ vị trí gọi của một cuộc gọi hàm, theo thứ tự ưu tiên của chúng. Hãy đặt những câu hỏi này theo thứ tự này và dừng lại khi quy tắc đầu tiên áp dụng.

1. Liệu hàm có được gọi với `new` không (**ràng buộc mới**)? Nếu có, `this` là đối tượng mới được xây dựng.

    `var bar = new foo()`

2. Liệu hàm có được gọi với `call` hoặc `apply` không (**ràng buộc tường minh**), ngay cả khi bị ẩn bên trong một *ràng buộc cứng* `bind`? Nếu có, `this` là đối tượng được chỉ định rõ ràng.

    `var bar = foo.call( obj2 )`

3. Liệu hàm có được gọi với một ngữ cảnh không (**ràng buộc ngầm định**), còn được gọi là đối tượng sở hữu hoặc chứa? Nếu có, `this` là *đối tượng ngữ cảnh* đó.

    `var bar = obj1.foo()`

4. Ngược lại, mặc định `this` (**ràng buộc mặc định**). Nếu trong chế độ nghiêm ngặt, chọn `undefined`, nếu không, chọn đối tượng `global`.

    `var bar = foo()`

Chỉ vậy thôi. Đó là *tất cả những gì cần* để hiểu các quy tắc ràng buộc `this` cho các cuộc gọi hàm thông thường. Chà... gần như vậy.

## Binding Exceptions

Như thường lệ, có một số *ngoại lệ* đối với các "quy tắc".

Trong một số trường hợp, hành vi ràng buộc `this` có thể gây ngạc nhiên, khi bạn dự định một ràng buộc khác nhưng bạn lại kết thúc với hành vi ràng buộc từ quy tắc *ràng buộc mặc định* (xem trước đó).

### Ignored `this`

Nếu bạn truyền `null` hoặc `undefined` làm tham số ràng buộc `this` cho `call`, `apply`, hoặc `bind`, các giá trị đó sẽ bị bỏ qua và thay vào đó, quy tắc *ràng buộc mặc định* sẽ áp dụng cho việc gọi.

```js
function foo() {
	console.log( this.a );
}

var a = 2;

foo.call( null ); // 2
```

Tại sao bạn lại cố tình truyền thứ gì đó như `null` cho ràng buộc `this`?

Việc sử dụng `apply(..)` để trải các mảng giá trị thành các tham số cho một cuộc gọi hàm là khá phổ biến. Tương tự, `bind(..)` có thể sử dụng currying cho các tham số (giá trị được cài đặt trước), điều này có thể rất hữu ích.

```js
function foo(a,b) {
	console.log( "a:" + a + ", b:" + b );
}

// spreading out array as parameters
foo.apply( null, [2, 3] ); // a:2, b:3

// currying with `bind(..)`
var bar = foo.bind( null, 2 );
bar( 3 ); // a:2, b:3
```

Cả hai tiện ích này đều yêu cầu ràng buộc `this` cho tham số đầu tiên. Nếu các hàm đang đề cập không quan tâm đến `this`, bạn cần một giá trị giữ chỗ và `null` có vẻ như là một lựa chọn hợp lý như được hiển thị trong đoạn code này.

**Lưu ý:** Chúng tôi không đề cập đến nó trong cuốn sách này, nhưng ES6 có toán tử trải `...` cho phép bạn trải cú pháp một mảng thành các tham số mà không cần `apply(..)`, chẳng hạn như `foo(...[1,2])`, tương đương với `foo(1,2)` - tránh ràng buộc `this` về mặt cú pháp nếu không cần thiết. Thật không may, không có thay thế cú pháp ES6 nào cho currying, vì vậy tham số `this` của cuộc gọi `bind(..)` vẫn cần được chú ý.

Tuy nhiên, có một "nguy hiểm" nhỏ ẩn giấu khi luôn sử dụng `null` khi bạn không quan tâm đến ràng buộc `this`. Nếu bạn từng sử dụng nó cho một cuộc gọi hàm (ví dụ, một hàm thư viện của bên thứ ba mà bạn không kiểm soát), và hàm đó *có* tham chiếu `this`, thì quy tắc *ràng buộc mặc định* có nghĩa là nó có thể vô tình tham chiếu (hoặc tệ hơn, thay đổi!) đối tượng `toàn cục` (`window` trong trình duyệt).

Rõ ràng, một cạm bẫy như vậy có thể dẫn đến nhiều lỗi *rất khó* để chẩn đoán/xác định.

#### Safer `this`

Có lẽ một cách thực hành "an toàn" hơn là truyền một đối tượng được thiết lập cụ thể cho `this` được đảm bảo không phải là một đối tượng có thể tạo ra các tác dụng phụ gây ra vấn đề trong chương trình của bạn. Mượn thuật ngữ từ mạng (và quân đội), chúng ta có thể tạo ra một đối tượng "DMZ" (khu phi quân sự) - không gì khác hơn là một đối tượng hoàn toàn trống, không được ủy quyền (xem Chương 5 và 6).

Nếu chúng ta luôn truyền một đối tượng DMZ cho các ràng buộc `this` bị bỏ qua mà chúng ta không nghĩ rằng mình cần quan tâm, chúng ta chắc chắn rằng bất kỳ cách sử dụng `this` ẩn/ngoài dự kiến nào sẽ bị giới hạn trong đối tượng trống, giúp cách ly đối tượng `toàn cục` của chương trình khỏi các tác dụng phụ.

Vì đối tượng này hoàn toàn trống, nên cá nhân tôi thích đặt cho nó tên biến là `ø` (ký hiệu toán học viết thường cho tập hợp rỗng). Trên nhiều bàn phím (như bố cục US trên Mac), ký hiệu này có thể dễ dàng nhập bằng `⌥` + `o` (option + `o`). Một số hệ thống cũng cho phép bạn thiết lập phím tắt cho các ký hiệu cụ thể. Nếu bạn không thích ký hiệu `ø` hoặc bàn phím của bạn không dễ gõ như vậy, tất nhiên bạn có thể gọi nó theo bất kỳ cách nào bạn muốn.

Bất kể bạn gọi nó là gì, cách dễ nhất để thiết lập nó thành **hoàn toàn trống** là `Object.create(null)` (xem Chương 5). `Object.create(null)` tương tự như `{ }`, nhưng không có sự ủy quyền cho `Object.prototype`, vì vậy nó "trống hơn" so với chỉ `{ }`.

```js
function foo(a,b) {
	console.log( "a:" + a + ", b:" + b );
}

// our DMZ empty object
var ø = Object.create( null );

// spreading out array as parameters
foo.apply( ø, [2, 3] ); // a:2, b:3

// currying with `bind(..)`
var bar = foo.bind( ø, 2 );
bar( 3 ); // a:2, b:3
```

Không chỉ an toàn hơn về mặt chức năng, `ø` còn có một lợi ích về mặt phong cách, ở chỗ nó truyền tải ngữ nghĩa "Tôi muốn `this` trống" rõ ràng hơn so với `null`. Nhưng một lần nữa, hãy đặt tên cho đối tượng DMZ của bạn theo ý thích của bạn.

### Indirection

Một điều khác cần lưu ý là bạn có thể (cố ý hoặc không!) tạo ra các "tham chiếu gián tiếp" đến các hàm và trong những trường hợp đó, khi tham chiếu hàm đó được gọi, quy tắc *ràng buộc mặc định* cũng áp dụng.

Một trong những cách phổ biến nhất mà các *tham chiếu gián tiếp* xảy ra là từ một phép gán:

```js
function foo() {
	console.log( this.a );
}

var a = 2;
var o = { a: 3, foo: foo };
var p = { a: 4 };

o.foo(); // 3
(p.foo = o.foo)(); // 2
```

*Giá trị kết quả* của biểu thức gán `p.foo = o.foo` chỉ là một tham chiếu đến chính đối tượng hàm bên dưới. Do đó, vị trí gọi hiệu quả chỉ là `foo()`, không phải `p.foo()` hay `o.foo()` như bạn có thể mong đợi. Theo các quy tắc ở trên, quy tắc *ràng buộc mặc định* sẽ áp dụng.

Nhắc lại: bất kể bạn gọi tới một hàm như thế nào bằng cách sử dụng quy tắc *ràng buộc mặc định*, trạng thái `strict mode` của **nội dung** của hàm được gọi, nơi tạo ra tham chiếu `this` - không phải vị trí gọi hàm - xác định giá trị *ràng buộc mặc định*: đối tượng `global` nếu không ở chế độ `strict mode` hoặc `undefined` nếu ở chế độ `strict mode`.

### Softening Binding

Chúng ta đã thấy trước đó rằng *ràng buộc cứng* là một chiến lược để ngăn chặn cuộc gọi hàm rơi vào quy tắc *ràng buộc mặc định* một cách vô tình, bằng cách buộc nó phải ràng buộc với một `this` cụ thể (trừ khi bạn sử dụng `new` để ghi đè nó!). Vấn đề là, *ràng buộc cứng* làm giảm đáng kể tính linh hoạt của một hàm, ngăn chặn việc ghi đè `this` thủ công bằng cả *ràng buộc ngầm định* hoặc thậm chí là các nỗ lực *ràng buộc tường minh* sau đó.

Sẽ rất tuyệt nếu có một cách để cung cấp một giá trị mặc định khác cho *ràng buộc mặc định* (không phải `global` hoặc `undefined`), trong khi vẫn cho phép hàm có thể được ràng buộc `this` thủ công thông qua các kỹ thuật *ràng buộc ngầm định* hoặc *ràng buộc tường minh*.

Chúng ta có thể xây dựng một tiện ích được gọi là *ràng buộc mềm* mô phỏng hành vi mong muốn của mình.

```js
if (!Function.prototype.softBind) {
	Function.prototype.softBind = function(obj) {
		var fn = this,
			curried = [].slice.call( arguments, 1 ),
			bound = function bound() {
				return fn.apply(
					(!this ||
						(typeof window !== "undefined" &&
							this === window) ||
						(typeof global !== "undefined" &&
							this === global)
					) ? obj : this,
					curried.concat.apply( curried, arguments )
				);
			};
		bound.prototype = Object.create( fn.prototype );
		return bound;
	};
}
```

Tiện ích `softBind(..)` được cung cấp ở đây hoạt động tương tự như tiện ích `bind(..)` tích hợp sẵn của ES5, ngoại trừ hành vi *ràng buộc mềm* của chúng ta. Nó bọc hàm được chỉ định trong logic kiểm tra `this` tại thời điểm gọi và nếu nó là `global` hoặc `undefined`, nó sẽ sử dụng một *mặc định* thay thế được chỉ định trước (`obj`). Ngược lại, `this` sẽ được giữ nguyên. Nó cũng cung cấp currying tùy chọn (xem thảo luận về `bind(..)` trước đó).

Hãy minh họa cách sử dụng của nó:

```js
function foo() {
   console.log("name: " + this.name);
}

var obj = { name: "obj" },
    obj2 = { name: "obj2" },
    obj3 = { name: "obj3" };

var fooOBJ = foo.softBind( obj );

fooOBJ(); // name: obj

obj2.foo = foo.softBind(obj);
obj2.foo(); // name: obj2   <---- look!!!

fooOBJ.call( obj3 ); // name: obj3   <---- look!

setTimeout( obj2.foo, 10 ); // name: obj   <---- falls back to soft-binding
```

Phiên bản ràng buộc mềm của hàm `foo()` có thể được ràng buộc `this` thủ công với `obj2` hoặc `obj3` như được hiển thị, nhưng nó sẽ quay trở lại `obj` nếu *ràng buộc mặc định* áp dụng.

## Lexical `this`

Các hàm thông thường tuân theo 4 quy tắc mà chúng ta vừa đề cập. Nhưng ES6 giới thiệu một loại hàm đặc biệt không sử dụng các quy tắc này: hàm mũi tên.

Hàm mũi tên được ký hiệu không phải bằng từ khóa `function` mà bằng toán tử "mũi tên béo" `=>`. Thay vì sử dụng bốn quy tắc `this` tiêu chuẩn, các hàm mũi tên áp dụng ràng buộc `this` từ phạm vi bao quanh (hàm hoặc toàn cục).

Hãy minh họa phạm vi từ vựng của hàm mũi tên:

```js
function foo() {
	// return an arrow function
	return (a) => {
		// `this` here is lexically adopted from `foo()`
		console.log( this.a );
	};
}

var obj1 = {
	a: 2
};

var obj2 = {
	a: 3
};

var bar = foo.call( obj1 );
bar.call( obj2 ); // 2, not 3!
```

Hàm mũi tên được tạo trong `foo()` bắt giữ theo từ vựng bất kỳ giá trị `this` của `foo()` tại thời điểm gọi. Vì `foo()` được ràng buộc `this` với `obj1`, nên `bar` (tham chiếu đến hàm mũi tên được trả về) cũng sẽ được ràng buộc `this` với `obj1`. Ràng buộc từ vựng của một hàm mũi tên không thể bị ghi đè (ngay cả với `new`!).

Trường hợp sử dụng phổ biến nhất có thể là trong việc sử dụng các hàm gọi lại, chẳng hạn như trình xử lý sự kiện hoặc bộ hẹn giờ:

```js
function foo() {
	setTimeout(() => {
		// `this` here is lexically adopted from `foo()`
		console.log( this.a );
	},100);
}

var obj = {
	a: 2
};

foo.call( obj ); // 2
```

Mặc dù các hàm mũi tên cung cấp một phương pháp thay thế cho việc sử dụng `bind(..)` cho một hàm để đảm bảo `this` của nó, điều này có vẻ hấp dẫn, nhưng cần lưu ý rằng chúng về cơ bản đang vô hiệu hóa cơ chế `this` truyền thống để ưu tiên phạm vi từ vựng được hiểu rộng rãi hơn. Trước ES6, chúng ta đã có một mô hình khá phổ biến để thực hiện điều đó, về cơ bản gần như không thể phân biệt được với tinh thần của các hàm mũi tên ES6:

```js
function foo() {
	var self = this; // lexical capture of `this`
	setTimeout( function(){
		console.log( self.a );
	}, 100 );
}

var obj = {
	a: 2
};

foo.call( obj ); // 2
```

Trong khi `self = this` và các hàm mũi tên đều có vẻ như là những "giải pháp" tốt để không muốn sử dụng `bind(..)`, chúng về cơ bản đang chạy trốn khỏi `this` thay vì hiểu và chấp nhận nó.

Nếu bạn thấy mình đang viết code theo phong cách `this`, nhưng hầu hết hoặc tất cả thời gian, bạn vô hiệu hóa cơ chế `this` bằng các "thủ thuật" từ vựng `self = this` hoặc hàm mũi tên, thì có lẽ bạn nên:

1. Chỉ sử dụng phạm vi từ vựng và quên đi sự giả vờ của code theo phong cách `this`.

2. Chấp nhận hoàn toàn các cơ chế theo phong cách `this`, bao gồm sử dụng `bind(..)` khi cần thiết, và cố gắng tránh các "thủ thuật" từ vựng `self = this` và hàm mũi tên "lexical this".

Một chương trình có thể sử dụng hiệu quả cả hai kiểu code (từ vựng và `this`), nhưng bên trong cùng một hàm và thực sự cho cùng một loại tra cứu, việc trộn lẫn hai cơ chế này thường dẫn đến code khó bảo trì hơn và có lẽ làm việc quá chăm chỉ để trở nên thông minh.

## Review (TL;DR)

Xác định ràng buộc `this` cho một hàm đang thực thi đòi hỏi việc tìm kiếm vị trí gọi trực tiếp của hàm đó. Sau khi xác định, bốn quy tắc có thể được áp dụng cho vị trí gọi theo thứ tự ưu tiên *này*:

1. Được gọi với `new`? Sử dụng đối tượng vừa được tạo.

2. Được gọi với `call` hoặc `apply` (hoặc `bind`)? Sử dụng đối tượng được chỉ định.

3. Được gọi với một đối tượng ngữ cảnh sở hữu cuộc gọi? Sử dụng đối tượng ngữ cảnh đó.

4. Mặc định: `undefined` trong chế độ `strict mode`, đối tượng toàn cục trong trường hợp khác.

Hãy cẩn thận với việc vô tình kích hoạt quy tắc *ràng buộc mặc định*. Trong các trường hợp bạn muốn "an toàn" bỏ qua ràng buộc `this`, một đối tượng "DMZ" như `ø = Object.create(null)` là một giá trị giữ chỗ tốt, bảo vệ đối tượng `toàn cục` khỏi các tác dụng phụ không mong muốn.

Thay vì bốn quy tắc ràng buộc tiêu chuẩn, các hàm mũi tên ES6 sử dụng phạm vi từ vựng cho ràng buộc `this`, có nghĩa là chúng áp dụng ràng buộc `this` (bất kể nó là gì) từ cuộc gọi hàm bao quanh. Về cơ bản, chúng là sự thay thế cú pháp của `self = this` trong lập trình trước ES6.
