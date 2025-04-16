# Utility Typesを使いこなして気持ちよくなろう

> **Typescript雑学メモ 目次**<br>
> Utility Typesを使いこなして気持ちよくなろう←ここ<br>
> [コールシグネチャとinterface](./call.md)<br>
> [宣言マージとモジュール拡張](./merge.md)<br>
> [リファクタサンプル：型で論理構造を表現する](./struct.md)<br>
> [二者択一は型にやらせる](./choice.md)<br>
> [Contextとレンダリング](./context.md)
> <br>
> [React v19](./v19.md)


`Omit`とか`Pick`とかあのへんのやつです。

[ユーティリティ型 (utility type) | TypeScript入門『サバイバルTypeScript』](https://www.typescriptlang.org/docs/handbook/utility-types.html)

実は他にも色々あって、一覧を見てると見たこともないようなものがあり簡単に気持ち悪くなれる。(？)

[TypeScript: Documentation - Utility Types](https://www.typescriptlang.org/docs/handbook/utility-types.html)

個人的にはExtractとかぱぱっと使いこなせてる人を見るとマトモにコード書ける人なんだな感出るので、そのあたりを自分なりに再整理しつつ、使ったことないものもまとめて実用例を探して記憶の片隅に置こうと思う。

使う頻度がありそうな(ある)順にまとめる。

> [!NOTE]
> 【目次】<br>
> - [最低限知っておきたいもの](#最低限知っておきたいもの)<br>
>   -  [Omit<T>](#Omit<T>)<br>
>   -  [Pick<T>](#Pick<T>)<br>
>   -  [Record<Key, Type>](#Record<Key, Type>)<br>
> - [たまに見かけるもの](#たまに見かけるもの)<br>
>   -  [ReturnType<T>](#ReturnType<T>)<br>
>   -  [Partial<T>](#Partial<T>)<br>
>   -  [Required<T>](#Required<T>)<br>
>   -  [NonNullable<T>](#NonNullable<T>)<br>
>   -  [Readonly<T>](#Readonly<T>)<br>
>   -  [おまけ: ReadonlyArray(=readonly T[])](#おまけ: ReadonlyArray(=readonly T[]))<br>
>   -  [おまけ２：satisfies演算子](#おまけ２：satisfies演算子)<br>
> - [知ってると便利なもの](#知ってると便利なもの)<br>
>   -  [Exclude<T, U>](#Exclude<T, U>)<br>
>   -  [Extract<T, U>](#Extract<T, U>)<br>
>   -  [Parameters<Type>](#Parameters<Type>)<br>
>   -  [おまけ：[number]](#おまけ：[number])<br>
> - [よくわからん奴ら](#よくわからん奴ら)<br>
>   -  [NoInfer<T>](#NoInfer<T>)<br>
>   -  [Awaited<T>](#Awaited<T>)<br>
>   -  [Uppercase<StringType>、Lowercase<StringType>](#Uppercase<StringType>Lowercase<StringType>)<br>
>   -  [Capitalize<StringType>、Uncapitalize<StringType>](#Capitalize<StringType>Uncapitalize<StringType>)<br>
>   -  [ConstructorParameters<Type>](#ConstructorParameters<Type>)<br>
>   -  [InstanceType<Type>](#InstanceType<Type>)<br>
>   -  [ThisParameterType<Type>](#ThisParameterType<Type>)<br>
>   -  [OmitThisParameter<Type>](#OmitThisParameter<Type>)<br>
>   -  [ThisType<Type>](#ThisType<Type>)<br>

#最低限知っておきたいもの

普通にめっちゃ使うやつ。

## Omit<T>

ある型から要らないものを取り除くもの。


```typescript
type Human={
  name: string;
  age: number;
  weight: number;
}

type Profile = Omit<Human, "age" | "weight">
```
みたいなノリ。なんだかんだ使います。ユニオン型で複数指定できる。

## Pick<T>

```typescript
type Human={
  name: string;
  age: number;
  weight: number;
}

type Profile = Pick<Human, "name">
```
Omitの逆。こっちは案外使う機会少ないけど、たまーーーーーに使う。

## Record<Key, Type>
キーとその内容をジェネリクスで指定できる。プリミティブ型だと恩恵を感じることがあんまりないが(`Record<string, string>`とか使うけど)、このジェネリクスに何でも入れることができるのがミソ。つまり自分で定義した型を使い、厳格な型を生成するのに使う。

```typescript
enum UserRole {
  Admin = 'admin',
  Editor = 'editor',
  Viewer = 'viewer'
}

const roleDescriptions: Record<UserRole, string> = {
  [UserRole.Admin]: '全機能へのアクセス',
  [UserRole.Editor]: '編集権限',
  [UserRole.Viewer]: '閲覧のみ'
};
```
chatGPTになんかいい例出して、って投げたもの。こんな風にオブジェクトの中身を制限したい時に使う。

余談ですが、**enumって実はミュータブルだからあんま使わない方がいい**んですけどね。as constで定義するのダルいから僕は好きですけど、本質的にはダメではある。少なくともこの例では使うとマズいっちゃマズい。

# たまに見かけるもの
どっちかっていうとライブラリの返り値とかに使われてるイメージ。自分で書くことはあんまりないかもだけど、知識としてないのは微妙。

## ReturnType<T>

関数の返り値の型を返す。`ReturnType<typeof ()=>string>`ならstring、みたいな。何に使うんだよって話ですが、

```typescript
function createUser(name: string) {
  return { id: Date.now(), name };
}

type User = ReturnType<typeof createUser>;
// { id: number, name: string } 型になる
```
どっちかっていうと、こういう**型推論に名前をつける**という意図で使われるイメージ。主導権がオブジェクトではなく関数の方にあるパターンだとありうるけど、まあ珍しい。とはいえ稀に見かけるので知っといた方がいいっちゃいい。

## Partial<T>

すべてをoptionalにする。**(※nullableではない)**
フォームの初期値とかに使えそう。例えば**名前はrequiredだけど初期値はないよね**みたいな定義は`Partial`を使うと一撃で定義可能。
~~でもだいたいのフォームでそういう柔軟な定義をさせてくれないイメージがある…~~

## Required<T>
Partialの逆。つまりundefinedが剥がれる。**nullはそのまま。**

・・・ってのがめっちゃ分かりづらいというか、僕もよく知らなかったので手作業で使うと混乱するかも。
あくまでも**フィールドとして必須**、っていうニュアンスを感じるのが大事。

~~でも、使うことあるんですかね？~~

## NonNullable<T>

上記`Required<T>`に加えて、nullも剥がすもの。
フォームで、初期値のほうをゆるい制約にして、すべてをrequiredにする型という意味でこれを使って定義する、みたいな使い方はできないこともない。
けど、フォームの型の主語は送信する時のインターフェースである方が自然なんだよなぁ感……

## Readonly<T>

全てにreadonly属性を付ける。**as constの方が再帰的で厳格**なのでほぼ使わない。
まあ、as constに包含されているわけでもないので型ヒント的につけてもいいケースはありそうではあるが、そもそもオブジェクトの中身を弄りまわすようなロジックってフォーム以外に組むことない気がする。

## おまけ: ReadonlyArray(=readonly T[])

[読み取り専用の配列 (readonly array) | TypeScript入門『サバイバルTypeScript』 ](https://typescriptbook.jp/reference/values-types-variables/array/readonly-array)

引数の形とかに指定するとイミュータブルな配列に限定できそう、とか思いますが、**そもそも配列の値を直接弄って返すようなコード書く奴とかおるんか？** 感はあるので使わなさそう……
そういうことするんなら新しい配列作って返すと思う……。元の値をこねくりまわすのってクラス/インスタンス的な考え方な気がする。<br>
いわゆる関数型プログラミング的な考え方だとそんな感じっぽいです。僕自身は意識したことないけど…(さっき関数型プログラミングについて調べてて初めて腑に落ちた…自分はめっちゃ感覚でやってました……)
Typescript(React)は割とそういう書き方がメジャーな気がします。しらんけど。

> [!NOTE]
> [【コード編2】とにかくイミュータブルを意識する](../review/code_2.md#とにかくイミュータブルを意識する) 

何が言いたいかっていうと、そもそも**readonlyであることを前提として書いてると思う**のでわざわざ型注釈でこんなのつけることは実際ないとは思う。僕は見たこと無い。

 
[純粋関数は最高だ｜こわくない関数型プログラミング ](https://zenn.dev/tockri/books/dcaf6c55e64448/viewer/pure_func_is_wonderful)
Readonlyの一応の活用例。大事なのは引数として渡される**配列やオブジェクト**はTypescriptにおいては**参照渡しとなっている**ことを忘れないでね、ということ。

## おまけ２：satisfies演算子
型アサーションの亜種で`satisfies`というのが存在する。あんま使わない方がいいとは思うけど、キャストを避けたい時、稀に使用する。

[satisfies演算子「satisfies operator」 | TypeScript入門『サバイバルTypeScript』 ](https://typescriptbook.jp/reference/values-types-variables/satisfies#as-const%E3%81%A8%E7%B5%84%E3%81%BF%E5%90%88%E3%82%8F%E3%81%9B%E3%82%8B)
```typescript
const array = [1, "2", 3] as const satisfies (string | number)[];
```
こういう例とか使えるんかな？でも同ページに書かれてる配列の共変性とかあるし実際こういう型推論って必要なケースあんまないかも……。有益なユースケースを言語化できたらまた追加します。

# 知ってると便利なもの
マイナーだが知ってると便利なもの。Zodとか使ってるとちょくちょく使うシーンがある。

関連：
リファクタサンプル：型で論理構造を表現する 

## Exclude<T, U>

あるインターフェースから**型**を削除するもの。`Omit`との違いは下記。

```typescript
type Status = 'success' | 'error' | 'loading' | null;
type ActiveStatus = Exclude<Status, null>;
```
つまり**ユニオン型に対するOmit**みたいなノリで使う。

Reactコンポーネントのnull回避とかもこれでシンプルに書ける。


```typescript
type Response= Data | null
interface Props{
  response: Response //コンポーネントに渡ってくる時点でnullの可能性がない実装
}
const Component=({response}:Props){
   //nullが渡ってくることはないが構文エラーが出るので考慮
   if(response==null){
    return null
   }
   return ......
}
```
みたいな型推論でnullableだと、**nullにならないことがわかっているのにキャストが苦しい**例に使える。

```typescript
type Response= Data | null
interface Props{
  response: Exclude<Response, null>
}
const Component=({response}:Props){
   //responseはある前提のコードが書ける
   return ......
}
```
こういうケース、**稀によく見る**ので覚えておいて損はない。ちゃんと勉強してないとこういう知識にたどり着かないからこそ便利ではある。


## Extract<T, U>

さっきの逆で、抽出する。`Pick`みたいなやつ。
こっちの利用ケースはわからんけど、何らかのエッジケースについての分岐のためのコンポーネント定義とかそういうのに使うイメージ。

```typescript
/**
 * 戻り値に型がきちんとつく, より賢い Object.keys
 */
export const keys = Object.keys as <T>(o: T) => Extract<keyof T, string>[];
```
今のプロジェクトのユーティリティー関数にこういうのが居た。キーとかふつうstringだしkeyof Tで十分では？って思ったけど、


```typescript
const obj = {
  name: "John",
  age: 30,
  [Symbol('id')]: 123
};
keys(obj); // keys は ["name", "age"]
```
といったようにシンボルキーとかよくわからない存在をストリップできるので、良い活用例といえる。

 

おまけ。似たようなもので面白いユーティリティーもある。

```typescript
/**
 * 戻り値に型がきちんとつく, より賢い Object.entries
 */
export const entries = Object.entries as <T>(
  o: T
) => (keyof T extends infer U ? (U extends keyof T ? [U, T[U]] : never) : never)[];
```
`infer`とか`Conditional Types`(型の条件分岐)とかが活用されている。

[infer | TypeScript入門『サバイバルTypeScript』](https://typescriptbook.jp/reference/type-reuse/infer)
[Conditional Types | TypeScript入門『サバイバルTypeScript』](https://typescriptbook.jp/reference/type-reuse/conditional-types)

## Parameters<Type>

Parameters<Type> は、関数型から引数の型をタプルとして抽出します。

基本的な使用例：
```typescript
// 関数の引数型を抽出
function greet(name: string, age: number) {
  return `Hello, ${name}. You are ${age} years old.`;
}
type GreetParams = Parameters<typeof greet>;
// [name: string, age: number] 型
```
マジで何に使うんだろう……。って思ってたけど、カスタムフック書いてる時に使ったので格上げ。
僕が使った用途としては**任意の関数を渡せるカスタムフックに型推論を効かせることができるように**、という用途。

```typescript
export const useSaveFetchDate = <T extends (...args: Parameters<T>) => ReturnType<T>>(
  fetcher: T
) => {
  const fetchDateRef = useRef<Date | null>(null);
  const fetch = useCallback(
    async (...args: Parameters<T>) => {
      const response = await fetcher(...args);
      fetchDateRef.current = new Date();
      return response;
    },
    [fetcher]
  );
  return {
    fetch,
    fetchDate: fetchDateRef.current,
  };
};
```
こうすることで、引数(および返り値)に正しく型推論が効き、特にジェネリクスを使用しなくてもtype-narrowingが効く。

## おまけ：[number]

似たような使い方するのでこのコラムにまとめる。**配列から要素の型を取り出すもの**って理解すればOK。
難しい言葉で言うと**インデックスアクセス型**とかいうんですけど、そのせいでぱっと見理解が難しくなりそう。

```typescript
type Parson=Human[]
type AAA=Parson[number] // Humanが取り出せてAAA=Humanになる
``` 
ちゃんとした例をあげると、例えばここを書くきっかけになった自分が書いた例をあげると(さすがにちょっと修正してます)

```typescript
export type Account = Exclude<Detail, 'personal'>['accounts'][number];
```
こういうのを書くと、**元となった配列にインターフェースが定義されていなくても取り出すことができる**という点で大変エコです。<br>
まあ大体は定義されてるものなんですけど、何らかの理由で定義していない場合とかたまーにあると思います。中身が若干不確定要素あったり。そういう場合も推論が効くので使いこなせると強いです。

# よくわからん奴ら

たぶん知らなくても困らないor雰囲気でわかるもの。非常に限定的なケースなら使えるかも。僕は見たことない。雑学レベルだけど、もしかしたら有意義なものもあるかも知れないので調べる。

## NoInfer<T>

読んで字のごとく推論を防ぐらしい。そもそも使わないで済むのがベストなので、ライブラリとかの兼ね合いでよっぽど苦しくなった時とかにしか使わなさそう。そのまま使うと変な型の可能性を推論されるけど、要件的にありえないからNoInferでいいや、みたいな。
**ユーザ定義型ガード**使えばお上品に書けるけど、そこまで堅牢ではなくてもいいやって時にしか使わなさそう。

たぶん、ロジック的にありえるせいで**`Omit`が使えないけど、`as`のように限定できないパターン**とかに使うんでしょうか？

## Awaited<T>

Promiseの返り値を取得する。いつ使うんだろう？asyncが使えない特定のケースとか？

[Awaited<T> | TypeScript入門『サバイバルTypeScript』](https://typescriptbook.jp/reference/type-reuse/utility-types/awaited)

↑を見る限り、**ありえないぐらいPromiseがネストしてる場合**に活用できるらしい。が、**そもそもそんなコード書くな**という話でもある…

## Uppercase<StringType>、Lowercase<StringType>

`Uppercase<StringType>` は、文字列型を大文字に変換する型ユーティリティです。

基本的な例：

```typescript
type Greeting = "hello";
type UpperGreeting = Uppercase<Greeting>; // "HELLO"
type Color = "red" | "green" | "blue";
type UpperColor = Uppercase<Color>; // "RED" | "GREEN" | "BLUE"
```
**マジで何に使うん？** って思ったんですけど、小文字なのか大文字なのかよくわからん定義をまとめたい時とかにギリ使えるんですかね。システム移行とか。

## Capitalize<StringType>、Uncapitalize<StringType>

`Capitalize<StringType>` は、文字列の最初の文字を大文字に変換する型ユーティリティです。

基本的な例：
```typescript
type Name = "john";
type CapitalizedName = Capitalize<Name>; // "John"
type Color = "red" | "green" | "blue";
type CapitalizedColor = Capitalize<Color>; // "Red" | "Green" | "Blue"
```
これもよくわかんないけど、まあそういうのがあるんですねということで……

## ConstructorParameters<Type>

`ConstructorParameters<Type>` は、クラスのコンストラクター引数の型をタプルとして抽出します。

基本的な例：

```typescript
class User {
  constructor(
    public name: string, 
    public age: number, 
    public email?: string
  ) {}
}
type UserConstructorParams = ConstructorParameters<typeof User>;
// [name: string, age: number, email?: string] 型
```
クラスとか全然使わなくなったなぁ……。たぶん用途としてはさっきと一緒な気がする。クラスを拡張する際とかにも使える？

## InstanceType<Type>

`InstanceType<Type>` は、クラスのインスタンス型を抽出します：

基本的な例：

```typescript
class User {
  name: string;
  age: number;
  constructor(name: string, age: number) {
    this.name = name;
    this.age = age;
  }
  greet() {
    return `Hello, ${this.name}!`;
  }
}
type UserInstance = InstanceType<typeof User>;// User クラスのインスタンス型
```
これはクラス使ってたら使うこともあるかも知れない？わからんけど。


## ThisParameterType<Type>

`ThisParameterType<Type>` は、関数の this パラメータの型を抽出します。

基本的な例：

```typescript
function example(this: { name: string }, arg: number) {
  return this.name + arg;
}
type ThisType = ThisParameterType<typeof example>;// { name: string } 型
```
そもそも今時thisとか使わないし、正直よくわからん。10年ぶりに見た。笑
これは覚えなくてもいいかな……　thisのスコープとかそういう話をまず思い出すところから始めないとだし、むしろそういうレガシーなコードの現場、行きたくないしなぁ…….


## OmitThisParameter<Type>

`OmitThisParameter<Type>` は、関数の this パラメータを削除した型を返します。

基本的な例：

```typescript
function method(this: { name: string }, arg: number) {
  return this.name + arg;
}
// thisパラメータを削除
type MethodWithoutThis = OmitThisParameter<typeof method>;
// (arg: number) => string 型
```
　これも上記と同じ。使う場面が想像できない。


## ThisType<Type>

`ThisType<Type>` は、オブジェクトリテラルの this コンテキストを指定するための型ユーティリティです。

基本的な例：

```typescript
type ObjectWithThis = {
  doSomething(): void;
  getData(): string;
} & ThisType<{ name: string }>;
const obj: ObjectWithThis = {
  doSomething() {
    console.log(this.name); // thisの型が保証される
  },
  getData() {
    return this.name;
  }
};
```
オブジェクトのthisの型を限定するってことですかね？

例は面白そうだけど、手で書くことはあんまりないのかな。ストアを自分で作らざるを得ない時とかに活用するんですかね？どっちにしろ今はよくわかんないっす。


とりあえずExcludeとかそのあたりをうまく使いこなせると、だいぶインターフェースの自由度が上がると思います。
