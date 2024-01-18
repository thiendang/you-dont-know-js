# You Don't Know JS: Scope & Closures
# Chapter 5: Scope Closure

Hy vọng đến phần này, tất cả chúng ta đều hiểu thấu đáo về cách scope làm việc.

We turn our attention to an incredibly important, but persistently elusive, *almost mythological*, part of the language: **closure**. If you have followed our discussion of lexical scope thus far, the payoff is that closure is going to be, largely, anticlimactic, almost self-obvious. *There's a man behind the wizard's curtain, and we're about to see him*. No, his name is not Crockford!

*Giờ chúng ta chuyển qua chú ý đến một vấn đề vô cùng quan trọng, nhưng cũng rất khó nắm bắt, là một phần của ngôn ngữ và ​gần như là truyền thuyết: ​closure​. Theo như những gì ta tìm hiểu về lexical scope cho đến nay, thì lợi ích của closure là hiển nhiên. Nhưng có một người đằng sau bức màn bí mật, chúng ta sẽ phải tìm anh ta.*

If however you have nagging questions about lexical scope, now would be a good time to go back and review Chapter 2 before proceeding.

## Enlightenment

For those who are somewhat experienced in JavaScript, but have perhaps never fully grasped the concept of closures, *understanding closure* can seem like a special nirvana that one must strive and sacrifice to attain. *Với những người có kinh nghiệm với JavaScript, nhưng có lẽ chưa bao giờ nắm bắt đầy đủ khái niệm của closure, việc **hiểu closure**​ có thể coi như đạt cảnh giới niết bàn cần phải có sự phấn đấu và hy sinh mới đạt được.*

I recall years back when I had a firm grasp on JavaScript, but had no idea what closure was. The hint that there was *this other side* to the language, one which promised even more capability than I already possessed, teased and taunted me. I remember reading through the source code of early frameworks trying to understand how it actually worked. I remember the first time something of the "module pattern" began to emerge in my mind. I remember the *a-ha!* moments quite vividly. *

What I didn't know back then, what took me years to understand, and what I hope to impart to you presently, is this secret: **closure is all around you in JavaScript, you just have to recognize and embrace it.** Closures are not a special opt-in tool that you must learn new syntax and patterns for. No, closures are not even a weapon that you must learn to wield and master as Luke trained in The Force.

Closures happen as a result of writing code that relies on lexical scope. They just happen. You do not even really have to intentionally create closures to take advantage of them. Closures are created and used for you all over your code. What you are *missing* is the proper mental context to recognize, embrace, and leverage closures for your own will.

The enlightenment moment should be: **oh, closures are already occurring all over my code, I can finally *see* them now.** Understanding closures is like when Neo sees the Matrix for the first time.

## Nitty Gritty

OK, enough hyperbole and shameless movie references.

Here's a down-n-dirty definition of what you need to know to understand and recognize closures:

> Closure là khi một function có thể ghi nhớ và truy cập lexical scope của nó ngay cả khi nó đó thực thi bên ngoài lexical scope đấy.

**Closure là khi một hàm có khả năng nhớ và truy cập lexical scope của nó ngay cả khi hàm đó được thực thi bên ngoài lexical scope của nó.**

Hãy xem ví dụ sau:

```js
function foo() {
	var a = 2;

	function bar() {
		console.log( a ); // 2
	}

	bar();
}

foo();
```

Đoạn code trên trông tương tự với những gì đã thảo luận ở phần Nested Scope (Scope lồng nhau). Function `bar()` có *quyền truy cập* đến variable `a` ở scope bao bên ngoài theo quy định về phép tìm kiếm của lexical scope (trong trường hợp này, đây là phép tìm bên phải).

Đoạn code trên có minh họa cho "closure" hay không? Có lẽ, về mặt kỹ thuật thì ... *có vẻ đúng*. But by our what-you-need-to-know definition above... *not exactly*. I think the most accurate way to explain `bar()` referencing `a` is via lexical scope look-up rules, and those rules are *only* (an important!) **part** of what closure is.

Về mặt lý thuyết thuần túy, những gì đoạn code trên thể hiện cho thấy function `bar()` có một *closure* over the scope of `foo()` (and indeed, even over the rest of the scopes it has access to, such as the global scope in our case). Put slightly differently, it's said that `bar()` closes over the scope of `foo()`. Why? Because `bar()` appears nested inside of `foo()`. Plain and simple.

But, closure defined in this way is not directly *observable*, nor do we see closure *exercised* in that snippet. We clearly see lexical scope, but closure remains sort of a mysterious shifting shadow behind the code.

Chà, về kỹ thuật thì... *có lẽ*. Nhưng những gì bạn cần biết như ở trên thì... *không chính xác*.​ Tôi nghĩ cách chính xác nhất để giải thích `bar()​` tham chiếu `a`​ thông qua quy tắc tìm kiếm lexical scope, và các quy tắc (quan trọng!) đó chỉ là **​một phần** của closure.

Từ góc độ hàn lâm thuần túy, hàm `bar()`​ ở trên được giải thích là nó có *​closure*​ trong phạm vi của `foo()`​ (và thậm chí thực ra là trong phần con lại của scope nó có quyền truy cập đến, phạm vi toàn cục chẳng hạn). Còn nói hơi khác thì `bar()`​ đóng kín trong phạm vi của `foo()`​. Vì sao? Vì `​bar()` ​xuất hiện lồng bên trong `foo()`,​ đơn giản thẳng đuột ruột ngựa.

