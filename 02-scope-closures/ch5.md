# You Don't Know JS: Scope & Closures
# Chapter 5: Scope Closure

Hy vọng đến phần này, tất cả chúng ta đều hiểu thấu đáo về cách scope làm việc.

Giờ chúng ta chuyển qua chú ý đến một vấn đề vô cùng quan trọng, nhưng cũng rất khó nắm bắt, là một phần của ngôn ngữ và *​gần như là truyền thuyết*: **​closure​.** Theo như những gì ta tìm hiểu về lexical scope cho đến nay, thì lợi ích của closure là hiển nhiên. Nhưng có *một người đằng sau bức màn bí mật, chúng ta sẽ phải tìm anh ta.*

If however you have nagging questions about lexical scope, now would be a good time to go back and review Chapter 2 before proceeding.

## Enlightenment

Với những người có kinh nghiệm với JavaScript, nhưng có lẽ chưa bao giờ nắm bắt đầy đủ khái niệm của closure, việc **hiểu closure**​ có thể coi như đạt cảnh giới niết bàn cần phải có sự phấn đấu và hy sinh mới đạt được.

Tôi nhớ vài năm trước, khi tôi đã nắm vững JavaScript, nhưng vẫn không hiểu closure là gì. Có một gợi ý châm chọc rằng ở *phía bên kia*​ của ngôn ngữ, có nhiều hứa hẹn hơn những gì tôi có. Tôi nhớ tôi đã đọc hết source code của một framework và cố gắng hiểu nó hoạt động ra sao. Tôi nhớ lần đầu tiên cái gì đó như “module pattern” đã gợi lên trong tâm trí. Tôi nhớ những khoảnh khắc a-ha!.​

Những gì tôi đã không biết sau đó, cái gì đã khiến tôi cả năm để hiểu, và những gì tôi hy vọng truyền đặt được cho bạn là bí mất: **closure luôn ở xung quanh JavaScript, bạn phải nhận ra và nắm bắt lấy nó**. Closure không phải công cụ đặc biệt mà bạn phải học thêm về cú pháp và pattern. Không, closure thậm chí cũng không phải vũ khí mà bạn phải học cách làm chủ như Luke đã luyện trong The Force (xem Star wars — người dịch).

Closure xảy ra như là kết quả của viết code dựa trên lexical scope. Đôi khi là nó chỉ xảy ra. Thậm chí bạn còn không thực sự có ý định tạo closure để tận dụng lợi thế của chúng. Closure được tạo ra suốt quá trình code. Những gì bạn *đang thiếu* là bối cảnh để nhận ra, nắm bắt và dùng như đòn bẩy cho ý riêng.

## Nitty Gritty

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

Đoạn code này nhìn tương tự như trong phần Nested Scope (phạm vi lồng nhau). Hàm `bar()`​ ​truy cập đến biến `a`​ ở trong phạm vi bao ngoài vì quy tắc tìm kiếm lexical scope (trường hợp này là một tìm kiếm RHS)

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

Hàm ​`bar()​` có lexical scope truy cập vào scope của `foo()​`. Nhưng sau đó, ta lấy chính bản thân hàm `bar()`​, và truyền nó đi như một giá trị. Trường hợp này, ta `return`​ chính hàm ​`bar​` như một tham chiếu.

Sau khi thực thi `foo()​`, chúng ta gán giá trị trả về (hàm `bar()` bên trong) cho một biến gọi là `baz`​, sau đó chúng ta gọi `baz()​`, và dĩ nhiên nó gọi hàm bên trong `bar()`​, chẳng qua là theo cách nhận diện tham chiếu khác mà thôi.

Chắc chắn `bar()​` được thực thi. Nhưng trong trường hợp này, nó lại thực thi *bên ngoài*​ lexical scope mà nó đã khai báo.

Sau khi ​`foo()​` thực thi, thông thường chúng ta cho rằng toàn bộ scope bên trong của `foo()`​ sẽ ra đi, vì *Engine*​ sử dụng công cụ Gom rác​ đi kèm và giải phóng bộ nhớ khi nó không còn sử dụng. Vì dường như nội dung của `foo()`​ đã không còn sử dụng, nó có thể coi là *mất*.

Nhưng “điều kỳ diệu” của closusre không để điều này xảy ra. Nghĩa là phần bên trong scope vẫn đang “sử dụng”, không đi đâu hết. Ai dùng nó? **Chính hàm​** ​bar()​.

Vì ​`bar()`​ đã được khai báo nên nó có lexical scope closure qua phạm vi bên trong của `foo()`,​ việc này đã giúp cho `bar()`​ tồn tại trong việc tham chiếu về sau.

**`bar()`​ vẫn có một tham chiếu đến phạm vi, và điều này được gọi là closure.**

