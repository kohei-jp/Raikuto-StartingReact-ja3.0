#### 第8章 何はともなくコンポーネント

・React は仮想 DOM をリアルな DOM にレンダリングすることで Web アプリケーションとして動作する。

・その仮想 DOM を構成するのが React Elements。

・Reactコンポーネントは、props を引数として受け取り、戻り値として React Elementsを返す関数。
　引数と戻り値があることが、関数に似ているが、違いとしては、個々に「状態」を持つ事ができる所。

・Reactの差分エンジンで検出される要素としては、propsとstateがある。これらが変化するとレンダリングが走る。
　→ ただ極力、stateは持たせない様にしたい。 React にとって理想的 なコンポーネントとは、props が同じなら必ずレンダリング結果が同じになるコンポーネントであるため。( = 純粋関数 )



・React 

```ts
// 親コンポーネント
import CharacterList, { Character } from './CharacterList';

const App: VFC = () => {
　const characters: Character[] = [
    { id: 1, name: '桜木花道', grade: 1, height: 189.2 },
    { id: 2, name: '流川 楓', grade: 1, height: 187 },
  ];

　return (
   <>
  	<CharacterList school="湘北高校" characters={characters} />
	 </>
　)
}
export default App;


// 子コンポーネント
import { VFC } from 'react';
import { Header, Icon, Item } from 'semantic-ui-react';

// 
export type Character = {
  id: number;
  name: string;
  grade: number;
  height?: number;
};

// ① Propsという型エイリアスを定義　＝> マウントにscheelとcharactersが必要ということが分かる。
type Props = {
  school: string;
  characters: Character[];
};

// VFCにPropsという型引数を渡すことで、propsの型を指定できる。
const CharacterList: VFC<Props> = (props) => {
  const { school, characters } = props;　// 分割代入して取り出している。

  return (
    <>
      <Header as="h2">{school}</Header>
      <Item.Group>
        {characters.map((character) => (
          <Item key={character.id}>
            <Icon name="user circle" size="huge" />
            <Item.Content>
              <Item.Header>{character.name}</Item.Header>
              <Item.Meta>{character.grade}年生</Item.Meta>
              <Item.Meta>
                {character.height ? character.height : '???'}
                cm
              </Item.Meta>
            </Item.Content>
          </Item>
        ))}
      </Item.Group>
    </>
  );
};

export default CharacterList;

```



・クラスコンポーネント

```ts
import { Component, ReactElement } from 'react';

type State = { count: number };

// 型引数にunknownとStateがある。 このコンポーネントにはpropsが必要ないので、unknownを渡している。
// Stateは今コンポーネントで使用するstateの型。 => 数値型プロパティのcountのこと。
class App extends Component<unknown, State> {
  
  // stateの初期化には、constructorが必要
  constructor(props: unknown) {
    super(props);
    this.state = { count: 0 }; // count = 0と初期化している。
    // constructor以外でcountを変更する場合は、setStateメソッドを使用する。
  }

  reset(): void {
    this.setState({ count: 0 });
  }

  increment(): void {
    this.setState((state) => ({ count: state.count + 1 }));
  }

  render(): ReactElement {
    const { count } = this.state;

    return (
      <div className="container">
        <header>
          <h1>カウンター</h1>
        </header>
        <Card>
          <Statistic className="number-board">
            <Statistic.Label>count</Statistic.Label>
            <Statistic.Value>{count}</Statistic.Value>
          </Statistic>
          <Card.Content>
            <div className="ui two buttons">
              <Button color="red" onClick={() => this.reset()}>
                Reset
              </Button>
              <Button color="green" onClick={() => this.increment()}>
                +1
              </Button>
            </div>
          </Card.Content>
        </Card>
      </div>
    );
  }
}

export default App;

```

stateに値をセットするときに使用する```setState```メソッドの使用方法
① state内の変更したい要素名のキーに、値をその値にしたオブジェクトを渡す。

```ts
 this.setState({ count: 0 });
```

② ```(prevState, props?) => newState``` 形式の、以前の state(必要であればpropsも)を引数として 受け取って新しい state を返す関数

```ts
this.setState((state) => ({ count: state.count + 1 }));
// 必要であれば、propsも引数で渡す。
```



