# よくあるレビュー指摘点【コード編2】

> [!NOTE]
> ある程度書けるけどコードがとっ散らかりがち…な、経験半年ぐらいの人向け。だいたい知らなければ1回は指摘される内容。
> これぐらいの内容はレビューでも妥協してスルーしがちですが、1回指摘して繰り返されると失望する…笑

これもなるべく取り入れやすい(難易度が低い順)に並べているつもり…

## そのコメント、必要？

コメントはあった方が嬉しいですが、コメントがなくても理解できるコードが最良です。
ちなみに宗派によってはコメントを減らせっていうレビューもされることあります。結局コメントがないと理解できないコードってよくないっていう話に落ち着くんですが。
だいたい命名が悪いか関数の責務が広がりすぎてるかのどっちかが原因です。

### 例：適した命名をすればコメントが不要になるケース


```typescript
// ID に一致するユーザーをリストから探す
function getUser(users: User[], id: number) {
    return list.find((u) => u.id === id);
}
```
ユーザーを取得、というつもりで書いてるっぽいですけど、これ下記のようにするとコメントなくてもわかりません？

```typescript
function findUserById(users: User[], id: number) {
    return users.find((u) => u.id === id);
}
```

(※↓の例、冷静に見たらあんまよくない例だったので後で差し替え予定)

他の例だと、chatGPT君はこういう例を教えてくれました。

