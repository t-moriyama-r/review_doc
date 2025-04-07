# 宣言マージとモジュール拡張

## 宣言マージ(declaration merging)

既存の型定義では賄えない独自のフィールドを付与したい時があります。
そういう時、まずは普通に交差型で定義することが考えられます。


```typescript
interface Fruit{}
interface Meat{}

// 両方を併せ持つインターフェース
type Food=Fruit & Meat
interface Food extends Fruit, Meat {} // interfaceを使う場合
```
みたいな。(雑な例ですが…)

ただ、typescriptの`interface`には1つ面白い仕組みがあって、それは**同じものを複数回定義するとマージになる**んですよね。


```typescript
// sample.d.ts
interface Food{
  weight: number
}

interface Food{
  tasty: number
}
// この時点で、Foodはweightとtasty両方持つインターフェース！
```
`type`は`interface`と似ていますが、こちらは複数回定義するとエラーになります。

これを**宣言マージ(declaration merging)** っていいます。
わざとらしく`d.ts`とかつけてるんですが、実際のとこローカルファイルでは使えず、**ライブラリのような外部モジュールにしか使えない**技法なので開発に直接的には必要ない知識ですが、あってもいいとは思います。

[オープンエンドと宣言マージ (open-ended and declaration merging) | TypeScript入門『サバイバルTypeScript』 ](https://typescriptbook.jp/reference/object-oriented/interface/open-ended-and-declaration-merging)

つまり、typescriptにおいて**型は後付けで拡張できる**、ということを認識した上で次の話です。

## モジュール拡張(module augmentation)

通常の開発で使えないのならいつ使うんだよって話ですが、**ライブラリの型に対する拡張を定義したい時**に使うわけですね。

これらを通常のtsファイルで型を拡張するための条件として、同じ名前の `interface` を `declare module` 内で再宣言することでのみ有効になります。

### 活用例
```typescript
import axios from 'axios';
declare module 'axios' {
  interface AxiosRequestConfig {
    fetchDate?: Date; // NOTE: 既存の型と矛盾が生じるため、requiredにはできない
  }
}
```
`AxiosRequestConfig`という既存の型に対してこうすることで、このファイル内では`AxiosRequestConfig`型は`fetchDate`を持っているものとして扱うことができる。定義は`optional`にしなければいけないという制約はあるのですが、既存の型にあとから自由にプロパティを追加できるわけですね。

これで何が嬉しくなるかというと、ライブラリ由来のメソッドに独自の定義を流しても構文エラーにならなくなるんですよね。


```typescript
//普通に書くと...
import axios from 'axios';
const config = {
  params: { id: 1 },
  headers: { Authorization: 'Bearer token' },
  fetchDate: new Date(),
};

axios.get('/api/data', config); // ❌ AxiosRequestConfigにfetchDateがない！
```
当たり前ですけど、こういうことはできないわけです。なぜなら`fetchDate`なんてオプションに定義されてないんだから。

```typescript
//普通に書くと...
import axios from 'axios';
declare module 'axios' {
  interface AxiosRequestConfig {
    fetchDate?: Date;
  }
}

const config = {
  params: { id: 1 },
  headers: { Authorization: 'Bearer token' },
  fetchDate: new Date(),
};
axios.get('/api/data', config); // OK！
```
ただ、このように型を拡張してやると構文エラーにならず、そのまま`config`を渡すことができるようになります。


このテクニックは、**ライブラリが絡んだ何かに独自処理を挟む必要がでてくる**際に活用できると思います。
