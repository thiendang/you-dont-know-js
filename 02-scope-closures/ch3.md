# You Don't Know JS: Scope & Closures
# Chapter 3: Function vs. Block Scope

Như đã nói trong Chương 2, scope là 1 series các "quả bóng" mà "bên trong quả bóng này", các variables và functions được khai báo. Các quả bóng này được lồng bên trong nhau một cách ngăn nắp, và việc sắp xếp này được xác định ngay từ lúc đoạn code được viết xong.

Nhưng liệu các quả bóng scope chỉ được tạo ra bởi các functions? còn 1 cấu trúc nào tạo ra bóng nữa hay không?

## Scope chỉ tạo ra bởi Functions?

Nếu hỏi câu trên, câu trả lời bạn thường gặp nhất là: "*Javascript là ngôn ngữ lập trình có scope tạo bởi function*". Điều đó có nghĩa là mỗi function được khai báo sẽ tạo ra 1 "quả bóng scope" cho chính nó, không còn cấu trúc nào khác tạo ra được bóng scope. Chúng ta sẽ cùng xem xét câu trả lời này qua các phân tích dưới đây, và sẽ thấy là nó không hoàn toàn đúng. 

Xem đoạn code dưới đây: 
```js
function foo(a) {
	var b = 2;

	// một đoạn code nào đó

	function bar() {
		// ...
	}

	// thêm vài đoạn code khác

	var c = 3;
}
```
Trong mẩu code trên, quả bóng scope của `foo(...)` sẽ chứa các variables `a`, `b`, `c` và function `bar`. Việc khai báo này nằm chính xác ở đâu bên trong scope **không quan trọng**, dù sao thì các variables và function kia vẫn thuộc về quả bóng scope `foo(...)` chứa nó. 

`bar(..)` có quả bóng scope của riêng nó. Global scope cũng vậy, nó chứa 1 định danh là `foo`.

Bởi vì `a`, `b`, `c`, và `bar` đều nằm bên trong scope của quả bóng `foo(..)`, ta không thể truy cập những variables và function đó từ bên ngoài của `foo(..)`. Dẫn đến đoạn code sau sẽ trả về thông báo lỗi `ReferenceError` do ở global scope không chứa các định danh `bar` hoặc `a`, `b`, `c`. (Lưu ý của người dịch: Xem lại chương 1, mục *Erros - Các lỗi thường gặp*, ở đây *Engine* thực hiện phép tìm bên phải - RHS - nhưng không thấy các variables và function này ở global scope, dẫn đến việc trả về thông báo lỗi `ReferenceError`).

```js
bar(); // báo lỗi

console.log( a, b, c ); // báo lỗi
```
Tuy vậy, `a`, `b`, `c`, `foo`, và `bar` có thể được gọi từ *bên trong* của `foo(..)`, thậm chí còn được gọi từ bên trong của `bar(..)` (với giả định rằng không tồn tại các định danh trùng tên ở trong `bar(..)`).

Function scope cổ vũ ý tưởng là mọi variables thuộc về 1 function thì có thể được sử dụng và tái sử dụng ở khắp nơi bên trong function (bao gồm cả ở trong những scopes bên trong nó). Cách tiếp cận này rất hữu dụng, nó tận dụng được bản chất "động" của các variables trong JavaScript nhằm lấy values về khi cần.

Mặt khác, nếu không cẩn trọng khi viết code, việc variables có thể được truy cập xuyên suốt bên trong function scope có thể dẫn đến những vấn đề không mong muốn.

## Hiding In Plain Scope

Khi nghĩ đến function, thường thì lập trình viên sẽ liên tưởng ngay đến việc khai báo function, rồi thêm các dòng code bên trong function. Tuy nhiên, hãy thử hình dung nếu ta làm ngược lại:
- viết trước những dòng code sẽ dùng bên trong hàm "tương lai".
- bọc những dòng code trên bằng 1 khai báo hàm.  

Bạn nhìn ra vấn đề ở đây không? Chúng ta vừa tạo ra 1 quả bóng scope để:
- bao lấy những dòng code trên, 
- "che dấu" variables và functions bên trong nó. 

Tại sao việc "che dấu" variables và functions lại là một kỹ thuật hữu ích? Thực ra là có rất nhiều lý do, nhưng chung quy đều xuất phát từ nguyên tắc thiết kế phần mềm tên là "Principle of Least Privilege" [^note-leastprivilege], một vài chỗ gọi nó là "Least Authority" hoặc "Least Exposure". Nguyên lý này nói rằng trong việc thiết kế phần mềm, ví dụ thiết kế API cho 1 module/object, bạn chỉ nên phơi bày tối thiểu những gì cần thiết, và "che" đi những thứ còn lại.

Nguyên lý này được áp dụng ngay vào việc chọn scope sẽ chứa variables và functions. Nếu như toàn bộ variables và functions đều thuộc về global scope thì mọi scope nhỏ bên trong đều sẽ truy cập được đến các variables và functions đấy. Việc này vi phạm nguyên tắc thiết kế nói trên. 

Ví dụ:

```js
function doSomething(a) {
	b = a + doSomethingElse( a * 2 );

	console.log( b * 3 );
}

function doSomethingElse(a) {
	return a - 1;
}

var b;

doSomething( 2 ); // 15
```

Trong mẩu code trên, varialbe `b` và hàm `doSomethingElse(..)` gần như là "tài sản cá nhân" để giúp riêng cho hoạt động của `doSomething(..)`. Việc để cho các các "tài sản" trên phơi bày ra với các scope khác không chỉ là không cần thiết, mà thậm chỉ là còn tiềm ẩn nguy cơ khiến "tài sản riêng" bị sử dụng theo cách ta không mong muốn (cả tốt lẫn xấu).

Để code hợp lý hơn, hãy giấu các tài sản riêng vào bên trong scope của `doSomething(..)`:

```js
function doSomething(a) {
	function doSomethingElse(a) {
		return a - 1;
	}

	var b;

	b = a + doSomethingElse( a * 2 );

	console.log( b * 3 );
}

doSomething( 2 ); // 15
```

Bây giờ ta không thể gọi thẳng `b` và `doSomethingElse(..)` từ các scope bên ngoài mà không thông qua `doSomething(..)`. Chức năng và kết quả cuối cùng của `doSomething(...)` không đổi, trong khi code được coi là chuẩn chỉ hơn. 

### Tránh nguy cơ xung đột 

Một lợi ích khác của việc "giấu" variables và functions bên trong scope là để tránh các nguy cơ xung đột giữa các variables hoặc functions trùng tên nhưng có mục đích sử dụng khác nhau. Một khi variables/ functiosn đã bị xung đột, nó sẽ dẫn đến việc kết quả trả về bị ghi đè theo cách không mong muốn. 

Xem ví dụ sau:

```js
function foo() {
	function bar(a) {
		i = 3; // thay đổi `i` in the enclosing scope's for-loop
		console.log( a + i );
	}

	for (var i=0; i<10; i++) {
		bar( i * 2 ); // oops, infinite loop ahead!
	}
}

foo();
```

Việc gán `i = 3` bên trong `bar(..)` đã chép đè lên `i` được khai báo trong vòng lặp for của `foo(..)`. Nó dẫn đến kết cục là vòng lặp chạy mãi không dừng bởi trong mỗi vòng lặp, `i` lại bị gán bằng `3` và sẽ luôn `< 10`.

`i` bên trong `bar(..)` cần được khai báo như một local variable, không cần biết là định danh của nó là gì. Để làm vậy, hãy thay đổi `i = 3;` thành `var i = 3;` thì vấn đề sẽ biến mất (and would create the previously mentioned "shadowed variable" declaration for `i`). Một cách *khác*, nhưng không nên dùng, đó là sử dụng một định danh hoàn toàn khác cho `i` bên trong vòng lặp for của `foo(...)`, ví dụ dùng `var j = 3;`. Nhưng trong thực tế việc sử dụng trùng tên variables là vô cùng thường gặp và tự nhiên, cho nên sử dụng scope để "che đi" các khai báo variables/ functions bên trong vẫn là cách làm tốt nhất.

#### Global "Namespaces"

Phần này, chúng ta sẽ đề cập đến một ví dụ điển hình của việc các variables bị xung đột trong global scope. Điều này xảy ra khi rất nhiều thư viện được gọi khi chạy chương trình, trong khi lập trình viên quên không ẩn đi các variables và functions vốn là tài sản riêng của từng thư viện. 

Mỗi thư viện sẽ tạo ra một khai báo variable đơn, thường là 1 object, tên gọi của object đó tương đối độc nhất trong global scope. Object này được sử dụng như một "namespace" cho thư viện đó, nơi mà all specific exposures of functionality are made as properties off that object (namespace), rather than as top-level lexically scoped identifiers themselves.

Ví dụ:

```js
var MyReallyCoolLibrary = {
	awesome: "stuff",
	doSomething: function() {
		// ...
	},
	doAnotherThing: function() {
		// ...
	}
};
```

#### Quản lý Module

Một cách làm khác để tránh việc bị xung đột chính là cách xây dựng "module" theo phương cách hiện đại, trong đó sử dụng rất nhiều các công cụ quản lý dependencies. Sử dụng các công cụ này sẽ giúp không thêm bất kỳ tên định danh (identifiers) nào của thư viện vào global scope, mà vẫn giúp những định danh đó được xuất hiện trong 1 scope đặc biệt khác quản lý của dependency managers. 

Lưu ý rằng những công cụ trên không hề có tính năng "thần kỳ" nào giúp tránh khỏi những quy định của lexical scope. Các công cụ đón đơn giản chỉ giúp thực hiện các quy định đã giải thích bên trên về scoping, đó là "cưỡng chế" không cho các định danh được chèn thẳng vào các không gian chung (shared scope), giữ chúng trong những không gian riêng tư hơn, không tiềm ẩn nguy cơ xung đột, từ đó phòng ngừa bất kỳ sự cố nào liên quan đến scope.

As such, you can code defensively and achieve the same results as the dependency managers do without actually needing to use them, if you so choose. See the Chapter 5 for more information about the module pattern.

## Functions As Scopes

