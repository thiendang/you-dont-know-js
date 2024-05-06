# You Don't Know JS: *this* & Object Prototypes
# Chapter 3: Objects

In Chapters 1 and 2, we explained how the `this` binding points to various objects depending on the call-site of the function invocation. But what exactly are objects, and why do we need to point to them? We will explore objects in detail in this chapter.

## Syntax

Các đối tượng trong JavaScript có hai dạng: dạng khai báo (literal) và dạng xây dựng (constructor).

Cú pháp khai báo cho một đối tượng trông như thế này:

```js
var myObj = {
  key: value
  // ...
};
```

Dạng xây dựng trông như thế này:

```js
var myObj = new Object();
myObj.key = value;
```

Cả hai dạng xây dựng và khai báo đều tạo ra cùng một loại đối tượng. Điểm khác biệt duy nhất là bạn có thể thêm một hoặc nhiều cặp key/value vào khai báo dạng literal, trong khi với các đối tượng dạng xây dựng, bạn phải thêm các thuộc tính từng cái một.

**Lưu ý:** Rất hiếm khi sử dụng "dạng xây dựng" để tạo đối tượng như cách vừa trình bày. Bạn gần như luôn muốn sử dụng cú pháp dạng khai báo. Điều tương tự cũng đúng với hầu hết các đối tượng được tích hợp sẵn (xem bên dưới).

## Type

Đối tượng là khối xây dựng cơ bản mà phần lớn JavaScript được xây dựng trên. Chúng là một trong 6 kiểu dữ liệu chính (được gọi là "kiểu ngôn ngữ" trong thông số kỹ thuật) trong JS:

* `string` (chuỗi)
* `number` (số)
* `boolean` (kiểu logic)
* `null`
* `undefined`
* `object` (đối tượng)

Lưu ý rằng các *kiểu nguyên thủy đơn giản* (`string`, `number`, `boolean`, `null`, và `undefined`) **không phải** là chính bản thân các `object`. `null` đôi khi được gọi là một kiểu object, nhưng quan niệm sai lầm này bắt nguồn từ một lỗi trong ngôn ngữ khiến `typeof null` trả về chuỗi `"object"` một cách không chính xác (và gây nhầm lẫn). Thực tế, `null` là kiểu nguyên thủy riêng của nó.

**Một tuyên bố sai lầm phổ biến là "mọi thứ trong JavaScript đều là một object". Điều này rõ ràng là không đúng.**

Ngược lại, có một vài **kiểu con đặc biệt của object**, mà chúng ta có thể gọi là *kiểu nguyên thủy phức hợp*.

`function` là một kiểu con của object (về mặt kỹ thuật, nó là một "object có thể gọi"). Các hàm trong JS được gọi là "hạng nhất" vì về cơ bản chúng chỉ là các object thông thường (với ngữ nghĩa hành vi có thể gọi được), vì vậy chúng có thể được xử lý giống như bất kỳ object đơn giản nào khác.

Mảng cũng là một dạng của object, với các hành vi bổ sung. Cách tổ chức nội dung trong mảng có cấu trúc hơn một chút so với các object thông thường.

### Built-in Objects

Có một số kiểu con khác của object, thường được gọi là các object tích hợp sẵn. Đối với một số trong số chúng, tên của chúng dường như ngầm định chúng có liên quan trực tiếp đến các đối tác nguyên thủy đơn giản của chúng, nhưng trên thực tế, mối quan hệ của chúng phức tạp hơn, điều mà chúng ta sẽ khám phá ngay sau đây.

* `String`
* `Number`
* `Boolean`
* `Object`
* `Function`
* `Array`
* `Date`
* `RegExp`
* `Error`

Những object tích hợp sẵn này có vẻ như là các kiểu thực tế, thậm chí là các lớp, nếu bạn dựa vào sự tương đồng với các ngôn ngữ khác như lớp `String` của Java.

Nhưng trong JS, chúng thực sự chỉ là các hàm tích hợp sẵn. Mỗi hàm tích hợp sẵn này có thể được sử dụng như một constructor (nghĩa là, một cuộc gọi hàm với toán tử `new` - xem Chương 2), kết quả là một object mới được *xây dựng* thuộc kiểu con đang đề cập. Ví dụ:

```js
var strPrimitive = "I am a string";
typeof strPrimitive;              // "string"
strPrimitive instanceof String;         // false

var strObject = new String( "I am a string" );
typeof strObject;                 // "object"
strObject instanceof String;          // true

// kiểm tra kiểu con của object
Object.prototype.toString.call( strObject );  // [object String]
```

Chúng ta sẽ xem chi tiết trong một chương sau chính xác cách thức hoạt động của `Object.prototype.toString...`, nhưng tóm lại, chúng ta có thể kiểm tra kiểu con bên trong bằng cách mượn phương thức `toString()` mặc định cơ bản, và bạn có thể thấy nó tiết lộ rằng `strObject` là một object thực sự được tạo bởi constructor `String`.

Giá trị nguyên thủy `"I am a string"` không phải là một object, nó là một giá trị nguyên thủy và không thể thay đổi. Để thực hiện các thao tác trên nó, chẳng hạn như kiểm tra độ dài của nó, truy cập vào từng ký tự nội dung của nó, v.v., thì cần có một object `String`.

May mắn thay, ngôn ngữ tự động ép kiểu một nguyên thủy `"string"` thành một object `String` khi cần thiết, điều đó có nghĩa là bạn hầu như không bao giờ cần phải tạo explicit object form. **Được khuyến khích mạnh mẽ** bởi phần lớn cộng đồng JS là sử dụng literal form cho một giá trị, khi có thể, thay vì constructed object form.

Ví dụ:

```js
var strPrimitive = "I am a string";

console.log( strPrimitive.length );     // 13

console.log( strPrimitive.charAt( 3 ) );  // "m"
```

Trong cả hai trường hợp, chúng ta gọi một thuộc tính hoặc phương thức trên một string nguyên thủy, và engine tự động ép kiểu nó thành một object `String`, để truy cập thuộc tính/phương thức hoạt động.

Loại ép kiểu tương tự cũng xảy ra giữa số nguyên thủy `42` và object wrapper `new Number(42)`, khi sử dụng các phương thức như `42.359.toFixed(2)`. Tương tự đối với các object `Boolean` từ các nguyên thủy `"boolean"`.

`null` và `undefined` không có object wrapper form, chỉ có các giá trị nguyên thủy của chúng. Ngược lại, các giá trị `Date` *chỉ có thể* được tạo với constructed object form của chúng, vì chúng không có literal form tương ứng.

