# よくあるレビュー指摘点【イディオム編】
> [親記事:よくあるレビュー指摘点](../README.md)<br>
>   ┗[コード編1(初学者向け)](./code_1.md)<br>
>   ┗[コード編2(1年目向け)](./code_2.md)<br>
>   ┗イディオム編(頻出ロジック)←ここ<br>
>   ┗[プルリク編](./pull.md)<br>
>   ┗[React・フロント編](./react.md)<br>
>   ┗[レベル別Gitフロー](./git.md)<br>

> [!TIP]
> 一目で何がしたいのかわかるコード、を目指してねという話。複雑なロジックをわざわざ組まなくてもなんとかなるように大体なってます。**こっちが頭を使う必要はない。**

> [!NOTE]
> 【目次】<br>
> - [時刻・日付を文字列で扱うことの禁止](#時刻日付を文字列で扱うことの禁止)<br>
> - [配列操作](#配列操作)<br>
> - [Map, Setの活用](#Map,Setの活用)<br>
> - [key-value値の活用](#key-value値の活用)<br>
> - [イテレーターの活用](#イテレーターの活用)<br>

## 時刻・日付を文字列で扱うことの禁止
これも今のプロジェクトで何人か指摘受けてました。僕も指摘したことあるかも…
普通に考えてそれやったらバグの温床になるのがわかると思うというか、`Date`型みたいな便利なのがどの言語でも大体存在するので活用してあげればいいのになって思います。ダルい計算は全部コンピューターにやらせましょう。

```typescript
const year = target.substring(0, 4);
const month = target.substring(4, 6)
```
例えばこういうの。targetの前4文字を年として、そこから2文字を月として取り扱う…みたいなコードですね。例えばこれ、12月を1月にしたいみたいな処理がしたくなったらどうするつもりなんでしょうね？良いビジョンが見えましたか？

ちなみにこういうコードを2年目とかの人が書いてきた瞬間、僕は**こいつバグ製造機だな…** って思っちゃいます。そんなに仕事が好きなんですかね？恥ずかしいので絶対にやめるべきです。
```typescript
const date = new Date(target)

// 12月に+1ヶ月する処理も標準でついてて、なんかいい感じにやってくれる
date.setMonth(date.getMonth() + 1);
```
`Date`オブジェクトを利用すれば、利用者側はなんも考えずにこんな感じのコードを書いとけば、2024年が2025年になる可能性があったとしてもいい感じに処理してくれます。

この手のものは言語に標準装備されてるか、有名なライブラリが確実にあると思います。PHPだとCarbonとか。javascriptだとLuxonとか。

## 配列操作
### 例：map・filterなど
`map`や`filter`など、配列操作に関わるメソッドはよくある。javascriptを例にすると…
```typescript
// 1000円以上の商品を抽出し、大文字にするメソッド
function getExpensiveItems(items) {
  const result = [];
  for (let i = 0; i < items.length; i++) {
    if (items[i].price >= 1000) {
      result.push(items[i].name.toUpperCase());
    }
  }
  return result;
}

// 使用例
const items = [
  { name: 'Apple', price: 500 },
  { name: 'Banana', price: 1200 },
  { name: 'Cherry', price: 1500 },
];

console.log(getExpensiveItems(items)); // ['BANANA', 'CHERRY']
```

愚直に書くとこのようになるが、こういう操作を得意とするメソッドがいる。
```typescript
function getExpensiveItems(items) {
  return items
    .filter(item => item.price >= 1000) // 条件に合う商品を抽出
    .map(item => item.name.toUpperCase()); // 名前を大文字に変換
}

// 使用例
const items = [
  { name: 'Apple', price: 500 },
  { name: 'Banana', price: 1200 },
  { name: 'Cherry', price: 1500 },
];

console.log(getExpensiveItems(items)); // ['BANANA', 'CHERRY']
```
こっちの方が何やってるかわかりやすいはず。このように**配列操作は専門のメソッドが居る**ので最大限活用すべきです。他の言語でも大体ある気がします。

### 例：someなど
配列の中から特定の値があるか、みたいなのもsomeとか使えばいいです。これもだいたいどのような言語でもあるはず。
こんなのAIに聞けば一発なんですが、自力で書いちゃう人って結構多いです。

## Map, Setの活用
重複しない組み合わせ、を示すのにこういうデータ型で宣言すれば、その意図が正確に伝わる。typescriptを例にするが、**どの言語でも大体それっぽいのがある**はず。pythonなんかだと**集合演算**もサポートされてる。(後で例を書くかも)

> [!TIP]
> 何度も同じことを言いますが、頑張って覚える必要ないです。こういうの書き始めて**違和感を感じてAIに聞く**というのが出来ればいいだけです。**違和感を感じないというのが問題**なのです。

### 例：配列内から重複したものを取り出す
何も考えないで書くとこうなる。

```typescript
function findDuplicates(arr: number[]): number[] {
  const duplicates: number[] = [];
  for (let i = 0; i < arr.length; i++) {
    for (let j = i + 1; j < arr.length; j++) {
      if (arr[i] === arr[j] && !duplicates.includes(arr[i])) {
        duplicates.push(arr[i]);
      }
    }
  }
  return duplicates;
}

const numbers = [1, 2, 3, 4, 5, 3, 2, 6, 2];
console.log(findDuplicates(numbers)); // [3, 2]
```
`i`とか`j`とかある時点で**考えることを放棄したくなる。** これ見てなんか難しそうなコードだなって思いません？書いててもそうだし、やっぱりこういう頭を使うコードは**レビューしたくない。**

ここで、こういうのは`Set`というもので書き下すことができる。

```typescript
function findDuplicates(arr: number[]): number[] {
  return [...new Set(
    arr.filter(num => arr.indexOf(num) !== arr.lastIndexOf(num))
  )];
}

const numbers = [1, 2, 3, 4, 5, 3, 2, 6, 2];
console.log(findDuplicates(numbers)); // [3, 2]
```
`Set`というのは**仕様として重複しない組み合わせを表現するもの**と決まっているので、まず**`Set`という文字を見た瞬間に「ああ、重複がないデータなんだな」** というのがわかる。

で、`findDuplicates`という関数名を見て、配列から重複しているものを見つけるものなのかなぁという想像がつく。で、実際にコードを見てみると、引数の前と後から走査して一致しないもの、つまりダブっているものをとりあえず新しいSetに入れてるし、これが重複していたとしてもSetの仕様で弾かれる。だから、結果としてダブってるデータが重複なく、配列形式に変換されて返すコードなのかなーと、**すーっと頭の中に入ってくるコード**になって認知が易しいわけです。

```typescript
function findDuplicates<T>(arr: T[]): T[] {
  return [...new Set<T>(
    arr.filter(num => arr.indexOf(num) !== arr.lastIndexOf(num))
  )];
}
```
ジェネリクスを活用すると、より使いやすいコードになりますね。

### 例：対応表をイミュータブルに実装する(flatMapの活用)
配列の長さを長くしたり短くしたりするのはflatMapを使うとスムーズだったりします。

> [!NOTE]
> イミュータブルとは：
> [【コード編2】よくあるレビュー指摘点(wip) | とにかくイミュータブルを意識する](./code_2.md#とにかくイミュータブルを意識する) 

例えば、APIとバックエンドで扱うステータスの表示が異なり、その変換が必要な例。

```typescript
// バックエンド
1: 未審査
2: 審査中
3: 審査通過
4: 審査NG

1: 削除フラグ

// フロント側
1: 審査中(バックエンドの未審査＋審査中)
2: 利用可能(審査通過)
3: 利用不可(審査NG)
4: 削除済み(削除フラグtrue)
```
これらを検索するとして、フロント側からバックエンド側に検索パラメータの配列を送ることを想定する。バックエンドの2値にフロントの1値が対応しており、さらにバックエンドで参照するカラムが2つに分かれている。このような、内部状態と実際にUIに見せるステータスが違うというのは往々にしてあると思います。

```typescript
//Bad Case: ありがちなミュータブルコード
function generateQuery(statuses: Status[]){
  const statusQuery: BackStatus[] = [];
  if (statuses?.includes(STATUS.審査中)) {
    statusQuery.push(BACK_STATUS.未審査);
    statusQuery.push(BACK_STATUS.審査中);
  }
  if (statuses?.includes(STATUS.利用可能)) {
    statusQuery.push(BACK_STATUS.審査通過);
  }
  if (statuses?.includes(STATUS.利用不可)) {
    statusQuery.push(BACK_STATUS.審査NG);
  }

  return {
    statuses: statusQuery, // BACK_STATUS[]は得られる
    delFlg: status?.includes(STATUS.削除済み),
  };
}
```
やってることは直感的で分かりやすいのですが、配列がミュータブルなのがちょっと宜しくない。まあ配列内で閉じてるから妥協してもいいっちゃいいんですけど、将来性を考えるとやはりイミュータブルなコードにしたい。

`Map`だと配列の長さが同じになってしまうが、このような配列の長さが違うものを対応させたい時`flatMap`が使いやすい。

```typescript
// Good Example
const STATUS_MAP={
  [STATUS.審査中]: [BACK_STATUS.未審査, BACK_STATUS.審査中],
  [STATUS.利用可能]: [BACK_STATUS.審査通過],
  [STATUS.利用不可]: [BACK_STATUS.審査NG],
} as const;

function generateQuery(statuses: Status[]){
  return {
    statuses: statuses
      .filter(s=>s!=STATUS.削除済み)
      .flatMap(s=>STATUS_MAP[s]),
    delFlg: statuses?.includes(STATUS.削除済み),
  };
}
```
1値に2値を対応させるというのは`map`内にて配列を返すことで実現し、二次元配列にする必要はないのでflatで均すことで実現している。このようにするとイミュータブルでかつ意図が伝わりやすいコードになる。また、対応表も別ファイルに退避することができるので、関心の分離もしやすくクリーンなコードにしやすい。

また、上記の例だと`statusQuery`という入れ物が必要なくなったことで使う変数が1つ減り、命名の枯渇がしにくくなっているというのも隠れた効用です。(まあこれぐらいならあんま影響ないんですけど)

> [!TIP]
> ミュータブルなコードを書くと、その時点で**分かってなさそうな人**という烙印を押されるかも知れません。(場合による)
> 学習過程としては一旦ミュータブルな実装で書き下して、**イミュータブルにするにはどうしたら？** とAIに聞くといい答えが返ってくるかも。

## key-value値の活用
イディオムというより設計とかリソース表現の話ではあるが、適したページがないのでここに。具体例は後で書く。

## イテレーターの活用
反復処理の話ばっかしてる気がしますが、イテレーターという概念があります。まあ実はあんまり実際書いて活用することはないかもですけど、概念としては知っておくべき話ではある。

小難しい話をすると、デザインパターンの一種ですね。