Nhưng, xác định closure theo cách này không trực tiếp observable (quan sát được), và chúng ta cũng không thấy closure ​thể hiện gì trong đoạn code trên. Ta thấy rõ lexical scope, nhưng closure vẫn là gì đó ẩn sâu bên trong.


Let us then consider code which brings closure into full light:

```js
function foo() {
	var a = 2;

	function bar() {
		console.log( a );
	}

	return bar;
}

var baz = foo();

baz(); // 2 -- Whoa, closure was just observed, man.
```

Function `bar()` có lexical scope truy cập đến scope bên trong `foo()`. But then, we take `bar()`, the function itself, and pass it *as* a value. Trong trường hợp này, we `return` the function object itself that `bar` references. *Hàm ​`bar()​` có lexical scope truy cập vào scope của `foo()​`. Nhưng sau đó, ta lấy chính bản thân hàm `bar()`​, và truyền nó đi như một giá trị. Trường hợp này, ta `return`​ chính hàm ​`bar​` như một tham chiếu.*

After we execute `foo()`, we assign the value it returned (our inner `bar()` function) to a variable called `baz`, and then we actually invoke `baz()`, which of course is invoking our inner function `bar()`, just by a different identifier reference. *Sau khi thực thi `foo()​`, chúng ta gán giá trị trả về (hàm `bar()` bên trong) cho một biến gọi là `baz`​, sau đó chúng ta gọi `baz()​`, và dĩ nhiên nó gọi hàm bên trong `bar()`​, chẳng qua là theo cách nhận diện tham chiếu khác mà thôi.*

Function `bar()` đã được thực thi, nhưng mà thực thi *bên ngoài* lexical scope mà nó được khai báo. *Chắc chắn bar()​ được thực thi. Nhưng trong trường hợp này, nó lại thực thi bên ngoài​ lexical scope mà nó đã khai báo.*

Sau khi `foo()` được thực thi, về mặt cảm tính chúng ta sẽ cho rằng toàn bộ scope bên trong của `foo()` sẽ biến mất, bởi ta biết *Engine* sẽ cho *Garbage Collector* hoạt động để giải phóng bộ nhớ một khi `foo()` không còn cần đến nữa. Since it would appear that the contents of `foo()` are no longer in use, it would seem natural that they should be considered *gone*. *Sau khi ​foo()​ thực thi, thông thường chúng ta cho rằng toàn bộ scope bên trong của foo()​ sẽ ra đi, vì Engine​ sử dụng công cụ Gom rác​ đi kèm và giải phóng bộ nhớ khi nó không còn sử dụng. Vì dường như nội dung của foo()​ đã không còn sử dụng, nó có thể coi là mất.*

But the "magic" of closures does not let this happen. That inner scope is in fact *still* "in use", and thus does not go away. Who's using it? **The function `bar()` itself**. *Nhưng “điều kỳ diệu” của closusre không để điều này xảy ra. Nghĩa là phần bên trong scope vẫn đang “sử dụng”, không đi đâu hết. Ai dùng nó? Chính hàm​ ​bar()​.*

By virtue of where it was declared, `bar()` has a lexical scope closure over that inner scope of `foo()`, which keeps that scope alive for `bar()` to reference at any later time. *Vì ​bar()​ đã được khai báo nên nó có lexical scope closure qua phạm vi bên trong của foo(),​ việc này đã giúp cho bar()​ tồn tại trong việc tham chiếu về sau.*

**`bar()` still has a reference to that scope, and that reference is called closure.** | **bar()​ vẫn có một tham chiếu đến phạm vi, và điều này được gọi là closure.**

So, a few microseconds later, when the variable `baz` is invoked (invoking the inner function we initially labeled `bar`), it duly has *access* to author-time lexical scope, so it can access the variable `a` just as we'd expect. *Vì vậy, vài micro giây sau đó, khi biến baz​ được gọi (gọi hàm bar​ bên trong), nó có quyền truy cập đến author-time của lexical scope, nên nó có thể tiếp cận biến a​ như ta mong muốn.*

The function is being invoked well outside of its author-time lexical scope. **Closure** lets the function continue to access the lexical scope it was defined in at author-time. *Hàm được gọi ngon lành cành đào từ bên ngoài của author-time lexical scope. ​**Closure​** cho phép hàm tiếp tục truy cập lexical scope đã xác định tại author-time.*

Of course, any of the various ways that functions can be *passed around* as values, and indeed invoked in other locations, are all examples of observing/exercising closure. *Tất nhiên bất kỳ phương cách nào mà hàm có thể truyền đi xung quanh​ như một giá trị, và được viện dẫn ở chỗ khác, đều là mô tả của việc observe/thực hiện closure.*

```js
function foo() {
	var a = 2;

	function baz() {
		console.log( a ); // 2
	}

	bar( baz );
}

function bar(fn) {
	fn(); // look ma, I saw closure!
}
```

We pass the inner function `baz` over to `bar`, and call that inner function (labeled `fn` now), and when we do, its closure over the inner scope of `foo()` is observed, by accessing `a`. *Ta truyền hàm `baz`​ bên trong cho `​bar`,​ và gọi hàm đó (tức là `fn​`), và như vậy closure của nó qua phạm vi bên trong `foo()`​ được observe bằng cách truy cập `a`​.*