`Object`, `Array`, `Function` và `RegExp` (biểu thức chính quy) đều là các object bất kể literal form hay constructed form được sử dụng. Constructed form cung cấp, trong một số trường hợp, nhiều tùy chọn hơn trong việc tạo so với literal form. Vì các object được tạo theo cả hai cách, nên literal form đơn giản hơn gần như luôn được ưu tiên. **Chỉ sử dụng constructed form nếu bạn cần các tùy chọn bổ sung.**

Các object `Error` hiếm khi được tạo explicit trong code, nhưng thường được tạo tự động khi các ngoại lệ được throw. Chúng có thể được tạo với constructed form `new Error(..)`, nhưng thường không cần thiết.

## Contents

Như đã đề cập trước đó, nội dung của một object bao gồm các giá trị (bất kỳ loại nào) được lưu trữ tại các *vị trí* được đặt tên cụ thể, mà chúng ta gọi là thuộc tính (properties).

Điều quan trọng cần lưu ý là trong khi chúng ta nói "nội dung" ngụ ý rằng các giá trị này *thực sự* được lưu trữ bên trong object, đó chỉ là bề ngoài. Engine lưu trữ các giá trị theo cách phụ thuộc vào implementation, và rất có thể không lưu trữ chúng *trong* một container object nào đó. Những gì *được* lưu trữ trong container là các tên thuộc tính này, chúng hoạt động như các con trỏ (về mặt kỹ thuật, *tham chiếu*) đến nơi lưu trữ các giá trị.

Ví dụ:

```js
var myObject = {
  a: 2
};

myObject.a;   // 2

myObject["a"];  // 2
```

Để truy cập giá trị tại *vị trí* `a` trong `myObject`, chúng ta cần sử dụng toán tử `.` hoặc toán tử `[ ]`. Cú pháp `.a` thường được gọi là truy cập "thuộc tính", trong khi cú pháp `["a"]` thường được gọi là truy cập "key". Trong thực tế, cả hai đều truy cập cùng một *vị trí* và sẽ lấy ra cùng một giá trị, `2`, vì vậy các thuật ngữ có thể được sử dụng thay thế cho nhau. Chúng ta sẽ sử dụng thuật ngữ phổ biến nhất, "truy cập thuộc tính" từ đây trở đi.

Sự khác biệt chính giữa hai cú pháp là toán tử `.` yêu cầu một tên thuộc tính tương thích với `Identifier` sau nó, trong khi cú pháp `[".."]` có thể lấy bất kỳ chuỗi UTF-8/unicode tương thích nào làm tên cho thuộc tính. Ví dụ, để tham chiếu đến một thuộc tính có tên "Super-Fun!", bạn sẽ phải sử dụng cú pháp truy cập `["Super-Fun!"]`, vì `Super-Fun!` không phải là một tên thuộc tính `Identifier` hợp lệ.

Ngoài ra, vì cú pháp `[".."]` sử dụng **giá trị** của một chuỗi để xác định vị trí, điều này có nghĩa là chương trình có thể xây dựng giá trị của chuỗi theo chương trình, chẳng hạn như:

```js
var wantA = true;
var myObject = {
  a: 2
};

var idx;

if (wantA) {
  idx = "a";
}

// later

console.log( myObject[idx] ); // 2
```

Trong các object, tên thuộc tính **luôn luôn** là chuỗi. Nếu bạn sử dụng bất kỳ giá trị nào khác ngoài `string` (nguyên thủy) làm thuộc tính, nó sẽ được chuyển đổi thành chuỗi trước tiên. Điều này thậm chí bao gồm cả số, thường được sử dụng làm index của mảng, vì vậy hãy cẩn thận không nhầm lẫn việc sử dụng số giữa object và mảng.

```js
var myObject = { };

myObject[true] = "foo";
myObject[3] = "bar";
myObject[myObject] = "baz";

myObject["true"];				// "foo"
myObject["3"];					// "bar"
myObject["[object Object]"];	// "baz"
```

### Computed Property Names

Cú pháp truy cập thuộc tính `myObject[..]` mà chúng ta vừa mô tả hữu ích nếu bạn cần sử dụng giá trị của một biểu thức được tính toán *làm* tên key, như `myObject[prefix + name]`. Nhưng nó không thực sự hữu ích khi khai báo các object bằng cú pháp object-literal.

ES6 thêm *tên thuộc tính được tính toán*, cho phép bạn chỉ định một biểu thức, được bao quanh bởi cặp `[ ]`, trong vị trí key-name của một khai báo object-literal:

```js
var prefix = "foo";

var myObject = {
  [prefix + "bar"]: "hello",
  [prefix + "baz"]: "world"
};

myObject["foobar"]; // hello
myObject["foobaz"]; // world
```

Cách sử dụng phổ biến nhất của *tên thuộc tính được tính toán* có lẽ là dành cho `Symbol` của ES6, mà chúng ta sẽ không đề cập chi tiết trong cuốn sách này. Nói tóm lại, chúng là một kiểu dữ liệu nguyên thủy mới có giá trị không thể đoán trước (về mặt kỹ thuật là một giá trị `string`). Bạn sẽ được khuyến khích mạnh mẽ không nên làm việc với *giá trị thực tế* của một `Symbol` (về mặt lý thuyết có thể khác nhau giữa các engine JS khác nhau), vì vậy tên của `Symbol`, như `Symbol.Something` (chỉ là một tên giả!), sẽ là thứ bạn sử dụng:

```js
var myObject = {
	[Symbol.Something]: "hello world"
};
```

### Property vs. Method

Một số developer thích phân biệt khi nói về truy cập thuộc tính trên một object, nếu giá trị được truy cập là một hàm. Bởi vì có vẻ như hàm "thuộc về" object, và trong các ngôn ngữ khác, các hàm thuộc về object (hay còn gọi là "class") được gọi là "method", nên việc nghe thấy "truy cập method" thay vì "truy cập thuộc tính" là điều không hiếm gặp.

**Thật thú vị, chính specification cũng phân biệt theo cách này.**

Về mặt kỹ thuật, các hàm không bao giờ "thuộc về" object, vì vậy nói rằng một hàm tình cờ được truy cập trên tham chiếu object tự động trở thành "method" có vẻ hơi quá đáng về mặt ngữ nghĩa.

Đúng là một số hàm có tham chiếu `this` bên trong chúng, và *đôi khi* các tham chiếu `this` này tham chiếu đến tham chiếu object tại vị trí gọi. Nhưng cách sử dụng này thực sự không làm cho hàm đó trở thành "method" hơn bất kỳ hàm nào khác, vì `this` được liên kết động tại thời gian chạy, tại vị trí gọi, và do đó mối quan hệ của nó với object là gián tiếp, tốt nhất.