Vì vậy, vài micro giây sau đó, khi biến `baz`​ được gọi (gọi hàm `bar`​ bên trong), nó có quyền truy cập đến author-time của lexical scope, nên nó có thể tiếp cận biến `a​` như ta mong muốn.

Hàm được gọi ngon lành cành đào từ bên ngoài của author-time lexical scope. ​**Closure​** cho phép hàm tiếp tục truy cập lexical scope đã xác định tại author-time.

Tất nhiên bất kỳ phương cách nào mà hàm có thể *truyền đi xung quanh*​ như một giá trị, và được viện dẫn ở chỗ khác, đều là mô tả của việc observe/thực hiện closure.

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

Ta truyền hàm `baz`​ bên trong cho `​bar`,​ và gọi hàm đó (tức là `fn​`), và như vậy closure của nó qua phạm vi bên trong `foo()`​ được observe bằng cách truy cập `a`​.

Cách truyền hàm như vậy có thể theo cách gián tiếp.

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

Dù chúng ta sử dụng phương tiện gì để *vận chuyển* hàm bên trong ra ngoài lexical scope, nó vẫn giữ một tham chiếu phạm vi tại nơi nó được khai báo, và dù ta thực thi nó ở đâu, closure sẽ được thực hiện.

## Now I Can See

Các đoạn code ở trên cũng chỉ để minh họa cấu trúc của việc sử dụng closure. Nhưng tôi hứa là nó còn hơn cả một đứa trẻ có đồ chơi mới. Tôi hứa là closure nó luôn hiện hữu xung quanh code. Hãy xem đoạn code sau:

```js
function wait(message) {

	setTimeout( function timer(){
		console.log( message );
	}, 1000 );

}

wait( "Hello, closure!" );
```

Ta lấy hàm bên trong (`timer`)​ và truyền nó vào `setTimeout(..)​`. Nhưng `​timer`​ có một phạm vi closure qua `wait()​`, và giữ một tham chiếu đến biến `message`​.

Một ngàn mili giây sau khi ta thực thi `wait()​`, scope bên trong nó sẽ thay vì mất đi, nhưng hàm bên trong `timer​` vẫn có closure trong phạm vi.

Đi sâu vào mấu chốt của Engine​, hàm tiện ích dựng sẵn `setTimeout(..)` t​ham chiếu đến vài tham số, tạm gọi là `fn` ​hay `func`​ hay đại loại tương tự vậy. Engine​ sẽ gọi hàm đó, nghĩa là hàm `timer`​ bên trong được gọi, và tham chiếu lexical scope vẫn nguyên vẹn.

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

Tôi không chắc bạn viết code kiểu nào, còn tôi thì thường viết code có trách nhiệm điều khiển toàn bộ “trang bị vũ trang” của closure, nên ví dụ trên là thực tiễn!

Về cơ bản, bất kỳ *khi nào*​ và *​tại đâu* bạn xử lý hàm (hàm truy vập vào lexical scope riêng của chúng) như một giá trị hạng nhất và truyền chúng đâu đó, bạn có thể thấy hàm đó thực hiện closure. Là timers, event handler, gọi Ajax, web worker, cửa sổ thông báo, hay bất kỳ nhiệm vụ động bộ, bất đồng bộ, khi bạn truyền trong một hàm *callback*,​ tức là đã dùng closure.

**Note:** Chapter 3 introduced the IIFE pattern. While it is often said that IIFE (alone) is an example of observed closure, I would somewhat disagree, by our definition above.

```js
var a = 2;

(function IIFE(){
	console.log( a );
})();
```

Code này hoạt động, nhưng nó hoàn toàn không phải là một observation của closure. Vì sao? bởi vì hàm `IIFE`​ không được thực thi bên ngoài lexical scope. Nó vẫn gọi ngay trong cùng một scope mà nó đã được khai báo (scope bên ngoài có biến `a​`). `a​` được tìm thế theo cách tra cứu lexical scope thông thường, không phải là qua closure.

Về mặt kỹ thuật, trong khi closure xảy ra ở thời điểm khai báo, IIFE không phải như vậy.

Mặc dù bản thân IIFE không phải là ví dụ của closure, nó lại tạo ra scope, và nó là một trong những công cụ thông thường nhất để ta tạo ra scope. Vì vậy IIFE có mỗi liên hệ mật thiết với closure, mặc dù bản thân nó không thực hiện closure.

Và giờ thì dừng đọc đi mấy bạn, tôi có nhiệm vụ cho bạn đây. Giờ bạn mở code của bạn lên, coi coi có hàm nào là `closure` hay giá trị là hàm không.

I'll wait.

Now... you see!

## Loops + Closure

Ví dụ điển hình để minh họa closure là vòng lặp for.

```js
for (var i=1; i<=5; i++) {
	setTimeout( function timer(){
		console.log( i );
	}, i*1000 );
}
```