These passings-around of functions can be indirect, too. *Cách truyền hàm như vậy có thể theo cách gián tiếp.*

```js
var fn;

function foo() {
	var a = 2;

	function baz() {
		console.log( a );
	}

	fn = baz; // assign `baz` to global variable
}

function bar() {
	fn(); // look ma, I saw closure!
}

foo();

bar(); // 2
```

Whatever facility we use to *transport* an inner function outside of its lexical scope, it will maintain a scope reference to where it was originally declared, and wherever we execute it, that closure will be exercised. *Dù chúng ta sử dụng phương tiện gì để vận chuyển hàm bên trong ra ngoài lexical scope, nó vẫn giữ một tham chiếu phạm vi tại nơi nó được khai báo, và dù ta thực thi nó ở đâu, closure sẽ được thực hiện.*

## Now I Can See

The previous code snippets are somewhat academic and artificially constructed to illustrate *using closure*. But I promised you something more than just a cool new toy. I promised that closure was something all around you in your existing code. Let us now *see* that truth. *Các đoạn code ở trên cũng chỉ để minh họa cấu trúc của việc sử dụng closure. Nhưng tôi hứa là nó còn hơn cả một đứa trẻ có đồ chơi mới. Tôi hứa là closure nó luôn hiện hữu xung quanh code. Hãy xem đoạn code sau:*

```js
function wait(message) {

	setTimeout( function timer(){
		console.log( message );
	}, 1000 );

}

wait( "Hello, closure!" );
```

We take an inner function (named `timer`) and pass it to `setTimeout(..)`. But `timer` has a scope closure over the scope of `wait(..)`, indeed keeping and using a reference to the variable `message`. *Ta lấy hàm bên trong (`timer`)​ và truyền nó vào `setTimeout(..)​`. Nhưng `​timer`​ có một phạm vi closure qua `wait()​`, và giữ một tham chiếu đến biến `message`​.*

A thousand milliseconds after we have executed `wait(..)`, and its inner scope should otherwise be long gone, that inner function `timer` still has closure over that scope. *Một ngàn mili giây sau khi ta thực thi `wait()​`, scope bên trong nó sẽ thay vì mất đi, nhưng hàm bên trong timer​ vẫn có closure trong phạm vi.*

Deep down in the guts of the *Engine*, the built-in utility `setTimeout(..)` has reference to some parameter, probably called `fn` or `func` or something like that. *Engine* goes to invoke that function, which is invoking our inner `timer` function, and the lexical scope reference is still intact. *Đi sâu vào mấu chốt của Engine​, hàm tiện ích dựng sẵn `setTimeout(..)` t​ham chiếu đến vài tham số, tạm gọi là `fn` ​hay `func`​ hay đại loại tương tự vậy. Engine​ sẽ gọi hàm đó, nghĩa là hàm `timer`​ bên trong được gọi, và tham chiếu lexical scope vẫn nguyên vẹn.*

**Closure.**

Or, if you're of the jQuery persuasion (or any JS framework, for that matter):

```js
function setupBot(name,selector) {
	$( selector ).click( function activator(){
		console.log( "Activating: " + name );
	} );
}

setupBot( "Closure Bot 1", "#bot_1" );
setupBot( "Closure Bot 2", "#bot_2" );
```

I am not sure what kind of code you write, but I regularly write code which is responsible for controlling an entire global drone army of closure bots, so this is totally realistic! *Tôi không chắc bạn viết code kiểu nào, còn tôi thì thường viết code có trách nhiệm điều khiển toàn bộ “trang bị vũ trang” của closure, nên ví dụ trên là thực tiễn!*

(Some) joking aside, essentially *whenever* and *wherever* you treat functions (which access their own respective lexical scopes) as first-class values and pass them around, you are likely to see those functions exercising closure. Be that timers, event handlers, Ajax requests, cross-window messaging, web workers, or any of the other asynchronous (or synchronous!) tasks, when you pass in a *callback function*, get ready to sling some closure around! *Về cơ bản, bất kỳ khi nào​ và ​tại đâu bạn xử lý hàm (hàm truy vập vào lexical scope riêng của chúng) như một giá trị hạng nhất và truyền chúng đâu đó, bạn có thể thấy hàm đó thực
hiện closure. Là timers, event handler, gọi Ajax, web worker, cửa sổ thông báo, hay bất kỳ nhiệm vụ động bộ, bất đồng bộ, khi bạn truyền trong một hàm callback,​ tức là đã dùng closure.*

**Note:** Chapter 3 introduced the IIFE pattern. While it is often said that IIFE (alone) is an example of observed closure, I would somewhat disagree, by our definition above.

```js
var a = 2;

(function IIFE(){
	console.log( a );
})();
```

This code "works", but it's not strictly an observation of closure. Why? Because the function (which we named "IIFE" here) is not executed outside its lexical scope. It's still invoked right there in the same scope as it was declared (the enclosing/global scope that also holds `a`). `a` is found via normal lexical scope look-up, not really via closure. *Code này hoạt động, nhưng nó hoàn toàn không phải là một observation của closure. Vì sao? bởi vì hàm IIFE​ không được thực thi bên ngoài lexical scope. Nó vẫn gọi ngay trong cùng một scope mà nó đã được khai báo (scope bên ngoài có biến a​). a​ được tìm thế theo cách tra cứu lexical scope thông thường, không phải là qua closure.*