Mỗi khi bạn truy cập một thuộc tính trên object, đó là **truy cập thuộc tính**, bất kể loại giá trị bạn nhận được. Nếu bạn *tình cờ* nhận được một hàm từ truy cập thuộc tính đó, thì nó không phải là một "method" một cách kỳ diệu. Không có gì đặc biệt (ngoài việc có thể liên kết `this` ngầm định như đã giải thích trước đó) về một hàm đến từ truy cập thuộc tính.

Ví dụ:

```js
function foo() {
  console.log( "foo" );
}

var someFoo = foo;  // tham chiếu biến đến `foo`

var myObject = {
  someFoo: foo
};

foo;        // hàm foo(){..}

someFoo;      // hàm foo(){..}

myObject.someFoo; // hàm foo(){..}
```

`someFoo` và `myObject.someFoo` chỉ là hai tham chiếu riêng biệt đến cùng một hàm, và không có tham chiếu nào ngầm định rằng hàm đó là đặc biệt hoặc "thuộc sở hữu" của bất kỳ object nào khác. Nếu `foo()` ở trên được định nghĩa để có tham chiếu `this` bên trong nó, thì sự *liên kết ngầm định* `myObject.someFoo` sẽ là sự khác biệt **duy nhất** có thể quan sát được giữa hai tham chiếu. Không có tham chiếu nào thực sự có nghĩa là được gọi là "method".

**Có lẽ người ta có thể cho rằng** một hàm *trở thành method*, không phải tại thời điểm định nghĩa, mà là trong thời gian chạy chỉ cho lần gọi đó, tùy thuộc vào cách nó được gọi tại vị trí gọi của nó (có ngữ cảnh tham chiếu object hay không - xem Chương 2 để biết thêm chi tiết). Ngay cả cách diễn giải này cũng hơi quá đáng.

Kết luận an toàn nhất có lẽ là "function" và "method" có thể thay thế cho nhau trong JavaScript.

**Lưu ý:** ES6 thêm một tham chiếu `super`, thường sẽ được sử dụng với `class` (xem Phụ lục A). Cách `super` hoạt động (liên kết tĩnh thay vì liên kết trễ như `this`) càng củng cố thêm ý tưởng rằng một hàm được liên kết `super` ở đâu đó là "method" hơn "function". Nhưng một lần nữa, đây chỉ là những sắc thái tinh tế về ngữ nghĩa (và cơ học).

Ngay cả khi bạn khai báo một biểu thức hàm là một phần của cú pháp object-literal, hàm đó cũng không tự nhiên *thuộc về* object hơn - vẫn chỉ là nhiều tham chiếu đến cùng một object hàm:

```js
var myObject = {
  foo: function foo() {
    console.log( "foo" );
  }
};

var someFoo = myObject.foo;

someFoo;    // hàm foo(){..}

myObject.foo; // hàm foo(){..}
```

**Lưu ý:** Trong Chương 6, chúng tôi sẽ đề cập đến một cách viết tắt ES6 cho cú pháp khai báo `foo: function foo(){ .. }` trong object-literal của chúng tôi.

### Arrays

Mảng cũng sử dụng cú pháp truy cập `[ ]`, nhưng như đã đề cập ở trên, chúng có tổ chức cấu trúc hơi chặt chẽ hơn về cách và nơi lưu trữ giá trị (mặc dù vẫn không có giới hạn về *kiểu* giá trị được lưu trữ). Mảng sử dụng *lập chỉ số theo số*, nghĩa là các giá trị được lưu trữ tại các vị trí, thường được gọi là *chỉ số*, ở các số nguyên không âm, chẳng hạn như `0` và `42`.

```js
var myArray = [ "foo", 42, "bar" ];

myArray.length;   // 3

myArray[0];     // "foo"

myArray[2];     // "bar"
```

Mảng *là* các object, vì vậy mặc dù mỗi chỉ số là một số nguyên dương, bạn *cũng có thể* thêm các thuộc tính vào mảng:

```js
var myArray = [ "foo", 42, "bar" ];

myArray.baz = "baz";

myArray.length; // 3

myArray.baz;  // "baz"
```

Lưu ý rằng việc thêm các thuộc tính được đặt tên (bất kể cú pháp toán tử `.` hoặc `[ ]`) không thay đổi `length` được báo cáo của mảng.

Bạn *có thể* sử dụng mảng như một object key/value đơn giản, và không bao giờ thêm bất kỳ chỉ số số nào, nhưng đây là một ý tưởng tồi vì mảng có hành vi và tối ưu hóa cụ thể cho mục đích sử dụng của chúng, và tương tự với các object thông thường. Sử dụng object để lưu trữ các cặp key/value và mảng để lưu trữ các giá trị tại các chỉ số số.

**Hãy cẩn thận:** Nếu bạn cố gắng thêm một thuộc tính vào một mảng, nhưng tên thuộc tính *trông giống* như một số, cuối cùng nó sẽ trở thành một chỉ số số (do đó sửa đổi nội dung của mảng):

```js
var myArray = [ "foo", 42, "bar" ];

myArray["3"] = "baz";

myArray.length;	// 4

myArray[3];		// "baz"
```

### Duplicating Objects

Một trong những tính năng được yêu cầu phổ biến nhất khi developer mới bắt đầu học ngôn ngữ JavaScript là cách sao chép một object. Có vẻ như nên có một phương thức `copy()` tích hợp sẵn, phải không? Hóa ra nó phức tạp hơn một chút so với suy nghĩ đó, vì không hoàn toàn rõ ràng rằng theo mặc định, thuật toán nào nên được sử dụng cho việc sao chép.

Ví dụ, hãy xem xét object này:

```js
function anotherFunction() { /*..*/ }

var anotherObject = {
  c: true
};

var anotherArray = [];

var myObject = {
  a: 2,
  b: anotherObject, // tham chiếu, không phải bản sao!
  c: anotherArray,  // một tham chiếu khác!
  d: anotherFunction
};

anotherArray.push( anotherObject, myObject );
```

Bản sao chính xác của `myObject` nên được biểu diễn như thế nào?

