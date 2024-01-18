# You Don't Know JS: Scope & Closures
# Appendix C: Lexical-this

Mặc dù chương này không đề cập đến cơ chế làm việc của `this` một cách chi tiết, nhưng do có 1 topic quan trọng trong ES6 liên quan đến `this` và lexical scope, nên chúng ta sẽ tóm tắt nhanh về nó ở đây. 

Trong ES6, có một cú pháp khai báo function đặc biệt tên là "arrow function" (hàm mũi tên), trông giống như sau:

```js
var foo = a => {
	console.log(a);
};

foo( 2 ); // 2
```

Ký hiệu mũi tên `=>` được thay thế cho từ khóa `function`. Nhưng hàm mũi tên không chỉ đơn thuần là cách viết ngắn gọn cho cách khai báo hàm thông thường, nó còn có những đặc điểm mà chúng ta sẽ xem xét bên dưới đây.

Hãy cùng nghiên cứu vấn đề đối với đoạn code bên dưới:

```js

var obj = {
	id: "awesome",
	cool: function coolFn() {
		console.log( this.id );
	}
};

var id = "not awesome";

obj.cool(); // awesome

setTimeout( obj.cool, 100 ); // not awesome
```

Vấn đề xảy ra với đoạn code trên, đó là `this` đã mất đi mối liên hệ với function `cool()`. Có một vài cách để xử lý chuyện này, nhưng một cách hay được nhắc đến đó là `var self = this;`, viết lại đoạn code thành:

```js
var obj = {
	count: 0,
	cool: function coolFn() {
		var self = this;

		if (self.count < 1) {
			setTimeout( function timer(){
				self.count++;
				console.log( "awesome?" );
			}, 100 );
		}
	}
};

obj.cool(); // awesome?
```

Without getting too much into the weeds here, the `var self = this` "solution" chỉ phân phối cho toàn bộ việc hiểu và dùng đúng ràng buộc `this` và quay về cái gì đó mà chúng ta cảm thấy thoải mái: lexical scope, `self` trở thành một định danh có thể giải quyết thông qua lexical scope và closure, không quan tâm những gì xảy ra với ràng buộc `this` trên đường đi.

Người ta không thích viết cái gì dài dòng, đặc biệt là khi cứ phải lặp lại mãi. Bởi vậy, một động lực của ES6 là hỗ giảm bớt hoàn cảnh này, thực tế là *fix* một vấn đề bản chất, như là cái này.

Giải pháp hàm mũi tên của ES6 giới thiệu một hành vi gọi là "lexical this".

```js
var obj = {
	count: 0,
	cool: function coolFn() {
		if (this.count < 1) {
			setTimeout( () => { // arrow-function ftw?
				this.count++;
				console.log( "awesome?" );
			}, 100 );
		}
	}
};

obj.cool(); // awesome?
```

Giải thích ngắn gọn ở trên cho thấy hàm mũi tên không hành xử tất cả như hàm thông thường khi nói đến ràng buộc `this`​. Nó loại bỏ tất cả các quy tắc thông thường của ràng buộc `this`​, và thay vào đó lấy ngay giá trị `this` ​của lexical scope bao ngoài, bất kể nó là gì.

Vì vậy, trong đoạn code trên, hàm mũi tên không nhận được loại bỏ ràng buộc `this`​ của nó theo một số cách không đoán được, nó chỉ "kế thừa" ràng buộc `this`​ của hàm `cool()`​ (điều này đúng nếu ta gọi nó như đã trình bày).

Trong khi nó làm cho code ngắn hơn, quan điểm của tôi là hàm mũi tên chỉ thực sự chỉ làm cho code thành một cú pháp gây sai lầm thường thấy cho lập trình viên nhầm lẫn và pha trộn quy tắc “this binding” với quy tắc “lexical scope”.

Ngược lại với quan điểm trên: tại sao lại đi gặp rắc rối và rườm rà bằng cách sử dụng mô hình `this`,​ chỉ cần cắt bớt và trộn nó với tham chiếu lexical. Việc nắm bắt một trong những cách tiếp cận cho bất kỳ đoạn code mà không phải trộn chúng trong cùng một đoạn code có vẻ tự nhiên hơn.

**Note:** one other detraction from arrow-functions is that they are anonymous, not named. See Chapter 3 for the reasons why anonymous functions are less desirable than named functions.

A more appropriate approach, in my perspective, to this "problem", is to use and embrace the `this` mechanism correctly.

```js
var obj = {
	count: 0,
	cool: function coolFn() {
		if (this.count < 1) {
			setTimeout( function timer(){
				this.count++; // `this` is safe because of `bind(..)`
				console.log( "more awesome" );
			}.bind( this ), 100 ); // look, `bind()`!
		}
	}
};

obj.cool(); // more awesome
```

Dù bạn thích hàm mũi tên với hành vi lexical-this hơn, hay bạn thích sử dụng `bind()`​ hơn, điều quan trọng cần ghi chú là hàm mũi tên k​ hông chỉđơn thuần là đỡ mất công gõ chữ "function". Nó có một *sự khác biệt về hành vi có chủ đích* mà chúng ta cần học và hiểu, nếu ta chọn.

They have an *intentional behavioral difference* that we should learn and understand, and if we so choose, leverage.

Now that we fully understand lexical scoping (and closure!), understanding lexical-this should be a breeze!