While closure might technically be happening at declaration time, it is *not* strictly observable, and so, as they say, *it's a tree falling in the forest with no one around to hear it.* *Về mặt kỹ thuật, trong khi closure xảy ra ở thời điểm khai báo, IIFE không phải như vậy.*

Though an IIFE is not *itself* an example of closure, it absolutely creates scope, and it's one of the most common tools we use to create scope which can be closed over. So IIFEs are indeed heavily related to closure, even if not exercising closure themselves. *Mặc dù bản thân IIFE không phải là ví dụ của closure, nó lại tạo ra scope, và nó là một trong những công cụ thông thường nhất để ta tạo ra scope. Vì vậy IIFE có mỗi liên hệ mật thiết với closure, mặc dù bản thân nó không thực hiện closure.*

Put this book down right now, dear reader. I have a task for you. Go open up some of your recent JavaScript code. Look for your functions-as-values and identify where you are already using closure and maybe didn't even know it before. *Và giờ thì dừng đọc đi mấy bạn, tôi có nhiệm vụ cho bạn đây. Giờ bạn mở code của bạn lên, coi coi có hàm nào là `closure` hay giá trị là hàm không.*

I'll wait.

Now... you see!

## Loops + Closure

The most common canonical example used to illustrate closure involves the humble for-loop.

```js
for (var i=1; i<=5; i++) {
	setTimeout( function timer(){
		console.log( i );
	}, i*1000 );
}
```

**Note:** Linters often complain when you put functions inside of loops, because the mistakes of not understanding closure are **so common among developers**. We explain how to do so properly here, leveraging the full power of closure. But that subtlety is often lost on linters and they will complain regardless, assuming you don't *actually* know what you're doing.

The spirit of this code snippet is that we would normally *expect* for the behavior to be that the numbers "1", "2", .. "5" would be printed out, one at a time, one per second, respectively. *Linh hồn của đoạn code trên là điều chúng ta mong muốn là các số “1”, “2”, .. “5” sẽ được in ra sau mỗi giây tương ứng.*

In fact, if you run this code, you get "6" printed out 5 times, at the one-second intervals. *Khi bạn chạy đoạn code này, bạn sẽ có kết quả là “6” được in ra 5 lần theo mỗi giây.*

**Huh?**

Đầu tiên, hãy cùng xem con số `6` đến từ đâu. Điều kiện để dừng vòng lặp là khi giá trị của `i` *không* `<=5`. Như vậy khi vòng lặp dừng thì `i` bằng 6. Vì thế, kết quả in ra chính là giá trị cuối cùng của `i` sau khi vòng lặp dừng.

Điều này mới đầu thì trông có vẻ hợp lý bởi các function callback trên đều được chạy chậm lại một quãng thời gian sau khi vòng lặp kết thúc. Tuy vậy, kể cả khi ta đặt `setTimeout(.., 0)` cho từng vòng lặp, thì kết quả vẫn là `6` giống như trên. Điều này chứng tỏ còn có gì nằm sâu hơn cần phải tìm hiểu. Chúng ta đã *quên* gì trong đoạn code trên để nó có thể dẫn đến kết quả như ta mong muống?

Vấn đề ở đây chính là vì chúng ta *tưởng* mỗi vòng lặp sẽ "lưu" lại bản sao của `i` tại thời điểm đó. Tuy nhiên, theo cái cách mà scope hoạt động, thì tất cả các functions trên, dẫu có được khai báo riêng biệt trong từng vòng lặp, thì đều **ở trong 1 scope chung là global scope**, mà scope này vốn chỉ có một `i` trong nó.

Khi hiểu như vậy thì *đương nhiên* là mọi functions đều chia sẻ cùng một tham chiếu đến `i`. Cấu trúc của vòng lặp thường khiến cho chúng ta bối rối, nghĩ rằng sẽ có thứ gì phức tạp hơn. Câu trả lời là không. Không có gì khác ngoài việc cứ như là không hề có vòng lặp, mà chỉ có 5 timeout callback được khai báo lần lượt từng function một. 

Ok, quay trở lại câu hỏi nhức đầu trên, chúng ta đang thiếu thứ gì? Chúng ta cần thêm closured scope. Chính xác hơn, chúng ta cần closured scope (hay là scope của riêng từng function) cho mỗi vòng lặp.

Trong chương 3, ta đã biết là IIFE tạo ra scope bằng cách khai báo và thực thi hàm ngay lập tức. Vậy hãy thử với IIFE xem sao:

```js
for (var i=1; i<=5; i++) {
	(function(){
		setTimeout( function timer(){
			console.log( i );
		}, i*1000 );
	})();
}
```

Nó có cho kết quả như ta cần không? **KHÔNG.** Tại sao? Rõ ràng chúng ta đã có thêm lexical scope. Mỗi timeout function callback thực tế đã được đóng gói trong scope tạo ra bởi từng IIFE trong từng vòng lặp. 

Câu trả lời là vì scope mà IIFE tạo ra ở trên là **scope rỗng**. Scope này cần thêm *dữ liệu* để có thể sử dụng. Nó cần variable của chính nó, chính là bản sao giá trị của `i` trong từng vòng lặp.