**①の注意事項 P.145**
**state の値の更新は、 React によるレンダリング最適化処理の中で非同期に行われる**ため、
②のサンプルでやっている カウントアップさせる際に、1の方法で「 **this.state.count + 1** 」をすると、stateが最新の値でない可能性がある。そのため、明示的に引数で現在の値を渡してあげる必要がある。

```ts
this.setState({ count: this.state.count + 1 })
```



関数を実行にイベントオブジェクトのオリジナルの動作を抑制する場合(aタグの遷移動作など)

```ts
reset = (): void =>  {
  this.setState({ count: 0 });
}


reset = (e: SyntheticEvent) => { 
  e.preventDefault(); 
  this.setState({ count: 0 });
};

SyntheticEvent: イベントハンドラのコールバックに引数として渡されるオブジェクトの型
e.preventDefault():  実行は該当要素のオリジナルの挙動を抑制している。
```



##### 8-4. コンポーネントのライフサイクル

コン ポーネントにおけるライフサイクルとは、
まずマウントして初期化され、次にレンダリングされた後、 何らかのきっかけで再レンダリングされ、
最後にアンマウントされるまでの過程をいう。

1. <u>Mounting フェーズ</u> ...... 

   コンポーネントが初期化され、仮想 DOM にマウントされるまでの フェーズ。このフェーズで初めてコンポーネントがレンダリングされる

   ex. componentDidMount: コンポーネントがマウントされた直後に呼ば れる

2. <u>Updating フェーズ</u> ...... 

   差分検出処理エンジンが変更を検知してコンポーネントが再レン ダリングされるフェーズ

   ex.  shouldComponentUpdate: 変更を検知してから再レンダリング処理の前 に呼ばれ、
   　　　　　　　　　　　　　　   false を返すことで再レンダリング を中止できる

   　　componentDidUpdate: 再レンダリングの完了直後に呼ばれる

3. <u>Unmounting</u> フェーズ ...... 

   コンポーネントが仮想 DOM から削除されるフェーズ

   componentWillUnmount(): コンポーネントがアンマウントされて破棄さ れる直前に呼ばれる

4. <u>Error Handling</u> フェーズ ...... 

   子孫コンポーネントのエラーを検知、捕捉するフェーズ

![lifecycle-methods-diagram](/Users/kohei/Desktop/8.4 lifecycle-methods-diagram.png)



##### 8-5. Presentational Component と Container Component

ひとつのコンポーネントを『Presentational Component』お よび『Container Component』というものの 2 種類に分割しようというデザインパターンがある。

・presentational component：見た目だけを責務とするコンポーネント。

・Container Component：ロジックを追加するためのコンポーネント。



もともと、
コンポーネントベースのアーキテクチャでは、MVC のようにアプリケーションを機能 で分割するのではなく、デザインとロジックを閉じ込めた独立性の高いコンポーネントを組み合わ せてアプリケーションを作る。
というものだが、
プロジェ クトの継続的な運用を考えると、個々のコンポーネントを見た目だけのコンポーネントと、
それに ロジックを追加するコンポーネントに分割しておくことは有益。



公式が推奨するコンポーネントの正しい作り方 ( P.156 )

・React の流儀(Thinking in React)

```
1. デザインモックから始め、その UI をコンポーネントの構造に分解して落とし込む
2. ロジックを除外した、静的に動作するバージョンを作成する。DEMOデータを用意して作る。
3. UI を表現するために最低限必要な「状態」を特定する
4. 3 の「状態」をどこに配置すべきかを決める
5. 階層構造を逆のぼって考え、データが上階層から流れてくるようにする
```

今回の件に落とし込むと、
１、2は、 見た目を作り込む presentational component の設計
 3〜 5は、ロジックを追加する container component の設計



#### 第 9 章 Hooks 、関数コンポーネントの合体強化 パーツ

・HOC が任意のコンポーネントを引数として受け取って、その戻り値にコンポーネントを返す関数。

```ts
type Props = { target: string };
const HelloComponent: FC<Props> = ({ target }) => <h1>Hello {target}!</h1>;

// withTargetがHOCで、外からtargetの実体を与えている。
// 以下は、targetに Patty という文字列を与えている。
const withTarget = (WrappedComponent: FC<Props>) => WrappedComponent({ target: 'Patty' });

export default withTarget(HelloComponent);
```