```typescript
function fetchUserAndProcess(id: number, callback: (user: User) => void) {
    fetch(`https://api.example.com/user/${id}`)
        .then((response) => response.json())
        .then((user) => callback(user))
        .catch((error) => console.error("エラー", error));
}
```
callbackとかいう最悪の命名が諸悪の根源なんですが、こういうのもコメントをつけて命名の悪さをごまかさずに…

```typescript
async function fetchUser(id: number): Promise<User> {
    const response = await fetch(`https://api.example.com/user/${id}`);
    if (!response.ok) {
        throw new Error("ユーザー取得に失敗");
    }
    return response.json();
}
async function handleUserProcessing(id: number, processUser: (user: User) => void) {
    try {
        const user = await fetchUser(id);
        processUser(user);
    } catch (error) {
        console.error("エラー", error);
    }
}
```
このように責務を分割することで、何がしたいかっていうのは見通しが出てくるんじゃないでしょうか。processUserはコールバックなので何やるか結局わかんないですけど、この関数にとっては知ったこっちゃないしまあこれでもいいか…といったところ。

ある程度のキャリアを経て基本に立ち返ると、見えてくる景色もあると思います。
例①→
【コード編1】よくあるレビュー指摘点(wip) | 例：トートロジー・命名の錯綜
例②→
【命名の話】コードを書く時はIQを3にして読む 

小難しい処理は、より適切な表現をすることで簡単化できるパターンもあります。
例→
【イディオム編】よくあるレビュー指摘点(wip) | 例：配列内から重複したものを取り出す 

コメントを消す努力をしてみると、こういうのは身につくかも知れないです。

### 例：WhatではなくWhyを意識

もう1つ、コメントについて意識する点としては「何をしているコード」を説明するのではなく「何のためにあるコード」なのかがコメントとしては欲しいところです。

そもそもWhat、つまり「何をしているか」はコードを見りゃわかるという状態がベストで、そうではなく何か仕様があって、それを満たすために敢えてこういうことをしている、という「なぜ」の部分が必要だからコメントというのがあるのです。

```typescript
// 0 以上の数値を取得する
const positiveNumbers = numbers.filter((n) => n >= 0);
```
例えばこれ。0以上の数値を取得する…っていうのは見りゃわかる。こういうコメントをしたくなる気持ちもわかる。でも、このコメントにぶっちゃけ意味はない。だって見りゃわかるじゃん。

```typescript
// 負の値は無効データとして扱うため、0 以上の数値のみを取得する
const positiveNumbers = numbers.filter((n) => n >= 0);
```
Whyを意識するとこういうコメントになる。これならコメントを残す価値があるといえる。

## ネストを下げる意識

ネストが深いコードは何をやってるか読みづらいので、とにかくネストは下げる意識。多段ネストしてるコードはレビューしてても、書いてて頭疲れないんかなって思う。みんな真面目なんだよな……。

### 例：早期リターン

昔にこの話見て目から鱗だった記憶がある。僕は初期の頃から意識してる。
よく指摘したコードとしては、下記のようなコード。

```typescript
function example(user){
  if(user){
    user.id=1
    user.name="テスト"
    user.email="example@test.com"
    ......
    console.log("処理ここまで")
  }
}
```
if文で囲ってますが、早期リターンするとネストを一段下げられます。

```typescript
function example(user){
  if(!user)return null;
  user.id=1
  user.name="テスト"
  user.email="example@test.com"
  ......
  console.log("処理ここまで")
}
```
ネストを下げられる他、userがnullだった場合そこから先のコードを読まなくてもよくなり、読みやすいコードになっていきます。

もうちょっと複雑な例としては…

```typescript
// 関数から特定の値を見つける ※ほんとはfindとか使うべきだと思う
function findValue(array, target) {
  let found:boolean = false;
  if (Array.isArray(array)) {
    for (let value of array) {
      if (value === target) {
        console.log(`見つかりました: ${target}`);
        found = true
      }
    }
    console.log('見つかりませんでした。');
  } else {
    // hint: ここを読んでる時点で、なんの条件分岐の中だったか覚えてる人います？
    console.error('引数は配列でなければなりません。');
  }
  return found;
}
```
(補足) 今どきletなんか使うと確実にレビューで突き返されます。
参考：
【コード編2】よくあるレビュー指摘点(wip) | とにかくイミュータブルを意識する 

初心者が書きがちなのはこのようなコード。愚直にif文書いてるとこうなるのですが、ネストが深くて読みづらいし、if文の中にどういう条件で入ってるのか読みながらわからなくなる。

早期リターンの何がいいかは、これをリファクタするとすぐ理解できる。

```typescript
function findValue(array, target) {
  if (!Array.isArray(array)) {
    console.error('引数は配列でなければなりません。');
    return false; // 早期リターン
  }
  // この時点でもう配列であることが確定している
  for (let value of array) {
    if (value === target) {
      console.log(`見つかりました: ${target}`);
      return true; // 条件を満たしたら早期リターン
    }
  }
  // この時点でもう正常系がないことが確定している
  console.log('見つかりませんでした。');
  return false; // 最後まで見つからなかった場合
}
```
排除したい事項の条件を満たした時点ですぐに関数を終了することで、そこから先のコードを読む必要がなくなるという大きな恩恵が得られます。例えばこういうのを意識してると…

```typescript
// ありがちな例
function validateUserInput(input) {
  let result: boolean = true;
  if (input) {
    if (input.length < 3) {
      result = false; // 長さが不十分
    }
  }
  // みんな結局ここまで読まされてますよね？
  return result;
}
```
```typescript
// 早期リターン
function validateUserInput(input) {
  if (!input) return false; // 入力が空
  if (input.length < 3) return false; // 長さが不十分
  return true; // 検証成功
}
```
条件を満たした場合、そこから先のコードを読む必要がなくなるのでそのぶん頭を使わなくて済むし、正常系の場合も排除すべき可能性を排除できてるかどうかがわかりやすい。何よりネストが浅く読みやすい。

僕はここは簡単にできるのに恩恵が大きく、ちょっとこだわりがちなのでかなりの頻度でレビュー指摘してました。

elseの使用を避ける意識があるとうまく行くかも知れません。

## 例：関数分割でネストを下げる
```typescript
function processOrders(orders) {
  if (orders && orders.length > 0) {
    let totalPrice = 0;
    for (const order of orders) {
      if (order && order.price > 0) {
        totalPrice += order.price;
      } else {
        console.error('無効な注文が含まれています。');
        return;
      }
    }
    if (totalPrice > 10000) {
      console.log('注文の合計金額が上限を超えています。');
    } else {
      console.log(`合計金額: ${totalPrice}円`);
    }
  } else {
    console.error('注文がありません。');
  }
}
```
オーダを処理する関数ですね。無効な注文があればバリデートし、正常な注文なら合計金額を表示する…って、1要素で説明できてない時点でよくない関数というのはわかるんですが、このままだとネストが2段あって読みづらいですね。分割してみます。

```typescript
function validateOrders(orders) {
  if (!orders || orders.length === 0) {
    console.error('注文がありません。');
    return false;
  }
  for (const order of orders) {
    if (!order || order.price <= 0) {
      console.error('無効な注文が含まれています。');
      return false;
    }
  }
  return true;
}
function calculateTotalPrice(orders) {
  return orders.reduce((total, order) => total + order.price, 0);
}
function printTotalPrice(totalPrice) {
  if (totalPrice > 10000) {
    console.log('注文の合計金額が上限を超えています。');
  } else {
    console.log(`合計金額: ${totalPrice}円`);
  }
}
function processOrders(orders) {
  if (!validateOrders(orders)) {
    return;
  }
  const totalPrice = calculateTotalPrice(orders);
  printTotalPrice(totalPrice);
}
```
chatGPTになんかいい例ない？って投げただけの例なので若干微妙な感じはするんですが、こんな風に要素を分割した関数を定義するとネストが下がるし、それぞれの関数の責務も分離されるので読みやすくなります。

個人的には、ネストは2段を超えるとシェイプアップできそうなにおいを感じます。絶対ではないですが、何か手抜きできないかを探し始めます。

## 条件分岐の工夫
### 例：離れたelseはなるべく使わない(とにかく近くに置く意識)

さっきの例でもヒントとして書きましたけど、elseとか何に対してのelseだったか覚えてられません。

```typescript
function processUser(user: User) {
  if (user) {
    if (user.isActive) {
      if (user.hasPermission) {
        // 深いネスト
        if (user.role === 'admin') {
          // 何か処理
        } else {
          // 別の処理
        }
      } else {
        // ...？
      }
    } else {
      // ......？？？？？？
    }
  } else {
    // 知るか！！！！！！！！！！！！！
  }
}
```
このパターンの大体は早期リターンで対応できますけど、そうでない場合は関数分割するべきシグナルです。

```typescript
function example(user: User){
  if(user.id == null && !user.isGuest || !user.hasAdmin){
    user.name="名無し"
    user.id=generate_uuid()
    user.isGuest=true
    user.hasAdmin=false
    user.loginCount=0
  }else{
    // ifの内容を思い出すのに大きな視線移動が必要で、処理を理解するのが大変......
    user.loginCount++
    user.logined=true
  }
}
```
超テキトーに書いたのでありえないコードではあるのですが、こういう類のコードがあったとして、なんか分かりづらいですよね。


```typescript
function example(user: User){
  if(user.id == null && !user.isGuest || !user.hasAdmin){
    setDefaultConfig(user)
  }else{
    // どういう条件の時にここに入ってくるんだっけ？っていうのはちょっと上を見るだけでいい
    login(user)
  }
}
```
else句の中を書く人にとって、関係ない条件の場合の具体的処理とか見る必要ないので、関数化することで折りたたまれていると認知がスムーズです。ああ、なんかゲストユーザーの場合はここでデフォルト値を設定してんだな(中身はよーわからんけど)、以上の情報は知る必要ないので、このように関数にして具体の処理を隠蔽するとわかりやすいコードに見えません？

絶対ってわけじゃなく、最終的には感覚の話です。
が、基本的に5行以上前の情報は覚えてないことを前提に書くと良いです。他にも条件そのものを整理したり関数化したり色んなアプローチがありますが、とにかく頭を使わず読めるコードを目指して下さい。

## 関数について
### 例：多すぎる引数

だいたい関数の引数が3～4つを超え始めると、ほんとかな？というにおいは感じます。

もちろん、依存関係にある変数が4つ以上あるパターンなんて全然あるんですけど、1つのインターフェースにまとめた方がいいんじゃないか？というのはよく考えた方がいいです。

```typescript
function processProduct(productId?: number, price?: number, secretFlg?: boolean){
  ...
}
```
例えばこういう関数。引数が3つあるだけでうわぁ～…って感じるんですけど、それぞれに目を向けてみると、商品ID、値段、あとなんかのフラグ…ってあります。
果たしてこれらは別々に管理すべきものでしょうか？概念として1つではないでしょうか。

```typescript
interface Product{
  productId: number;
  price: number;
  secretFlg: boolean;
}
function processProduct(product: Product | null){
  ...
}
```
別にこれでよくない？って話です。だらだらした記述が減ってコンパクトになりましたね。引数を減らす、というのはこういうことでもあります。

え、IDがなくて値段がある要件がある場合ですか？

```typescript
interface Product{
  productId?: number; // optionalにした
  price: number;
  secretFlg: boolean;
}
function processProduct(product: Product | null){
  ...
}
```
別にこれでよくないですか？あくまでもこれらは商品という1つの概念に紐づけて管理できますよね、っていう話です。バラバラである「必然性」がない。
まあ、IDがなくて値段だけが渡ってくるみたいな設計の方がこの場合クソではありますが、例だし許して


```typescript
interface Product{
  productId?: number; // optionalにした
  price: number;
  secretFlg: boolean;
}
function processProduct(userId:number, product: Product | null){
  ...
}
```
一方、ユーザーIDと商品を紐づけるみたいな関数の場合は、当然商品とユーザーって同じ概念ではないので、引数として分けるのは至極当然です。このような必然性を持って説明できるかどうかで分けるといいと思います。

```typescript
function processProduct(userId:number, productId?: number, price?: number, secretFlg?: boolean){
  ...
}
```
ただ少なくともこれは確実にナシです。引数が4つある関数なんてそもそもが異常で、分割を検討するレベルです。読んでてしんどいです。あと単純に拡張性低いし。で、この後仕様変更あって泣きを見るのは、あなた自身かも知れません＾＾

別にこの例が常に正解とは限らないです。ただ、引数が増えてきたら、本当にその構造が「直感的」なのかどうかを考えると良いです。ずっと繰り返してますが、コードは寝ながらでも読めるレベルに簡単化すべきものです。

### 例：1要素で説明できない関数は危険信号
「困難は分割せよ」「1つのことをうまくやる」とかいわれるやつですね。
まあ正直あんまり分けすぎても目線があっち行ったりこっち行ったりでウザいので僕はそこまで強くこだわらずレビューするんですけど、20行以上の関数とかはちょっとコードのにおいは探し始めます。

```typescript
function getUser(id: number): User {
    // ユーザーデータを取得
    const response = fetch(`https://api.example.com/users/${id}`);
    if (!response.ok) {
        throw new Error("Failed to fetch user data");
    }
    const data = response.json();
    // データのバリデーション
    if (!data.name || !data.email) {
        throw new Error("Invalid user data");
    }
    return data
}
```
ユーザーデータを取得し、バリデーションを行い返す関数ですね……
ユーザーを取得するという関数名ですが、中で色々やってますね。このような実装の仕方をしてると困る典型例が、バリデーションを通過しないデータも管理画面用に取得したい！みたいなのが後から出てきたパターンです。

管理画面的には「ユーザーデータを取得する」以上のことは求めてないのに、なんか余計な要素くっついてるせいで邪魔だなぁ…ってなるわけです。で、結局分割するか条件分岐を挟むかの選択を迫られるわけですね。

これもこのgetUserが1要素、つまりユーザーを取得する、以外の余計な要素があるから困ることになったわけですね。

じゃあ、1要素で説明できる関数っていうのはどういうものか？

```typescript
// ユーザーを取得する
function getUserById(id: number): User {
    const data = fetchUserData(user.id);
    if (!response.ok) {
        throw new Error("Failed to fetch user data");
    }
    return response.json();
}
// ユーザーのバリデーションをする
function validateUserData(data: { name?: string; email?: string }) {
    if (!data.name || !data.email) {
        throw new Error("Invalid user data");
    }
}
// バリデーションが必要な時は組み合わせて呼ぶ
function isValidUser(id: number):boolean {
    const user = getUserById(id)
    validateUserData(user)
    return true
}
// 不要な場合はデータだけ取得する
function admin(id: number):User{
  const user = getUserById(id)
  ...
}
```
こんな風に、やってることが分割されてると扱いやすいコードになっていきます。サービス層とリポジトリ層を分けて実装しろみたいなのも、結局こういう話に対応するためだったりしますね。

## N+1問題を意識しているか？

あんまりバックエンドにしか関わりがない話は取り上げるつもりなかったんですけど、N+1問題を指摘されてるレビューが何個かあって驚いたので……。初めてバックエンドにアサインされたとかなら分かるんですが、何年もやってるはずがこれを指摘されてると、普通にやばいなって思います。っていうレベルなんですけど、何故かレビュー指摘受けてる人多い……謎です。

正直N+1問題を意識できない人はバックエンドとか書くべきじゃない気がします。常識だし……

そもそもクエリの発行回数はなるべく抑える(というか基本1回で全部取得する)ことを意識して書くものだと思います。

(とはいえ初学者向けのドキュメントの下書きだし、後で例は探して追記予定…)

## 引数の定義に意思はあるか？
### 例：オブジェクト引数パターン
```typescript
const user = getUserData(userId, true)
```
これを見て、この関数が何をやっているか理解できる人は誰もいないと思います。trueって何だよ。

```typescript
function getUserData(id: number, withDeleted: boolean=false)
```
定義を見て初めて、このtrueって削除済も同時に取得することだったんだ、と理解できる。のだが…

```typescript
const user = getUserData({
  id: userId
  withDeleted: true
})
```
こういう関数だとどうですか？記述量は増えましたけど、何やってるか超わかりやすくないですか？

こんな風に、引数にオブジェクトを渡すパターンをOptions Objectパターンって言います。

[参考：キーワード引数とOptions Objectパターン | TypeScript入門『サバイバルTypeScript』 ](https://typescriptbook.jp/reference/functions/keyword-arguments-and-options-object-pattern)

大体どんな言語でもこういうことができるようになってると思います。Goとかだと工夫が必要だったりした気がしますが。
とにかく呼び出し元から何やってるかを理解できないといけないし、呼び出す側が疑問を持つコードはすべて悪です。

そして、さらなる問題としてこの関数にアクティブユーザーに限定する機能をつけたい！ってなった時により状況は悪化します。

```typescript
function getUserData(id: number, withDeleted: boolean, withInactive: boolean)
```
何も考えないとこういうコードになるんですが……


```typescript
const user = getUserData(userId, true, false)
```
こういう地獄みたいな関数が生まれます。こんなん書いた暁には終身刑に処されます。しかも更に問題となるのはwithInDeletedのみに初期値を与えたいと思った時、後続が必須パラメータだとそれも不可能になるんですよね。

ですが、オブジェクトにしてると……


```typescript
type FetchUserParams = {
  id: number,
  withDeleted?: boolean,
  withInActive: boolean,
}

