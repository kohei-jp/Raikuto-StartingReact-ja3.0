#### 第4章 TypeScript で型をご安全に

Typescriptの特徴
　静的型付け、型推論、Null 安全性

動的型付け言語の型を書かなくていいというアドバンテージは薄まり、いっぽうで堅牢なコードが効率 よく書ける静的型付け言語のメリットが強調されるようになった。



TypeScript は大規模なアプリケーション開発のために作られた、JavaScript に静的型付けの型システムとクラスベースのオブジェクト指向を加えた完全上位互換の言語。

< プリミティブ型 >
・Boolean 型 ...... true および false の 2 つの真偽値を扱うデータ型。型名は boolean   
・Number 型 ...... 数値を扱うためのデータ型。型名は number
・BigInt 型 ...... number 型では表現できない大きな数値(253 以上)を扱う型。型名は bigint 
・String 型 ...... 文字列を扱うためのデータ型。型名は string
・Symbol 型 ......「シンボル値」という固有の識別子を表現する値の型。型名は symbol
・Null 型 ...... 何のデータも含まれない状態を明示的に表す値。型名は null
・Undefined 型 ......「未定義」であることを表す値。型名は undefined



問1  P.160

以下、Statusの [attr: string]: number; はどういう機能か。

```ts
interface Status { 
  level: number;
	maxHP: number;
	maxMP: number;
	[attr: string]: number;
}

const myStatus: Status = { 
  level: 99,
	maxHP: 999,
	maxMP: 999,
	attack: 999,
	defense: 999
};
```

A. 
インデック スシグネチャ(Index Signature)と言って、任意のキーとプロパティを定義している。
ちなみにインデックスシグネチャのキーに使える型は、文字列と数値の 2 種類のみ。

問2 P.160  
以下のenum型を条件型のリテラル型で書き換えて下さい。

```ts
enum Pet{
 Cat='Cat',
 Dog='Dog',
 Rabbit='Rabbit'
}
```

A.

```ts
type Pet = 'Cat'|'Dog'|'Rabbit'
```



問3 P.163
個々の要素の型と、その順番や要素数に制約を設けられる特殊な配列の型を何というか。
A.

タプル型

 ```ts
const charAttrs: [number, string, boolean] = [1, 'patty', true];
 ```



問4 P.163
以下のタプル型でレストパラメータを使用しているが、具体的な代入例をしてして下さい。

```ts
const spells: [number, ...string[]] = ・・・
```

A.

```ts
const spells: [number, ...string[]] = [1, "A", "B", "C"]
```



問5 P.168

以下は、inlineで型を定義しているが、引数を纏めて定義し、関数を「呼び出し可能オブジェクト」として定義して下さい。

```ts
const add = function(n: number, m: number): number { 
  return n + m;
};
console.log(add(5,7)); //12 
```

A.

```ts
interface NumOp {
  (n:number, m:number): number
}

const add:NumOp = function(n, m) { 
  return n + m;
};
```



問6 P.169
以下①、②はどの様な結果になるか。

```ts
const toArray = <T>(arg1: T, arg2: T): T[] => [arg1, arg2];
① toArray('foo', 'bar');
② toArray(5, 'bar');
```

A.

① [foo, bar]　② error

関数の型宣言にジェネリクスを用いた記法であり、関数を実行時に引数を渡し型が決まる。
< >として型を渡している。Tは、Type Parameter(型引数)のT。
→ 事前に型を決め打ちしない分、再利用性が上がる。
上記の例では、arg1とarg2が同じ型にしないといけないので、②では失敗している。



問7 P.170

問6と同様に、ジェネリクスを用いて可変長の引数を受け取れるような関数を定義して下さい。
A. 

```ts
const toArrayVariablly = <T>(...args: T[]): T[] => [...args];
toArrayVariablly(1,2,3,4,5) #=> [1,2,3,4,5]
```

問8 P.171
以下のnameプロパティの役割は何か。

```ts
class Rectangle {
  readonly name = 'rectangle'; 
}  
```

A.
『プロパティ初期化子(Property Initializer)』という機能。
引数を渡さなくても、name = 'rectangle'という初期値が設定出来る。
つまり、コンストラクタに引数がないクラスでは、インスタンスの初期化をこれだけで済ませてコンストラクタを省略できる。
以下とやっていることは一緒。