Đầu tiên, chúng ta cần trả lời xem đó là bản sao * nông * hay * sâu *? Một bản sao * nông * sẽ kết thúc với `a` trên object mới là bản sao của giá trị `2`, nhưng các thuộc tính `b`, `c` và `d` chỉ là các tham chiếu đến cùng một vị trí như các tham chiếu trong object gốc. Một bản sao * sâu * sẽ không chỉ sao chép `myObject`, mà còn cả `anotherObject` và `anotherArray`. Nhưng sau đó chúng ta gặp vấn đề là `anotherArray` có tham chiếu đến `anotherObject` và `myObject` trong đó, vì vậy *chúng* cũng nên được sao chép thay vì giữ nguyên tham chiếu. Bây giờ chúng ta có một vấn đề sao chép vòng lặp vô hạn vì tham chiếu vòng tròn.

Chúng ta có nên phát hiện tham chiếu vòng tròn và chỉ dừng việc duyệt vòng tròn (không sao chép hoàn toàn phần tử sâu)? Chúng ta có nên báo lỗi hoàn toàn không? Hay một cách nào đó ở giữa?

Hơn nữa, việc "sao chép" một hàm thực sự có nghĩa gì không? Có một số thủ thuật như lấy ra chuỗi hóa `toString()` của mã nguồn của hàm (mà thay đổi tùy theo implementation và thậm chí không đáng tin cậy trong tất cả các engine tùy thuộc vào loại hàm được kiểm tra).

Vậy làm thế nào để chúng ta giải quyết tất cả những câu hỏi棘手 này? Các framework JS khác nhau đã chọn cách giải thích riêng của họ và đưa ra quyết định riêng. Nhưng trong số đó (nếu có) thì JS nên áp dụng cái nào làm *tiêu chuẩn*? Trong một thời gian dài, không có câu trả lời rõ ràng.

Một giải pháp phụ là các object an toàn với JSON (có thể được tuần tự hóa thành chuỗi JSON và sau đó được phân tích lại thành một object có cùng cấu trúc và giá trị) có thể dễ dàng được *sao chép* với:

```js
var newObj = JSON.parse( JSON.stringify( someObj ) );
```

Tất nhiên, điều đó đòi hỏi bạn phải đảm bảo object của bạn an toàn với JSON. Đối với một số tình huống, điều đó rất đơn giản. Đối với những người khác, nó không đủ.

Đồng thời, bản sao nông khá dễ hiểu và ít vấn đề hơn, vì vậy ES6 hiện đã định nghĩa `Object.assign(..)` cho nhiệm vụ này. `Object.assign(..)` lấy một object *mục tiêu* làm tham số đầu tiên và một hoặc nhiều object *nguồn* làm các tham số tiếp theo. Nó lặp qua tất cả các *có thể đếm được* (xem bên dưới), *các khóa sở hữu* (có ngay lập tức) trên object *nguồn* và sao chép chúng (chỉ thông qua gán `=`) sang *mục tiêu*. Nó cũng hữu ích, trả về *mục tiêu*, như bạn có thể thấy bên dưới:

```js
var newObj = Object.assign( {}, myObject );

newObj.a;           // 2
newObj.b === anotherObject;   // true
newObj.c === anotherArray;    // true
newObj.d === anotherFunction; // true
```

**Lưu ý:** Trong phần tiếp theo, chúng tôi mô tả "mô tả thuộc tính" (đặc điểm thuộc tính) và cho thấy việc sử dụng `Object.defineProperty(..)`. Tuy nhiên, việc sao chép xảy ra cho `Object.assign(..)` chỉ là gán theo kiểu `=`, vì vậy bất kỳ đặc điểm đặc biệt nào của một thuộc tính (như `writable`) trên object nguồn **không được bảo toàn** trên object mục tiêu.

### Property Descriptors

Trước ES5, ngôn ngữ JavaScript không cung cấp cách trực tiếp để code của bạn kiểm tra hoặc phân biệt các đặc điểm của thuộc tính, chẳng hạn như thuộc tính đó có chỉ đọc hay không.

Nhưng kể từ ES5, tất cả các thuộc tính đều được mô tả dưới dạng **mô tả thuộc tính**.

Xem xét đoạn code này:

```js
var myObject = {
  a: 2
};

Object.getOwnPropertyDescriptor( myObject, "a" );
// {
//    value: 2,
//    writable: true,
//    enumerable: true,
//    configurable: true
// }
```

Như bạn có thể thấy, mô tả thuộc tính (được gọi là "mô tả dữ liệu" vì nó chỉ dành cho việc giữ giá trị dữ liệu) cho thuộc tính object thông thường `a` của chúng ta còn nhiều hơn chỉ là giá trị `value` của nó là `2`. Nó bao gồm 3 đặc điểm khác: `writable`, `enumerable` và `configurable`.

Mặc dù chúng ta có thể thấy các giá trị mặc định cho các đặc điểm của mô tả thuộc tính là gì khi chúng ta tạo một thuộc tính thông thường, nhưng chúng ta có thể sử dụng `Object.defineProperty(..)` để thêm một thuộc tính mới hoặc sửa đổi một thuộc tính hiện có (nếu nó có thể cấu hình được!), với các đặc điểm mong muốn.

Ví dụ:

```js
var myObject = {};

Object.defineProperty( myObject, "a", {
  value: 2,
  writable: true,
  configurable: true,
  enumerable: true
} );

myObject.a; // 2
```

Sử dụng `defineProperty(..)`, chúng tôi đã thêm thuộc tính `a` thông thường, đơn giản vào `myObject` theo cách thủ công rõ ràng. Tuy nhiên, bạn thường sẽ không sử dụng cách tiếp cận thủ công này trừ khi bạn muốn sửa đổi một trong các đặc điểm của mô tả so với hành vi thông thường của nó.

#### Writable

Khả năng thay đổi giá trị của một thuộc tính được kiểm soát bởi đặc điểm `writable`.

Ví dụ:

```js
var myObject = {};

Object.defineProperty( myObject, "a", {
  value: 2,
  writable: false, // không thể ghi!
  configurable: true,
  enumerable: true
} );

myObject.a = 3;

myObject.a; // 2
```

Như bạn có thể thấy, việc sửa đổi `value` của chúng ta đã thất bại một cách âm thầm. Nếu chúng ta thử trong chế độ `strict mode`, chúng ta sẽ gặp lỗi:

```js
"use strict";

var myObject = {};

Object.defineProperty( myObject, "a", {
  value: 2,
  writable: false, // không thể ghi!
  configurable: true,
  enumerable: true
} );

myObject.a = 3; // TypeError
```

Lỗi `TypeError` cho chúng ta biết rằng chúng ta không thể thay đổi một thuộc tính không thể ghi.