function getUserData({
  id,
  withDeleted = false,
  withInactive
}:FetchUserParams)

const user = getUserData({id: userId, withInactive: true})
```
後からこういう修正もできるようになります。なので、位置引数の使用はなるべく控えた方がいいです。
まあそもそも引数が多い場合は関数に意味が複数持たされていることを示唆するパターンも多いと思うので、分割も検討すべきですけどね。

### 例：逆に位置引数が正当化される・活かされるパターン

じゃあ位置引数ってダメなのかっていうと別にそういうわけじゃない。まず単純な例として引数が1つであることが明確な場合。

```typescript
const user=getUserById(3)
```
誰が見ても最初に来るのがIDなのはわかりきってるので、いちいち{id:3}とか書くのはダルいので、これが正解。
え、じゃあさっきの例と違うじゃん、削除済ユーザーを除外するオプションをつけたい時どうしたらいいのって話ですね。

```typescript
function getUserById(id:number, withDeleted:boolean)//これはダメ
function getUserById(id:number, option?:{withDeleted?:boolean})//オプションオブジェクト
```
これはよくやる例です。つまり、その関数にとって絶対的なコアとなるものを位置引数とし、オプションを第二引数にすれば、わかりやすく拡張性の高い関数が作れます。これなら、例えばwithInactiveを追加する要件が追加されても…

```typescript
//オプションを追加したい時はここに入れれば良い
type Option={
  withDeleted?:boolean,
  withInactive?:boolean
}

