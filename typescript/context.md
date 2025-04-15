# Contextのレンダリングについて
Contextは便利だが、リレンダリングを誘発するものという印象がある。その部分を言語化して教えられるレベルに持っていこうという話。

## Contxtとリレンダリングの基本

まずは下記サイトを読んでみましょう。
[React の Context の更新による不要な再レンダリングを防ぐ 〜useContext を利用した時に発生する不要な再レンダリングを防ぐ方法に関して〜 - Qiita](https://qiita.com/soarflat/items/b154adc768bb2d71af21) 

**`Provider` 内のすべての `Consumer` は、`Provider` の`value`プロパティが更新される度に再レンダリングされる。**

```typescript
import React, { createContext, useContext, useReducer } from "react";
const CountContext = createContext();
function countReducer(state, action) {
  switch (action.type) {
    case "increment": {
      return { count: state.count + 1 };
    }
    case "decrement": {
      return { count: state.count - 1 };
    }
    default: {
      throw new Error(`Unhandled action type: ${action.type}`);
    }
  }
}
function CountProvider({ children }) {
  const [state, dispatch] = useReducer(countReducer, { count: 0 });
  const value = {
    state,
    dispatch
  };
  return (
    // value（state か dispatch のどちらか）が更新したら、
    // CountContext.Provider 内のすべての Consumer が再レンダリングされる。
    <CountContext.Provider value={value}>{children}</CountContext.Provider>
  );
}
function Count() {
  console.log("render Count");
  // CountContext からは state のみを取得しているが、
  // dispatch が更新されても再レンダリングされる
  const { state } = useContext(CountContext);
  return <h1>{state.count}</h1>;
}
function Counter() {
  console.log("render Counter");
  // CountContext からは dispatch のみを取得しているが、
  // state が更新されても再レンダリングされる
  const { dispatch } = useContext(CountContext);
  return (
    <>
      <button onClick={() => dispatch({ type: "decrement" })}>-</button>
      <button onClick={() => dispatch({ type: "increment" })}>+</button>
    </>
  );
}
export default function App() {
  return (
    <CountProvider>
      <Count />
      <Counter />
    </CountProvider>
  );
}
```
ここでコアとなるのは下記の部分。

```typescript
// CountContext からは dispatch のみを取得しているが、
// state が更新されても再レンダリングされる
const { dispatch } = useContext(CountContext);
```
`state`が更新されることで、`CountContext`の`value`が更新される、つまり`CountContext`を使っている全てのコンポーネントがリレンダリングされる。
一方、上記コードは`state`を使っていないのにもかかわらずリレンダリングされる、ということでパフォーマンスの観点から宜しくない。

これが一般的にいわれる、**Contextが無用なリレンダリングを引き起こす**という例である。

### 対応策

## ①コンポーネント分割
```typescript
import React, { createContext, useContext, useReducer } from "react";
const CountStateContext = createContext();
const CountDispatchContext = createContext();
function countReducer(state, action) {
  switch (action.type) {
    case "increment": {
      return { count: state.count + 1 };
    }
    case "decrement": {
      return { count: state.count - 1 };
    }
    default: {
      throw new Error(`Unhandled action type: ${action.type}`);
    }
  }
}
function CountProvider({ children }) {
  const [state, dispatch] = useReducer(countReducer, { count: 0 });
  // CountStateContext.Provider の value が更新したら、
  // CountStateContext の値を取得している全ての Consumer が再レンダリングされる。
  // CountDispatchContext.Provider の value が更新したら、
  // CountDispatchContext の値を取得している全ての Consumer が再レンダリングされる。
  return (
    <CountStateContext.Provider value={state}>
      <CountDispatchContext.Provider value={dispatch}>
        {children}
      </CountDispatchContext.Provider>
    </CountStateContext.Provider>
  );
}
function Count() {
  console.log("render Count");
  // state と dispatch を保持する Context オブジェクトが異なるので、
  // dispatch が更新されてもこのコンポーネントは再レンダリングされない。
  const state = useContext(CountStateContext);
  return <h1>{state.count}</h1>;
}
function Counter() {
  console.log("render Counter");
  // state と dispatch を保持する Context オブジェクトが異なるので、
  // state が更新されてもこのコンポーネントは再レンダリングされない。
  const dispatch = useContext(CountDispatchContext);
  return (
    <>
      <button onClick={() => dispatch({ type: "decrement" })}>-</button>
      <button onClick={() => dispatch({ type: "increment" })}>+</button>
    </>
  );
}
export default function App() {
  return (
    <CountProvider>
      <Count />
      <Counter />
    </CountProvider>
  );
}
```
引用元のコピペですが、重要なのはこれ。

```typescript
function CountProvider({ children }) {
  const [state, dispatch] = useReducer(countReducer, { count: 0 });
  // CountStateContext.Provider の value が更新したら、
  // CountStateContext の値を取得している全ての Consumer が再レンダリングされる。
  // CountDispatchContext.Provider の value が更新したら、
  // CountDispatchContext の値を取得している全ての Consumer が再レンダリングされる。
  return (
    <CountStateContext.Provider value={state}>
      <CountDispatchContext.Provider value={dispatch}>
        {children}
      </CountDispatchContext.Provider>
    </CountStateContext.Provider>
  );
}
```
このように`state`と`dispatch`をそれぞれ`Context`を2つに分けて`value`に渡せばいいよね、という思想。
`value`の更新がリレンダリングを引き起こすのである、というのが原理なので、その`value`自体が分かれていれば大丈夫ですよね。

```typescript
function CountProvider({ children }) {
  const [state, dispatch] = useReducer(countReducer, { count: 0 });
  const value = {
    state,
    dispatch
  };
  return (
    // value（state か dispatch のどちらか）が更新したら、
    // CountContext.Provider 内のすべての Consumer が再レンダリングされる。
    <CountContext.Provider value={value}>{children}</CountContext.Provider>
  );
}
```
元の形と見比べてみましょう。`value`の中に`state`と`dispatch`が存在してる。これは直感的に理解しやすい一方、このようなパフォーマンス上のリスクがあるわけですね。

### ②メモ化する
`React.memo`で`props`レベルでメモ化するか、`useMemo`で計算結果をメモするかの二択が存在するが、とりあえずメモ化でも対応可能ではある。
例は･･････別に理解するのは難しくないと思うので、いいか……。


とりあえずこれが`Context`が引き起こす問題と、その対応策である。本来、`Context`を定義する際はこのようなチューニングを見据えて実装されるべきではある。

もちろん、全ての場合においてパフォーマンスを気にするかっていうとこれもまた別問題で、**可読性を落とすというトレードオフも存在する**ので、このへんは影響範囲を考えてチューニングすべき。

例えば、頻繁に状態が変わるとか関連する状態が多いのならセッターと分けるとかやってもいいけど、規模がでかくなるならストアでいいとか･･････そのへんはちゃんと考えないといけないなあという意識があると良さそうですね。
とはいえ、Reactの流儀としてはパフォーマンスチューニングは必要に迫られて、計測し初めて考慮されるべきで、基本的には可読性を優先すべきだとは思ってはいる。公式もそんな感じのこと言ってるし。

## constate

ちなみに、どうやらこのあたりをラップして使いやすくするライブラリがあるらしい。

[GitHub - diegohaz/constate: React Context + State](https://github.com/diegohaz/constate) 

使い方は下記。

[ReactのContext APIを簡単実装！「constate」の使い方](https://aizulab.com/blog/react-constate/)

モーダルの例があったりしてドンピシャって感じ。**もう全部こいつでいいんじゃないか？**<br>
ただし思考停止でこういうことをすると二重モーダルとか素直にできなくなりそうなので、そのへんは仕様と相談って感じですね。

## おまけ

[React ContextAPIのアンチパターン](https://zenn.dev/sora_kumo/articles/72fae8a8244adf)

↑みたいなトリッキーなパターンがあるにはあるが、ぱっと見意味が分からなかったし、やはりDiscussionが炎上してる。(笑)

```typescript
const dispatches = useContext(context);
  useEffect(() => {
    dispatches[name] = setValue;
    return () => {
      delete dispatches[name];
    };
  }, [name]);
```
どうやら、`Context`から受け取った`Ref`オブジェクトに`State`をぶち込む思想らしいが、これってもはやストアなのでは･･････？ということで、有害図書扱いされてるみたい。(笑)