```ts
class Rectangle {
  name: string;
  
  constructor() {
    name = 'rectangle'
  }
}  
```

問9 P.174
Typescriptでクラスの型を抽象化して定義する方法
A.
・abstract 修飾子 を用いて抽象クラスを定義する
　→ 抽象クラス自身はインスタンスが生成できず、継承されることを前提としている。

・インターフェース
　→ 実装を伴わずに型だけを定義できる。
　※ 実装を伴うと親クラスで何をしているか把握する必要があり、クラス間の依存を産む。保守性悪い。

抽象クラスをよりも、インターフェースが推奨される。

```ts
interface Shape { 
  readonly name: string; 
  getArea: () => number; // () => number; メンバーメソッド(アロー関数)の型を定義している。
}

interface Quadrangle { 
  sideA: number; 
  sideB?: number; 
  sideC?: number; 
  sideD?: number;
}
// 複数のinterfaceを適応
class Rectangle implements Shape, Quadrangle { 
  readonly name = 'rectangle';
  sideA: number;
  sideB: number;
  
  constructor(sideA: number, sideB: number) { 
    this.sideA = sideA;
    this.sideB = sideB;
  }
  getArea = (): number => this.sideA * this.sideB; 
}
```

問10 P.176
クラスはインターフェースとして使用できるか。
A.

可能。
クラスは、通常のクラスとしての振る舞いと、型のコンテキスト(文脈)ではインターフェースとして扱われる。

```ts
class Point {
  x: number = 0; 
  y: number = 0;
}
const pointA = new Point();
const pointB: Point = { x: 2, y: 4 };　// Pointというインターフェースとして扱われている
```



cf.

・型エイリアス(type-aliase)

```ts
type Unit = 'USD' | 'EUR' | 'JPY' | 'GBP'; // ① リテラルに別名を与えている

type TCurrency = { // ② 型を定義
  unit: Unit; 
  amount: number;
};
interface ICurrency { 
  unit: Unit; 
  amount: number;
}
const priceA: TCurrency = { unit: 'JPY', amount: 1000 }; // ②
const priceB: ICurrency = { unit: 'USD', amount: 10 };
```

型エイリアスは、マップ型や条件付き型といった高度な型構文が記述 できるけど、
インターフェースではそれができない。
→ 現在は型エイリアスの方が表現力が高く、インフーフェースを優先して使用する理由がなくなっている。



問11 P.181
以下AorB、AorCのような型は共用体(Union Types)と言われるが、どのような型になっているか。

```ts
typeA={
  foo: number; 
  bar?: string;
};
type B = { foo: string }; 
type C = { bar: string };

type AorB = A|B;
type AorC = A|C;
```

A.
共用体型は、AまたはBのように適応範囲を増やしていく。

```ts
AorB // {foo:number|string; bar?:string} bar?の理由は、typeBであるならbarは不要のため
AorC // {foo:number;bar?:string} or {bar:string}

// また、参考までに、nullを許容する型を定義する際は、以下のように表記する。
type foo = string | null
```



問12 P.181
AかつBのような型は交差型( Intersection Types )と言われるが、
AnBやAnCはどのような型になっているか。

```ts
type A = { foo: number }; 
type B = { bar: string };
type C = {
  foo?: number;
  baz: boolean; 
};
type AnB = A & B;
type AnC = A & C;
type CnAorB = C & (A | B);
```

A.

```ts
type AnB = A & B; // { foo: number, bar: string }; 
type AnC = A & C; // { foo: number, baz: boolean; }
type CnAorB = C & (A | B); 
// {foo: number, baz: boolean; } or {bar: string, foo?: number, baz: boolean;}
```

問13 P.186
100 から型を抽出して下さい。
A

**typeof**を使用すると、型推論で既存の変数から抜き出せるので便利。

```ts
console.log(typeof 100); // 'number'
```



問14 P.187

以下のval、val2はコンパイルが成功するか。

```ts
const arr = [1, 2, 3];
type NumArr = typeof arr;

const val: NumArr = [4, 5, 6]; 
const val2:NumArr=['foo','bar','baz']; 
```

A.

```ts
NumArr は、Number[] になるので、val2はエラーになる。
```



問15 P.187
以下の **in演算子** の役割は何か。