//当然、初期値も柔軟に設定できる
function getUserById(id:number. {
  withDeleted=false,
  witInactive=true,
}:Option){...}

const user = getUserById(5); //当然、IDだけでも呼び出せる
const user = getUserById(5,{withInactive:false}); //コメントなくても意味が理解できる
```
ね、わかりやすいでしょ？

もう1つ例を挙げると、座標のように順番が明確な場合や、順番を逆にしても問題ない場合ですね。

```typescript
//座標なんてx,y,zの順番に決まってますので(1,2,3)みたいな形の方がわかりやすい。
function getCoordinates(x: number, y: number, z: number): string {
  return `x: ${x}, y: ${y}, z: ${z}`;
}

//1+2も2+1も一緒だし、add(3,4)という字面を見ても処理が想像しやすい。
function add(a: number, b: number): number {
  return a + b;
}

function setValue(name: string, value: any){...}//emailというキーにvalueをセットする関数なのは明らか
setValue("email", "aaa@test.com")
```
こういうケースであれば位置引数も正当化されます。

別に位置引数を絶対使うなというわけではないです。引数が2つある関数も全然許容されます。ただ、呼び出す側が処理を想像できない引数だけはダメです。

### 例：そもそも要らない引数を減らせ
(記述予定)

### 例：引数の順番に意思を持つ
(記述予定)

## とにかくイミュータブルを意識する
> 変数に入っている値は後からコロコロ変わると、結局何が入ってるのかわからんからやめて欲しい。言ってることが昨日と違う人、イヤでしょ？ｗ

> [!NOTE]
> mutable: 変更可能な　⇔  immutable : 不変な

### 例1：後から変わる変数
業務でコード書き始めた頃は、イミュータブルってなんだ？何の意味があるんだ？って思ってたんですけど、例えばこういうコード。

```typescript
let memos[] = ["aaa","bbb","ccc"]
~~~~
memos[] = ["あああ", "いいい", "ううう"]
```
memosの中身が後から変わってますね。これ、書いてるときはわかるけど1年後に見た時、ある時点でのmemosにはどっちが入ってるんだ？ってわからなくなることが容易に想像できます。

さすがにこういうコードは業務だと使わないけど、実際に指摘されがちなのは配列のループ操作かな。

```typescript
const memos = [];
for (let i = 0; i < 10; i++) {
  memos.push({ id: i, value: `Memo ${i}` });
}
```
配列にループ処理でpushしている。これ、何がマズいかってループごとにmemos配列の中身が違うことなんですよね。
これぐらいならいいだろ……と思うかもしれませんが、こういうものを許容してしまうと、例えばもっと後でさらに追加しようとしたらどうなりますか？

```typescript
for (let i = 0; i < 10; i++) {
  memos.push({ id: `${i}_new`, value: `後から追加したMemo ${i}` });
}
```
このmemosの内容、追えるビジョン見えますか？影響範囲探すのダルくないっすか？

だから、こういうコードを書きたかったらこうすべきなのです。

```typescript
const memos = Array.from({ length: 10 }, (_, i) => ({ id: i, value: `Memo ${i}` }));
～～～
const newMemos = [
  ...memos, // 既存の要素を展開
  ...Array.from({ length: 10 }, (_, i) => ({
    id: `${i}_new`,
    value: `後から追加したMemo ${i}`,
  })),
];
```
こうすると、宣言した時点で値が確定しているというメンテしやすいコードになりますよね。イミュータブルとは、そういうことだと思っています。要は変数の中身を変えるな、変えたい時は新しく宣言しろという話です。

### 例2：意図せぬ副作用
配列がポインタであるような言語(ほとんどの言語でそうだった気がする？確証はない…)だと、こういうトラブルがあります。

```typescript
const originalArray = [1, 2, 3];
const newArray = originalArray; // 同じ参照を持つ
newArray.push(4); // 配列を直接変更
console.log(originalArray); // [1, 2, 3, 4] <- 副作用が発生
```
こんなんされたらもうコードはメンテできないです。これが副作用を起こすのかどうかっていうのは知識に依存してるのでしょうがないんですが、配列とかインスタンスを操作する際はこういうのを気をつけるべきという話です。PHPだとインスタンスもこういう副作用起こしますよね確か。

## スコープについての理解
> 自分の机の上にあった物をどかしただけで「勝手にいじらないでよ！」とか急に言われてもムカつきません？これ俺の机なんだけどｗ

```typescript
export function displayData(user:User){
  ...
}
```
ユーザーデータを表示する関数ですね。ここに、メモを追加する修正を施したい。


```typescript
export function displayData(user:User, memo:str){...}
```
ある関数を修正したいが、これがexportされる関数なので、これを修正した結果どっかがバグったらダルいなぁ……というケースは多々あると思います。一応文字列検索とかするんですけど、まあわかんないじゃないですか。修正した結果コンパイルエラーが起きると目が当てられません。最悪なのは、これを使った修正が別ブランチで進行してて、マージした時に初めて問題が発覚する、というパターン。こうなると非常にキツい。どっちが悪いかの責任のなすりつけ合いが始まります。

このトラブルをなくすにはどうしたらいいか？という話です。

答えは**影響範囲を限定する**、です。

### Step1：とにかくすべてをprivateにする

> [!NOTE]
> プログラミングの大原則です。

とにかく、コードを書き始める時はあらゆるものをプライベートにしてください。つまり他ファイルに公開しない。

```typescript
class User{
  private id: number; // IDを外部から好き放題変えられない
  function getUserData(){...}
  private setUserData(user:User){...}
}
const user:User
user.id=4; //idを好き勝手いじることを禁止できる。
user.setUserData(dummy); //privateメソッドは外部から呼び出せない
const userData=user.getUserData(); //データの取得だけは許可する
```
クラスの基本的な例ですが、とにかくあらゆるものをprivateにする癖が必須です。こうすれば、少なくともprivateなものは修正を加えてもどっか別のとこがバグることはないので、余計なこと考えなくて済みます。

```typescript
const KEY=1; //このキーって命名が微妙なので変更したい。
export const KEY=1 // 変えたくても変えづらい......修正中に別ブランチで誰かが使っちゃったら...
```
なので、余計なものをexportしないっていうのは大原則で、確定でレビュー指摘が入ります。

 

### Step2：package化
とは言っても、ファイル分割してたら外から読み込むしかないし、特にコンポーネントとかは外部ファイルに公開するのは必須なはずです。

```typescript
export const DomesticPhone=(props)=>{
  return <Phone/>
}
```
例えば、国内の電話番号を表示するようなコンポーネントですね。わざわざPhoneを拡張して定義してるぐらいなので、何らかのページの特殊なコンポーネントのつもりで書いており、他の人が使うことをあんまり想定しないで書いてます。ただ、コンポーネントはファイルの切り出しなので、当然exportしないと使えないです。しかしexportはしたくない……

ここで影響範囲をもうちょっと広げて同じ親ディレクトリ配下に限定したい、というのがpackageです。


```typescript
/**
* @package
**/
export const DomesticPhone=(props)=>{
  return <Phone/>
}