Chúng ta đã thấy rằng có thể lấy bất kỳ đoạn code và bao nó bằng một hàm, điều này tạo ra hiệu quả “giấu” bất kỳ biến hoặc hàm đã khai báo với phía bên ngoài phạm vi hoặc hàm khác nằm bên trong phạm vi.

For example:

```js
var a = 2;

function foo() { // <-- insert this

	var a = 3;
	console.log( a ); // 3

} // <-- and this
foo(); // <-- and this

console.log( a ); // 2
```

While this technique "works", it is not necessarily very ideal. There are a few problems it introduces. The first is that we have to declare a named-function `foo()`, which means that the identifier name `foo` itself "pollutes" the enclosing scope (global, in this case). We also have to explicitly call the function by name (`foo()`) so that the wrapped code actually executes.
Đầu tiên là chúng ta phải khai báo một tên hàm foo()​, nghĩa là bản thân định danh foo​ "làm bẩn" phạm vi bên trong, chúng ta đồng thời gọi tên hàm để nó thực thi.

It would be more ideal if the function didn't need a name (or, rather, the name didn't pollute the enclosing scope), and if the function could automatically be executed.
Lý tưởng hơn nếu hàm không cần tên (hoặc tên không làm dơ phạm vi bên trong), và nếu hàm tự động thực thi.

Fortunately, JavaScript offers a solution to both problems.
May mắn là JavaScript có cả giải pháp cho hai vấn đề.

```js
var a = 2;

(function foo(){ // <-- insert this

	var a = 3;
	console.log( a ); // 3

})(); // <-- and this

console.log( a ); // 2
```

Let's break down what's happening here.

