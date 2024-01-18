# You Don't Know JS: Scope & Closures
# Appendix B: Polyfilling Block Scope

Trong Chương 3, chúng ta khám phá Block Scope. Ta thấy rằng mệnh đề `with` and the `catch` đều là hai ví dụ nhỏ của block scope tồn tại trong JS từ ES3.

Khi ES6 giới thiệu `let` cuối cùng cũng cung cấp đầy đủ khả năng mở rộng của block-scoping cho code của chúng ta. Có rất nhiều điều hấp dẫn, cả hàm và phong cách code mà block scope sẽ cho phép.

But what if we wanted to use block scope in pre-ES6 environments?

Consider this code:

```js
{
	let a = 2;
	console.log( a ); // 2
}

console.log( a ); // ReferenceError
```

This will work great in ES6 environments. But can we do so pre-ES6? `catch` is the answer.

```js
try{throw 2}catch(a){
	console.log( a ); // 2
}

console.log( a ); // ReferenceError
```

Whoa! Code nhìn xấu vãi cả dị. Ta thấy `try/catch` xuất hiện để ép việc ném ra lỗi, nhưng "error" chỉ quăng ra giá trị 2​, và sau đó khai báo biến sẽ nhận giá trị bên trong mệnh đề `catch(a)`.

That's right, the `catch` có block-scoping của nó, nghĩa là nó có thể sử dụng như một polyfill cho block scope trong môi trường tiền ES6.

## Traceur

What does Traceur produce from our snippet? You guessed it!

```js
{
	try {
		throw undefined;
	} catch (a) {
		a = 2;
		console.log( a );
	}
}

console.log( a );
```

Với việc sử dụng các công cụ như vậy, chúng ta bắt đầu khai thác lợi thế của block scope bất kể ta có nhắm đến ES6 hay không, bởi vì `try/catch` đã tồn tại và làm việc theo cách này từ hồi ES3.

## Implicit vs. Explicit Blocks

Trong Chương 3, chúng ta nhận diện vài cạm bẫy tiềm tàng với tính bảo trì/refactor của code khi giới thiệu về block-scoping. Có cách nào khác để tận dụng lợi thế của block scope mà giảm thiểu nhược điểm này?

Xem xét hình thức thay thế của `let`, called the "let block" or "let statement" (contrasted with "let declarations" from before).

```js
let (a = 2) {
	console.log( a ); // 2
}

console.log( a ); // ReferenceError
```

Thay vì ngầm chiếm lấy block hiện hữu, lệnh let tạo một block rõ ràng cho ràng buộc scope của nó. Không chỉ làm nổi bật các block minh bạch hơn, mà có lẽ thiết thực hơn trong việc refactor code, nó tạo ra gì đó sạch hơn về mặt ngữ pháp, buộc tất cả các khai báo lên trên đầu block. Nó làm cho bấy kỳ block nào cũng dễ thấy và scope gì của nó hay không.

Là một khuôn mẫu, nó phản ánh cách tiếp cận của nhiều người trong việc scope hóa hàm khi họ tự dời/hoist tất cả khai báo `var` ên trên đầu hàm. Lệnh let đặt chúng lên đầu block theo ý định, và nếu bạn không sử dụng khai báo `let` rải rác, khai báo block scoping của bạn sẽ dễ dàng nhận diện để bảo trì hơn.

We have two options. Ta có thể bắng bằng cách sử dụng cú pháp ES6 hợp lệ và một chút khuôn phép:

```js
/*let*/ { let a = 2;
	console.log( a );
}

console.log( a ); // ReferenceError
```

Nhưng các công cụ tồn tại để giải quyết vấn đề của chúng ta. Vì vậy lựa chọn khác là viết các khối lệnh let rõ ràng và để công cụ chuyển chúng thành code hợp lệ

Vì vậy tôi đã xây dụng một công cụ gọi là "let-er" [^note-let_er] để xử lý vấn đề này. ​*let-er*​ là một trình phiên dịch code ở bước build, nhiệm vụ duy nhất của nó là tìm dạng lệnh let và phiên dịch , những dạng code khác nó vẫn để nguyên, kể cả khai báo let. Bạn có thể sử dụng *let-er*​ một cách an toàn như là bước phiên dịch ES6 ban đầu, sau đó chuyển code của bạn qua gì đó như Traceur nếu cần thiết.

Hơn nữa, *let-er* có một cờ cấu hình (configuration flag) `--es6`, có thể bật (mặc định là tắt), thay đổi kiểu code được tạo. Thay vì dùng `try/catch` ES3 polyfill hack, *let-er* sẽ chuyển đoạn code tuân thủ ES6 đầy đủ mà không phải hack gì cả:

```js
{
	let a = 2;
	console.log( a );
}

console.log( a ); // ReferenceError
```

Bạn có thể dùng *let-er*​ ngay bây giờ và quan trọng nhất là, **bạn có thể sử dụng dạng lệnh let một cách thích hợp và rõ ràng**​ mặc dù nó không (chưa) phải là phần chính thức của ES.

## Performance

Tôi nêu nhanh về hiệu suất `try/catch`, một chút để giải thích câu hỏi "tại sao sử dụng IIFE để tạo scope?"

Trước tiên, hiệu suất của `try/catch` *is* slower, chậm hơn nhưng lại không có một giả định hợp lý rằng phải dùng cách này, hay nó vốn phải vậy. Từ khi TC39 chính thức chấp nhận trình phiên dịch ES6 sử dụng `try/catch`, nhóm Traceur đã yêu cầu Crome cải thiện hiệu suất của `try/catch`, và rõ ràng là họ có động cơ để đòi hỏi.

Tiếp đến, IIFE không phải là cách so sánh ngang bằng với `try/catch`, vì bất kỳ đoạn code nào được bao bởi hàm đều thay đổi ý nghĩa bên trong code, ý nghĩa của `this`, `return`, `break`, and `continue`. IIFE không phải là một thay thế phù hợp. Nó chỉ có thể dùng thủ công tùy trường hợp.

Câu hỏi cụ thể là: bạn có thực sự muốn block-scoping hay không? Nếu có thì công cụ cho bạn một phương án. Nếu không, hãy cứ dùng `var` và tiếp tục code thôi!

[^note-traceur]: [Google Traceur](http://traceur-compiler.googlecode.com/git/demo/repl.html)

[^note-let_er]\: [let-er](https://github.com/getify/let-er)