pages
┗UserData
  ┗DomesticPhone.tsx  // package export
　┗UserData.tsx  //当然、同じディレクトリ配下なのでimport可能
 ┗Contact
   ┗Contact.tsx  // DomesticPhoneはUserData配下にしか公開されていないので使用不可
```
このようにすると、意図せぬ場所で勝手にDomesiticPhoneが使われるリスクがなくなります。使おうとすると警告が出ます。

え、じゃあ他の場所で使いたくなったらどうすんの？って思うかもしれませんが、その時に初めて共通コンポーネントディレクトリとかに移動するのを検討すればいいのです。移動した結果の影響範囲は、少なくともUserData配下に限定されるので、仮にバグが出てもその範囲内に収まります。

Goとかだと標準でこういうコンセプトで作られてますね。他の言語でも大体こういう機能はあるはずなので、最大限活用して事故を未然に防ぐことです。

## ジェネリクスを活用する
Array<string>みたいなやつです。ちょっと中級者向け。ジェネリクスが何かは知ってる前提にします。

これも考えることを減らしたり、バグを減らしたりするために最大限活用すべきです。

### 例：テーブルの中身を限定する

例えば、テーブルを描画するためにこういうものがあったとします。

```typescript
//GridColDefは行の情報が入ってるものだと思ってください
const TableRow({column}:{column:GridColDef}){
  return(
      <tr>
        <td>{column.name}</td>
        <td>{column.emairu}</td>
        <td>{column.age}</td>
      </tr>
    )
}
```
名前、メール、年齢を表示するテーブルの行みたいですね。ここで動作確認したら、なんかメールアドレスが表示されない……って思ったらemairuって何だよ！？
…なんてことは多々あると思います。ダルいっすね。バグフィクスって一番嫌いです。防ぐためにはどうしたらいいでしょうか？

GridColDefはMUIで使われてるカラムの型なんですが、これジェネリクスが効くんですよ。

![参考画像](https://github.com/t-moriyama-r/review_doc/blob/25da8725c79e2e598b427d31523104c2a3c1a66c/80c266c7-6e52-4afc-830c-039e7eaea3d9.png)
全然関係ない例なんですが、エディタとかでオンマウスで関数の説明とか出てくるんですけど、例えばこのrefとかいう関数はジェネリクス(<T>の部分)を受け付けてるらしいみたいな情報が得られるわけですね。このジェネリクスが何を示すかはだいたい雰囲気でわかります。わからなかったらドキュメントを見るか、chatGPTに説明させればいいです。

話を戻すと、GridColDefのジェネリクスはカラムの型を指定できます。テーブルのカラムにくっついたジェネリクスとかそれ以外に何があるんだよって話ですね。

```typescript
type Column={
  name: string,
  email: string,
  age: number,
}