```ts
type Fig = 'one' | 'two' | 'three'; 
type FigMap = { [k in Fig]?: number };

const figMap: FigMap = { 
  one: 1,
  two: 2,
  three: 3,
};
```

A.
・[k in Fig] では、文字列リテラル型の共用体である Fig から各要素の型の値を抜き出す。
→ 'one' か  'two' か 'three'を許可する。
　今回は、{ [k in Fig]?: number } なので、hashのキーがこれで、valueはnumberになる型。

```ts
// ちなみに、以下のin演算子は obj のキーに 'a' があるかどうか。
const obj = { a:1, b:2, c:3 };
console.log( 'a' in obj );
//=> true

// 以下のin演算子は、objからインクリメンタルにキーを抽出している。
for( const key in obj ){ console.log(key); } //abc

→ どの用途の in演算子 なのか区別する。
```



問16 P.187

以下の **keyof** を使用したPermsCharの結果はどうなるか。

```ts
const permissions = { 
  r: 0b100,
  w: 0b010,
  x: 0b001,
};

type PermsChar = keyof typeof permissions; // => ?
```

A.

オブジェクトからキーのみを抽出する。

```ts
type PermsChar = keyof typeof permissions;
=> 'r'|'w'|'x'

cf. 
typeof permissions では以下のように型推論される。これにkeyofすると、キーの型が抽出できるということ

const permissions: {
  r: number;
  w: number;
  x: number;
}
```



問17 P.188
上記の permissions から、PermsChar を使用してvalueの型のみを抽出して下さい。

A.

```ts
type PermsChar = keyof typeof permissions;
type PermsNum = typeof permissions[PermsChar]

前問のcfにあるように、typeof permissions で、このオブジェクトの型が抽出出来る。
このオブジェクトに対して、obj[key] のようにすることで、value(の型)が抽出出来る。
```



問18 P.188
上記で作成したオブジェクトからvalueの型を抽出した以下の型をジェネリクス<>を用いて汎用的に表現して下さい。

```tsx
type PermsNum = typeof permissions[PermsChar]
```

A.

```ts
type ValueOf<T> = T[keyof T]
// Tには型引数を渡す。今回の例では、typeof permissions になり、keyof Tでは、キーのみが抽出される。
// そのキーを使用して、obj[key]の形で、valueの型のみを抽出している。

type PermsNum = ValueOf< typeof permissions >; // 1 | 2 | 4
```



問19 P.189

問18では省略していたが、PermsNumは、実際には以下のままでは、numberと型推論されてしまう。
これを、```0b100 | 0b010 | 0b001```とするにはどうすれば良いか。

```ts
const permissions = { 
  r: 0b100,
  w: 0b010,
  x: 0b001,
} 
type ValueOf<T> = T[keyof T]
type PermsNum = ValueOf< typeof permissions >; // => number;
```

A.
as const構文である**Const アサーション**を使用して、定数としての型注釈を付与する。

```ts
const permissions = { 
  r: 0b100,
  w: 0b010,
  x: 0b001,
}as const; 

type ValueOf<T> = T[keyof T]
type PermsNum = ValueOf< typeof permissions >; // => 0b100 | 0b010 | 0b001;
```



問20 P.189
以下の species で定義されている4つのプロパティのどれかを許容する型を作成して下さい。

```ts
const species = ['rabbit', 'bear', 'fox', 'dog']
```

A.

species の配列に**as const**を付与する => 付与しないと、*string[]* と推論される。
*typeof species [number]*とする。 
=> [number]を抜かすと、この *['rabbit', 'bear', 'fox', 'dog']* の配列を許可する型になる。

```ts
const species = ['rabbit', 'bear', 'fox', 'dog'] as const;
type Species = typeof species[number];
```



問21 P.190
以下①、②のoverride関数を実行すると、どのような結果になるか。

```ts
const override = <T, U extends T>(obj1: T, obj2: U): T & U => ({
  ...obj1,
  ...obj2,
});

① override( {a:1}, {a:24, b:8} ); 
② override( {a: 2}, {x: 73} );
```

A.

