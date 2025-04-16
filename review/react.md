# よくあるレビュー指摘点【React･フロント編】

> [親記事:よくあるレビュー指摘点](../README.md)<br>
>   ┗[コード編1(初学者向け)](./code_1.md)<br>
>   ┗[コード編2](./code_2.md)<br>
>   ┗[イディオム編(頻出ロジック)](./idiom.md)<br>
>   ┗[プルリク編](./pull.md)<br>
>   ┗React・フロント編←ここ<br>
>   ┗[レベル別Gitフロー](./git.md)<br>

> [!TIP]
> Reactはマトモに書ける人が少ない(っていうか**一介の作業者で「普通に」書ける人を僕は見たことない**)のでこのレベルの指摘は多々します。逆にいうと、これができるだけで単価って割と上げられそうな気がします。<br>
> 体感ですが、このレベルの指摘が不要(=人に教えられる)になれば単価80～85万いける水準だと思います。<br>
> Reactは単なるライブラリだし公式ドキュメントを読むだけで割とマスターできます。<br>
> 逆に、定期的に読み直して理解を深める姿勢がない人のコードってほんとにわかりやすいです……


【ちょっと高度な内容は個別のメモにまとめた】
リファクタサンプル：型で論理構造を表現する 
リファクタサンプル：副作用フックの従属関係はきちんと考える 
二者択一は型にやらせる 