```js
for (var i=1; i<=5; i++) {
	(function(){
		var j = i;
		setTimeout( function timer(){
			console.log( j );
		}, j*1000 );
	})();
}
```

**Eureka! Nó đã chạy!**

Một phiên bản khác được ưu thích hơn là:

```js
for (var i=1; i<=5; i++) {
	(function(j){
		setTimeout( function timer(){
			console.log(j);
		}, j*1000 );
	})(i);
}
```

Rõ ràng, bởi IIFEs thực ra chỉ là các functions, chúng ta có thể truyền tham số `i` cho nó, và gọi tham số truyền vào là `j` nếu muốn, hoặc thậm chí vẫn để là `i` cũng được. Cả hai cách này đều cho kết quả giống nhau. Việc dùng IIFE bên trong mỗi vòng lặp đã tạo ra một scope mới ở từng vòng lặp, trong mỗi scope này có chứa variable mà ta có quyền truy cập giá trị của variable đấy.

Vậy là vấn đề trên đã được giải quyết!

### Cùng xem lại "Block Scoping"

Khi đọc kỹ lại phần trên, chúng ta thấy rằng ta đã sử dụng IIFE để tạo ra scope mới trong mỗi vòng lặp. Vậy thực ra để giải quyết vấn đề, chúng ta *đã cần* những **block scope** (scope có phạm vi trong từng khối code) trong các vòng lặp. Việc này gợi lại chương 3, chương đã đề cập đến khai báo dùng `let`, một từ khóa đã thay đổi đặc tính của 1 khối code, giúp khai báo variable ngay trong khối code đấy. 

**Từ đó,  essentially turns a block into a scope that we can close over.** So, the following awesome code "just works":

**Cơ bản nó tạo một block trong scope để chúng ta có thể đóng nó.** V​ì vậy đoạn code dưới đây chạy ngon:

```js
for (var i=1; i<=5; i++) {
	let j = i; // yay, block-scope for closure!
	setTimeout( function timer(){
		console.log( j );
	}, j*1000 );
}
```

*But, that's not all!* (in my best Bob Barker voice). There's a special behavior defined for `let` declarations used in the head of a for-loop. This behavior says that the variable will be declared not just once for the loop, **but each iteration**. And, it will, helpfully, be initialized at each subsequent iteration with the value from the end of the previous iteration.

Nhưng chưa hết! Nó có một hành vi đặc biệt được xác định cho khai báo `​let​` được sử dụng trên đầu của vòng lặp for. Hành vi này cho biết rằng biến sẽ được khai báo không chỉ một lần cho vòng lặp, mà còn là mỗi lần lặp. Và nó được khởi tạo tại mỗi lần lặp với giá trị từ cuối cho đến lần lặp trước. (mới thấy `var` và `​let`​ khác biệt rõ rệt nhé - người dịch)


```js
for (let i=1; i<=5; i++) {
	setTimeout( function timer(){
		console.log( i );
	}, i*1000 );
}
```

How cool is that? Block scoping and closure working hand-in-hand, solving all the world's problems. I don't know about you, but that makes me a happy JavaScripter. *Block scoping và closure đã tay trong tay hoạt động cùng nhau, giải quyết mọi thứ trên thế giới. Tôi không biết bạn sao chớ nó làm cho tôi vui với JavaScript.*
## Modules

There are other code patterns which leverage the power of closure but which do not on the surface appear to be about callbacks. Let's examine the most powerful of them: *the module*. *Có những mẫu code sử dụng sức mạnh của closure nhưng không xuất hiện trên bề mặt mà thường là callback. Hãy kiểm tra một trong những kiểu mạnh nhất trong đó: mô-đun​.*

```js
function foo() {
	var something = "cool";
	var another = [1, 2, 3];

	function doSomething() {
		console.log( something );
	}

	function doAnother() {
		console.log( another.join( " ! " ) );
	}
}
```

As this code stands right now, there's no observable closure going on. We simply have some private data variables `something` and `another`, and a couple of inner functions `doSomething()` and `doAnother()`, which both have lexical scope (and thus closure!) over the inner scope of `foo()`. *Hiện tại, đoạn code trên chưa có observable closure. Chúng ta chỉ có biến riêng `something`​, `​another`​, và hàm `​doSomething()​` và `doAnother()`​ bên trong, cả biến và hàm đều có lexical scope bên trong phạm vi của `foo()`​.*

But now consider:

```js
function CoolModule() {
	var something = "cool";
	var another = [1, 2, 3];

	function doSomething() {
		console.log( something );
	}

	function doAnother() {
		console.log( another.join( " ! " ) );
	}

	return {
		doSomething: doSomething,
		doAnother: doAnother
	};
}

var foo = CoolModule();

foo.doSomething(); // cool
foo.doAnother(); // 1 ! 2 ! 3
```

This is the pattern in JavaScript we call *module*. Một trong những cách thông dụng nhất để áp dụng module pattern là mô hình "Revealing Module", và chúng ta sẽ xem xét biến thể của pattern này bên dưới đây. *Đây là mẫu trong JavaScript gọi là `module​`. Cách thông thường nhất để viết một mẫu module thường gọi là “Revealing Module”, cách viết như ở trên.*

Let's examine some things about this code. *Hãy kiểm xem có gì trong đoạn code trên.*