```ts
① override( {a:1}, {a:24, b:8} ); //  {a:24, b:8}
② override({ a: 2 }, { x: 73 }); // compile error!

第一引数の型はT、 第二引数の型はTを拡張したUである。
UはTと同じか拡張したものでないといけないので、②の例ではUはaのキーを持っていないオブジェクトで
あるので、コンパイルで弾かれる。
```



問22 P.190
以下の型はどういう挙動を示すか。

```ts
T extends U ? X : Y
ex)
type User = { id: unknown };
type NewUser = User & { id: string }; 
type OldUser = User & { id: number }; 
type Book = { isbn: string };

type IdOf<T> = T extends User ? T['id'] : never;

type NewUserId = IdOf<NewUser>; // ?
type OldUserId = IdOf<OldUser>; // ?
type BookId = IdOf<Book>;       // ?
```

A.
**条件付き型(Conditional Types)**と呼ばれ、
型 T が 型 U を拡張していた場合は型 X を、それ以外の場合は型 Y となる。

オブジェクトの型から任意のプロパティの型を抽出したりするとき なんかに使える。
他にも関数の型から任意の引数の型を抽出したりといった使いみちも考えられる。

```ts
type NewUserId = IdOf<NewUser>; // string
type OldUserId = IdOf<OldUser>; // number
type BookId = IdOf<Book>;       // never
```



問23 P.191
以下の type AとN の結果はどうなるか。

```ts
type Flatten<T> = T extends Array<infer U> ? U : T;
const num = 5;
const arr = ["3", "6", "9"]; 

type N = Flatten<typeof num>; // ?
type A = Flatten<typeof arr>; // ?
```

A.
**infer** は、マッチングした型の出力ができる。
例えば、Flatten では、T がなんらかの配列だった場合、その配列の中身の型を **infter U**で型Uとして取得出来る。
→ A では、Tが配列で中身が文字列なので、Uにはstringが入る。

→ N では、Tが5で数字なので U にはマッチせず、条件式はfalseなので、5



問24 P.192

以下のDateFormatの型を定義して下さい。

```ts
const date1: DateFormat = '2020-12-05';
```

A.
テンプレートリテラル型を用いる。

```ts
type DateFormat = `${number}-${number}-${number}`
```



問25 P.193
組み込みユーティリティ型は、以下の種類があるが、これを自作するとどうなるか。

```ts
Partial<T>  ...... T のプロパティをすべて省略可能にする
Required<T> ...... T のプロパティをすべて必須にする
Readonly<T> ...... T のプロパティをすべて読み取り専用にする

type Partial = ?
type Required = ?
type Readonly = ?
```

A.

```ts
type Partial<T> = { [K in keyof T]?: T[K] }
type Required<T> = { [K in keyof T]: T[K] }
type Readonly<T> = { reaonly [K in keyof T]: T[K]}

const fig = { a: "1", b: "2", c: "3" }
type figk = keyof (typeof fig); // a | b | c  # ユニオン型
figの様なオブジェクトから抽出する場合は、一旦、tyoepf objで型推論が必要だが、
keyof T とすることで、オブジェクトのkeyの型を抽出する。
in演算子は、ユニオン型( a|b|c )からどれか抜き出す。
```



問26 P.194

以下、PickedTodo、OmittedTodoの型はどの様になるか。

```ts
type Todo = {
  title: string; 
  description: string; 
  isDone: boolean;
};

type PickedTodo = Pick<Todo, 'title' | 'isDone'>;
type OmittedTodo = Omit<Todo, 'description'>;
```

A.

```ts
PickedTodoは、 Todoから 'title' | 'isDone'を抜き出している。
//=> { title: string, isDone: boolean } 

OmittedTodoは、Todoから 'description' を除いている。
//=> { title: string, isDone: boolean } 
```



cf. そのほかのユーティリティ

・列挙的な型を加工するユーティリティ型

　・Extract<T,U> ...... T から U の要素だけを抽出する 

　・Exclude<T,U> ...... T から U の要素を省く

```ts
type Permission = 'r' | 'w' | 'x';
type RW1 = Extract<Permission, 'r' | 'w'>;　// 'r' | 'w'
type RW2 = Exclude<Permission, 'x'>;　　　　 // 'r' | 'w'
```



・任意の型から null と undefined だけを省いて null 非許容にするためのユーティリティ型

　・NonNullable< T > ...... T から null と undefined を省く