> [!NOTE]
> 【目次】<br>
> - [CSS(SCSS)](#CSSSCSS)<br>
> - [コンポーネント(書き方)](#コンポーネント書き方)<br>
> - [コンポーネント(ロジック)](#コンポーネントロジック)<br>
> - [状態管理関連](#状態管理関連)<br>
> - [useEffect(副作用)](#useEffect副作用)<br>
> - [フック関連](#フック関連)<br>
> - [ReactHookForm](#ReactHookForm)<br>
> - [Key](#Key)<br>
> - [デバッグ関連](#デバッグ関連)<br>
> - [実装パターン](#実装パターン)<br>

## CSS(SCSS)
### importantの使用禁止
!importantは**使わなくてもなんとかなるように頑張って**下さい。クラスなどを詳細に指定することで優先順位を上げたりもできます。**いかに使わないで実装するかが腕の見せ所**です。
そもそも、なぜ使用しているか、というコメントがないと**確実にリジェクト**します。

参考サイト：[【初めてのHTML/CSS】CSS（スタイルシート）の優先順位を覚えておこう！](https://www.support.makeshop.jp/design/?p=10390 )

何故使ってはいけないのかというと、単純に**影響範囲が不透明になるか**らですね。(例が必要そうなら後で書きます...)

## タグはちゃんと指定する
パフォーマンスもそうですし、読み手に優しくなるのでつけてほしいです。あと、想定していないとこにスタイルが当たるという事故も防げます。

```scss
// これだけ見ても、何のスタイルかよくわからん
.deleted{
  color: red;
}

// これなら、何のスタイルか直感的に分かる
tr.deleted{
  color: red;
}

// 複数指定したい場合も、なるべくちゃんと指定すること。(common的なスタイルでない限り)
tr, td{
  &.deleted{
    color: red;
  }
}

// こっちでも可
tr.deleted, td.deleted {
  color: red;
}
```

## わかりやすく階層をネストする
これも一般的かどうかは不明で僕個人の単純な好み、なんですけど、ちゃんとネストして書いてくれた方が構造が頭に入ってきやすくて好きです。

```scss
// こう書くより...
table{
  ...
}
tbody{
  ...
}
tr{
  ...
}
tr.deleted{
  ...
}

// こう書いて欲しい(ネストがあまりにも深くなるような指定はそもそもが問題ある気がする)
table{
  ...
  tbody{
    ...
    tr{
      ...
      &.deleted{
        ...
      }
    }
  }
}
```
> [!NOTE]
> これについては、厳密には空気を読むみたいなとこあって、ある程度センテンスで分けたりもするかもです。ただ、全てを並列に書かれても正直見づらいので僕は好きじゃないです。

## コンポーネント(書き方)
### 飾り文字は使わない
【タイトル】とか＜タイトル＞みたいな装飾は、コンポーネントやCSSでやるべきで、プレーンなHTMLではなるべく避けるべきです。統一的なUIを提供するのが難しくなります。

### 絶対パスはなるべく避ける
同じ親ディレクトリ配下のコンポーネント移動のリンクは相対パスで書く。これは、大きな意味単位の親コンポーネントから見てどの位置に居るかというのはディレクトリのルートとなるページから相対的に決まるのであって、webアプリのルートからのパスを子コンポーネントが知っているというのは本来持つ必要のない知識の漏洩でしかない。

逆に絶対パスが容認されるのは、全く別のディレクトリへのリンク、例えばその日のニュース記事一覧からマイページに飛ぶといったような、管理している親ページが全く別種のものの場合は、webアプリのルートからのパスで指定するというのは問題ないことだと思う。

(文章で書くとちと小難しい表現になったので、また今度具体例を追記します)

### コンポーネントの上/下にあるべきものを意識する
> [!INFO]
> これは中級者向けかも知れないです。

コンポーネントを見た時に、最も関心の強いものが **「構造について」** なので、コンポーネントの構造を表すもの、たとえば`props`インターフェースや定数の定義などはコンポーネントの上にあるべきで、テーブルのカラムやCSSなどの具体性を伴ったものはコンポーネントの構造には関係ないためコンポーネントの下に追いやってしまうべき。

```typescript
// Bad Example
const Example=()=>{
  const navigate = useNavigate(); // React Router
  const colmuns:: GridColDef<Item>[] =()=>[
   {
    field: 'column1',
    headerName: '項目1',
    sortable: true,
  },
   {
    field: 'column2',
    headerName: '項目2',
    sortable: true,
  },
  {
    field: 'button',
    headerName: ' ',
    renderCell: (params) => (
        <Button
          label="詳細"
          onClick={() => navigate(`./${params.row.id}`)}
        />
    ),
  },
  ]
  return <Table columns={columns}/>
}
```
こういうコードを書く人って結構います。`navigate`がカスタムフックなので、それをリンクに使うためにコンポーネント内に書きたくなる気持ちもわかります。でも、カラムの中がどうとかって**全体の構造よりも先に理解することではない**わけですね。

```typescript
// まずこのコンポーネント全体の構造がわかる
const Example=()=>{
  const navigate = useNavigate(); // React Router
  return <Table columns={columns(navigate)}/>
}

// カラムの内容が知りたくなれば下を見ればいい
const colmuns: GridColDef<Item>[] =(navigate: NavigateFunction)=>[
   {
    field: 'column1',
    headerName: '項目1',
    sortable: true,
  },
   {
    field: 'column2',
    headerName: '項目2',
    sortable: true,
  },
  {
    field: 'button',
    headerName: ' ',
    renderCell: (params) => (
        <Button
          label="詳細"
          onClick={() => navigate(`./${params.row.id}`)}
        />
    ),
  },
  ]
```
`columns`を生成をカスタムフックにすればcolumns内でも呼び出せますが、これでもシンプルでいいでしょう。
他にもCSSの類などはコンポーネントについて**真っ先に知る必要のあるものではない**ため、このように下に分けて定義するのが望ましいです。

## コンポーネント(ロジック)
### Propsは可能な限り減らす

propsで渡す必要のないものは渡さないというか、とにかくpropsは減らす意識を持つとコンポーネント分割がうまく行ったりします。
よくある例が、子コンポーネントでしか使わないデータを親データで取得しちゃう例。


```typescript
interface Props{
  userId: number;
  articleList: Article[];
}

const ArticleList=({userId, articleList}:Props)=>{
  return (<>
    <p>ID:{userId}</p>
    {articleList.map((article, index)=><Article key={index} article={article}/>)}
  </>)
}
```
こういうコンポーネントもよくあると思うんですけど、この`articleList`ってどう考えても`userId`から取れるもので、propsで受け渡すんじゃなくて**このコンポーネント内で取得しちゃった方が良くないですか？**
そうしたらpropsが1つ減らせるし、親コンポーネントにとって不要なロジックを1つ掃除することができます。

```typescript
// 本来こうあるべき
interface Props{
  userId: number;
}

const ArticleList=({userId}:Props)=>{
  const [articleList, setArticleList]=useState<Article[]>([])
  useEffect(()=>{
    const response= await fetchArticleList(userId)
    setArticleList(response.data)
  },[userId])
  return (<>
    <p>ID:{userId}</p>
    {articleList.map((article, index)=><Article key={index} article={article}/>)}
  </>)
}
```
記事の取得というロジックがこのコンポーネント内に隠蔽され、親コンポーネントから関心が切り離されます。こちらの方がpropsが減り、本当に必要なものだけを受け渡しているので分かりやすいコードになります。 

## Propsのnullableを減らす

よくある例として…

```typescript
interface Props{
  user?:User
}

const UserPage=({user}:Props)=>{
  if (user==null) return null;
  return(<div>{user.name}</div>)
}
```
こういうコンポーネント、`User`がnullableな必要ないので、最初から`User`型しか受け付けないようにすべきです。いやそんなん当たり前じゃんって思うんですけど、なんとなくてnullableにして無益な分岐を発生させるケース、ほんとに多いです。

この辺の話はReactに限らないです。→
【コード編1】よくあるレビュー指摘点(wip) | nullableについて 

### 100行を超えるコンポーネントは分割を意識
個人的に**100行を超えるコンポーネント**は**ロジックが肥大化してる**か**責務が大きくなりすぎてる**かしてるので、リファクタする余地がありそうと思ってレビューで見ます。言っても伝わらない人多いのでこのへんはレビューしてても妥協しがちですが……メンテする人のことを考えたら、**短く直感的に何やってるか分かるコンポーネント**というのを意識すべきです。

```typescript
const UserProfile = () => {
    const [user, setUser] = useState({
        name: "Alice",
        email: "alice@example.com",
        bio: "Frontend developer",
    });
    const [isEditing, setIsEditing] = useState(false);
    const [newBio, setNewBio] = useState(user.bio);
    const handleEditClick = () => setIsEditing(true);
    const handleSaveClick = () => {
        setUser((prev) => ({ ...prev, bio: newBio }));
        setIsEditing(false);
    };

    return (
        <div className="user-profile">
            <h2>{user.name}</h2>
            <p>Email: {user.email}</p>
            <div className="bio-section">
                <h3>Bio</h3>
                {isEditing ? (
                    <div>
                        <textarea value={newBio} onChange={(e) => setNewBio(e.target.value)} />
                        <button onClick={handleSaveClick}>Save</button>
                    </div>
                ) : (
                    <p>{user.bio}</p>
                )}
                <button onClick={handleEditClick}>Edit</button>
            </div>
        </div>
    );
};
```
AIに吐き出させたコンポーネントです。ユーザープロフィールを表示するコンポーネントっぽいですね。実際はもっとだらだら書かれてるはずで、そうなると見通しが悪くて何やってんのかよくわかんないコードになります。

なので、もう少し意味単位でコンポーネントを分割すること考えましょう。

```typescript
const UserProfile = () => {
    const [user, setUser] = useState({
        name: "Alice",
        email: "alice@example.com",
        bio: "Frontend developer",
    });
    const updateBio = (newBio: string) => setUser((prev) => ({ ...prev, bio: newBio }));
    return (
        <div className="user-profile">
            <UserInfo name={user.name} email={user.email} />
            <BioSection bio={user.bio} onUpdateBio={updateBio} />
        </div>
    );
};
```
ユーザー情報と、プロフィール部分が分離しました。なんかそういう画面なんだなっていうのが直感的に分かるようになりました。

```typescript
const UserInfo = ({ name, email }: { name: string; email: string }) => (
    <div>
        <h2>{name}</h2>
        <p>Email: {email}</p>
    </div>
);
```
```typescript
const BioSection = ({
    bio,
    onUpdateBio,
}: {
    bio: string;
    onUpdateBio: (newBio: string) => void;
}) => {
    const [isEditing, setIsEditing] = useState(false);
    const [newBio, setNewBio] = useState(bio);
    const handleSave = () => {
        onUpdateBio(newBio);
        setIsEditing(false);
    };

    return (
        <div className="bio-section">
            <h3>Bio</h3>
            {isEditing ? (
                <div>
                    <textarea value={newBio} onChange={(e) => setNewBio(e.target.value)} />
                    <button onClick={handleSave}>Save</button>
                </div>
            ) : (
                <p>{bio}</p>
            )}
            <button onClick={() => setIsEditing(true)}>Edit</button>
        </div>
    );
};
```
コンポーネントの関心がより細かいパーツに行ったことで、このコンポーネントが何をしようとしているのか、そして各種ロジックの影響範囲が狭まりコードの認知が易しくなっていきますね。
このBioは、どうやら編集ボタンがあって書き換え可能らしいということがわかるし、親コンポーネントそのものはこの機能に関心がなく、**レイアウトのみに関心を持つことができる。** という風に、**関心を分離しメンテナンスしやすいコード**になっていきます。

### 複雑なロジックはコンポーネント内に隠蔽する
分割するのがいいって言われても分割しすぎてもなぁ感あるし、よくわかんないって人多いと思うので個人的な目安の話をすると、**ロジックが過集中してるところは分割対象**だと思っています。

例えばめっちゃ簡単な例で言うと、押したらAPIからデータを取得するという機能があるボタンがあるコンポーネントとか。

```typescript
const ParentComponent = () => {
    const [loading, setLoading] = useState(false);
    const [data, setData] = useState<string | null>(null);
    const fetchData = async () => {
        setLoading(true);
        try {
            const response = await fetch("https://api.example.com/data");
            const result = await response.json();
            setData(result.message);
        } catch (error) {
            console.error("データ取得に失敗しました", error);
        } finally {
            setLoading(false);
        }
    };

    return (
        <div>
            <button onClick={fetchData} disabled={loading}>
                {loading ? "Loading..." : "Fetch Data"}
            </button>
            {data && <p>取得したデータ: {data}</p>}
        </div>
    );
};
```
このコンポーネントのロジック、**大部分がボタンそのものに関わる**もので、親コンポーネントにとっては正直あんま関係ないので余計なものがあることで認知を難しくしてますね。ここで、ボタンだけをコンポーネントに切り出すとどうなるでしょうか。

```typescript
interface Props {
    onDataFetched: (data: string) => void;
}

const FetchButton = ({ onDataFetched }:Props) => {
    const [loading, setLoading] = useState(false);
    const fetchData = async () => {
        setLoading(true);
        try {
            const response = await fetch("https://api.example.com/data");
            const result = await response.json();
            onDataFetched(result.message);
        } catch (error) {
            console.error("データ取得に失敗しました", error);
        } finally {
            setLoading(false);
        }
    };
    return (
        <button onClick={fetchData} disabled={loading}>
            {loading ? "Loading..." : "Fetch Data"}
        </button>
    );
};
```
こんな風に、データ取得に関わるロジックはごっそりこっちに持ってこれますね。

```typescript
const ParentComponent = () => {
    const [data, setData] = useState<string | null>(null);
    return (
        <div>
            <FetchButton onDataFetched={(value)=>setData(value)} />
            {data && <p>取得したデータ: {data}</p>}
        </div>
    );
};
```
すると、親コンポーネントがこうなります。`FetchButton`について、**なんかよーわからんけど押したらデータ持ってきてくれるコンポーネント**、という扱いになって、考えることが減ってラクですね。何よりこのコンポーネントのStateが1つ減るので見通しも良くなります。

つまり、コンポーネント分割は意味単位というより、**ロジックが集中してる箇所を分割する**という意識でやった方が美しくなると思います。

### 子コンポーネントで使わないPropsはレスト構文にまとめる
これもよくある例です。

```typescript
interface ButtonProps{
  onClick:()=>void;
  color: Color;
}

interface Props{
  message: string;
}

export const ExampleComponent({message, ...rest}:Props & ButtonProps)=>{
  return(<>
    {message}
    <SubmitButton {...rest}/>
  </>)
}
```
子コンポーネントでしか使わない、たらい回しされてるものはこうやってまとめて渡せます。(この例だとありがたみが伝わらないかもなのでもうちょっといい例見つけたら更新します)

### コールバックにSetStateを直接渡さない
昔これやってたんですが、割とこれお行儀が悪いらしい。

```typescript
interface Props{
  onClick: Dispatch<SetStateAction<boolean>>
}
```
クリックすると親から渡されたセッターに値をセットしてリレンダリングを発火させられるPropsっぽいですが、この子コンポーネントにとってはその知識は不要で関係ない。
また、もし何らかの事情でbooleanではないStateになった場合、全部書き換えなきゃいけない側面もある。

```typescript
interface Props{
  onClick: (value: boolean)=> void
}
```
ので、親の実装に依存しないという面で、このように無毒化するのが良いコードということなんだろうと思う。

## 状態管理関連
> [!INFO]
> SetStateを実行する・もしくはpropsの値が変更されると、コンポーネントが上から再実行される。**それだけの話でリレンダリングは説明できる**んですが、そういうことが理解できていないコードが割とある。

これも初心者あるあるなのですがStateを**ただの変数と思ってないか？** という話です。

たまに見かけるものとして、

```typescript
const [count]=useState<number>()
```
というコードを見かけます。セッターがないStateとかありえないんですが、何人かこういうの出してくる人がいます。

Stateとは **「更新時にリレンダリングを引き起こす(必要がある)もの」** なので、数が多ければ多いほどレンダリング負荷が増大していきます。なので、**妙にStateが多いコンポーネントはそれだけでにおうな**、って思って見ます。

### Stateにジェネリクスをつける

ついてた方が明らかに分かりやすいのでつけた方がいいと思ってます。

```typescript
// 見た目の認知が厳しいし、もしinitialの型が変わったら静かに崩壊するリスク
const [user, setUser]=useState(initial)

// 見てすぐわかるし、stringであることを前提としていることがわかる
const [user, setUser]=useState<string>(initial)

// これも取るべき状態が分かりやすい。
const [user, setUSer]=useState<string|null>(null)

// ちなみにこれを許容するかは宗教問題。見りゃ分かる派と、undefined書いて欲しい派がある。
const [user, setUser]=userState<string>()
```

### 既存stateから計算可能な値の区別をつける
例えばこういうコード。

```typescript
const Userdata()=>{
  const [user, setUser]=useState<User>(initialUser)
  const [email, setEmail]=useState<string>(initialUser.email)
  const [name, setName]=useState<string>(initialUser.name)

  const onClick(newUser)=>{
    setUser(newUser)]
    setEmail(newUser.email)
    setName(newUser.name)
  }
}
```
こういうコード出してくる人ほんと多いんですよ。**stateはただの変数じゃない**です。
セッターが呼び出されるとコンポーネントが上から再実行されるという原則に立ち返ると、

```typescript
const Userdata()=>{
  const [user, setUser]=useState<User>(initialUser)
  const email = user.email
  const name = user.name
  const onClick(newUser)=>{
    setUser(newUser)]
  }
}
```
これでいいんです。`email`も`name`も可変の値なので状態かな？って思っちゃうから先程のようなコードがよく出てくるのですが、stateがトリガーとなって画面の再描画が走るだけなのでそれに付随した計算(つまり`user`から`email`を抜き出すような)も再計算され、以後の処理に使われます。

まあ……僕もこれ理解するのに結構時間かかった記憶あるんでアレなんですけど、何年もやってる人がこういうコードを出してきたりします。

### 依存関係にあるStateはまとめる
```typescript
const Example=()=>{
  const [name,setName]=useState<string>()
  const [email,setEmail]=useState<string>()
  const [role,setRole]=useState<Role>()
}
```
こういうコードも散見されるんですが、**状態が多いコンポーネントは何かにおうな**と思いながら見ています。果たしてこれらが独立したパラメータである必然性はあるのでしょうか？

```typescript
type User={
  name: string;
  email: string;
  role: Role;
}

const Example=()=>{
  const [user,setUser]=useState<User>()
}
```
このように1つにまとめても何も問題ない場合が非常に多いです。**とにかくStateは減らせ**ということです。

### レンダリングに必要な変数の区別
> [!INFO]
> useStateとuseRefの違いが理解・説明できますか？

```typescript
// Bad Example
const Example=()=>{
  [isCalled, setIsCalled]=useState<boolean>(false);

  useEffect(()=>{
    await api();
    setIsCalled(false)
  },[])

  const handleClick= async()=>{
    if(!isCalled) return;
    ......
  }
  ......
}
```
こういう例だと、`isCalled`を呼び出したかどうかってレンダリング結果に影響を与えないので**`useRef`を使うのが正解**です。

ちなみに、こういうケースで**平然とState使う人ってマジで多いです。** 歴が半年とかならまだわかるんですけどね、`useRef`ってドキュメント見てないとあんま遭遇しないので……

(補足)
```typescript
//下記2つは全く別の用途なので注意っていう話はv19でなくなったらしい
const ref = useRef()
const ref = useRef(null)
```
上記の話をついでに触れようと思ったんですけど、v19から**すべてのRefObjectがミュータブルになったらしい。** へえ。

React v19の話：
React v19 

## useEffect(副作用)
### useEffectの使用は「なるべく避ける」
あくまでも**副作用というのは仕方なく使う**というつもりで書く。基本的に使うのを避ける意識がないと、コードは取っ散らかっていきます。

やたらめったら`useEffect`を使って変数同士の関連を定義しようとすると、結局**やってることはたらい回しでしかなくなる**ので非常に見通しが悪いコードになります。そもそもそのようなコードである必然性も低い場合がほとんどなので、これもコードとしては「なんかにおう」感がします。

また、これは多少流派があると思いますが`useEffect`を利用最小限にするという思想でいえば、例えばフォームの値に連動してなにか別の値が変わる際、`useEffect`ではなく`onChange`で手続き的に処理をする方がベターでもあります。

```typescript
//before
const toggle=watch("toggle")

useEffect(()=>{
  setValue("other_field","new value")
},[toggle])

return(
  <RadioButton
    name="toggle"
    values={values}
  />
)

//after
return(
  <RadioButton
    name="toggle"
    values={values}
    onChange={()=>{
      setValue("other_field","new value")
    }}}
  />
)
```
`useEffect`(副作用)のデメリットとしては、**宣言した箇所と実際のトリガーが物理的に離れる**ということです。
上記の例でいうと、`toggle`の変更をキャッチして新しい値をセットしていますが、これを修正する人がまずこれを見て **「`toggle`ってどこだ？」** と探すところから始めなければなりません。実際のコードはこんなに短くないし、もし多段的に影響を与える可能性があったら…とか考えると、依存関係を探すのも大変です。

一方、`onChange`内に記述した場合は、このラジオボタンの変更がトリガーとなり指定したフィールドが更新されるということが一目瞭然なので、認知が易しく良いコード、となります。こう書けるものはこう書いた方がみんなに優しいです。

 
もちろん、すべてのパターンでこのように書かなくてはいけないわけではなく、上にまとめた方が読みやすいパターンもありうる(例えばコンポーネントが分散していれば親でキャッチした方がわかりやすいこともありそう)のでケースバイケースですが、とにかく**`useEffect`は認知の難易度を上げる**ので、使用を避けられないか一度考える、というのは当然やるべきことです。

### 非同期関数は匿名関数で実装する
これも若干諸説あるんですが、頻出パターンとして`useEffect`内でAPIを発火してデータを取得する、というケースがあります。しかし`useEffect`はasync関数が使えないので、まあ大体こういう実装になります。

```typescript
// `fetch` 関数を useCallback でメモ化(これをしないと無限ループする)
const fetch = useCallback(async (id: number) => {
    const response = await callApi(id);
    setData(response.data);
}, [id]);

// `selectedId` が変わったら `fetch` を実行
useEffect(() => {
    if (selectedId) {
        fetch(selectedId);
    }
}, [selectedId, fetch]); // 本来依存関係にないfetchを含めなくてはいけないのが気持ち悪い...
```
**Exhaustive Deps Rule**を取り入れてるプロジェクトだと、`fetc`hを`useEffect`に入れないといけないです。僕はこのルール、難読化するのであんまり好きじゃないんですけど、公式が推奨してるから従った方がいいんだろうなあ……

で、リレンダリングのたびに`fetch`関数は新しく作られてしまい無限ループを起こすのでメモ化が必要……みたいな、不必要に複雑化した実装になってしまいます。あと、`useEffect`読んでたのに`fetch`関数を読むために目線を上に動かす必要があって、これもコードリーディングを無益に複雑化してて気に入りません。

ここで、匿名関数を使って実装すると…

```typescript
useEffect(() => {
    (async () => {
        const response = await callApi(selectedId);
        setData(response);
    })();
}, [selectedId]);
```
これだと、**`useEffect`を上から下に読むだけでだいたい何やりたいかわかる**ので、認知が易しいコードになります。最後に余計な()がくっついちゃいますが、メリットの方が大きいので、もうそういうイディオムとして取り扱ったほうがいいです。

```typescript
useEffect(() => {
    const fetchDataAsync = async () => {
        const response = await fetchData();
        setData(response);
    };
    fetchDataAsync();
}, [selectedId]);
```
ちなみに、こういうこと書いてくる人結構居るんですけど、**僕は確実にこれをリジェクト**します。
だって目線移動が意味不明すぎません？なんでuseEffectの処理内容を理解するのに、関数定義部分をわざわざスルーして下まで読んで、また戻ってこなきゃいけないんですか？こういう頭を使う必要があるパズルみたいなコード、読んでてしんどいので僕は嫌ですね……。匿名関数とどっちが皆さん読みやすいですか？って話です。

### 補足：一行で済むレベルだったら普通にPromiseを使う
```typescript
// 例としてこう書いたが...
useEffect(() => {
    (async () => {
        const response = await callApi(selectedId);
        setData(response);
    })();
}, [selectedId]);

// これで済むならこっちの方が見通しいい。
useEffect(() => {
    callApi(selectedId).then(setData);
}, [selectedId]);
```
あんまりPromiseって僕は見た目好きじゃなくて避ける傾向あるんですけど(笑)、さすがにこれぐらいならPromiseで書いたほうがシンプルなので優先されるべきかと思います。
目安としてはthen内の処理が二行以上に渡る必要があるかどうかで考えるといいかなと思います。まあ正直好みの問題もあるかと思います。

##フック関連
### カスタムフックはuseCallBackを使う
メモ化に関連した話ですね。ドキュメントにそう書いてあるので使うべきです。
メモ化とか理解してる人はレビュアークラスにないと居ない気がするし、たぶんレビューしててもスルーすると思うんですが(そもそもカスタムフック書ける人がどれだけ居るかという話ですが)、指摘はしたくなるので置いときます。

[useCallback – React ](https://ja.react.dev/reference/react/useCallback#optimizing-a-custom-hook)

## ReactHookForm
> [!NOTE]
> 困ったらドキュメントを見るのが一番→[公式ドキュメント](https://www.react-hook-form.com/)

### watchとuseWatch
超似てるこの2つ、混同しがちなんですがよくわかんなかったら**`useWatch`を使っておけばいい**です。
で、具体的な差分ですが**`watch`は問答無用でフォームルートコンポーネントからリレンダリングが走ります。**
なので、パフォーマンスにおいて`useWatch`に劣る、という理解で問題ないです。要は分からないなら使うなぐらいでいいです。

逆に、フォームを定義(`useForm`)しているコンポーネントそのもので使う分には、この2つに差分はないです。

## Key
### ループでindexの使用は避ける

```typescript
{items.map((item, index) => (
  <li key={index}>{item}</li>  // index を key に使用する例はよく見られる。
))}
```
よくこういう実装をする人がいる。まあ、避けられない例であれば仕方がないが、**基本的にはNG。**
何故keyを設定することを強制されるのかというと、このループ処理で生み出されるコンポーネントをこの`key`で判断しているらしいからですね。
で、この配列の順番を入れ替えて新しい`index`を作成した際に`index`の値が変わってしまい、表示したい位置と誤認が生じる…みたいな話があります。

他にも配列の追加・削除など行った場合、バグったりリレンダリングが起きなかったりと**想定外の動き**をします。

```typescript
const ListComponent = () => {
  const [items, setItems] = useState(["Apple", "Banana", "Cherry"]);
  const removeItem = (indexToRemove: number) => {
    setItems(items.filter((_, index) => index !== indexToRemove));
  };
  return (
    <ul>
      {items.map((item, index) => (
        <li key={index}>
          {item} <button onClick={() => removeItem(index)}>Remove</button>
        </li>
      ))}
    </ul>
  );
};
```

* `index` を `key` に使うと、削除後の index がズレてしまう。
* 例えば `"Banana"` を削除すると、`"Cherry" `の index が 1 → 0 に変わる。
* React は `key` を `index` にしているため、 `"Cherry"` は `"Banana"` の後にあったものだと認識せずに、「要素が変更された」と誤解する。

これにより、意図しない UI のバグが発生する可能性がある。

まあ**正直理解とかしなくていい**です。**indexを使った上で配列操作すると不具合が起きるからやめとけ**って意識があれば困ることはなんもないです。ちなみにこれを知らないと無限にハマります、っていうかハマってる人いました。

回避策としては各要素について**一意の識別子(IDなど)をkeyに設定すると良い**です。なかったら**値から作ればいい**し、なんなら値そのものでもいいです。とりあえずindexを使わなければいいです。

レビュアークラスになると、indexでもいいかどうかは判断できる(要は後から更新されなければ関係ない話です)が、特にIDとかあるパターンだとindexをあえて使う必要はないんじゃないかなーと思いながら、細かすぎるのでレビューではスルーしたりする話ではあります……。(が、そのようなケースが想定される場合は指摘します)

> [!TIP]
> こういうよくわからない警告は、なんでそんなこと強制されるんだろう？って思って調べると良いです。惰性で設定してるとこういうケースでどハマりします。

### コンポーネントの初期化
あるコンポーネントを初期化したいって時どうしますか？リセットボタン的なものを実装したい時ですね。初期値を覚えておくとか処理が煩雑になるんですけど、コンポーネントのkeyを変えれば新しいコンポーネントとして再描画される、つまり**実質的な初期化**になります。これが必要なケース、たまにでてきます。

```typescript
const UserComponent = ({ userId }: { userId: string }) => {
  const [userData, setUserData] = useState<{ name: string } | null>(null);
  useEffect(() => {
    fetchUserData(userId).then(setUserData);
  }, []); // 🔴 `userId` の変更時にリクエストされない！（[] のため最初の1回しか実行されない）
  return <div>{userData ? `User: ${userData.name}` : "Loading..."}</div>;
};

const App = () => {
  const [userId, setUserId] = useState("1");
  return (
    <div>
      <button onClick={() => setUserId("2")}>Switch to User 2</button>
      <UserComponent userId={userId} />
    </div>
  );
};
```
こういうのも知ってると秒殺です。


```typescript
const App = () => {
  const [userId, setUserId] = useState("1");
  return (
    <div>
      <button onClick={() => setUserId("2")}>Switch to User 2</button>
      <UserComponent key={userId} userId={userId} /> {/* ✅ `key` を userId に設定 */}
    </div>
  );
};
```
`key`が変更されたことにより`UserComponent`そのものを作り変えるので、`useEffect`込みで再実行されます。また、**意図が伝わりやすいのでコードも読みやすくなります。**
これを知らないとえらくごちゃごちゃしたコードを書いちゃうと思うので、そういう場合はレビュー指摘するかも知れないです。(これぐらいなら吸収してくれそうな人じゃなかったらスルーするかもですが……)

## デバッグ関連
### リレンダリングを起きているタイミングを知る
Chromeの拡張機能に[React Developer Tools](https://chromewebstore.google.com/detail/react-developer-tools/fmkadmapgofadopljbjfkapdkoienihi?hl=ja) というのがある。

 
![](https://github.com/t-moriyama-r/review_doc/blob/0bcc3b1c1d21ecbc8cafe9be58ef6424abce18a7/fa16fe2e-df07-492d-988f-8d00e96f1b5e.png)
Componentsというのが追加される。

![](https://github.com/t-moriyama-r/review_doc/blob/0bcc3b1c1d21ecbc8cafe9be58ef6424abce18a7/2b9be499-1b76-4cbb-9554-eb4f6a903f18.png)
Highlight updates…のチェックをonにする

![](https://github.com/t-moriyama-r/review_doc/blob/d9a9471f0bdf156019ac04969c59c348643b46be/169c5927-5268-4ef3-8d47-938b1d0d81bb.png)
と、デバッグコンソールを開いている時にリレンダリングされているコンポーネントがハイライトされ、視覚的にわかる。意図しないコンポーネントを起点としてリレンダリングが走ってたり、やけにリレンダリングが起きてるな…とかを観測できるので、開発の助けになります。

### コンポーネント内の値を知る
![](https://github.com/t-moriyama-r/review_doc/blob/0bcc3b1c1d21ecbc8cafe9be58ef6424abce18a7/6886f486-6166-4d4b-b4cc-33896f13302b.png)
Devtoolに、CSSと似たようなこういう選択機能がある。使い方は見たまんまです。

## 実装パターン
### フロント・バックエンドにとって都合のいい形で取り扱う
[【コード編2】フロント・バックエンドにとって都合のいい形で取り扱う](./code_2.md#フロントバックエンドにとって都合のいい形で取り扱う)<br>
【コード編2】よくあるレビュー指摘点(wip) | フロント・バックエンドにとって都合のいい形で取り扱う 

どっちに置くか迷った話ですが、一般化しうる話なのでコード編2に置きました。

### ダイアログにおけるテンプレートリテラルの活用例
ダイアログの表示を考えてみる。ダイアログは基本的に排他的なので1つのページに1つしか要らず、中のメッセージを状態によって変える。

```typescript
// 普通につくるとこうなる
type DialogType={
  open: boolean,
  subject: "SaveConfirm" | "SaveFinish" | "DeleteConfirm" | "DeleteFinish"
}

const Page=()=>{
  const [dialog, setDialog] = useState<DialogType | null>(null);
  return (<>
    <Dialog
      open={dialog.open}
      subject="保存確認"
      message={DIALOG_MESSAGE["saveConfirm"]}
    />
    <Dialog
      open={dialog.open}
      subject="保存完了"
      message={DIALOG_MESSAGE["SaveFinish"]}
    />
    <Dialog
      open={dialog.open}
      subject="削除確認"
      message={DIALOG_MESSAGE["DeleteConfirm"]}
    />
    <Dialog
      open={dialog.open}
      subject="削除完了"
      message={DIALOG_MESSAGE["DeleteFinish"]}
    />
  </>)
}
```
ダイアログが4種あって微妙ですね。これを1つにまとめたい。

```typescript
type DialogType = "SaveConfirm" | "SaveFinish" | "DeleteConfirm" | "DeleteFinish"
type DialogType = `${"Save" | "Delete"}${"Confirm" | "Finish"}`;//別例。こっちの方がいいかも

const Page = () => {
  const [dialog, setDialog] = useState<DialogType | null>(null);
  if(dialog==null) return null
  return (
    <Dialog
      open={!!dialog}
      title={
        (dialog?.startsWith("Save") ? "保存" : "削除") + 
        (dialog?.endsWith("Confirm") ? "確認" : "完了")
      }
      message={DIALOG_MESSAGE[dialog]}
    />
  );
};
```
排他的な制御だとこのようなパターンが使えたりする。(例がちょっとエレガントさが足りないのでもっとしっくりくる例見つけたら修正するかも)

> [親記事:よくあるレビュー指摘点](../README.md)<br>
>   ┗[コード編1(初学者向け)](./code_1.md)<br>
>   ┗[コード編2](./code_2.md)<br>
>   ┗[イディオム編(頻出ロジック)](./idiom.md)<br>
>   ┗[プルリク編](./pull.md)<br>
>   ┗React・フロント編←ここ<br>
>   ┗[レベル別Gitフロー](./git.md)<br>