**Lưu ý:** Chúng ta sẽ thảo luận về getter/setter sớm, nhưng nói ngắn gọn, bạn có thể quan sát thấy rằng `writable:false` có nghĩa là giá trị không thể thay đổi, điều này tương đương phần nào với việc bạn định nghĩa một setter không hoạt động (no-op setter). Trên thực tế, setter không hoạt động của bạn sẽ cần phải throw một lỗi `TypeError` khi được gọi, để thực sự phù hợp với `writable:false`.

#### Configurable

As long as a property is currently configurable, we can modify its descriptor definition, using the same `defineProperty(..)` utility.

```js
var myObject = {
	a: 2
};

myObject.a = 3;
myObject.a;					// 3

Object.defineProperty( myObject, "a", {
	value: 4,
	writable: true,
	configurable: false,	// not configurable!
	enumerable: true
} );

myObject.a;					// 4
myObject.a = 5;
myObject.a;					// 5

Object.defineProperty( myObject, "a", {
	value: 6,
	writable: true,
	configurable: true,
	enumerable: true
} ); // TypeError
```

The final `defineProperty(..)` call results in a TypeError, regardless of `strict mode`, if you attempt to change the descriptor definition of a non-configurable property. Be careful: as you can see, changing `configurable` to `false` is a **one-way action, and cannot be undone!**

**Note:** There's a nuanced exception to be aware of: even if the property is already `configurable:false`, `writable` can always be changed from `true` to `false` without error, but not back to `true` if already `false`.

Another thing `configurable:false` prevents is the ability to use the `delete` operator to remove an existing property.

```js
var myObject = {
	a: 2
};

myObject.a;				// 2
delete myObject.a;
myObject.a;				// undefined

Object.defineProperty( myObject, "a", {
	value: 2,
	writable: true,
	configurable: false,
	enumerable: true
} );

myObject.a;				// 2
delete myObject.a;
myObject.a;				// 2
```

As you can see, the last `delete` call failed (silently) because we made the `a` property non-configurable.

`delete` is only used to remove object properties (which can be removed) directly from the object in question. If an object property is the last remaining *reference* to some object/function, and you `delete` it, that removes the reference and now that unreferenced object/function can be garbage collected. But, it is **not** proper to think of `delete` as a tool to free up allocated memory as it does in other languages (like C/C++). `delete` is just an object property removal operation -- nothing more.

#### Enumerable

Đặc điểm cuối cùng của mô tả thuộc tính mà chúng ta sẽ đề cập ở đây (còn hai đặc điểm khác, chúng ta sẽ đề cập đến chúng khi thảo luận về getter/setter) là `enumerable`.

Tên gọi có lẽ đã cho bạn biết điều hiển nhiên, đặc điểm này kiểm soát xem một thuộc tính có xuất hiện trong các liệt kê thuộc tính object nhất định hay không, chẳng hạn như vòng lặp `for..in`. Đặt thành `false` để ngăn nó xuất hiện trong các liệt kê như vậy, mặc dù nó vẫn hoàn toàn có thể truy cập được. Đặt thành `true` để giữ nó hiển thị.

Tất cả các thuộc tính do người dùng định nghĩa thông thường đều được mặc định thành `enumerable`, vì đây thường là điều bạn muốn. Nhưng nếu bạn có một thuộc tính đặc biệt muốn ẩn khỏi liệt kê, hãy đặt nó thành `enumerable:false`.

Chúng tôi sẽ trình bày chi tiết hơn về khả năng liệt kê trong thời gian ngắn nữa, vì vậy hãy ghi nhớ chủ đề này.

### Immutability

Đôi khi bạn muốn tạo các thuộc tính hoặc object không thể thay đổi (vô tình hay cố ý). ES5 bổ sung hỗ trợ xử lý điều đó theo nhiều cách tinh tế khác nhau.

Điều quan trọng cần lưu ý là **tất cả** các cách tiếp cận này đều tạo ra tính bất biến nông (shallow immutability). Nghĩa là, chúng chỉ ảnh hưởng đến object và các đặc điểm trực tiếp của thuộc tính của nó. Nếu một object có tham chiếu đến một object khác (mảng, object, function, v.v.), thì *nội dung* của object đó không bị ảnh hưởng và vẫn có thể thay đổi.

```js
myImmutableObject.foo; // [1,2,3]
myImmutableObject.foo.push( 4 );
myImmutableObject.foo; // [1,2,3,4]
```

Trong đoạn mã này, chúng ta giả định rằng `myImmutableObject` đã được tạo và được bảo vệ là không thể thay đổi. Nhưng, để bảo vệ nội dung của `myImmutableObject.foo` (là một object riêng của nó - mảng), bạn cũng cần phải làm cho `foo` không thể thay đổi, bằng cách sử dụng một hoặc nhiều chức năng sau.

**Lưu ý:** Việc tạo ra các object không thể thay đổi sâu trong các chương trình JS không phải là quá phổ biến. Các trường hợp đặc biệt chắc chắn có thể yêu cầu nó, nhưng như một mô hình thiết kế chung, nếu bạn muốn *niêm phong* hoặc *đóng băng* tất cả các object của mình, bạn có thể muốn lùi lại một bước và xem xét lại thiết kế chương trình của mình để mạnh mẽ hơn trước những thay đổi tiềm ẩn trong giá trị của object.

#### Object Constant

By combining `writable:false` and `configurable:false`, you can essentially create a *constant* (cannot be changed, redefined or deleted) as an object property, like:

```js
var myObject = {};

Object.defineProperty( myObject, "FAVORITE_NUMBER", {
	value: 42,
	writable: false,
	configurable: false
} );
```

#### Prevent Extensions

If you want to prevent an object from having new properties added to it, but otherwise leave the rest of the object's properties alone, call `Object.preventExtensions(..)`:

```js
var myObject = {
	a: 2
};

Object.preventExtensions( myObject );

myObject.b = 3;
myObject.b; // undefined
```

In `non-strict mode`, the creation of `b` fails silently. In `strict mode`, it throws a `TypeError`.

#### Seal

`Object.seal(..)` tạo ra một object "được niêm phong", có nghĩa là nó lấy một object hiện có và về cơ bản gọi `Object.preventExtensions(..)` trên object đó, nhưng đồng thời cũng đánh dấu tất cả các thuộc tính hiện có của nó là `configurable:false`.

Vì vậy, bạn không chỉ không thể thêm bất kỳ thuộc tính nào nữa, mà bạn cũng không thể cấu hình lại hoặc xóa bất kỳ thuộc tính hiện có nào (mặc dù bạn *vẫn có thể* sửa đổi giá trị của chúng).

