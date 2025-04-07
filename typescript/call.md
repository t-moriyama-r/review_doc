# コールシグネチャとinterface

## コールシグネチャ

…とは何か、という基本をまずおさらい。

```typescript
type Greeter = (message: string) => string
```
こういうやつのことをコールシグネチャって言います。つまり、関数に対して型を定義しているものです。これは何気なく使っていますね。



```typescript
type Human = {
  sayHello: Greeter 
  sayGoodbye: Greeter 
}
こういうこともやりますね。
```


```typescript
interface Greeter {
  (message: string): void
}
```
ここからだんだんあまり使わない話になってくるのですが、interfaceを用いて定義するとこうなります。

応用例

で、本題です。ライブラリの型追ってたらこういうのを見つけました。



```typescript
interface GenericMapper {
  <T, U>(array: T[], callback: (item: T) => U): U[];
}
```
ジェネリクスが活用されてる、おどろおどろしいこと書いてる感ありますが、一体これは何に使えるんでしょう？

とはいえ、冷静に見てみると、TをUに変換する(マッピングする)だけの関数、を表現する型であることは分かります。



```typescript
const mapper: GenericMapper = (array, callback) => {
  return array.map(callback);
};

// 返り値の型が同じでもいいし...
const numbers = mapper([1, 2, 3], x => x * 2);  // [2, 4, 6]
const strings = mapper(['a', 'b', 'c'], x => x.toUpperCase());  // ['A', 'B', 'C']

// 異なってもいい
const noList = mapper([1, 2, 3], (num) => `No.${num}`); // ["No.1", "No.2", "No.3"]
```

もっと言うと、**オーバーロードも表現できる。**

```typescript
interface Calculator {
  (a: number, b: number): number;
  (a: string, b: string): string;
}

const add: Calculator = (a, b) => {
  if (typeof a === 'number' && typeof b === 'number') {
    return a + b;
  }
  return a + ' ' + b;
};

console.log(add(1, 2));        // 3
console.log(add('Hello', 'World')); // 'Hello World'
```
`Calculator`は、引数が数字同士なら数字を返すし、文字列同士なら文字列を返す、ということが定義できる。これはどういうことかというと、

```typescript
// こう書くか
function add(a:number|str, b:number|str){
  return a+b // numberなのかstrなのかはっきりしない......
}

// こうとも書けるが
function add<T extends number|str>(a:T, b:T){
  return a+b //型推論は効くが、自由度が高く限定的ではない(number|str型も許容してしまう)
}

// number同士、str同士を"強制"できる
const add:Calculator=(a,b)=>return a + b ！
```
こういうことが言える。これは型で論理構造を表現する思想に応用できることを示唆してる。

> [!NOTE]
> リファクタサンプル：型で論理構造を表現する

もっと身近な例だと、Reactの`SetState`もこれで表現できる。

```typescript
// ReactのSetStateだとこんな感じ
interface SetState<T> {
  (value: T): void;
  (updater: (prev: T) => T): void;
}

// イベントハンドラも似たような感じで定義できる！
interface EventHandler {
  (event: MouseEvent): void;
  (event: KeyboardEvent): void;
}
```
何気なく使ってる型も、実はこのように定義されてたのか、って驚きませんか？笑

ヘルパ関数とかそういうのを定義する際にこういうのを活用できると良さそうだなーって思います。