**Note:** Linters often complain when you put functions inside of loops, because the mistakes of not understanding closure are **so common among developers**. We explain how to do so properly here, leveraging the full power of closure. But that subtlety is often lost on linters and they will complain regardless, assuming you don't *actually* know what you're doing.

Linh hồn của đoạn code trên là điều chúng ta *mong muốn* là các số “1”, “2”, .. “5” sẽ được in ra sau mỗi giây tương ứng.

Khi bạn chạy đoạn code này, bạn sẽ có kết quả là “6” được in ra 5 lần theo mỗi giây.

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

Có những mẫu code sử dụng sức mạnh của closure nhưng không xuất hiện trên bề mặt mà thường là callback. Hãy kiểm tra một trong những kiểu mạnh nhất trong đó: *modules.*

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

Hiện tại, đoạn code trên chưa có observable closure. Chúng ta chỉ có biến riêng `something`​, `​another`​, và hàm `​doSomething()​` và `doAnother()`​ bên trong, cả biến và hàm đều có lexical scope bên trong phạm vi của `foo()`​.

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

Đây là mẫu trong JavaScript gọi là `module​`. Cách thông thường nhất để viết một mẫu module thường gọi là “Revealing Module”, cách viết như ở trên.

Hãy kiểm xem có gì trong đoạn code trên.

Trước hết, `CoolModule()`​ chỉ là một hàm, nhưng nó phải được gọi để khởi tạo một module. Nếu không thực thi hàm bên ngoài, việc tạo scope bên trong và closure sẽ không xảy ra.

Tiếp theo, hàm `CoolModule()`​ trả về một object, biểu thị bởi cú pháp object-literal ​`{ key: value, ... }​`. Object mà ta trả có tham chiếu trong nó đến các hàm phía trong, giữ cho chúng ẩn và riêng biệt. Bạn nên nghĩ object này trả giá trị như một **API công khai của module**.

Việc trả giá trị cho object cuối cùng gán cho biến bên ngoài foo​, sau đó ta có thể truy cập những phương thức theo API, ví dụ `foo.doSomething()​`.

**Note:** It is not required that we return an actual object (literal) from our module. We could just return back an inner function directly. jQuery is actually a good example of this. The `jQuery` and `$` identifiers are the public API for the jQuery "module", but they are, themselves, just a function (which can itself have properties, since all functions are objects).

Hàm ​`doSomething()​` và ​`doAnother()`​ có closure thông qua scope bên trong của module (có được nhờ gọi `CoolModule()`​). Khi ta chuyển hàm trong lexical scope ra ngoài, ta thiếu lập một điều kiện rằng closure nào có thể được observe và thực hiện bằng cách tham chiếu vào object chúng ta trả.*

Để cho đơn giản, hãy nhớ rằng có hai "điều kiện" để một đoạn code được gọi là module pattern: *Mô tả đơn giản hơn, có hai yêu cầu cho một module pattern thực hiện:*

1. Nó phải có một function bao bên ngoài, và function này phải được gọi ít nhất một lần (mỗi lần gọi function này sẽ tạo ra 1 instance của module đó).

2. Function bao bên ngoài (tạm gọi là function `A`) phải trả về ít nhất 1 function khác (tạm gọi là function `a`) vốn khai báo bên trong nó (tức là function `a` được khai báo bên trong `A`). Nhờ vậy mà function `a` có thể truy cập & thay đổi các variables bên trong `A` như chương 3 đã đề cập.

Bản thân một object với thuộc tính hàm bên trong nó không thực sự là một module. Trong bối cảnh observable, một object được trả từ một hàm chỉ có các thuộc tính dữ liệu bên trong nó và không có hàm closure nào thì ​*không thực sự* là một module.

Đoạn code ở trên cho thấy một trình tạo module độc lập gọi là `CoolModule()​`, có thể gọi bao nhiêu lần cũng được, mỗi lần gọi thì nó tạo một module tức thì. Một thay đổi nhỏ với mẫu này là khi bạn quan tâm đến việc chỉ có một lần tạo, đây là một dạng "sigleton":

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

Chúng ta đã chuyển hàm module thành IIFE và gọi nó ngay lập tức và gán giá trị trả lại của nó trực tiếp vào module instance đơn ​`foo`​.

Một biến thể mạnh mẽ của module pattern là đặt tên object và bạn trả lại như một API công khai:

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

Vài trình tải/quản lý module phụ thuộc bao lấy mẫu module thành một API gần gũi hơn. Thay vì kiểm tra bất kỳ một thư viện cụ thể, tôi sẽ giới thiệu một chứng minh đơn giản cho khái niệm này nhằm **mục đích minh họa**:

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

Mấu chốt của đoạn code trên là `modules[name] = impl.apply(impl, deps)`.​ Nó sẽ chạy hàm xác định bên ngoài để tạo module, và lưu giá trị trả lại (API của module) thành một danh sách các module được theo dõi theo tên.