Đầu tiên, `CoolModule()` chỉ đơn giản là một function, nhưng nó *cần phải được invoked* để tạo tạo ra instance của module. Không thực thi function bên ngoài (`CoolModule()`) thì sẽ không có inner scope và closures. *Trước hết, `CoolModule()`​ chỉ là một hàm, nhưng nó phải được gọi để khởi tạo một module. Nếu không thực thi hàm bên ngoài, việc tạo scope bên trong và closure sẽ không xảy ra.*

Thứ hai, function `CoolModule()` trả về một object dưới dạng `{ key: value, ... }`. Object được trả về có tham chiếu đến functions bên trong của `CoolModule()`, nhưng *không* tham chiếu đến các variables của `CoolModule()` như `something` hoặc `another`. Chúng ta giữ những variable này, không để lộ ra ngoài. Hãy coi object trả về kia như một **public API cho module của chúng ta**. *Tiếp theo, hàm `CoolModule()`​ trả về một object, biểu thị bởi cú pháp object-literal ​`{ key: value, ... }​`. Object mà ta trả có tham chiếu trong nó đến các hàm phía trong, giữ cho chúng ẩn và riêng biệt. Bạn nên nghĩ object này trả giá trị như một **API công khai của module**.*

Khi variable `foo` được gán với `CoolModule()`, đồng nghĩa với việc nó cũng được gán với object trả về của `CoolModule()`, sau đó ta có thể truy cập vào các thuộc tính của API, như là `foo.doSomething()`. *Việc trả giá trị cho object cuối cùng gán cho biến bên ngoài foo​, sau đó ta có thể truy cập những phương thức theo API, ví dụ `foo.doSomething()​`.*

**Note:** It is not required that we return an actual object (literal) from our module. We could just return back an inner function directly. jQuery is actually a good example of this. The `jQuery` and `$` identifiers are the public API for the jQuery "module", but they are, themselves, just a function (which can itself have properties, since all functions are objects).

Hai functions `doSomething()` và `doAnother()` có closure ứng với scope bên trong của module "instance" (có được nhờ vào việc invoke `CoolModule()`). Khi chúng ta gọi các hàm đó bên ngoài lexical scope của nó (thông qua các tham chiếu đến object được trả về kia), ta đã thiết lập một môi trường để quan sát và triển khai closure. *Hàm ​`doSomething()​` và ​`doAnother()`​ có closure thông qua scope bên trong của module (có được nhờ gọi `CoolModule()`​). Khi ta chuyển hàm trong lexical scope ra ngoài, ta thiếu lập một điều kiện rằng closure nào có thể được observe và thực hiện bằng cách tham chiếu vào object chúng ta trả.*

Để cho đơn giản, hãy nhớ rằng có hai "điều kiện" để một đoạn code được gọi là module pattern: *Mô tả đơn giản hơn, có hai yêu cầu cho một module pattern thực hiện:*

1. Nó phải có một function bao bên ngoài, và function này phải được gọi ít nhất một lần (mỗi lần gọi function này sẽ tạo ra 1 instance của module đó).

2. Function bao bên ngoài (tạm gọi là function `A`) phải trả về ít nhất 1 function khác (tạm gọi là function `a`) vốn khai báo bên trong nó (tức là function `a` được khai báo bên trong `A`). Nhờ vậy mà function `a` có thể truy cập & thay đổi các variables bên trong `A` như chương 3 đã đề cập.

Một object có chứa property là function thì không *thực sự* là module. An object which is returned from a function invocation which only has data properties on it and no closured functions is not *really* a module, in the observable sense. *Bản thân một object với thuộc tính hàm bên trong nó không thực sự là một module. Trong bối cảnh observable, một object được trả từ một hàm chỉ có các thuộc tính dữ liệu bên trong nó và không có hàm closure nào thì ​không thực sự là một module.*

The code snippet above shows a standalone module creator called `CoolModule()` which can be invoked any number of times, each time creating a new module instance. A slight variation on this pattern is when you only care to have one instance, a "singleton" of sorts: *Đoạn code ở trên cho thấy một trình tạo module độc lập gọi là `CoolModule()​`, có thể gọi bao nhiêu lần cũng được, mỗi lần gọi thì nó tạo một module tức thì. Một thay đổi nhỏ với mẫu này là khi bạn quan tâm đến việc chỉ có một lần tạo, đây là một dạng "sigleton":*

```js
var foo = (function CoolModule() {
	var something = "cool";
	var another = [1, 2, 3];

	function doSomething() {
		console.log( something );
	}

	function doAnother() {
		console.log( another.join( " ! " ) );
	}

	return {
		doSomething: doSomething,
		doAnother: doAnother
	};
})();

foo.doSomething(); // cool
foo.doAnother(); // 1 ! 2 ! 3
```

Ở đây, chúng ta chuyển module function thành một IIFE (xem Chương 3), rồi gọi nó *ngay lập tức*, gán giá trị trả về trực tiếp cho 1 instance của module này (instance có định danh là `foo`). *Chúng ta đã chuyển hàm module thành IIFE và gọi nó ngay lập tức và gán giá trị trả lại của nó trực tiếp vào module instance đơn ​`foo`​.*