#### Freeze

`Object.freeze(..)` tạo ra một object "đông cứng", có nghĩa là nó lấy một object hiện có và về cơ bản gọi `Object.seal(..)` trên object đó, nhưng nó cũng đánh dấu tất cả các thuộc tính "truy cập dữ liệu" (data accessor) là `writable:false`, do đó giá trị của chúng không thể thay đổi.

Cách tiếp cận này là mức độ bất biến cao nhất mà bạn có thể đạt được cho chính object, vì nó ngăn chặn mọi thay đổi đối với object hoặc bất kỳ thuộc tính trực tiếp nào của nó (mặc dù, như đã đề cập ở trên, nội dung của bất kỳ object khác được tham chiếu đến vẫn không bị ảnh hưởng).

Bạn có thể "đông cứng sâu" một object bằng cách gọi `Object.freeze(..)` trên object đó, sau đó lặp lại đệ quy qua tất cả các object mà nó tham chiếu (chưa bị ảnh hưởng cho đến nay) và gọi `Object.freeze(..)` trên chúng nữa. Tuy nhiên, hãy cẩn thận, vì điều đó có thể ảnh hưởng đến các object khác (được chia sẻ) mà bạn không định ảnh hưởng đến.


### `[[Get]]`

There's a subtle, but important, detail about how property accesses are performed.

Consider:

```js
var myObject = {
	a: 2
};

myObject.a; // 2
```

The `myObject.a` is a property access, but it doesn't *just* look in `myObject` for a property of the name `a`, as it might seem.

According to the spec, the code above actually performs a `[[Get]]` operation (kinda like a function call: `[[Get]]()`) on the `myObject`. The default built-in `[[Get]]` operation for an object *first* inspects the object for a property of the requested name, and if it finds it, it will return the value accordingly.

However, the `[[Get]]` algorithm defines other important behavior if it does *not* find a property of the requested name. We will examine in Chapter 5 what happens *next* (traversal of the `[[Prototype]]` chain, if any).

But one important result of this `[[Get]]` operation is that if it cannot through any means come up with a value for the requested property, it instead returns the value `undefined`.

```js
var myObject = {
	a: 2
};

myObject.b; // undefined
```

This behavior is different from when you reference *variables* by their identifier names. If you reference a variable that cannot be resolved within the applicable lexical scope look-up, the result is not `undefined` as it is for object properties, but instead a `ReferenceError` is thrown.

```js
var myObject = {
	a: undefined
};

myObject.a; // undefined

myObject.b; // undefined
```

From a *value* perspective, there is no difference between these two references -- they both result in `undefined`. However, the `[[Get]]` operation underneath, though subtle at a glance, potentially performed a bit more "work" for the reference `myObject.b` than for the reference `myObject.a`.

Inspecting only the value results, you cannot distinguish whether a property exists and holds the explicit value `undefined`, or whether the property does *not* exist and `undefined` was the default return value after `[[Get]]` failed to return something explicitly. However, we will see shortly how you *can* distinguish these two scenarios.

### `[[Put]]`

Since there's an internally defined `[[Get]]` operation for getting a value from a property, it should be obvious there's also a default `[[Put]]` operation.

It may be tempting to think that an assignment to a property on an object would just invoke `[[Put]]` to set or create that property on the object in question. But the situation is more nuanced than that.

When invoking `[[Put]]`, how it behaves differs based on a number of factors, including (most impactfully) whether the property is already present on the object or not.

If the property is present, the `[[Put]]` algorithm will roughly check:

1. Is the property an accessor descriptor (see "Getters & Setters" section below)? **If so, call the setter, if any.**
2. Is the property a data descriptor with `writable` of `false`? **If so, silently fail in `non-strict mode`, or throw `TypeError` in `strict mode`.**
3. Otherwise, set the value to the existing property as normal.

If the property is not yet present on the object in question, the `[[Put]]` operation is even more nuanced and complex. We will revisit this scenario in Chapter 5 when we discuss `[[Prototype]]` to give it more clarity.

### Getters & Setters

The default `[[Put]]` and `[[Get]]` operations for objects completely control how values are set to existing or new properties, or retrieved from existing properties, respectively.

**Note:** Using future/advanced capabilities of the language, it may be possible to override the default `[[Get]]` or `[[Put]]` operations for an entire object (not just per property). This is beyond the scope of our discussion in this book, but will be covered later in the "You Don't Know JS" series.

ES5 introduced a way to override part of these default operations, not on an object level but a per-property level, through the use of getters and setters. Getters are properties which actually call a hidden function to retrieve a value. Setters are properties which actually call a hidden function to set a value.

When you define a property to have either a getter or a setter or both, its definition becomes an "accessor descriptor" (as opposed to a "data descriptor"). For accessor-descriptors, the `value` and `writable` characteristics of the descriptor are moot and ignored, and instead JS considers the `set` and `get` characteristics of the property (as well as `configurable` and `enumerable`).

Consider:

```js
var myObject = {
	// define a getter for `a`
	get a() {
		return 2;
	}
};

Object.defineProperty(
	myObject,	// target
	"b",		// property name
	{			// descriptor
		// define a getter for `b`
		get: function(){ return this.a * 2 },

		// make sure `b` shows up as an object property
		enumerable: true
	}
);

myObject.a; // 2

myObject.b; // 4
```

Either through object-literal syntax with `get a() { .. }` or through explicit definition with `defineProperty(..)`, in both cases we created a property on the object that actually doesn't hold a value, but whose access automatically results in a hidden function call to the getter function, with whatever value it returns being the result of the property access.

```js
var myObject = {
	// define a getter for `a`
	get a() {
		return 2;
	}
};

myObject.a = 3;

myObject.a; // 2
```

Since we only defined a getter for `a`, if we try to set the value of `a` later, the set operation won't throw an error but will just silently throw the assignment away. Even if there was a valid setter, our custom getter is hard-coded to return only `2`, so the set operation would be moot.

To make this scenario more sensible, properties should also be defined with setters, which override the default `[[Put]]` operation (aka, assignment), per-property, just as you'd expect. You will almost certainly want to always declare both getter and setter (having only one or the other often leads to unexpected/surprising behavior):

```js
var myObject = {
	// define a getter for `a`
	get a() {
		return this._a_;
	},

	// define a setter for `a`
	set a(val) {
		this._a_ = val * 2;
	}
};

myObject.a = 2;

myObject.a; // 4
```

**Note:** In this example, we actually store the specified value `2` of the assignment (`[[Put]]` operation) into another variable `_a_`. The `_a_` name is purely by convention for this example and implies nothing special about its behavior -- it's a normal property like any other.