以下のカウントアップアプリの例では、
targetは、max・( count・reset・ increment )にあたる。

```ts
import React, { FC, Component, ReactElement } from 'react';

type InjectedProps = { 
  count: number;
  reset: () => void; 
  increment: () => void;
};
type Props = { max: number }; 

type State = { count: number };

// 必要な stateやロジックのためのメンバーメソッドを持つ「クラスコンポーネント」を生成してる。
const withCounter = (WrappedComponent: FC<Props & Partial<InjectedProps>>) => 

	class EnhancedComponent extends Component<Props, State> {
    constructor(props: Props) { super(props);
      this.state = { count: 0 };
    }
    
		reset = (): void => this.setState({ count: 0 });
		increment = (): void => this.setState((state) => ({ count: state.count + 1 }));
    componentDidUpdate = (): void => {
    	if (this.state.count > this.props.max) this.reset();
    };
		render = (): ReactElement => ( 
      // withCounterが引数として受け取ったコンポーネントに props としてそれらを渡してあげてる
      <WrappedComponent
        max={this.props.max} 
				count={this.state.count}
        reset={this.reset}
        increment={this.increment} 
			/>
     ); 
	};


// 状態やロジックを持たない 純粋な presentational component 
// FCの型引数<T>には、JSXの表示に必要なProps(max)と HOCによってロジックを注入できる様にしておく必要があるので、省略可能なPartial型を使用して、InjectedPropsを合成(&)している。
const CounterComponent: FC<Props & Partial<InjectedProps>> = ({ 
  max, 
  count = 0,
	reset = () => undefined, 
  increment = () => undefined,
	})=>( 
    <div>
      <div> {count} / {max} </div>
      <button onClick={reset} type="button"> Reset </button>
      <button onClick={increment} type="button"> +1 </button>
     </div> 
);

// withCounterはHOCで、この中の count、reset、increment という 3 つの props に、
// 外から状態やロジックを注入してるだけ
export default withCounter(CounterComponent);
```



・CounterComponentの引数
　・HOC 適用後の外側のコン ポーネントの props をうまく辻褄が合うように型合成してあげる必要がある
　・内側のコンポーネントはそれ単体でも成立するよう、HOCで注入予定のpropsには
　　デフォルト値を設定する必要がある。



・Render Props
　React Elements を返す関数を props と して受け取ってそれを自身のレンダリングに利用するコンポーネント

```ts
type Props = { target: string };
// HelloComponent はコンポーネントだけど、React Elements を返す関数でもある
const HelloComponent: FC<Props> = ({ target }) => <h1> Hello {target}!</h1>;


const TargetProvider: FC<{ render: FC<Props> }> = ({ render }) => render({ target: 'Patty' });

// renderというpropsにHelloComponentが渡されている。
<TargetProvider render={HelloComponent} />
```



```ts
import React, { Component, FC, ReactElement } from 'react';

type ChildProps = { 
  count: number;
	reset: () => void; 
  increment: () => void;
};

type Props = {
	max: number;
	children: (props: ChildProps) => ReactElement;
};

type State = { count: number };

class CounterProvider extends Component<Props, State> { 
  constructor(props: Props) {
    super(props);
		this.state = { count: 0 }; 
  }
  
	reset = (): void => this.setState({ count: 0 });
	increment = (): void => this.setState((state) => ({ count: state.count + 1 }));
	componentDidUpdate = (): void => {
		if (this.state.count > this.props.max) this.reset();
	};
  
  render = (): ReactElement => 
    this.props.children({
      count: this.state.count, 
      reset: this.reset,
      increment: this.increment
    });
}

const Counter: FC<{ max: number }> = ({ max }) => ( 
  <CounterProvider max={max}>
		{({ count, reset, increment }) => ( 
      <div>
      	<div> {count} / {max} </div>
        <button onClick={reset} type="button">Reset</button>
        <button onClick={increment} type="button"> +1 </button>
       </div> 
		)}
	</CounterProvider> 
);

export default Counter;
```





##### 9-2. Hooks で state を扱う

```ts
const [count, setCount] = useState(0);
useStateは、state変数とstate更新関数をタプル型で返すので、配列の分割代入の形で定義している。
```

上記の例では、初期値0から 型推論でnumberとなっているが、初期値がないパターンなどは、
型引数を渡してあげる必要がある。