Do modules chỉ đơn giản là functions, vì vậy mà chúng ta có thể truyền tham số cho modules: *Một biến thể mạnh mẽ của module pattern là đặt tên object và bạn trả lại như một API công khai:*

```js
function CoolModule(id) {
	function identify() {
		console.log( id );
	}

	return {
		identify: identify
	};
}

var foo1 = CoolModule( "foo 1" );
var foo2 = CoolModule( "foo 2" );

foo1.identify(); // "foo 1"
foo2.identify(); // "foo 2"
```

Một phiên bản khác (nhưng mạnh mẽ hơn) của modele pattern đó là việc đặt tên object trả về là publicAPI (Ghi chú của người dịch: Việc đặt tên này giúp người đọc - một lập trình viên khác, hiểu ngay đây đâu là giá trị trả về được công khai cho người dùng):

```js
var foo = (function CoolModule(id) {
	function change() {
		// modifying the public API
		publicAPI.identify = identify2;
	}

	function identify1() {
		console.log(id);
	}

	function identify2() {
		console.log(id.toUpperCase());
	}

	var publicAPI = {
		change: change,
		identify: identify1
	};

	return publicAPI;
})("foo module");

foo.identify(); // foo module
foo.change();
foo.identify(); // FOO MODULE
```

Bằng cách giữ một tham chiếu bên trong tới object "publicAPI", mà object này lại đặt bên trong của module, ta có thể thay đổi các thuộc tính của module instance **từ bên trong**, kể cả việc thêm, bớt methods, thuộc tính, *và* thay đổi các giá trị của chúng.

### Modern Modules

Various module dependency loaders/managers essentially wrap up this pattern of module definition into a friendly API. Rather than examine any one particular library, let me present a *very simple* proof of concept **for illustration purposes (only)**: *Vài trình tải/quản lý module phụ thuộc bao lấy mẫu module thành một API gần gũi hơn. Thay vì kiểm tra bất kỳ một thư viện cụ thể, tôi sẽ giới thiệu một chứng minh đơn giản cho khái niệm này nhằm **mục đích minh họa**:*

```js
var MyModules = (function Manager() {
	var modules = {};

	function define(name, deps, impl) {
		for (var i=0; i<deps.length; i++) {
			deps[i] = modules[deps[i]];
		}
		modules[name] = impl.apply( impl, deps );
	}

	function get(name) {
		return modules[name];
	}

	return {
		define: define,
		get: get
	};
})();
```

The key part of this code is `modules[name] = impl.apply(impl, deps)`. This is invoking the definition wrapper function for a module (passing in any dependencies), and storing the return value, the module's API, into an internal list of modules tracked by name. *Mấu chốt của đoạn code trên là `modules[name] = impl.apply(impl, deps)`.​ Nó sẽ chạy hàm xác định bên ngoài để tạo module, và lưu giá trị trả lại (API của module) thành một danh sách các module được theo dõi theo tên.*

And here's how I might use it to define some modules: *Và đây là cách tôi có thể sử dụng để tạo module:*

```js
MyModules.define( "bar", [], function(){
	function hello(who) {
		return "Let me introduce: " + who;
	}

	return {
		hello: hello
	};
} );

MyModules.define( "foo", ["bar"], function(bar){
	var hungry = "hippo";

	function awesome() {
		console.log( bar.hello( hungry ).toUpperCase() );
	}

	return {
		awesome: awesome
	};
} );

var bar = MyModules.get( "bar" );
var foo = MyModules.get( "foo" );

console.log(
	bar.hello( "hippo" )
); // Let me introduce: hippo

foo.awesome(); // LET ME INTRODUCE: HIPPO
```

Both the "foo" and "bar" modules are defined with a function that returns a public API. "foo" even receives the instance of "bar" as a dependency parameter, and can use it accordingly. *Cả module “foo” và “bar” đều được xác định bằng một hàm trả một API công khai. “foo” thậm chí nhận được instance của “bar” như một tham số phụ thuộc, và có thể sử dụng nó tùy ý.*

Spend some time examining these code snippets to fully understand the power of closures put to use for our own good purposes. The key take-away is that there's not really any particular "magic" to module managers. They fulfill both characteristics of the module pattern I listed above: invoking a function definition wrapper, and keeping its return value as the API for that module. *Bạn hãy dành thời gian để kiểm tra đoạn code trên cho đến khi hiểu hoàn toàn sức mạnh của closure để phục vụ cho mục đích của mình. Không có “ma thuật” nào trong trình quản lý module, nó đáp ứng hai đặc tính của module pattern mà tôi đã liệt kê ở trên: gọi một hàm xác định bao ngoài, và trả giá trị như là API của module đó.*

In other words, modules are just modules, even if you put a friendly wrapper tool on top of them. *Nói cách khác, module là module, kể cả khi bạn tạo một công cụ trên nó.*

### Future Modules

ES6 adds first-class syntax support for the concept of modules. When loaded via the module system, ES6 treats a file as a separate module. Each module can both import other modules or specific API members, as well export their own public API members. *ES6 bổ sung cú pháp cao cấp cho khái niệm module. Khi được tải bởi hệ thống module, ES6 xử lý một file như một module riêng lẻ. Mỗi module có thể vừa nhập các module khác hoặc thành viên API cụ thể, đồng thời xuất các thành viên API công khai của chính nó.*

