# frontend-style-guide

トリコ株式会社の JavaScript / TypeScript におけるコード規約

## Table of Contents

1. [References](#references)
1. [Objects](#objects)
1. [Arrays](#arrays)
1. [Destructuring](#destructuring)
1. [Strings](#strings)
1. [Functions](#functions)
1. [Arrow Functions](#arrow-functions)
1. [Classes & Constructors](#classes--constructors)
1. [Modules](#modules)
1. [Comparison Operators & Equality](#comparison-operators--equality)
1. [Comments](#comments)
1. [Whitespace](#whitespace)
1. [Semicolons](#semicolons)
1. [Type Casting & Coercion](#type-casting--coercion)
1. [Naming Conventions](#naming-conventions)
1. [Standard Library](#standard-library)
1. [React](#react)

## References

- [1.1](#1-1) 全ての変数は `const` を使用し、 `var` は使用しません。

> 参照を再割り当てできないことで、バグに繋がりやすく理解しにくいコードになることを防ぎます。

```ts
// bad
var a = 1;
var b = 2;

// good
const a = 1;
const b = 2;
```

- [1.2](#1-2) 参照の再割り当てを行う場合、 `var` の代わりに `let` を使用しましょう。

```ts
// bad
var a = 1;
a = 2;

// good
let a = 1;
a = 2;
```

## Objects

- [2.1](#2-1) 動的にプロパティ名を持つオブジェクトを作成する場合もオブジェクトの中で定義を行いましょう。

```ts
function getKey(key: string): string {
  return `key-is-${key}`;
}

// bad
const obj = {
  id: 1,
};
obj[getKey('enabled')] = true;

// good
const obj = {
  id: 1,
  [getKey('enabled')]: true,
};
```

- [2.2](#2-2) プロパティの短縮構文を使用しましょう。

```ts
const key = 'this is key';

// bad
const obj = {
  key: key,
};

// good
const obj = {
  key,
};
```

- [2.3](#2-3) object の key として無効な識別子のみプロパティを文字列として定義しましょう。

```ts
// bad
const bad = {
  foo: 1,
  'data-foo': 2,
};

// good
const good = {
  foo: 1,
  'data-foo': 2,
};
```

- [2.4](#2-4) オブジェクトを shallow-copy する場合、 [`Object.assign`](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Object/assign) よりも[スプレッド構文](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Operators/Spread_syntax)を使用しましょう。特定のプロパティを省略した新しいオブジェクトを取得するには、オブジェクトの[残余引数](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Functions/rest_parameters)を使用しましょう。

```ts
// bad
const original = { a: 1, b: 2 };
const copy = Object.assign(original, { c: 3 });
delete copy.a; // copy から a が消えてしまう

// good
const original = { a: 1, b: 2 };
const copy = {...original, { c:3 }};
const {a, ...noA} = copy; // noA => {b: 2, c:3}
```

## Arrays

- [3.1](#3-1) 配列をコピーする場合は、[スプレッド構文](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Operators/Spread_syntax)を使用しましょう。

```ts
// bad
const len = items.length;
const itemsCopy = [];

for (let i = 0; i < len; i++) {
  itemsCopy[i] = items[i];
}

// good
const itemsCopy = [...items];
```

- [3.2](#3-2) [反復処理プロトコル](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Iteration_protocols) を配列に変換する場合はスプレッド構文の代わりに[`Array.from`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/from) を使用しましょう。
  > 元の変数が配列で単純に shallow copy をしたいのか、iterable を Array に変換したいのか明確にするためです。

```ts
const foo = document.querySelectorAll('.foo');

// good
const nodes = [...foo];

// best
const nodes = Array.from(foo);
```

- [3.3](#3-3) 反復処理プロトコル(iterables)へのマッピングにはスプレッド構文の代わりに `Array.from` を使用しましょう。
  > 理由は中間配列の作成を防ぐためです。

```ts
// bad
const baz = [...foo].map(bar);

// good
const baz = Array.from(foo, bar);
```

- [3.4](#3-4) 配列のコールバック関数の中では、 return 構文を使用しましょう。もし関数の中が副作用のない式を返す一行で構成されている場合は、return を省略してもかまいません。

```ts
// good
[1, 2, 3].map(x => {
  const y = x + 1;
  return x * y;
});

// good
[1, 2, 3].map(x => x + 1);
```

- [3.5](#3-5) 配列の返り値がオブジェクトになる場合、副作用のない形であれば return を省略しましょう。

```ts
// bad
[1, 2, 3].map(x => {
  return {
    x,
    y: x + 1,
  };
});

// good
[1, 2, 3].map(x => ({
  x,
  y: x + 1,
}));
```

## Destructuring

- [4.1](#4-1) 複数のプロパティからなるオブジェクトにアクセスする際は、オブジェクト構造化代入を使用しましょう。

```ts
// bad
function getFullName(user: User): string {
  const firstName = user.firstName;
  const lastName = user.lastName;

  return `${firstName} ${lastName}`;
}

// good
function getFullName(user: User): string {
  const {firstName, lastName} = user;

  return `${firstName} ${lastName}`;
}
```

- [4.2](#4-2) 配列の構造化代入を使用しましょう。

```ts
const array = [1, 2];

// bad
const first = array[0];
const second = array[1];

// good
const [first, second] = array;
```

- [4.3](#4-3) 複数の値を返却する場合は、配列の構造化代入ではなく、オブジェクトの構造化代入を使用しましょう。
  > こうすることで、後で新しいプロパティを追加したり、呼び出し元に影響することなく順序を変更することができます。

```ts
// bad
function processInput(input: any) {
  return [left, right, top, bottom];
}

const [left, _, top] = processInput(input);

// good
function processInput(input: any) {
  return {left, right, top, bottom};
}

const {left, top} = processInput(input);
```

## Strings

- [5.1](#5-1) 文字列にはシングルクオート `''` を使用しましょう。

```ts
// bad
const name = "JJ"; // prettier-ignore
const name = `JJ`; // もし内部で変数を参照する場合はテンプレートリテラルを使いましょう。

// good
const name = 'JJ';
```

- [5.2](#5-2) 100 文字以上になるような文字列は、文字列連結を使用して複数行にまたがって記述しないようにしましょう。
  > 文字列の検索容易性を損なうためです。

```ts
// bad
const errorMessage =
  'This is a super long error that was thrown because \
of Batman. When you stop to think about how Batman had anything to do \
with this, you would get nowhere \
fast.';

// bad
const errorMessage =
  'This is a super long error that was thrown because ' +
  'of Batman. When you stop to think about how Batman had anything to do ' +
  'with this, you would get nowhere fast.';

// good
const errorMessage =
  'This is a super long error that was thrown because of Batman. When you stop to think about how Batman had anything to do with this, you would get nowhere fast.';
```

- [5.3](#5-3) プログラムで文字列を生成する場合、文字列連結ではなく template string を使いましょう。

```ts
// bad
function sayHi(name: string): string {
  return 'How are you, ' + name + '?';
}

// bad
function sayHi(name: string): string {
  return ['How are you, ', name, '?'].join();
}

// good
function sayHi(name: string): string {
  return `How are you, ${name}?`;
}
```

- [5.4](#5-4) 絶対に `eval()` を使用しないでください。
- [5.5](#5-5) 文字列の中で文字を不必要にエスケープしないでください。
  > バックスラッシュは可読性を損なうため、必要な時にだけ存在するべきです。

```ts
// bad
const foo = '\'this\' is "quoted"';

// good
const foo = '\'this\' is "quoted"';
const foo = `my name is '${name}'`;
```

## Functions

- [6.1](#6-1) 関数の定義をする場合は、名前をつけるのではなく関数宣言を行いましょう。

```ts
// bad
const foo = function () {
  // do something
};

// good
function foo() {
  // do something
}
```

- [6.2](#6-2) 即時関数(IIFE)はかっこで囲みましょう。

  > 即時関数式は単一の単位であり、その両方とその呼び出しかっこを括弧で囲んで明確に表しましょう。

  ```ts
  // immediately-invoked function expression (IIFE)
  (function () {
    console.log('Welcome to the Internet. Please follow me.');
  })();
  ```

- [6.3](#6-3) 関数ではないブロック(`if`, `while` など)の中で関数を定義しないようにしましょう。代わりに変数に関数を割り当てるようにしましょう。

```ts
// bad
if (currentUser) {
  function test() {
    console.log('oh...');
  }
}

// good
let test;
if (currentUser) {
  test = () => {
    console.log('bar');
  };
}
```

- [6.4](#6-4) パラメータに `arguments` を指定しないようにしましょう。
  > 関数スコープに渡される [`arguments`](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Functions/arguments) オブジェクトの参照を上書きしてしまうためです。

```ts
// bad
function foo(name, options, arguments) {
  // ...
}

// good
function foo(name, options, args) {
  // ...
}
```

- [6.5](#6-5) `arguments` を使用しないようにしましょう。代わりに rest syntax を使用しましょう。
  > rest syntax を使用することで、いくつかのパラメータを利用したいことを明確にするためです。また rest パラメータは `arguments` のような Array-like なオブジェクトでなく正真正銘の Array です。

```ts
// bad
function concatenateAll() {
  const args = Array.prototype.slice.call(arguments);
  return args.join('');
}

// good
function concatenateAll(...args) {
  return args.join('');
}
```

- [6.6](#6-6) 関数のパラメータを変える場合は、デフォルトパラメータを使用しましょう。

```ts
// really bad
function handleThings(opts) {
  // もし、optsがfalsyだった場合は、望んだようにオブジェクトが設定されます。
  // しかし、微妙なバグを引き起こすかもしれません。
  opts = opts || {};
  // ...
}

// still bad
function handleThings(opts) {
  if (opts === void 0) {
    opts = {};
  }
  // ...
}

// good
function handleThings(opts = {}) {
  // ...
}
```

- [6.7](#6-7) 常にデフォルトパラメータは末尾に配置しましょう。

```ts
// bad
function handleThings(opts = {}, name) {
  // ...
}

// good
function handleThings(name, opts = {}) {
  // ...
}
```

- [6.8](#6-8) パラメータを再割り当てしないようにしましょう。

```ts
// bad
function f1(a) {
  a = 1;
  // ...
}

function f2(a) {
  if (!a) {
    a = 1;
  }
  // ...
}

// good
function f3(a) {
  const b = a || 1;
  // ...
}

function f4(a = 1) {
  // ...
}
```

- [6.9](#6-9) 可変引数の関数を呼び出す場合はスプレッド演算子を使用しましょう。

```ts
// bad
const x = [1, 2, 3, 4, 5];
console.log.apply(console, x);

// good
const x = [1, 2, 3, 4, 5];
console.log(...x);

// bad
new (Function.prototype.bind.apply(Date, [null, 2016, 8, 5]))();

// good
new Date(...[2016, 8, 5]);
```

## Arrow Functions

- [7.1](#7-1) 無名関数を使用する場合、アロー関数表記を使用しましょう。

```ts
// bad
[1, 2, 3].map(function (x) {
  const y = x + 1;
  return x * y;
});

// good
[1, 2, 3].map(x => {
  const y = x + 1;
  return x * y;
});
```

- [7.2](#7-2) 関数が単一の引数を取り、中括弧を使用しない場合は括弧を省略しましょう。

```ts
// bad
[1, 2, 3].map((x) => x * x); // prettier-ignore

// good
[1, 2, 3].map(x => x * x);
```

## Classes & Constructors

- [8.1](#8-1) メソッドの戻り値で `this` を返すことでメソッドチェーンを助けること。

```ts
// bad
Jedi.prototype.jump = function () {
  this.jumping = true;
  return true;
};

Jedi.prototype.setHeight = function (height) {
  this.height = height;
};

const luke = new Jedi();
luke.jump(); // => true
luke.setHeight(20); // => undefined

// good
class Jedi {
  jump() {
    this.jumping = true;
    return this;
  }

  setHeight(height) {
    this.height = height;
    return this;
  }
}

const luke = new Jedi();

luke.jump().setHeight(20);
```

- [8.2](#8-2) 1 つも指定されてない場合、クラスにはデフォルトのコンストラクタがあります。空のコンストラクタ関数やただ親クラスに移譲するだけのものは不要です。

```ts
// bad
class Jedi {
  constructor() {}

  getName() {
    return this.name;
  }
}

// bad
class Rey extends Jedi {
  constructor(...args) {
    super(...args);
  }
}

// good
class Rey extends Jedi {
  constructor(...args) {
    super(...args);
    this.name = 'Rey';
  }
}
```

## Modules

- [9.1](#9-1) 常にモジュール(`import`/`export`)を使用しましょう。またモジュールシステムの指定がある場合はそれに従いましょう。(例えば、 `next.config.js` などは動く Node.js のバージョンや package.json の type によって `import`/`export` または `require`/`module.exports`を使い分けましょう)

```ts
// bad
const AirbnbStyleGuide = require('./AirbnbStyleGuide');
module.exports = AirbnbStyleGuide.es6;

// ok
import AirbnbStyleGuide from './AirbnbStyleGuide';
export default AirbnbStyleGuide.es6;

// best
import {es6} from './AirbnbStyleGuide';
export default es6;
```

- [9.2](#9-2) ワイルドカードインポートは使用しないようにしましょう。

```ts
// bad
import * as AirbnbStyleGuide from './AirbnbStyleGuide';

// good
import AirbnbStyleGuide from './AirbnbStyleGuide';
```

- [9.3](#9-3) 可変のバインディングをエクスポートしないようにしましょう。

```ts
// bad
let foo = 3;
export {foo};

// good
const foo = 3;
export {foo};
```

- [9.4](#9-4) モジュールのエクスポートでは、名前付きエクスポートを使いましょう。ただし、このファイルを dynamic import する可能性がある場合はデフォルトエクスポートを使いましょう。
  > 基本的に名前付きエクスポートを使った方が、インポート元に対し変数名を強制できるためです。

```ts
// bad
export default function foo() {}

// good
export function foo() {}
```

## Comparison Operators & Equality

- [10.1](#10-1) 比較演算子(`==`/`!=`)を使う場合は、なるべく厳密比較演算子(`===`/`!==`)を使用してください。
- [10.2](#10-2) `if`文のような条件式は`ToBoolean`メソッドによる強制型変換で評価され、常にこれらのシンプルなルールに従います。
  - **オブジェクト** は **true** と評価されます。
  - **undefined** は **false** と評価されます。
  - **null** は **false** と評価されます。
  - 真偽値は **boolean** 型の値として評価されます。
  - 数値は **true** と評価されます。しかし、**+0, -0, or NaN** の場合は **false** です。
  - 文字列は **true** と評価されます。しかし、空文字''の場合は **false** です。
- [10.3](#10-3) 真偽値にはショートカットを使用しますが、文字列と数値には明示的な比較を使用してください。

```ts
// bad
if (isValid === true) {
  // ...
}

// good
if (isValid) {
  // ...
}

// bad
if (name) {
  // ...
}

// good
if (name !== '') {
  // ...
}

// bad
if (collection.length) {
  // ...
}

// good
if (collection.length > 0) {
  // ...
}
```

- [10.4](#10-4) null / undefined の比較を行う際は厳密比較演算子ではなく、比較演算子を使用してください。

```ts
// bad
if (foo !== undefined || foo !== null) {
}

// good
if (foo != null) {
}
```

- [10.5](#10-5) 三項演算子は入れ子にせず、常に単一行にしてください。

```ts
// bad
const foo = maybe1 > maybe2 ? 'bar' : value1 > value2 ? 'baz' : null;

// split into 2 separated ternary expressions
const maybeNull = value1 > value2 ? 'baz' : null;

// better
const foo = maybe1 > maybe2 ? 'bar' : maybeNull;

// best
const foo = maybe1 > maybe2 ? 'bar' : maybeNull;
```

## Comments

- [11.1](#11-1) 問題に対する注釈は `// FIXME: @<github account>` を使用してください。

```ts
class Foo {
  constructor() {
    // FIXME: @konojunya total はオブションパラメータとして設定されるべき
    total = 0;
  }
}
```

- [11.2](#11-2) 後ほど修正する予定の注釈は `// TODO: @<github account>` を使用してください。

```ts
class Foo {
  constructor() {
    // TODO: @konojunya total はオブションパラメータとして設定する
    total = 0;
  }
}
```

- [11.3](#11-3) コード上に注釈を付ける場合、 複数行のコメントをつけますが特に注意しておかなければならないものに関しては `// NOTE:` を使用してください。

```ts
// NOTE: foo する
function doFoo() {}
```

## Whitespace

- [12.1](#12-1) import 文のかっこの中にスペースを含めないようにしましょう。

```ts
// bad
import { foo } from './foo'; // prettier-ignore

// good
import {foo} from './foo';
```

- [12.2](#12-2) ブロックの間には改行を含めましょう。ただし変数定義の場合は含めなくてよいです。

```ts
// bad
const foo = 1;

const bar = 1;

// bad
const foo = 1;
function bar() {}

// good
const foo = 1;
const bar = 1;

// good
const foo = 1;

function bar() {}
```

## Semicolons

- [13.1](#13-1) つけましょう。
  > JavaScript はセミコロンなしで改行を検出すると、[自動セミコロン挿入(Automatic Semicolon Insertion)](https://tc39.es/ecma262/#sec-automatic-semicolon-insertion)と呼ばれる一連の規則を使用して、その改行をステートメントの終わりとしてみなし、改行の前にセミコロンを入れなければその行が壊れると考えた場所に、セミコロンを配置するかどうかを決定します。ただし ASI は必ずしも私たちの望んだ通りにセミコロンを挿入してくれるわけではないので、コードが壊れる可能性があります。

## Type Casting & Coercion

- [14.1](#14-1) 文字列の場合は `String constructor` を用いてキャストしましょう。

```ts
// bad
const totalScore = new String(this.reviewScore); // typeof totalScore is "object" not "string"

// bad
const totalScore = this.reviewScore + ''; // invokes this.reviewScore.valueOf()

// bad
const totalScore = this.reviewScore.toString(); // isn’t guaranteed to return a string

// good
const totalScore = String(this.reviewScore);
```

- [14.2](#14-2) 数値の場合は `Number constructor` か `parseInt` を用いてキャストしましょう。

```ts
const inputValue = '4';

// bad
const val = new Number(inputValue);

// bad
const val = +inputValue;

// bad
const val = inputValue >> 0;

// bad
const val = parseInt(inputValue);

// good
const val = Number(inputValue);

// good
const val = parseInt(inputValue, 10);
```

## Naming Conventions

- [15.1](#15-1) 極端に省略した変数の命名は避けましょう。名前から意図が読み取れるようにしましょう。

```ts
// bad
function q() {
  // ...
}

// good
function query() {
  // ...
}
```

- [15.2](#15-2) オブジェクト、関数、インスタンスにはキャメルケース（小文字から始まる）を使用しましょう。

```ts
// bad
const this_is_my_object = {};
function c() {}

// good
const thisIsMyObject = {};
function thisIsMyFunction() {}
```

- [15.3](#15-3) クラスやコンストラクタにはパスカルケース（大文字から始まる）を使用しましょう。

```ts
// bad
function user(options) {
  this.name = options.name;
}

const bad = new user({
  name: 'nope',
});

// good
class User {
  constructor(options) {
    this.name = options.name;
  }
}

const good = new User({
  name: 'yup',
});
```

## Standard Library

- [16.1](#16-1) グローバルな `isNaN` の代わりに `Number.isNaN` を使用しましょう。
  > グローバルな isNaN は数字ではないものを数字に強制変換し、NaN に強制変換されたものすべてに true を返します。 この動作が望ましい場合は、それを明示にしてください。

```ts
// bad
isNaN('1.2'); // false
isNaN('1.2.3'); // true

// good
Number.isNaN('1.2.3'); // false
Number.isNaN(Number('1.2.3')); // true
```

- [16.2](#16-2) グローバルな `isFinite` の代わりに `Number.isFinite` を使用しましょう。
  > グローバルな isFinite は数字ではないものを数字に強制変換し、有限数に強制変換されたものすべてに true を返します。 この動作が望ましい場合は、それを明示にしてください。

```ts
// bad
isFinite('2e3'); // true

// good
Number.isFinite('2e3'); // false
Number.isFinite(parseInt('2e3', 10)); // true
```

## React

- [17.1](#17-1) コンポーネントは名前付きアロー関数を使用しましょう。
  > function を使うとその他の関数また hooks とあいまいになってしまう可能性があるためです。

```ts
// bad
function FooComponent(props: Props) {}

// good
const FooComponent: React.FC<Props> = () => {};
```

- [17.2](#17-2) コンポーネント名はパスカルケースを使用しましょう。

```tsx
// bad
const fooComponent: React.FC<Props> = () => {};

// good
const FooComponent: React.FC<Props> = () => {};
```