### Existence

We showed earlier that a property access like `myObject.a` may result in an `undefined` value if either the explicit `undefined` is stored there or the `a` property doesn't exist at all. So, if the value is the same in both cases, how else do we distinguish them?

We can ask an object if it has a certain property *without* asking to get that property's value:

```js
var myObject = {
	a: 2
};

("a" in myObject);				// true
("b" in myObject);				// false

myObject.hasOwnProperty( "a" );	// true
myObject.hasOwnProperty( "b" );	// false
```

The `in` operator will check to see if the property is *in* the object, or if it exists at any higher level of the `[[Prototype]]` chain object traversal (see Chapter 5). By contrast, `hasOwnProperty(..)` checks to see if *only* `myObject` has the property or not, and will *not* consult the `[[Prototype]]` chain. We'll come back to the important differences between these two operations in Chapter 5 when we explore `[[Prototype]]`s in detail.

`hasOwnProperty(..)` is accessible for all normal objects via delegation to `Object.prototype` (see Chapter 5). But it's possible to create an object that does not link to `Object.prototype` (via `Object.create(null)` -- see Chapter 5). In this case, a method call like `myObject.hasOwnProperty(..)` would fail.

In that scenario, a more robust way of performing such a check is `Object.prototype.hasOwnProperty.call(myObject,"a")`, which borrows the base `hasOwnProperty(..)` method and uses *explicit `this` binding* (see Chapter 2) to apply it against our `myObject`.

**Note:** The `in` operator has the appearance that it will check for the existence of a *value* inside a container, but it actually checks for the existence of a property name. This difference is important to note with respect to arrays, as the temptation to try a check like `4 in [2, 4, 6]` is strong, but this will not behave as expected.

#### Enumeration

Previously, we explained briefly the idea of "enumerability" when we looked at the `enumerable` property descriptor characteristic. Let's revisit that and examine it in more close detail.

```js
var myObject = { };

Object.defineProperty(
	myObject,
	"a",
	// make `a` enumerable, as normal
	{ enumerable: true, value: 2 }
);

Object.defineProperty(
	myObject,
	"b",
	// make `b` NON-enumerable
	{ enumerable: false, value: 3 }
);

myObject.b; // 3
("b" in myObject); // true
myObject.hasOwnProperty( "b" ); // true

// .......

for (var k in myObject) {
	console.log( k, myObject[k] );
}
// "a" 2
```

You'll notice that `myObject.b` in fact **exists** and has an accessible value, but it doesn't show up in a `for..in` loop (though, surprisingly, it **is** revealed by the `in` operator existence check). That's because "enumerable" basically means "will be included if the object's properties are iterated through".

**Note:** `for..in` loops applied to arrays can give somewhat unexpected results, in that the enumeration of an array will include not only all the numeric indices, but also any enumerable properties. It's a good idea to use `for..in` loops *only* on objects, and traditional `for` loops with numeric index iteration for the values stored in arrays.

Another way that enumerable and non-enumerable properties can be distinguished:

```js
var myObject = { };

Object.defineProperty(
	myObject,
	"a",
	// make `a` enumerable, as normal
	{ enumerable: true, value: 2 }
);

Object.defineProperty(
	myObject,
	"b",
	// make `b` non-enumerable
	{ enumerable: false, value: 3 }
);

myObject.propertyIsEnumerable( "a" ); // true
myObject.propertyIsEnumerable( "b" ); // false

Object.keys( myObject ); // ["a"]
Object.getOwnPropertyNames( myObject ); // ["a", "b"]
```

`propertyIsEnumerable(..)` tests whether the given property name exists *directly* on the object and is also `enumerable:true`.

`Object.keys(..)` returns an array of all enumerable properties, whereas `Object.getOwnPropertyNames(..)` returns an array of *all* properties, enumerable or not.

Whereas `in` vs. `hasOwnProperty(..)` differ in whether they consult the `[[Prototype]]` chain or not, `Object.keys(..)` and `Object.getOwnPropertyNames(..)` both inspect *only* the direct object specified.

There's (currently) no built-in way to get a list of **all properties** which is equivalent to what the `in` operator test would consult (traversing all properties on the entire `[[Prototype]]` chain, as explained in Chapter 5). You could approximate such a utility by recursively traversing the `[[Prototype]]` chain of an object, and for each level, capturing the list from `Object.keys(..)` -- only enumerable properties.

## Iteration

The `for..in` loop iterates over the list of enumerable properties on an object (including its `[[Prototype]]` chain). But what if you instead want to iterate over the values?

With numerically-indexed arrays, iterating over the values is typically done with a standard `for` loop, like:

```js
var myArray = [1, 2, 3];

for (var i = 0; i < myArray.length; i++) {
	console.log( myArray[i] );
}
// 1 2 3
```

This isn't iterating over the values, though, but iterating over the indices, where you then use the index to reference the value, as `myArray[i]`.

ES5 also added several iteration helpers for arrays, including `forEach(..)`, `every(..)`, and `some(..)`. Each of these helpers accepts a function callback to apply to each element in the array, differing only in how they respectively respond to a return value from the callback.

`forEach(..)` will iterate over all values in the array, and ignores any callback return values. `every(..)` keeps going until the end *or* the callback returns a `false` (or "falsy") value, whereas `some(..)` keeps going until the end *or* the callback returns a `true` (or "truthy") value.

These special return values inside `every(..)` and `some(..)` act somewhat like a `break` statement inside a normal `for` loop, in that they stop the iteration early before it reaches the end.

If you iterate on an object with a `for..in` loop, you're also only getting at the values indirectly, because it's actually iterating only over the enumerable properties of the object, leaving you to access the properties manually to get the values.

**Note:** As contrasted with iterating over an array's indices in a numerically ordered way (`for` loop or other iterators), the order of iteration over an object's properties is **not guaranteed** and may vary between different JS engines. **Do not rely** on any observed ordering for anything that requires consistency among environments, as any observed agreement is unreliable.

But what if you want to iterate over the values directly instead of the array indices (or object properties)? Helpfully, ES6 adds a `for..of` loop syntax for iterating over arrays (and objects, if the object defines its own custom iterator):

```js
var myArray = [ 1, 2, 3 ];

for (var v of myArray) {
	console.log( v );
}
// 1
// 2
// 3
```

The `for..of` loop asks for an iterator object (from a default internal function known as `@@iterator` in spec-speak) of the *thing* to be iterated, and the loop then iterates over the successive return values from calling that iterator object's `next()` method, once for each loop iteration.

