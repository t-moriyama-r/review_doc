
# 目次
- [交差型とユニオン型](#交差型とユニオン型)
  - [応用例：受け取る側にとって最も都合のいい形で定義する](#応用例受け取る側にとって最も都合のいい形で定義する)
    - [都合のいいインターフェースを定義する](#都合のいいインターフェースを定義する)
- [副作用フックの従属関係はきちんと考える](#副作用フックの従属関係はきちんと考える)

>[!TIP]
> neverとかを扱えるとより良いです。(例は後で追記するかも...)

# 交差型とユニオン型

例えば、あんまりいい例じゃない気がするが**APIのレスポンス**をインターフェースに落とし込むことにする。

レスポンスとしてステータスコード・レスポンスデータ・エラー内容が考えられる。

```typescript
interface Response{
  status: number
  data: object | null
  error: object | null
}
```

例えばこんな感じかな。何も考えなければこういうインターフェースになるが、もう一歩進んだ型定義としてこのようなものが考えられる。

```typescript
type Response = {
  status: Status
  data: object
} | {
  status: Status
  error: object
}
```

こうすれば、dataとerrorが両立しえないことを**型が教えてくれる。**
恩恵としては、**余計なnull判定が不要になる**ことが挙げられる。

ユニオン型以外にも交差型を使ってみると

```typescript
type Response = {
  status: Status
} & ({
  data: object
} | {
  error: object
})
```

若干読みづらい感はありますが、このように型で論理構造を限定していくと、いわゆる**コードが教えてくれる**という状態になっていく。

statusについても制限を加えたければ、

```typescript
type Response = {
  status: SuccessStatus
  data: object
} | {
  status: ErrorStatus
  error: object
}
```

こんな風に定義できると、不適な組み合わせをそもそも排除できるので、**コーダーは余計な事を考えなくて済む。**
ということを考えながらインターフェースを定義できると、コードのノイズが減るかなと思います。



まあ、必ずしも型をガチガチに固める必要があるのかどうかはケースバイケースですが、このような思考をワンクッション挟む癖がついてると、コードの品質が上がっていくんじゃないかなと考えています。この例も採用するかどうかはプロジェクトによると思いますけど、考え方としては大事だと思ってます。

## 応用例：受け取る側にとって最も都合のいい形で定義する

例えば、下記のようなレスポンスに応じてダイアログを出しわけるコンポーネントを考える。

```typescript
interface Response{
  a: Code
  b: Code
}

type Code= "1" | "2" | "3"
```

AとBというデータがあり、1,2,3というフラグが同時に渡される。1は問題なし、2は警告を出す、3はエラー処理のイメージ。

この1,2,3というのに別名をつけたい人は定義したものとして読み替えてください。
(このようなコード値はバッドプラクティスと言われたりしますが、慣習として採用される例も多いイメージ...)


*悪いコード例*

```typescript
type Key = keyof Response; // "a" | "b"
interface DialogState{
  open: bool //ダイアログの開閉状態
  warn: Key [] | null
  error: Key [] | null
}

export const SamplePage = () => {
  // ダイアログ表示制御関連
  const [dialogState, setDialogState] = useState<DialogState | null>(null);
  
  const onDuplicateCheck = async (value:FormValue) => {
    const response = await api(value);
    if (
      response.a=== "1" &&
      response.b=== "1"
    ) {
      //重複なしの場合はそのまま遷移
      next()
      return;
    }
    //エラーが含まれる場合はエラーダイアログを表示する
    if (
      response.a=== "3" ||
      response.b=== "3"
    ) {
      setDialogState({
        open: true,
        warn: null, // エラーが優先されるので、警告はnullでOK
        error: generateErrorArray(response.a, response.b)//いい感じに配列化する関数があるとする
      });
      return;
    }
    // 警告の場合
    setDialogState({
      open: true,
      warn: generateWarnArray(response.a, response.b)
      error: null,
    });
  };

  return (
    <>
      { dialogState.warn !== null && (
        <Dialog
          open={dialogState.open}
          meg={hogehoge}
          onClose={closeDialog}
        />
      )}
      { dialogState.error !== null && (
        <Dialog
          open={dialogState.open}
          msg={fugafuga}
          onClose={closeDialog}
        />
      )}
    </>
  );
};
```

愚直に書くとこのようなコードになるが、このコードの問題点を考えてみます。



まず、コンポーネント内のロジックが**なーーんか妙に長い気がする**、というコードのにおいを感じ取れるか、です。



大事な考え方として、**フロントのデータ構造はバックエンドに準拠する必要は一切ない**、ということです。
この場合、バックエンドからのレスポンスをそのまま受け取っていますが、この時点でロジックが短くなるようにデータ構造を整理しておけないか？という思考のプロセスを挟んでみます。

```typescript
const response = await api(value);
```

つまり、この時点で**もっと都合のいい形でデータくれよ**、っていうのをやっておけば良いんじゃないか？ということで、ここを関数化してこのあたりのロジックはまとめられそうではあります。
ただ、結局この問題の本質はフロントにとって**都合のいいインターフェース(論理構造を具体化した型)が存在しないことにある**のを突き止めなければなりません。

### 都合のいいインターフェースを定義する

では具体的に落とし込んでいくと、上記コンポーネントにとってダイアログ表示ロジックにとって都合のいいインターフェースを考える。

```typescript
interface Output{
  result: "success" | "warning" | "error"
  subject: ("a" | "b")[] // (keyof Response)[]
}
```

まず、ダイアログを表示するのに本来必要なインターフェースってこんな感じだと思うんですよね。
出し分けに必要な「成功/警告/エラー」がわかればいいし、それはバックエンドのものをそのまま使う必要もない。で、aとbという項目のどっち、もしくは両方で起きてるかを配列で受け取ればいい。もちろんレスポンスとしては`{warn:[a], error[b]}`という形で返ってくる可能性はあるが、両方を表示する要件がなければコンポーネント目線では両方は要らないのです。
openという開閉状態も、実は別に持っておく必要がなく、状態がなければ表示しない、状態によって出すべきダイアログは排他的なので中を見て決めればOK、よって開閉状態は別に保つ必要ないこともわかります。

で、更に一歩進んで、警告・エラーがない場合、つまり`”success”`を受け取っている場合、subjectに値が入るワケがないので、これもインターフェースに落とし込みたい。

```typescript
type Output={
  result: "success"
} | {
  result: "warning" | "error"
  subject: ("a" | "b")[]
}
```

こっちが正しい論理構造であるといえる。

この状態で、レスポンスを関数なりスキーマなり何らかの手段でこの`Output`型で受け取れるようにすると…

```typescript
type DialogState = Output;

export const SamplePage = () => {
  // ダイアログ表示制御関連
  const [dialogState, setDialogState] = useState<DialogState | null>(null);
  
  const onDuplicateCheck = async (value:FormValue) => {
    const output: Output= convert(await api(value));
    if (output.result === "sucess") {
      //重複なしの場合はそのまま遷移
      next()
      return;
    }
    setDialogState(output);
  };

  return (
    <>
      <Dialog
        open={dialogState.result == "warning"}
        meg={hogehoge}
        onClose={closeDialog}
      />
      <Dialog
        open={dialogState.result == "success"}
        msg={fugafuga}
        onClose={closeDialog}
      />
    </>
  );
};
```

めっちゃシンプルになりましたね。(笑)

これで終わってもいいのですが、もう一段階リファクタが可能なポイントがあります。どこだか分かりますかね？



```typescript
type DialogState = Output;
```

これ、何のためにわざとらしく書いてたのかというと、実際の論理構造の理想は実は下記になります。


```typescript
type DialogState = Exclude<Output,{result: "success"}>;
```

ダイアログの表示にsuccess状態ってこの例だと関係ないですよね。なので、`Exclude`で除外可能です。例がシンプルなのでここで恩恵があるかっていうと特にないんですが、このように正確な型が定義できるようになると、よりコードの品質が上がり、**シンプルでメンテしやすいコード**が生み出せるようになっていくんじゃないかなと思います。



# 副作用フックの従属関係はきちんと考える
propsで会社IDを受け取り、その支店一覧を共通化されたカスタムフックから取得し、それが複数あればチェーン店、1つしかなければ個人店とする。
そして、その判定結果をフォームの値として埋め込む、というのを考える。 

まず、支店の一覧を取得する。対象としてcompanyIdがあると仮定すると、こんな感じで取得できるのが普通。Stateということがわかりやすくなるよう、ローディング状態も取得できるものとする。今回は別に使わないが。

```typescript
const [branchList, branchLoading] = useBranchList(companyId) // 支店一覧
```
そして、その変更を副作用として受け取る副作用を考える。

```typescript
useEffect(()=>{
  const isFranchise = branchList.length > 1
  setValue("category", isFranchise?"チェーン店":"個人店")
},[branchList, setValue])
```

つまり、まとめると…


```typescript
const [branchList, branchLoading] = useBranchList(companyId) // 支店一覧
useEffect(()=>{
  const isFranchise = branchList.length > 1 // 取得した支店が個人店なのかどうか判定
  setValue("category", isFranchise?"チェーン店":"個人店")
},[branchList, setValue])
```
こういうコードができあがる。


**どう思いますか？**
 

って聞いてる時点で改善案があるわけですが(笑)、まあ安直なリファクタ案としてはsetValue部分というかisFranchiseに詰め直すのが不要ではとかそういう話もありますが、**もっと本質的なこと**があります。

 

ここで、この要件を満たすために組んだロジックを日本語化してみます。

> propsで会社IDを受け取り、その支店一覧を共通化されたカスタムフックから取得し、それが複数あればチェーン店、1つしかなければ個人店とする。
> そして、その判定結果をフォームの値として埋め込む、というのを考える。

- 受け取った会社IDから支店一覧を取得する
- 支店一覧の長さを見て、1つなら個人店・複数ならチェーン店
- その値をフォームの値にセットする

 

うまいこと説明できないので、結論から書きます。

>  **その値**をフォームの値にセットする

この部分がミソで、副作用でやるべきこと、つまりフックとして監視すべきなのって「その値」でしかないということです。
どういうことかというと、この「その値」というのは、支店の長さから算出された判別値であり、フォームの値の更新は**支店一覧ではなく判別値の方に依存している**、ということです。

 

これをコードに落とし込むと、正しくはこうあるべき、です。

```typescript
// before
const [branchList, branchLoading] = useBranchList(companyId) // 支店一覧

useEffect(()=>{
  const isFranchise = branchList.length > 1 // 取得した支店が個人店なのかどうか判定
  setValue("category", isFranchise?"チェーン店":"個人店")
},[branchList, setValue])
```

```typescript
// after
const [branchList, branchLoading] = useBranchList(companyId) // 支店一覧
const category = branchList.length > 1 ? "チェーン店" : "個人店"

useEffect(()=>{
  setValue("category", category)
},[category, setValue])
```

端的にいうと、 **従属関係が整理された上で副作用は管理されているか？** というのは意識して書くべきかなと思います。

useEffect内の処理が妙に長い場合、こういうのを**コードのにおい**として整理できるかを見てみるとシンプルなコードになっていくかと思います。