```ts
const [author, setAuthor] = useState<User>(); // undefined. 引数はUserオブジェクト
const [articles, setArticles] = useState<Article[]>([])
ジェネリクスで型引数を渡している。
※ 初期値に、nullを入れたい場合は、 useState<User | null>(null) 
```

・stateが更新される注意点

```ts
const plusThreeDirectly = () => [0, 1, 2].forEach((_) => setCount(count + 1));

const plusThreeWithFunction = () => [0, 1, 2].forEach((_) => setCount((c) => c + 1));
```

上記のコードの場合、<u>plusThreeDirectly</u>では、+1しかされない。
　=> state 変数はそのコンポーネントのレンダリングごとで一定のため！

なので、state 変数を相対的に変更する処理を行うときは、前の値を直接参照・変更 するのは、
避けて必ず *setCount((c) => c + 1)* のように関数で書くべき。



##### 9-3. Hooks で副作用を扱う

コンポーネントの状態を変化させ、それ以降の出力を変えてしまう処理のこと。
ネットワークを介したデータの取 得やそのリアクティブな購読、ログの記録、リアル DOM の手動での書き換えといったもの。

React におけるコンポーネントとは、状態を持った関数のようなもので、
関数 y = f(x) は本来なら x が同じなら出力値 y も同じはずだけど、 状態を抱える関数であれば必ずしもそうとは限らない。

Effect Hook とは、レンダリング のタイミングに同期させて実行するための Hooks API のこと。

```ts
const SampleComponent: VFC = () => { 
  const [data, setData] = useState(null); 
  …
  useEffect(() => {
    doSomething(); 
    
    return () => clearSomething();  // ① unMount時に、発火させたいものがあれば指定する。
  }, [someDeps]);
  …
};
```

① setTimeoutによるタイマー処理は、cleartimeoutして終了させないとエラーが出る。
　 useEffect内で戻り値として任意の関数を返す様にしておくと、そのコンポーネントがアンマウントされるときにその戻り値の関数を実行してくれる。



###### Effect Hook とライフサイクルメソッドの相違点

1. 実行されるタイミング
   ```componentDidMount``` はその コンポーネントがマウントされてレンダリング内容が仮想 DOM からリアル DOM へ反映される前 に、ブラウザへの表示をブロックして実行される。

   いっぽう ```useEffect``` が初回実行されるのは、最初のレンダリングが行われてその内容がブラウザ に反映された直後。

   → useEffectではレスポンス性を高めるために、とりあえず、初期値を設定する様にしている。

   ※ ただ、```useLayoutEffect ```は、```componentDidMountやcomponentDidUpdate```と同じタイミングで実行される。ユースケースとしては、た とえば特定の DOM 要素の位置やブラウザの幅とかを取得してレンダリングに利用するなど。
   だから使っていいのは副作用処理にかかる時間のほうが、React が仮想 DOM の 差分を検知して再描画する時間よりも明らかに短くなるようなレアな場合に限定される。　

   

2. props と state の値の即時性
   マウントによってインスタンスが生成されてアンマウントまでそれが生き続けるクラスコンポーネント。
   レンダリングのたびに実行されては破棄される関数コンポー ネントが根本的に異なる。
   → 関数コンポーネントはレンダリングの度に破棄されるので、クラスコンポーネントと違い、インスタンス変数の保持ができないが、Hooksによってコンポーネント外部で保存されて、次回のレンダリングで改めて渡される。
   ・クラスCは、レンダリングと関係なく変数を内部のミュータブルな値として保持している。
   ・関数C　は、レンダリングの度に、外からイミュータブルな値として与えられる。

　　cf.「関数コンポーネントはクラスとどう違うのか? —Overreacted」
　　https://overreacted.io/ja/how-are-function-components-different-from-classes/

3. 凝集の単位
   ・クラスコンポーネントは、
   ```componentDidMount```・```componentDidUpdate```・```componentWillUnmount```と、
   3つのライフサイクルメソッドに分散されてしまう。これが、機能的凝集度が低い。
   ・関数コンポーネントは、
   一つの```useEffect```内に、上記の全ての処理を纏めて描くことができる。

   機能的凝集度が高いということは、
   同じ機能が分散して記述されないためコードの可読性が高いのはもちろん、
   機能によってまとまったロジックをコンポーネントから切り離して再利用しやすい。