```ts
type T1 = NonNullable<string | number | undefined>; 
type T2 = NonNullable<number[] | null | undefined>;
const str:T1 = undefined; // compile error!
const arr:T2 = null;      // compile error!
```



・列挙タイプの型をキーとしたオブジェクト の型を作成するもの

　・Record<K,T> ...... K の要素をキーとしプロパティ値の型を T としたオブジェクトの型を作成する

```ts
type Animal = 'cat' | 'dog' | 'rabbit'; 
type AnimalNote = Record<Animal, string>;　// keyがanimal、valueはstring

const animalKanji: AnimalNote = { 
  cat:    '猫',
  dog:    '犬',
  rabbit: '兎',
};
```



・関数を扱うユーティリティ型

　・Parameters< T > ...... T の引数の型を抽出し、タプル型で返す 

　・ReturnType< T > ...... T の戻り値の型を返す

```ts
const f1 = (a: number, b: string) => { console.log(a, b); }; 
const f2 = () => ({ x: 'hello', y: true });
type P1 = Parameters<typeof f1>; // [number, string] 
type P2 = Parameters<typeof f2>; // [] 
type R1 = ReturnType<typeof f1>; // void
type R2 = ReturnType<typeof f2>; // {x: string; y: boolean}
```



・文字列リテラル型と組み合わせて便利に使える型

　・Uppercase< T >    ...... T の各要素の文字列をすべて大文字にする

　・Lowercase< T >    ...... T の各要素の文字列をすべて小文字にする

　・Capitalize< T >      ...... T の各要素の文字列の頭を大文字にする

　・Uncapitalize< T >  ...... T の各要素の文字列の頭を小文字にする

```ts
type Company = 'Apple' | 'IBM' | 'GitHub';
type C1 = Lowercase<Company>;    // 'apple' | 'ibm' | 'github' 
type C2 = Uppercase<Company>;    // 'APPLE' | 'IBM' | 'GITHUB' 
type C3 = Uncapitalize<Company>; //'apple'|'iBM'|'gitHub'
type C4 = Capitalize<C3>;        // 'Apple' | 'IBM' | 'GitHub';
```



cf. 関数のオーバーロード P.197

同じ関数名でも引数の型や個数が違うと別のメソッドとして実行出来る。

```ts
// オーバーロードのインターフェース 引数違いで複数定義できる。
function transform(): void;
function transform(item: Brooch): void;
function transform(item: Compact): void;

// 実際の関数の実装は一つに集約する。 全てのパターンを網羅しておかないといけない。
// 引数の型でif文で条件分岐。
function transform(item?: Brooch | Compact): void {
  if (item instanceof Brooch) {
    console.log('Mooncrystalpower ,makeup!!'); } 
  else if (item instanceof CosmicCompact) {
    console.log('Mooncosmicpower ,makeup!!!'); } 
  else if (item instanceof CrisisCompact) { 
    console.log('Mooncrisis ,makeup!');
  } else if (!item) { 
    console.log('Moonprisimpower ,makeup!');
  }else{ 
    console.log('Itemisfake... ');
  } 
}

transform();
transform(new Brooch()); transform(new CosmicCompact());
transform(new CrisisCompact());
```



cf. ユーザ定義の型ガード

```ts
type Address = { zipcode: string; town: string };
type User = { username: string; address: Address };

// isUserの戻り値が「arg is User」となっている。
// これは「型述語」と言って、trueを返す場合は、argはUserであることを示す。
const isUser = (arg: unknown): arg is User => {
  const u = arg as User;

  return (
    typeof u?.username === 'string' &&
    typeof u?.address?.zipcode === 'string' &&
    typeof u?.address?.town === 'string'
  );
};

const u1: unknown = JSON.parse('{}');
const u2: unknown = JSON.parse('{ "username": "patty", "address": "Maple Town" }');
const u3: unknown = JSON.parse(
  '{ "username": "patty", "address": { "zipcode": "111", "town": "Maple Town" } }',
);

[u1, u2, u3].forEach((u) => {
  if (isUser(u)) { // 各 u1, u2, u3 の値がUserであれば、trueを返す。
    console.log(`${u.username} lives in ${u.address.town}`);
  } else {
    console.log("It's not User");
//  console.log(`${u.username} lives in ${u.address.town}`);  /* compile error */
  }
})
```