const TableRow({column}:{column:GridColDef<Column>}){
  return(
      <tr>
        <td>{column.name}</td>
        <td>{column.emairu}</td> //←Columnにemairuなんてないので構文エラーになる
        <td>{column.age.toUpperCase()}</td> // ←numberに使えない関数を適用してるからエラー
      </tr>
    )
}
```
こんな風に、絶対にエラーになるような行動を抑制してくれるので最大限に活用すべきです。プログラムは頭を使わないで書くものです。

他にもこういうのの恩恵といて、column.って打った段階で候補が出てくるので入力がラクになったり、あるいは存在するのかどうかわからないnullableな変数なのかどうかも全部コードが教えてくれます。活用できるジェネリクスは活用して、余計な可能性を排除すれば、やっぱり考えることが減ります。

フロント・バックエンドにとって都合のいい形で取り扱う

これについてはかなり重要な話ではあるものの、難易度は高いかもです。(が、呼んで吸収できない人は即戦力たり得ないです…)

基本的に自分のスコープ内では自分にとって一番都合のいいデータ型で書くべきです。つまり、あるロジック内では自分にとって最も都合のいいデータ型を定義してコードを書くべきです。

例：
リファクタサンプル：型で論理構造を表現する 

で、頻出なのがバックエンドで2つに分かれているパラメータがフロントでは1つのパラメータでまとめられるパターンですね。例えば･･････
```typescript
// 審査結果およびデータの削除状況を示す
interface Response{
  status: Status; // 1: Pending, 2:Accepted, 3: Refused
  deleted: boolean;
}
// フロントでは「審査中」「審査完了」「審査NG」「削除済み」のステータスを表示する
```
つまり、deletedは何においても最優先で表示されるべきで、deletedがfalseであればStatusに応じたステータスを表示する、というバッジコンポーネントを考えましょう。
```typescript
// やりがちな例
interface Props{
  status: Response // 上記のレスポンスを流用
}
const Badge=({status}:Props)=>{
  return (<Badge>
    {status.deleted?"削除済み":StatusMsg[status.status]}
  </Badge>)
}
```
テキトーに書きましたが、普通に書くとこういうコードになるんじゃないですかね。が、単にステータスに応じたバッジを表示するだけなのに妙に複雑なことというか、あんまり本質的でないことをしているコードだなって思いませんか。というか、そう思って欲しいです。

何故このようなことになっているのかというと、本来コンポーネントにとって関心のないバックエンドからのレスポンスインターフェースの知識にコンポーネントの動作が左右されているというのが問題なのです。
バッジコンポーネント目線で欲しいのは表示すべきステータスそのものですし、なんならフロント側で表示すべきステータスがこの2値から計算される値なのであれば、フロント目線ではプロパティは1つでいいわけです。

つまり･･････
```typescript
// こんな風にAPIレイヤを定義してたとして...
async function getStatus(id:number): Response{
  const response = await axios.get(URL, {id})
  return response.data
}
// APIレイヤ時点でフロントにとって都合のいいデータに組み替えた方がいい
interface FrontStatus{
  status: Status; // 1: Pending, 2:Accepted, 3: Refused, 4: Deleted
}
async function getStatus(id:number): FrontStatus{
  const response = await axios.get(URL, {id})
  return {
    status: !response.data.deleted ? response.data.status : Status.Deleted
  }
}
```
APIレイヤ時点でこのようなインターフェース(FrontStatus)を返してやれば論理構造が正確に表現できており、コンポーネントはただこれを利用すればいいだけ、というのがわかりますよね。

```typescript
// ベストな例
interface Props{
  status: Status
}
// 余計なstatusの分岐が消える
const Badge=({status}:Props)=>{
  return (<Badge>
    {StatusMsg[status]}
  </Badge>)
}
```
最初から最後まで一貫して余計なことを考えずに済むコード、というのが意識されたコードになったかなと思います。こういう例って案外多く存在すると思います。

バックとフロントのインターフェースの差異の吸収、っていうのが頻出パターンだと思いますが、バック・フロントそれぞれの内部でもこのようなメソッドに入ってきた時点で2値を1値に確定できる論理構造が表現できる、という例は存在しうると思います。こういうのをどのように簡単化していくか、というのを意識できると優秀なコーダーたりえると思います。