First, notice that the wrapping function statement starts with `(function...` as opposed to just `function...`. While this may seem like a minor detail, it's actually a major change. Instead of treating the function as a standard declaration, the function is treated as a function-expression.
Đầu tiên, để ý rằng bao lệnh hàm được bắt đầu bằng (function...​ đối lập với function...​. Trong khi nhìn nó là chi tiết phụ nhưng nó lại là thay đổi chính. Thay vì ứng xử hàm như một khai báo chuẩn, thì hàm lại coi như là một biểu thức của hàm

**Note:** The easiest way to distinguish declaration vs. expression is the position of the word "function" in the statement (not just a line, but a distinct statement). If "function" is the very first thing in the statement, then it's a function declaration. Otherwise, it's a function expression.
Ghi chú: Cách đơn giản để phân biết khai báo hay biểu thức là vị trí của từ “function” trong câu lệnh (không chỉ khác một dòng, mà là cả câu lệnh). Nếu “function” là cái trước nhất của biểu thức, thì nó là khai báo. Ngược lại, nó là một biểu thức hàm.

The key difference we can observe here between a function declaration and a function expression relates to where its name is bound as an identifier.
Mấu chốt khác biệt mà ta thấy ở đây là một khai báo hàm và biểu thức hàm liên quan đến nơi tên của nó được ràng buộc như một định danh.

Compare the previous two snippets. In the first snippet, the name `foo` is bound in the enclosing scope, and we call it directly with `foo()`. In the second snippet, the name `foo` is not bound in the enclosing scope, but instead is bound only inside of its own function.
So sánh hai đoạn code trên. Nếu đoạn code đầu, cái tên foo được ràng buộc với phạm vi của nó, và ta gọi trực tiếp foo()​. Trong đoạn thứ hai, cái tên foo​ không ràng buộc với phạm vi của nó, mà chỉ ràng buộc với bên trong chính hàm của nó.

In other words, `(function foo(){ .. })` as an expression means the identifier `foo` is found *only* in the scope where the `..` indicates, not in the outer scope. Hiding the name `foo` inside itself means it does not pollute the enclosing scope unnecessarily.
Nói khác đi, ​(function foo(){ .. })​ là một biểu thức nghĩa là định danh foo​ chỉ được tìm thấy trong phạm vi ..​chỉ ra, không phải phạm vi bên ngoài. Ẩn foo​ bên trong chính nó nghĩa là không làm ảnh hưởng phạm vi của nó.

### Anonymous vs. Named

You are probably most familiar with function expressions as callback parameters, such as:
Bạn đã quen thuộc với biểu thức hàm là tham chiếu callback, như là:

```js
setTimeout(function(){
	console.log("I waited 1 second!");
}, 1000 );
```

This is called an "anonymous function expression", because `function()...` has no name identifier on it. Function expressions can be anonymous, but function declarations cannot omit the name -- that would be illegal JS grammar.
Cái này được gọi là “biểu thức hàm vô danh”, bởi vì function()...​ không có tên định danh. Biểu thức hàm có thể vô danh, nhưng khai báo hàm phải có tên.

Anonymous function expressions are quick and easy to type, and many libraries and tools tend to encourage this idiomatic style of code. However, they have several draw-backs to consider:
Hàm vô danh nhanh và tiện gõ, và nhiều thư viện và công cụ có xu hướng khuyến khích điều này, nhưng có vài vấn đề cần phải nắm rõ:

1. Anonymous functions have no useful name to display in stack traces, which can make debugging more difficult.Hàm vô danh sẽ không có tên trong truy dấu, khó debug.

2. Without a name, if the function needs to refer to itself, for recursion, etc., the **deprecated** `arguments.callee` reference is unfortunately required. Another example of needing to self-reference is when an event handler function wants to unbind itself after it fires. Không có tên, nếu hàm muốn tham chiếu đến nó, hoặc đệ quy, ..., tham chiếu đã bị bỏ ​arguments.callee​ lại cần thiết. Ví dụ khác nữa là khi một hàm điều khiển sự kiện cần unbind chính nó sau khi chạy.

3. Anonymous functions omit a name that is often helpful in providing more readable/understandable code. A descriptive name helps self-document the code in question. Hàm vô danh làm code khó đọc hơn. Một cái tên còn chính là document của code.

**Inline function expressions** are powerful and useful -- the question of anonymous vs. named doesn't detract from that. Providing a name for your function expression quite effectively addresses all these draw-backs, but has no tangible downsides. The best practice is to always name your function expressions:
Biểu thức hàm trực tiếp rất mạnh và hữu dụng — câu hỏi giữa hàm ẩn danh vs. hàm có tên cũng không ảnh hưởng. Đặt tên tất nhiên là tốt hơn, nhưng không có nhược điểm. Dù gì tốt nhất vẫn phải luôn đặt tên hàm:

```js
setTimeout( function timeoutHandler(){ // <-- Look, I have a name!
	console.log( "I waited 1 second!" );
}, 1000 );
```

### Invoking Function Expressions Immediately

```js
var a = 2;

(function foo(){

	var a = 3;
	console.log( a ); // 3

})();

console.log( a ); // 2
```

Now that we have a function as an expression by virtue of wrapping it in a `( )` pair, we can execute that function by adding another `()` on the end, like `(function foo(){ .. })()`. The first enclosing `( )` pair makes the function an expression, and the second `()` executes the function.
Ta có một hàm như một biểu thức được đặt trong ( )​, chúng ta có thể xử lý hàm đó bằng thêm ()​ vào phía cuối (function foo(){ .. })()​. ​( )​đằng trước tạo ra một biểu thức cho hàm, và ()​ thực thi hàm.

This pattern is so common, a few years ago the community agreed on a term for it: **IIFE**, which stands for **I**mmediately **I**nvoked **F**unction **E**xpression.
Mẫu này rất thông thường, một vài năm trước cộng đồng đã đồng ý cho cụm từ: IIFE, đại diện cho Immediately Invoked Function Expression.

Of course, IIFE's don't need names, necessarily -- the most common form of IIFE is to use an anonymous function expression. While certainly less common, naming an IIFE has all the aforementioned benefits over anonymous function expressions, so it's a good practice to adopt.
Đương nhiên, IIFE không cần tên — dạng thông thường của IIFE được sử dụng theo cách vô danh. Việc đặt tên IIFE không phổ biến nhưng lợi ích đối với hàm ẩn danh nói trên thì đây cũng là việc tốt để thực hành.

```js
var a = 2;

(function IIFE(){

	var a = 3;
	console.log( a ); // 3

})();

console.log( a ); // 2
```

There's a slight variation on the traditional IIFE form, which some prefer: `(function(){ .. }())`. Look closely to see the difference. In the first form, the function expression is wrapped in `( )`, and then the invoking `()` pair is on the outside right after it. In the second form, the invoking `()` pair is moved to the inside of the outer `( )` wrapping pair.
Có một biến tấu nhỏ trong dạng IIFE truyền thống, một số người thích: (function(){ .. }())​. Nhìn kỹ để thấy sựa khác biệt. Trong dạng đầu tiên, biểu thức hàm được bao trong ( )​, và sau đó ​()​ gọi hàm nằm ngay bên ngoài. Trong dạng thứ 2, () lại được bỏ vào trong ( )​.

These two forms are identical in functionality. **It's purely a stylistic choice which you prefer.**
Hai dạng này giống hệt nhau. Tùy theo phong cách bạn chọn thôi.

Another variation on IIFE's which is quite common is to use the fact that they are, in fact, just function calls, and pass in argument(s).
Biến tấu khác của IIFE cũng hay thấy là sử dụng sự kiện trong sự kiện, chỉ gọi hàm, và truyền vào tham số.

For instance:

```js
var a = 2;

(function IIFE( global ){

	var a = 3;
	console.log( a ); // 3
	console.log( global.a ); // 2

})( window );

console.log( a ); // 2
```

We pass in the `window` object reference, but we name the parameter `global`, so that we have a clear stylistic delineation for global vs. non-global references. Of course, you can pass in anything from an enclosing scope you want, and you can name the parameter(s) anything that suits you. This is mostly just stylistic choice.
Ta gọi tham chiếu window​ nhưng đặt tên tham số là global​, nên ta có một cách mô tả rõ ràng giữa đại diện toàn cục vs. không toàn cục. Đương nhiên bạn có thể truyền bất cứ gì vào phạm vi bên trong bạn cần, và bạn có thể đặt tên tham số bất kỳ. Nó hầu như cũng chỉ là phong cách.

Another application of this pattern addresses the (minor niche) concern that the default `undefined` identifier might have its value incorrectly overwritten, causing unexpected results. By naming a parameter `undefined`, but not passing any value for that argument, we can guarantee that the `undefined` identifier is in fact the undefined value in a block of code:
Ứng dụng khác của mẫu này cũng giải quyết một hiệu ứng phụ là định danh undefined​ mặc định có thể có giá trị không hợp lệ ghi đè dẫn đến kết quả không mong muốn. Bằng cách đặt tên tham số undefined,​ nhưng không truyền bất kỳ giá trị đối số
nào, ta có thể đảm bảo giá trị không xác định ở trong khối code:

```js
undefined = true; // setting a land-mine for other code! avoid!

(function IIFE( undefined ){

	var a;
	if (a === undefined) {
		console.log( "Undefined is safe here!" );
	}

})();
```

Still another variation of the IIFE inverts the order of things, where the function to execute is given second, *after* the invocation and parameters to pass to it. This pattern is used in the UMD (Universal Module Definition) project. Some people find it a little cleaner to understand, though it is slightly more verbose.
Và biến thể khác của IIFE là đảo ngược thứ tự các thứ, hàm có thể thực thi sau khi viện dẫn tham số để truyền vào nó. Mẫu này được sử dụng trong dự án UDM (Universal Module Definition). Dù nó hơi dài dòng, một số người lại thấy vậy lại dễ hiểu.

```js
var a = 2;

(function IIFE( def ){
	def( window );
})(function def( global ){

	var a = 3;
	console.log( a ); // 3
	console.log( global.a ); // 2

});
```

The `def` function expression is defined in the second-half of the snippet, and then passed as a parameter (also called `def`) to the `IIFE` function defined in the first half of the snippet. Finally, the parameter `def` (the function) is invoked, passing `window` in as the `global` parameter.
Hàm ​`def`​ được xác định trong phần thứ hai của đoạn code, và sau đó truyền như một tham số (gọi là `def`​) vào hàm ​`IIFE​` ở phần đầu của đoạn code. Cuối cùng, tham số `def`​ (hàm) được gọi, truyền `window`​ vào với tham số `global`​.

## Blocks As Scopes

While functions are the most common unit of scope, and certainly the most wide-spread of the design approaches in the majority of JS in circulation, other units of scope are possible, and the usage of these other scope units can lead to even better, cleaner to maintain code.
Trong khi hàm là đơn vị thường thấy của phạm vi, và quy mô của nó trong thiết kế JS cũng lan rộng khắp chương trình, các đơn vị khác thì lại có thể dẫn đến sự rõ ràng, sạch sẽ để bảo trì code.

Many languages other than JavaScript support Block Scope, and so developers from those languages are accustomed to the mindset, whereas those who've primarily only worked in JavaScript may find the concept slightly foreign.
Nhiều ngôn ngữ khác JavaScript hỗ trợ Block Scope, và người lập trình các ngôn ngữ đó thường quen với khái niệm này, còn dân chỉ chơi JavaScript có thể thấy khái niệm này hơi lạ lẫm.

But even if you've never written a single line of code in block-scoped fashion, you are still probably familiar with this extremely common idiom in JavaScript:
Nhưng mặc dù bạn chưa bao giờ viết bất kỳ dòng code nào theo lối block-scoped, bạn vẫn quen với kiểu này như một thành ngũ trong JavaScript:

```js
for (var i=0; i<10; i++) {
	console.log( i );
}
```

We declare the variable `i` directly inside the for-loop head, most likely because our *intent* is to use `i` only within the context of that for-loop, and essentially ignore the fact that the variable actually scopes itself to the enclosing scope (function or global).
Ta khai báo biến i​ trực tiếp trong đầu vòng lặp for, bởi vì ý định của chúng ta là chỉ sử dụng i​ cho ngữ cảnh của vòng lặp này, và cơ bản bỏ qua ảnh hưởng của phạm vi ngoài (hàm hay toàn cục).

That's what block-scoping is all about. Declaring variables as close as possible, as local as possible, to where they will be used. Another example:
Đó là tất cả nội dung của block scope: khai báo biến tại nơi nó được sử dụng càng cục bộ càng tốt. Ví dụ khác:

```js
var foo = true;

if (foo) {
	var bar = foo * 2;
	bar = something( bar );
	console.log( bar );
}
```

We are using a `bar` variable only in the context of the if-statement, so it makes a kind of sense that we would declare it inside the if-block. However, where we declare variables is not relevant when using `var`, because they will always belong to the enclosing scope. This snippet is essentially "fake" block-scoping, for stylistic reasons, and relying on self-enforcement not to accidentally use `bar` in another place in that scope.
Ta dùng biến bar​ chỉ trong ngữ cảnh của lệnh if, nên nó tạo ra cảm giác rằng ta có thể khai báo nó bên trong khối if. Tuy nhiên, nơi khai báo lại không liên quan khi sử dụng var​, vì nó luôn thuộc về scope bên trong nó. Đoạn code này cơ bản là "giả" block-scoping, và nên dựa vào việc tự thực thi chứ đừng vô tình sử dụng bar​ ở chỗ khác trong phạm vi.

Block scope is a tool to extend the earlier "Principle of Least ~~Privilege~~ Exposure" [^note-leastprivilege] from hiding information in functions to hiding information in blocks of our code.
Block scope là công cụ để mở rộng “Principle of Least Privilege Exposure” (Nguyên tắc tối thiểu) để ẩn thông tin trong hàm thành ẩn thông tin trong khối của code.

Consider the for-loop example again:

```js
for (var i=0; i<10; i++) {
	console.log( i );
}
```

Why pollute the entire scope of a function with the `i` variable that is only going to be (or only *should be*, at least) used for the for-loop?
Vì sao ảnh hưởng toàn bộ phạm vi hàm với biến i​ lại chỉ sử dụng (và chỉ nên sử dụng) cho vòng lặp for?

But more importantly, developers may prefer to *check* themselves against accidentally (re)using variables outside of their intended purpose, such as being issued an error about an unknown variable if you try to use it in the wrong place. Block-scoping (if it were possible) for the `i` variable would make `i` available only for the for-loop, causing an error if `i` is accessed elsewhere in the function. This helps ensure variables are not re-used in confusing or hard-to-maintain ways.
Điều quan trọng nhất là nhà phát triển muốn nó tự kiểm để tránh vô tình sử dụng biến ngoài mục đích, chẳng hạn như báo
lỗi biến không xác định nếu bạn sử dụng biến sai chỗ. Block scope cho biến i​ làm cho ​i​ chỉ khả dụng cho vòng lặp for, sẽ lỗi nếu i​ truy cập chỗ khác trong hàm. Việc này chắc chắn biến không được tái sử dụng nhầm lẫn và khó bảo trì.

But, the sad reality is that, on the surface, JavaScript has no facility for block scope.
Thực tế đáng buồn là xét ở bề mặt thì JavaScript không có cơ sở cho block scope, bạn khai thác thêm mới có.

That is, until you dig a little further.

### `with`

We learned about `with` in Chapter 2. While it is a frowned upon construct, it *is* an example of (a form of) block scope, in that the scope that is created from the object only exists for the lifetime of that `with` statement, and not in the enclosing scope.

### `try/catch`

It's a *very* little known fact that JavaScript in ES3 specified the variable declaration in the `catch` clause of a `try/catch` to be block-scoped to the `catch` block.
Một chi tiết nhỏ rằng từ JavaScript ES3 đã chỉ định khai báo biến trong mệnh đề catch​ của try/catch​ để block-scoped cho khối catch​.

For instance:

```js
try {
	undefined(); // illegal operation to force an exception!
}
catch (err) {
	console.log( err ); // works!
}

console.log( err ); // ReferenceError: `err` not found
```

As you can see, `err` exists only in the `catch` clause, and throws an error when you try to reference it elsewhere.
Như bạn thấy, err​ chỉ tồn tại trong mệnh đề catch,​ và báo lỗi khi bạn muốn thao chiếu nó đâu đó.

**Note:** While this behavior has been specified and true of practically all standard JS environments (except perhaps old IE), many linters seem to still complain if you have two or more `catch` clauses in the same scope which each declare their error variable with the same identifier name. This is not actually a re-definition, since the variables are safely block-scoped, but the linters still seem to, annoyingly, complain about this fact.
**Ghi chú:** Trong khi hành vi này được xác định và đúng với tất cả môi trường JS tiêu chuẩn (có thể ngoại trừ một số trình duyệt IE cũ), nhiều linter có vẻ vẫn khó chịu với nhiều hơn hai mệnh đề catch​ trong cùng một phạm vi mà mỗi khai báo biến lỗi cùng với tên định danh, mặc dù việc này không cần định nghĩa lại vì các biến đã block-scoped an toàn.

To avoid these unnecessary warnings, some devs will name their `catch` variables `err1`, `err2`, etc. Other devs will simply turn off the linting check for duplicate variable names.
Để tránh những cảnh báo không cần thiết, một số lập trình viên sẽ đặt tên biến `catch`​ với `err1`​, `​err2`​... Một số khác chỉ đơn giản tắt báo trùng tên biên của linter.

The block-scoping nature of `catch` may seem like a useless academic fact, but see Appendix B for more information on just how useful it might be.
Bản chất của catch​ block-scoping trông có vẻ vô dụng, nhưng trong Phụ Lục B sẽ giải thích vì sao nó hữu dụng.

### `let`

Thus far, we've seen that JavaScript only has some strange niche behaviors which expose block scope functionality. If that were all we had, and *it was* for many, many years, then block scoping would not be terribly useful to the JavaScript developer.
Chúng ta đã thấy JavaScript cũng chỉ có vài hành vi lạ phơi bày chức năng block scope. Nếu đó là những gì ta có (điều đã xảy ra trong nhiều năm), thì block scoping chẳng có lợi ích cho lập trình viên JavaScript.

Fortunately, ES6 changes that, and introduces a new keyword `let` which sits alongside `var` as another way to declare variables.
May mắn là ES6 đã thay đổi điều này, từ khóa let​ ra như một cách khai báo biến khác bên cạnh var​.

The `let` keyword attaches the variable declaration to the scope of whatever block (commonly a `{ .. }` pair) it's contained in. In other words, `let` implicitly hijacks any block's scope for its variable declaration.
Từ khóa let​ gắn liền việc khai báo với phạm vi của bất kỳ khối nào (thường là trong { .. }​) chứa nó.

```js
var foo = true;

if (foo) {
	let bar = foo * 2;
	bar = something( bar );
	console.log( bar );
}

console.log( bar ); // ReferenceError
```

Using `let` to attach a variable to an existing block is somewhat implicit. It can confuse you if you're not paying close attention to which blocks have variables scoped to them, and are in the habit of moving blocks around, wrapping them in other blocks, etc., as you develop and evolve code.
Sử dụng let​ để gắn một biến vào một block hiện hữu có gì đó hơi ngầm. Nó có thể làm bạn nhầm nếu bạn không để ý block nào có biến nào suốt quá trình phát triển code bằng việc di chuyển block, bao nó trong block khác...

Creating explicit blocks for block-scoping can address some of these concerns, making it more obvious where variables are attached and not. Usually, explicit code is preferable over implicit or subtle code. This explicit block-scoping style is easy to achieve, and fits more naturally with how block-scoping works in other languages:
Tạo các block biệt lập cho block-scoping có thể giải quyết một số mối lo, cho thấy rõ nó có được gắn liền hay không. Thông thường, đoạn code ngầm được ưa dùng hơn code biệt lập, nhưng kiểu tách block-scoping này dễ diễn đạt, và tự nhiên phù hợp hơn cách block-scoping hoạt động như trong các ngôn ngữ khác:

```js
var foo = true;

if (foo) {
	{ // <-- explicit block
		let bar = foo * 2;
		bar = something( bar );
		console.log( bar );
	}
}

console.log( bar ); // ReferenceError
```

We can create an arbitrary block for `let` to bind to by simply including a `{ .. }` pair anywhere a statement is valid grammar. In this case, we've made an explicit block *inside* the if-statement, which may be easier as a whole block to move around later in refactoring, without affecting the position and semantics of the enclosing if-statement.
Ta có thể tạo một block ngẫu nhiên cho let​ đơn giản bằng cách thêm cặp { .. }​ bất cứ chỗ nào trong một cú pháp hợp lệ. Trong trường hợp này, bạn tạo một block biệt lập trong lệnh if, nó giúp dễ dàng hơn khi di chuyển trong quá trình refactor mà không ảnh hưởng vị trí và ngữ nghĩa của lệnh if đi kèm.

**Note:** For another way to express explicit block scopes, see Appendix B.

In Chapter 4, we will address hoisting, which talks about declarations being taken as existing for the entire scope in which they occur.
Trong Chương 4, ta sẽ tìm hiểu hoisting, việc khai báo được đưa ra trước cho toàn bộ phạm vi chứa nó.

However, declarations made with `let` will *not* hoist to the entire scope of the block they appear in. Such declarations will not observably "exist" in the block until the declaration statement.
Tuy nhiên, khai báo được tạo ra với let​ sẽ không​ đưa lên trong toàn bộ block nó xuất hiện. Bởi việc khai báo sẽ không "tồn tại" cho đến khi có biểu thức khai báo.

```js
{
   console.log( bar ); // ReferenceError!
   let bar = 2;
}
```

#### Garbage Collection

Another reason block-scoping is useful relates to closures and garbage collection to reclaim memory. We'll briefly illustrate here, but the closure mechanism is explained in detail in Chapter 5.
Lý do khác cho block-scoping là sự hữu ích liên quan đến closures và gom rác để lấy lại bộ nhớ. Tôi sẽ minh họa ngắn ở đây, cơ chế closure sẽ được giải thích chi tiết trong Chương 5.

Consider:

```js
function process(data) {
	// do something interesting
}

var someReallyBigData = { .. };

process( someReallyBigData );

var btn = document.getElementById( "my_button" );

btn.addEventListener( "click", function click(evt){
	console.log("button clicked");
}, /*capturingPhase=*/false );
```

The `click` function click handler callback doesn't *need* the `someReallyBigData` variable at all. That means, theoretically, after `process(..)` runs, the big memory-heavy data structure could be garbage collected. However, it's quite likely (though implementation dependent) that the JS engine will still have to keep the structure around, since the `click` function has a closure over the entire scope.
Hàm callback điều khiển `click`​ không cần biến `someReallyBigData`​ gì hết. Nghĩa là về mặt lý thuyết, sau khi `process(..)`​ chạy, cấu trúc dữ liệu hao bộ nhớ được gom lại. Tuy nhiên, nó kiểu giống như JS engine vẫn giữ cấu trúc đâu đó, khi hàm ​`click`​ có một closure trên toàn bộ scope.

Block-scoping can address this concern, making it clearer to the engine that it does not need to keep `someReallyBigData` around:
Block-scoping làm cho engine hiểu rõ nó không cần giữ `someReallyBigData` ​xung quanh:

```js
function process(data) {
	// do something interesting
}

// anything declared inside this block can go away after!
{
	let someReallyBigData = { .. };

	process( someReallyBigData );
}

var btn = document.getElementById( "my_button" );

btn.addEventListener( "click", function click(evt){
	console.log("button clicked");
}, /*capturingPhase=*/false );
```

Declaring explicit blocks for variables to locally bind to is a powerful tool that you can add to your code toolbox.
Khai báo các khối riêng biệt cho biến ràng buộc cục bộ là một công cụ mạnh mẽ.

#### `let` Loops

A particular case where `let` shines is in the for-loop case as we discussed previously.
Một trường hợp đặc biệt mà let​ tỏa sáng là trong vòng lặp for.

```js
for (let i=0; i<10; i++) {
	console.log( i );
}

console.log( i ); // ReferenceError
```

Not only does `let` in the for-loop header bind the `i` to the for-loop body, but in fact, it **re-binds it** to each *iteration* of the loop, making sure to re-assign it the value from the end of the previous loop iteration.
Không chỉ có `let`​ trong đầu vòng lặp ràng buộc `i`​ vào thân của vòng lặp, mà còn tái ráng buộc nó vào mỗi lần lặp, đảm bảo gán lại giá trị của nó từ cuối của lần lặp trước.

Here's another way of illustrating the per-iteration binding behavior that occurs:
Đây là cách khác để minh họa hành vi mỗi lần lặp:

```js
{
	let j;
	for (j=0; j<10; j++) {
		let i = j; // re-bound for each iteration!
		console.log( i );
	}
}
```

The reason why this per-iteration binding is interesting will become clear in Chapter 5 when we discuss closures.

Because `let` declarations attach to arbitrary blocks rather than to the enclosing function's scope (or global), there can be gotchas where existing code has a hidden reliance on function-scoped `var` declarations, and replacing the `var` with `let` may require additional care when refactoring code.
Vì khai báo ​`let`​ đính kèm khối tùy ý chứ không phải phạm vi hàm bao quanh (hay toàn cục), nó có thể tạo được tại nơi code hiện hữu có một khai báo ẩn var​ trong function scoped, việc thay thế `var`​ bằng `let`​ có thể cần chú ý khi refactor code.

Consider:

```js
var foo = true, baz = 10;

if (foo) {
	var bar = 3;

	if (baz > bar) {
		console.log( baz );
	}

	// ...
}
```

This code is fairly easily re-factored as:

```js
var foo = true, baz = 10;

if (foo) {
	var bar = 3;

	// ...
}

if (baz > bar) {
	console.log( baz );
}
```

But, be careful of such changes when using block-scoped variables:

```js
var foo = true, baz = 10;

if (foo) {
	let bar = 3;

	if (baz > bar) { // <-- don't forget `bar` when moving!
		console.log( baz );
	}
}
```

See Appendix B for an alternate (more explicit) style of block-scoping which may provide easier to maintain/refactor code that's more robust to these scenarios.
Xem phụ lục B cho kiểu khác (cụ thể hơn) của block-scoping, có thể cung cấp một hướng để bảo trì/refactor code dễ dàng hơn.

### `const`

In addition to `let`, ES6 introduces `const`, which also creates a block-scoped variable, but whose value is fixed (constant). Any attempt to change that value at a later time results in an error.
Bên cạnh `let`,​ ES6 cũng giới thiệu `const`​, cũng là tạo ra biến trong block-scoped, nhưng giá trị được cố định (constant - hằng số). Bất kỳ ý định thay đổi giá trị của nó sẽ bị lỗi.

```js
var foo = true;

if (foo) {
	var a = 2;
	const b = 3; // block-scoped to the containing `if`

	a = 3; // just fine!
	b = 4; // error!
}

console.log( a ); // 3
console.log( b ); // ReferenceError!
```

## Review (TL;DR)

Hàm là nơi tạo ra scope thông dụng nhất trong JavaScript. Những variables và functions dược khai báo bên trong 1 function A sẽ bị ẩn đi, chỉ có thể được truy xuất nội bộ bên trong scope đóng gói bởi fuction A, và đây là nguyên tắc thiết kế chuẩn trong ngành phần mềm. 

Tuy vậy, scope không chỉ được tạo ra bởi các functions mà còn bởi các block-scope, nghĩa là bởi bất kỳ khối code nào được đóng gói bởi cặp `{ .. }`.

Kể từ ES3, cấu trúc `try/catch` tạo ra một block-scope bên trong mệnh đề `catch`.

Trong ES6, từ khoá `let` (chị em họ với từ khoá `var`) được đưa vào cho phép khai báo variables trong bất kỳ khối code nào. `if (..) { let a = 2; }` sẽ khai báo một variable tên `a` và các lập trình viên có thể gọi nó bên trong cặp `{ .. }` của  `if`.

Though some seem to believe so, block scope should not be taken as an outright replacement of `var` function scope. Both functionalities co-exist, and developers can and should use both function-scope and block-scope techniques where respectively appropriate to produce better, more readable/maintainable code.
Mặc dù vậy, block scope cũng không nên được thực hiện như là thay thế hoàn toàn cho `var`​ trong phạm vi hàm. Cả hai đều có thể cùng tồn tại, và người lập trình có thể sử dụng cả kỹ thuật function-scope và block-scope để phù hợp với việc bảo trì, đọc code.

[^note-leastprivilege]: [Principle of Least Privilege](http://en.wikipedia.org/wiki/Principle_of_least_privilege)
