## (面接担当者様へ)
このページは、自社で書いていた技術コラムのうち、フロントエンド側のものをある程度抜粋したものです。<br>
(自分自身はバックエンドも書いており同様のコラムを書いているのですが、移植がちょっと大変なので...)<br>
<br>
(技術選定・比較の話などは、調査的な側面が強いので移植はしていないです)

【特に普段意識していること】<br>
(基本的なこと)<br>
[修正の根拠となったチケット・概要を載せる](/review/pull.md#修正の根拠となったチケット概要を載せる)→コミットを意味単位に整形し、プルリクも可能な限りちゃんと書く<br>
[引数の定義に意思はあるか？](/review/code_2.md#引数の定義に意思はあるか)→保守性の低いプログラムを書かない<br>
[フロント・バックエンドにとって都合のいい形で取り扱う](/review/code_2.md#フロントバックエンドにとって都合のいい形で取り扱う)→思考の手続きをそのまま書かず、合理的なコードを目指す<br>
[useEffect(副作用)について](/review/react.md#useEffect副作用)→useEffectは難読化につながることを意識する<br>

(応用的なこと)<br>
[Contextとレンダリング](/typescript/context.md)→リレンダリングのタイミングを理解する(あとちょっとしたライブラリ紹介)<br>
[二者択一は型にやらせる](/typescript/choice.md)→論理構造を型レベルで限定することで余計な分岐を減らす(neverとか活用できます)<br>
[Utility Typesを使いこなして気持ちよくなろう](/typescript/util.md)→typescriptの汎用型システムの理解<br>
[コールシグネチャとinterface](/typescript/call.md)→引数の形でtype-narrowingも可能という話<br>
[宣言マージとモジュール拡張](/typescript/merge.md)→ライブラリを拡張する話<br>
<br>
[React v19](/typescript/v19.md)→自主的にキャッチアップした時の記事<br>

このあたりをご覧になられると、作業者としての姿勢を伝えられるかと思っています。<br>より深いレベルの話は面談で聞いてもらえると答えられるかと思います。

***

# 【About】

元々レビューする時に気をつけてたポイントの話だったが、肥大化してレビューで指摘したこと・されたこと集になってきたのでそのようなメモにする。<br>
自分がレビューしていた時の指摘点のシェア、および他の人のレビュー指摘も見て、同じ轍を踏まないために陥りがちなポイントをアウトプットし言語化したもの。

> [!NOTE]
> 【目次】<br>
> [コード編1(初学者向け)](/review/code_1.md)<br>
> [コード編2](./code_2.md)<br>
> [イディオム編(頻出ロジック)](/review/idiom.md)<br>
> [作業の進め方・プルリク編](/review/pull.md)<br>
> [React・フロント編](/review/react.md)<br>
> [レベル別Gitフロー](/review/git.md)<br>
> <br>
> [typescript・React雑学集(一歩進んだ話)](/typescript/)<br>

> [!TIP]
> (初学者がこれ見るとビビるかもなのでアドバイス)<br>
> 小難しいこと書き並べてますけど、大事なのは**とにかく手を抜くこと**だと思ってます。脳の使用メモリを最小限にして、なんか書いててダルいからもっとラクな方法ないの？ってChatGPTとかに聞けばこういうのは自然に身につく気がします。



## (参考1) 僕が過去に読んだ本
| タイトル | 感想 |
| ---- | ---- |
| [リーダブルコード ―より良いコードを書くためのシンプルで実践的なテクニック (Theory in practice) ](https://amzn.to/3EbhhTT) | 有名なベストセラー。初学者はまず一読してみると良さそう。レビュー指摘されてその部分を読み直すみたいな使い方だと定着しそう。<br>当たり前のことが書いてある感あるが、**当たり前のことが当たり前にできる、という人がこの業界結構少ない**ので、特に1～2年目の人とかこれ読んで **「別に普通じゃね？」** って思えるかどうか試すと良さそう。個人的にそれだけで単価80万レベルはいける。<br>中級者以上でも感覚でやってるのと知識があった上でやってるのとでは差異があるので、ある程度の人でも読んでみると**確固たる自信につながる。** |
| [SQLアンチパターン](https://amzn.to/3PVjO7g)  | これも有名な本っぽい。初学者にはおすすめ。<br>隣接リストとか定義しちゃいがちなので知識としては大事。買ってよかった。<br>僕が最も参考になった例→[木構造の表現]() | 
| [Go言語 100Tips ありがちなミスを把握し、実装を最適化する impress top gearシリーズ ](https://amzn.to/4hyFMIY) | Goは現状独学なので、なんか本でも読もうかなと思って買ったもの。Goという言語の特性やTipsを理解するのに良かった。[関数オプションパターンとか目から鱗だった]() |
| [Joel on Software ](https://amzn.to/40OUW7h) | 客先のフリーランスと~~ド平日の深夜1時半まで飲んでた時に~~おすすめされた本。(笑)<br> Excelの開発者の本かな？技術というより価値観の本。興味深い内容がいくつかある。 |
| [ザ・ゴール ](https://amzn.to/3PTLARt) | 勧められたので読んだ本。どっちかというとマネジメント寄りの本だが、小説みたいに読める。プロジェクトは炎上しがちなので、それに対するナレッジとかそういうのを得るために読んだ。 |

Kindleとかで買って、特にやることがない電車移動中とかバスの中とかでテキトーに読むのがオススメです。
僕はわざわざプライベートな時間に仕事のこととか勉強したくないです。が、どうせ他にやることないから...っていう時に読むと、仕事してる感なくインプットできます。

##  (参考2)React学習時に僕がよく見てたサイト
| タイトル | 感想 |
| ---- | ---- |
| [TypeScript入門『サバイバルTypeScript』 ](https://typescriptbook.jp/) | 参画した当初はReactもイミフでjavascriptアレルギー持ってたのもあり(フロントリーダーになった時は絶望した…)、最初いきなり全ては理解できなかったが、理解できるようになったら何度も読み返しては**頭に入ってこないものは後回し**にしてそのうちまた読み返す…というのを繰り返してたらなんか定着してた。 |
| [React Developer Roadmap: Learn to become a React developer ](https://roadmap.sh/react) | 自分がどのくらいのレベルに居るのか知るために見てた。使ったことがなくても、そういうライブラリがあるんだ～みたいな知識を軽く持っておくだけで技術選定が将来的にラクになると思う。ちなみに他言語版もあるっぽい。 <br> 例：https://roadmap.sh/php |
| [React リファレンス概要 – React ](https://ja.react.dev/reference/react) | あらゆるライブラリがそうなのですが、**とにかく一次情報を読むことが重要**です。これもよくわからないものは後回しにして、しばらく経ったら読み返して足りない知識をアップデートしてた。特にフック周り。<br>正直Reactは覚えることが少ないし、何より日本語のドキュメントがあるから学習は圧倒的にラクだった。|

こういうの勉強するのダルくて嫌いなんですが、知識があるとラクができるシーンが多々あるので、スキマ時間にちょろっと読むのがオススメ。
仕事してて一番嫌いなバグフィクスという時間は、**知識があれば圧倒的に短縮できる。**

> [!NOTE]
> (レビューする側について)<br>
> 技術に関係ない話としては **「褒められる点があれば褒める・感想を伝える」** というのもレビューするにあたって大事かなと思います。自分は当時できなかったことなのですが、印象はだいぶ変わるかなと思います。