cf. 「useEffect 完全ガイド —Overreacted」https://overreacted.io/ja/a-complete-guide-to-useeffect/



##### 9-4. Hooks におけるメモ化を理解する

・useMemo

関数の実行結果をメモ化するHooks API。
レンダリングの度に、関数を実行しても戻り値が変わらない場合に再実行を抑える。
useMemo の第二引数の値が変化した場合のみ、第一引数の関数を実行する。

```ts
・引数の数字の中で、素数を含む配列を生成する関数。
 => 引数が同じなら、何度計算しても結果は一緒。
export const getPrimes = (maxRange: number): number[] =>
  [...Array(maxRange + 1).keys()].slice(2).filter((n) => {
    for (let i = 2; i < n; i += 1) {
      if (n % i === 0) return false;
    }
   return true;
	});

// limitが変更があった時のみ、getPrimes(limit)を計算し、primesに値を代入する。
const primes = useMemo(() => getPrimes(limit), [limit]);

<Statistic.Value
　className={primes.includes(timeLeft) ? 'prime-number' : undefined} 
>
  {timeLeft} 
</Statistic.Value>
```



・useCallback

関数定義そ のものをメモ化するためのもの。
関数はレンダリングの度に異なるPCメモリの箇所に定義されるので、useEffectの第二引数に渡す場合など、Memo化していないと、毎回変更検知されてしまう。

```ts
 - const reset = (): void => setTimeLeft(limit);
// limitが変更した場合のみ、reset()が再定義される。
 + const reset = useCallback(() => setTimeLeft(limit), [limit]);
```



cf. useRef

リアル DOM への参照に用いる ref オブジェクトを生成する ためのもの。

```ts
import { VFC, SyntheticEvent, useEffect, useRef } from 'react';

const TextInput: VFC = () => {
  const inputRef = useRef<HTMLInputElement>();  // ① refオブジェクトを生成
  const handleClick = (e: SyntheticEvent): void => { 
    e.preventDefault();
    // ③ inputRef.current で <input> のリアル DOM のノードが参照可能に
    if (inputRef.current) alert(inputRef.current.value);
  };
  
  useEffect(() => {
    if (inputRef.current) inputRef.current.focus();
  }, []);

  return ( 
    <>
      <input ref={inputRef} type="text" />  // ② ref 属性に代入
      <button onClick={handleClick} type="button">Click</button>
		</> 
	);
};

export default TextInput;
```

useRefには上記のような「リアルDOMを参照する」以外に、「あらゆる書き換え可能な値を保持しておく」ことができる。

以下の例では、カウントダウンアプリのtimerIDを保持している。
useStateではなく**useRefで値を保持している理由**としては、**値が変更しても再レンダリングが走らない。**
ため、リソースを削減出来る。

```ts
const timerId = useRef<NodeJS.Timeout>(); // この型は、setIntervalの戻り値。

const clearTimer = () => {
  if (timerId.current) clearInterval(timerId.current);
};

const reset = useCallback(() => {
    clearTimer(); // timerIDを初期化
    timerId.current = setInterval(tick, 1000); // 新しいtimerIDを代入
    setTimeLeft(limit);
}, [limit]);

useEffect(() => {
  reset();
  
  return clearTimer;  // timerIDを初期化
}, [reset]);
```



##### 9-5. Custom Hook でロジックを分離・再利用する

Hooks のロジック部分だけを抽出して、再利用可能な形にuseTimerというHooks関数で切り出す。

・src/Timer.jsx

```ts
import { VFC } from 'react';
import { Button, Card, Icon, Statistic } from 'semantic-ui-react';
import useTimer from 'hooks/use-timer';
import 'components/Timer.css';

const Timer: VFC<{ limit: number }> = ({ limit }) => {
  // 戻り値をタプル型で分割代入で受け取っている。(タプル型は必須ではない。)
  const [timeLeft, isPrime, reset] = useTimer(limit);

  return (
    <Card>
      <Statistic className="number-board">
        <Statistic.Label>time</Statistic.Label>
        <Statistic.Value className={isPrime ? 'prime-number' : undefined}>
          {timeLeft}
        </Statistic.Value>
      </Statistic>
      <Card.Content>
        <Button color="red" fluid onClick={reset}>
          <Icon name="redo" />
          Reset
        </Button>
      </Card.Content>
    </Card>
  );
};

export default Timer;
```