Arrays have a built-in `@@iterator`, so `for..of` works easily on them, as shown. But let's manually iterate the array, using the built-in `@@iterator`, to see how it works:

```js
var myArray = [ 1, 2, 3 ];
var it = myArray[Symbol.iterator]();

it.next(); // { value:1, done:false }
it.next(); // { value:2, done:false }
it.next(); // { value:3, done:false }
it.next(); // { done:true }
```

**Note:** We get at the `@@iterator` *internal property* of an object using an ES6 `Symbol`: `Symbol.iterator`. We briefly mentioned `Symbol` semantics earlier in the chapter (see "Computed Property Names"), so the same reasoning applies here. You'll always want to reference such special properties by `Symbol` name reference instead of by the special value it may hold. Also, despite the name's implications, `@@iterator` is **not the iterator object** itself, but a **function that returns** the iterator object -- a subtle but important detail!

As the above snippet reveals, the return value from an iterator's `next()` call is an object of the form `{ value: .. , done: .. }`, where `value` is the current iteration value, and `done` is a `boolean` that indicates if there's more to iterate.

Notice the value `3` was returned with a `done:false`, which seems strange at first glance. You have to call the `next()` a fourth time (which the `for..of` loop in the previous snippet automatically does) to get `done:true` and know you're truly done iterating. The reason for this quirk is beyond the scope of what we'll discuss here, but it comes from the semantics of ES6 generator functions.

While arrays do automatically iterate in `for..of` loops, regular objects **do not have a built-in `@@iterator`**. The reasons for this intentional omission are more complex than we will examine here, but in general it was better to not include some implementation that could prove troublesome for future types of objects.

It *is* possible to define your own default `@@iterator` for any object that you care to iterate over. For example:

```js
var myObject = {
	a: 2,
	b: 3
};

Object.defineProperty( myObject, Symbol.iterator, {
	enumerable: false,
	writable: false,
	configurable: true,
	value: function() {
		var o = this;
		var idx = 0;
		var ks = Object.keys( o );
		return {
			next: function() {
				return {
					value: o[ks[idx++]],
					done: (idx > ks.length)
				};
			}
		};
	}
} );

// iterate `myObject` manually
var it = myObject[Symbol.iterator]();
it.next(); // { value:2, done:false }
it.next(); // { value:3, done:false }
it.next(); // { value:undefined, done:true }

// iterate `myObject` with `for..of`
for (var v of myObject) {
	console.log( v );
}
// 2
// 3
```

**Note:** We used `Object.defineProperty(..)` to define our custom `@@iterator` (mostly so we could make it non-enumerable), but using the `Symbol` as a *computed property name* (covered earlier in this chapter), we could have declared it directly, like `var myObject = { a:2, b:3, [Symbol.iterator]: function(){ /* .. */ } }`.

Each time the `for..of` loop calls `next()` on `myObject`'s iterator object, the internal pointer will advance and return back the next value from the object's properties list (see a previous note about iteration ordering on object properties/values).

The iteration we just demonstrated is a simple value-by-value iteration, but you can of course define arbitrarily complex iterations for your custom data structures, as you see fit. Custom iterators combined with ES6's `for..of` loop are a powerful new syntactic tool for manipulating user-defined objects.

For example, a list of `Pixel` objects (with `x` and `y` coordinate values) could decide to order its iteration based on the linear distance from the `(0,0)` origin, or filter out points that are "too far away", etc. As long as your iterator returns the expected `{ value: .. }` return values from `next()` calls, and a `{ done: true }` after the iteration is complete, ES6's `for..of` can iterate over it.

In fact, you can even generate "infinite" iterators which never "finish" and always return a new value (such as a random number, an incremented value, a unique identifier, etc), though you probably will not use such iterators with an unbounded `for..of` loop, as it would never end and would hang your program.

```js
var randoms = {
	[Symbol.iterator]: function() {
		return {
			next: function() {
				return { value: Math.random() };
			}
		};
	}
};

var randoms_pool = [];
for (var n of randoms) {
	randoms_pool.push( n );

	// don't proceed unbounded!
	if (randoms_pool.length === 100) break;
}
```

This iterator will generate random numbers "forever", so we're careful to only pull out 100 values so our program doesn't hang.

## Review (TL;DR)

Object trong JS có cả dạng literal (như `var a = { .. }`) và dạng constructed (như `var a = new Array(..)`). Dạng literal gần như luôn được ưu tiên, nhưng dạng constructed trong một số trường hợp cung cấp nhiều tùy chọn tạo object hơn.

Nhiều người lầm tưởng "mọi thứ trong JavaScript đều là object", nhưng điều này không chính xác. Object là một trong 6 (hoặc 7, tùy theo quan điểm của bạn) kiểu dữ liệu nguyên thủy. Object có các kiểu con, bao gồm `function`, và cũng có thể được chuyên biệt về hành vi, như `[object Array]` là nhãn nội bộ đại diện cho kiểu con object mảng.

Object là tập hợp các cặp key/value. Giá trị có thể được truy cập như các thuộc tính, thông qua cú pháp `.propName` hoặc `["propName"]`. Bất cứ khi nào truy cập một thuộc tính, engine thực sự gọi hoạt động mặc định nội bộ `[[Get]]` (và `[[Put]]` để đặt giá trị), không chỉ tìm kiếm trực tiếp thuộc tính trên object mà còn sẽ duyệt qua chuỗi `[[Prototype]]` (xem Chương 5) nếu không tìm thấy.

Thuộc tính có một số đặc điểm nhất định có thể được kiểm soát thông qua các mô tả thuộc tính, chẳng hạn như `writable` và `configurable`. Ngoài ra, tính bất biến (mutability) của object và các thuộc tính của chúng có thể được kiểm soát ở các mức độ bất biến khác nhau bằng cách sử dụng `Object.preventExtensions(..)`, `Object.seal(..)` và `Object.freeze(..)`.

Thuộc tính không nhất thiết phải chứa giá trị - chúng cũng có thể là "thuộc tính accessor" với getter/setter. Chúng cũng có thể là *enumerable* hoặc không, điều này điều khiển việc chúng có xuất hiện trong các vòng lặp `for..in` hay không.

Bạn cũng có thể lặp qua **các giá trị** trong các cấu trúc dữ liệu (mảng, object, v.v.) bằng cách sử dụng cú pháp ES6 `for..of`, tìm kiếm object `@@iterator` tích hợp hoặc tùy chỉnh bao gồm phương thức `next()` để di chuyển qua các giá trị dữ liệu từng cái một.