**Note:** Function-based modules aren't a statically recognized pattern (something the compiler knows about), so their API semantics aren't considered until run-time. That is, you can actually modify a module's API during the run-time (see earlier `publicAPI` discussion).

**Ghi chú:​** Module nền hàm không phải là nhận dạng mẫu tĩnh (cái mà trình biên dịch hiểu rõ), vì vậy API của nó sẽ không được nhận thấy cho đến run-time. Vì thế, bạn có thể thay đổi một API của module trong quá trình run-time (xem thảo luận về `publicAPI`​ ở trên).

By contrast, ES6 Module APIs are static (the APIs don't change at run-time). Since the compiler knows *that*, it can (and does!) check during (file loading and) compilation that a reference to a member of an imported module's API *actually exists*. If the API reference doesn't exist, the compiler throws an "early" error at compile-time, rather than waiting for traditional dynamic run-time resolution (and errors, if any). *Ngược lại, ES6 API của module là tĩnh (API không thay đổi tại run-time). Do đó, trình biên dịch kiểm tra trong quá trình tải và biên dịch có một tham chiếu đến thành của một API module đã nhập có thực sự tồn tại. Nếu tham chiếu API không tồn tại, trình biên dịch báo lỗi “trước” trong thời gian biên dịch (compiler-time) thay vì chờ cho run-time (báo lỗi, nếu có) theo thông thường.*

ES6 modules **do not** have an "inline" format, they must be defined in separate files (one per module). The browsers/engines have a default "module loader" (which is overridable, but that's well-beyond our discussion here) which synchronously loads a module file when it's imported. *ES6 module không có định dạng “trực tiếp”, nó cần được xác định theo các file riêng lẻ (với mỗi module). Trình duyệt/engine có một trình tải module mặc định (có thể ghi đè, không thuộc vấn đề nói tới ở đây) đồng bộ tải module file khi nó được nhập (import).*

Consider:

**bar.js**
```js
function hello(who) {
	return "Let me introduce: " + who;
}

export hello;
```

**foo.js**
```js
// import only `hello()` from the "bar" module
import hello from "bar";

var hungry = "hippo";

function awesome() {
	console.log(
		hello( hungry ).toUpperCase()
	);
}

export awesome;
```

```js
// import the entire "foo" and "bar" modules
module foo from "foo";
module bar from "bar";

console.log(
	bar.hello( "rhino" )
); // Let me introduce: rhino

foo.awesome(); // LET ME INTRODUCE: HIPPO
```

**Note:** Separate files **"foo.js"** and **"bar.js"** would need to be created, with the contents as shown in the first two snippets, respectively. Then, your program would load/import those modules to use them, as shown in the third snippet.
**Ghi chú:​** Các file riêng lẻ **“foo.js​”** và **“​bar.js”**​ cần được tạo, với nội dung như hai đoạn code trên. Sau đó chương trình của bạn có thể tải/nhập các module đó để sử dụng chúng như đã trình bày ở đoạn code thứ ba.

`import` imports one or more members from a module's API into the current scope, each to a bound variable (`hello` in our case). `module` imports an entire module API to a bound variable (`foo`, `bar` in our case). `export` exports an identifier (variable, function) to the public API for the current module. These operators can be used as many times in a module's definition as is necessary.
*`import`​ một hay nhiều thành viên từ API của module trong phạm vi gần nhất, mỗi cái là một biến (`hello​` trong trường hợp của ta). `module`​ nhập toàn bộ API module thành các biến (`foo`​, `bar`​ trong trường hợp ví dụ). `export` ​xuất một định danh (biến, hàm) thành API công khai cho module gần nhất. Các biểu thức này có thể sử dụng nhiều lần trong việc xác định module nếu cần thiết.*

The contents inside the *module file* are treated as if enclosed in a scope closure, just like with the function-closure modules seen earlier. *Nội dung bên trong file module​ được xử lý như một scope closure bên ngoài, cũng như hàm closure đã xem ở trên.*

## Review (TL;DR)

Closure dường như là một thế giới JavaScript riêng bí ẩn trong bóng tối mà chỉ có vài chiến binh lập trình viên dũng cảm nhất mới dám bước tới. Tuy vậy, nó thực ra không gì hơn một tiêu chuẩn về cách chúng ta viết code trong môi trường lexical scope, nơi mà functions là những giá trị được truyền qua lại theo ý muốn.

**Closure là đặc tính của một function có thể ghi nhớ và truy cập lexical scope của nó kể cả khi function đó được gọi bên ngoài lexical scope đấy.**

Closures khi sử dụng trong vòng lặp có thể khiến kết quả của đoạn code không đúng như ta mong đợi, đơn gian vì ta bất cẩn không chú ý đến cách closurs hoạt động. Tuy vậy, closures cũng đồng thời là một công cụ mạnh mẽ, cho phép triển khai các patterns như *modules pattern* dưới nhiều hình thức khác nhau.

Để được gọi là module, đoạn code đó phải đảm bảo có hai yếu tố sau: (1) function bọc bên ngoài phải được gọi để giúp tạo ra một scope bao bên ngoài, (2) giá trị trả về của function bọc bên ngoài phải chứa tham chiếu đến ít nhất một hàm bên trong, hàm bên trong này có scope là private inner scope của function bọc bên ngoài.

Now we can see closures all around our existing code, and we have the ability to recognize and leverage them to our own benefit!