・src/hooks/use-timer.tsx (Custome Hooks)

```ts
import { useCallback, useEffect, useMemo, useRef, useState } from 'react';
import { getPrimes } from 'utils/math-tool';

// Custom Hooksは、use***という関数名にする決まりがある。
const useTimer = (limit: number): [number, boolean, () => void] => {
  const [timeLeft, setTimeLeft] = useState(limit);
  const primes = useMemo(() => getPrimes(limit), [limit]);
  const timerId = useRef<NodeJS.Timeout>();
  const tick = () => setTimeLeft((t) => t - 1);

  const clearTimer = () => {
    if (timerId.current) clearInterval(timerId.current);
  };

  const reset = useCallback(() => {
    clearTimer();
    timerId.current = setInterval(tick, 1000);
    setTimeLeft(limit);
  }, [limit]);

  useEffect(() => {
    reset();

    return clearTimer;
  }, [reset]);

  useEffect(() => {
    if (timeLeft === 0) reset();
  }, [timeLeft, reset]);

  return [timeLeft, primes.includes(timeLeft), reset];
};

export default useTimer;

```



最後に、上記のTimer.jsxをpresentational component と container componentに分けてみる。

・(Base) Timer.jsx <before>

```ts
import { VFC } from 'react';
import { Button, Card, Icon, Statistic } from 'semantic-ui-react';
import useTimer from 'hooks/use-timer';
import 'components/Timer.css';

const Timer: VFC<{ limit: number }> = ({ limit }) => {
  // 戻り値をタプル型で分割代入で受け取っている。(タプル型は必須ではない。)
  const [timeLeft, isPrime, reset] = useTimer(limit);

  return (
    <Card>
      <Statistic className="number-board">
        <Statistic.Label>time</Statistic.Label>
        <Statistic.Value className={isPrime ? 'prime-number' : undefined}>
          {timeLeft}
        </Statistic.Value>
      </Statistic>
      <Card.Content>
        <Button color="red" fluid onClick={reset}>
          <Icon name="redo" />
          Reset
        </Button>
      </Card.Content>
    </Card>
  );
};

export default Timer;
```

<after>

・src/containers/Timer.tsx ( container component )

　変数の取得し描画については子コンポーネントに委ねている。
　今後も、表示内容が増えればここで定義して、値は子コンポーネントに渡すだけ。

```ts
import { VFC } from 'react';
import { Button, Card, Icon, Statistic } from 'semantic-ui-react';
import useTimer from 'hooks/use-timer';
import Timer from 'components/Timer';

const Timer: VFC<{ limit: number }> = ({ limit }) => {
  
  const [timeLeft, isPrime, reset] = useTimer(limit);

  return (
    <Timer timeLeft={timeLeft} isPrime={isPrime} reset={reset} />
  );
};

export default Timer;
```

・src/components/Timer.tsx ( presentational component )

　変数を取得するようなHooksは含まない。propsとして親コンポーネントから渡す。
　ここでは、見た目の実行のみ。

```ts
import { VFC } from 'react';
import { Button, Card, Icon, Statistic } from 'semantic-ui-react';
import './Timer.css';

type Props = {
  timeLeft?: number,
  isPrime?: boolean,
  reset?: () => void,
}

const Timer: VFC<Props> = ({ 
	timeLeft = 0;
  isPrime = false;
  reset = () => undefined;
}) => {

  return (
    <Card>
      <Statistic className="number-board">
        <Statistic.Label>time</Statistic.Label>
        <Statistic.Value className={isPrime ? 'prime-number' : undefined}>
          {timeLeft}
        </Statistic.Value>
      </Statistic>
      <Card.Content>
        <Button color="red" fluid onClick={reset}>
          <Icon name="redo" />
          Reset
        </Button>
      </Card.Content>
    </Card>
  );
};

export default Timer;
```

・App.tsx

```ts
import { VFC } from 'react';
import Timer from 'containers/Timer'; import './App.css';

constApp: VFC = () => (
  <div className="container">
    <header> 
      <h1>タイマー</h1>
    </header>
    <Timer limit={60} /> 
  </div>
);
export default App;
```