Và đây là cách tôi có thể sử dụng để tạo module:

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

Cả module “foo” và “bar” đều được xác định bằng một hàm trả một API công khai. “foo” thậm chí nhận được instance của “bar” như một tham số phụ thuộc, và có thể sử dụng nó tùy ý.

Bạn hãy dành thời gian để kiểm tra đoạn code trên cho đến khi hiểu hoàn toàn sức mạnh của closure để phục vụ cho mục đích của mình. Không có “ma thuật” nào trong trình quản lý module, nó đáp ứng hai đặc tính của module pattern mà tôi đã liệt kê ở trên: gọi một hàm xác định bao ngoài, và trả giá trị như là API của module đó.

Nói cách khác, module là module, kể cả khi bạn tạo một công cụ trên nó.

### Future Modules

ES6 bổ sung cú pháp cao cấp cho khái niệm module. Khi được tải bởi hệ thống module, ES6 xử lý một file như một module riêng lẻ. Mỗi module có thể vừa nhập các module khác hoặc thành viên API cụ thể, đồng thời xuất các thành viên API công khai của chính nó.

**Ghi chú:​** Module nền hàm không phải là nhận dạng mẫu tĩnh (cái mà trình biên dịch hiểu rõ), vì vậy API của nó sẽ không được nhận thấy cho đến run-time. Vì thế, bạn có thể thay đổi một API của module trong quá trình run-time (xem thảo luận về `publicAPI`​ ở trên).

Ngược lại, ES6 API của module là tĩnh (API không thay đổi tại run-time). Do đó, trình biên dịch kiểm tra trong quá trình tải và biên dịch có một tham chiếu đến thành của một API module đã nhập có *thực sự tồn tại*. Nếu tham chiếu API không tồn tại, trình biên dịch báo lỗi “trước” trong thời gian biên dịch (compiler-time) thay vì chờ cho run-time (báo lỗi, nếu có) theo thông thường.

ES6 module không có định dạng “trực tiếp”, nó cần được xác định theo các file riêng lẻ (với mỗi module). Trình duyệt/engine có một trình tải module mặc định (có thể ghi đè, không thuộc vấn đề nói tới ở đây) đồng bộ tải module file khi nó được nhập (import).

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

**Ghi chú:​** Các file riêng lẻ **“foo.js​”** và **“​bar.js”**​ cần được tạo, với nội dung như hai đoạn code trên. Sau đó chương trình của bạn có thể tải/nhập các module đó để sử dụng chúng như đã trình bày ở đoạn code thứ ba.

`import`​ một hay nhiều thành viên từ API của module trong phạm vi gần nhất, mỗi cái là một biến (`hello​` trong trường hợp của ta). `module`​ nhập toàn bộ API module thành các biến (`foo`​, `bar`​ trong trường hợp ví dụ). `export` ​xuất một định danh (biến, hàm) thành API công khai cho module gần nhất. Các biểu thức này có thể sử dụng nhiều lần trong việc xác định module nếu cần thiết.

Nội dung bên trong file module​ được xử lý như một scope closure bên ngoài, cũng như hàm closure đã xem ở trên.

## Review (TL;DR)

Closure dường như là một thế giới JavaScript riêng bí ẩn trong bóng tối mà chỉ có vài chiến binh lập trình viên dũng cảm nhất mới dám bước tới. Tuy vậy, nó thực ra không gì hơn một tiêu chuẩn về cách chúng ta viết code trong môi trường lexical scope, nơi mà functions là những giá trị được truyền qua lại theo ý muốn.

**Closure là đặc tính của một function có thể ghi nhớ và truy cập lexical scope của nó kể cả khi function đó được gọi bên ngoài lexical scope đấy.**

**Khi một hàm có thể ghi nhớ và truy cập lexical scope của nó kể cả khi nó được gọi ngoài lexical scope được gọi là closure.**

Closures khi sử dụng trong vòng lặp có thể khiến kết quả của đoạn code không đúng như ta mong đợi, đơn gian vì ta bất cẩn không chú ý đến cách closurs hoạt động. Tuy vậy, closures cũng đồng thời là một công cụ mạnh mẽ, cho phép triển khai các patterns như *modules pattern* dưới nhiều hình thức khác nhau.

Để được gọi là module, đoạn code đó phải đảm bảo có hai yếu tố sau: (1) function bọc bên ngoài phải được gọi để giúp tạo ra một scope bao bên ngoài, (2) giá trị trả về của function bọc bên ngoài phải chứa tham chiếu đến ít nhất một hàm bên trong, hàm bên trong này có scope là private inner scope của function bọc bên ngoài.

Now we can see closures all around our existing code, and we have the ability to recognize and leverage them to our own benefit!
