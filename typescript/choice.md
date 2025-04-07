# 二者択一は型にやらせる

なんか盗める知識ないかな～と思って客先の過去のプルリク覗いてたらおもしろいのがあったのでまとめる。

【React･フロント編】よくあるレビュー指摘点(wip) に盛り込もうと思ったけど、ちょっと高度な内容なのでこっちで。レビューしててもこのレベルだとおそらく妥協すると思うので。
逆に言うと、こういうのを見て血肉にできる人だとレビューしてて楽しいんだよなー……。

ぶっちゃけ言ってること自体は同じ→
リファクタサンプル：型で論理構造を表現する 

二者択一とは

要は、AでなければB、というPropsを用意したい。単一のpropsだとこういうやつです。

```typescript
interface Props{
  documentType: "csv"|"pdf"
}
```
これもリテラル型っていう立派な型なんですけど、こういう書き方をするとdocumentTypeはcsvかpdfのどちらかというのが確実に分かるわけですね。なので、型推論とかでも…

```typescript
if(documentType== "csv"){
  return;
}
// この時点でdocumentType="pdf"という型推論が効く！
```
こういうことができて非常にハッピーなわけですね。

フィールドが2つある場合

今までは単一のフィールドについてでしたけど、これが2つになるとどうでしょうか？


```typescript
// telかmobile、どちらかを必須にしたい......
interface Props{
  name: string;
  tel?: string;
  mobile?: string;
}
```
これをどう表現するかって話ですね。これパッと言える人いますかね？

```typescript
type Props={
  name:string;
} & ({tel:string}|{mobile:string})
```
正解はこれです。

この定義で肝になってるのはユニオン型を交差しているとこですね。こうすることでどちらか片方しか満たせない、というのを定義できます。

これは簡単な例ですが、同様のことを3つにしたり、2*2の組み合わせをチェインしたりして、型で論理構造を限定でき、それを型推論に活かすことができます。typescriptの面白いとこです。

### 活用例

これが実際に僕が見た例に近いやつ。ダイアログにtextをもたせるか、childrenでコンポーネントを渡すかというのができる。


```typescript
// 知らないとこう書かざるを得ない
interface DialogProps{
  color: Color;
  onClose: ()=>coid;
  content: Exclude<ReactNode,null>;
}

// 別例としてstring | JSX.Elementでも可。どっちにしろこういう形にせざるを得ない。
// メッセージを出す例
<Dialog content="更新完了しました"/>

// でかいコンポーネントを埋め込みたいが、なんかイマイチ...
<Dialog content={<OtherDialog/>}}/>
```
stringとReactNodeが包含関係にあるせいでpropsとしてはまとめられるんですけど、やっぱり分かりづらいし、contentの中身がプリミティブなstringかどうかで分岐させたい時とかどうするんだろうっていう。typeofとか使えばいいんでしょうけど。
あと普通にcontentっていう命名もなんか気に入らないんですけど、textにしちゃうとコンポーネントが渡せるかどうかわかんなくなっちゃうし、って考えるとやっぱ苦しいですね。

 

ですが、さっきの知識を活かすとこのようなコンポーネントの定義にできます。


```typescript
type DialogProps={
  color: Color;
  onClose:()=>void;
} & ({text: string} | {children: Exclude<ReactNode,null>})
```
こんな感じのダイアログコンポーネントを用意すると、


```typescript
// 雑なメッセージダイアログを作れる
<Dialog text="更新完了しました"/>

// これもできる。直感的でわかりやすい。
<Dialog>
  <OtherComponent/>
</Dialog>

// こういう意味わかんないことは型が弾いてくれる
<Dialog text="ダミー">あああ</Dialog>
```
こんな風に、型で制限を加えることで直感的なコンポーネント、およびインターフェースの命名を行うことができる。活かせるパターンは結構あると思います。
