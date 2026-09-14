# ex02 解説 — `dynamic_cast`による実型判別

## 関連資料

- [モジュール全体](./CPP06_テーマと発展.md)
- [クラス関係図](./CPP06_ex02_class_overview_diagram.md)
- [判別フロー図](./CPP06_ex02_identify_flow_diagram.md)

---

## 1. 課題の構造

必要な型は4つである。

```text
Base
├─ A
├─ B
└─ C
```

`Base`はpublicな仮想デストラクタだけを持つ。`A`、`B`、`C`は空で、`Base`をpublic継承する。

この4クラスは、課題書によってOrthodox Canonical Formを免除されている。

必要な関数は次の3つである。

```cpp
Base *generate(void);
void identify(Base *p);
void identify(Base &p);
```

- `generate`: A、B、Cのいずれかを作り、`Base*`として返す
- ポインタ版`identify`: 実際の型をA、B、Cとして表示する
- 参照版`identify`: ポインタを使わずに実際の型を表示する

---

## 2. 静的型と動的型

```cpp
Base *p = new C;
```

この式には二つの型が関係する。

- 静的型: コンパイル時に式へ付く型。ここでは`Base*`
- 動的型: 実際に作られたオブジェクトの型。ここでは`C`

`p->`から直接利用できるメンバは、静的型`Base`のpublicインターフェースで決まる。`dynamic_cast`は、実行時に動的型と継承関係を確認する。

---

## 3. `Base`の仮想デストラクタ

### 3.1 ポリモーフィック型にする

C++では、少なくとも一つの仮想関数を持つクラスをポリモーフィック型として扱う。実行時検査を伴う`dynamic_cast`には、元の型がポリモーフィックであることが必要になる。

`Base`の仮想デストラクタが、その条件を満たす。

### 3.2 基底ポインタから削除する

`generate()`は具体型を作るが、戻り値は`Base*`である。

```text
new A / new B / new C
        ↓ upcast
      Base*
```

呼び出し側が`delete base;`を実行したとき、仮想デストラクタによって動的型に対応するデストラクタから順に破棄される。

---

## 4. `generate()`

`generate()`は、A、B、Cを偏りなく厳密に選ぶことまでは要求されていない。3候補のいずれかをランダムに生成する。

疑似コード:

```text
choice = rand() % 3
if choice == 0: return new A
if choice == 1: return new B
return new C
```

### seedを設定する場所

`srand`は、最初の`rand()`より前に一度呼ぶ。

```text
main:
    srand(...)
    繰り返し generate()
```

`generate()`の呼び出しごとに秒単位で得た同じseedを設定すると、短時間の呼び出しが同じ乱数列の先頭を使い、同じ型を返し続ける場合がある。

`std::time()`が返す`time_t`の表現と分解能は処理系定義である。「C++規格上、必ず秒単位の整数」とは説明できない。対象環境では時刻をseed材料として使う例が多い。

`rand() % 3`は、`RAND_MAX + 1`が3で割り切れない処理系では候補ごとの出現確率に小さな差が出る。この課題は統計的品質や暗号用途を求めていないため、3型を選ぶ方法として使用できる。テストでA、B、Cを確実に確認する場合は、乱数とは別に各型を直接作る。

---

## 5. ポインタ版`identify`

ポインタへの`dynamic_cast`は、失敗時にnullポインタを返す。

疑似コード:

```text
if dynamic_cast<A*>(p) がnullではない:
    "A"
else if dynamic_cast<B*>(p) がnullではない:
    "B"
else if dynamic_cast<C*>(p) がnullではない:
    "C"
else:
    課題外の型またはnull
```

A、B、C以外の派生型は課題に登場しないが、nullポインタを渡した場合の処理を決めておくと関数の挙動が明確になる。

`static_cast<A*>(p)`でもコンパイルできる場合があるが、実際のオブジェクトがAか確認しない。結果をAとして利用できる保証を作れないため、この判別には適さない。

---

## 6. 参照版`identify`

参照への`dynamic_cast`にはnullという失敗結果がない。変換に失敗すると`std::bad_cast`に一致する例外が送出される。

課題は、`identify(Base& p)`内でポインタを使うことを禁止している。

許可されない骨格:

```text
identify(Base& p):
    identify(&p)
```

許可される考え方:

```text
identify(Base& p):
    A&へのdynamic_castを試す
      成功: "A"を出してreturn
      失敗: 次へ
    B&へのdynamic_castを試す
      成功: "B"を出してreturn
      失敗: 次へ
    C&へのdynamic_castを試す
      成功: "C"を出す
```

`<typeinfo>`が禁止されているため、`std::bad_cast`を名前で使う実装は避ける。失敗だけを次候補へ進めるなら`catch (...)`を使える。`<exception>`から利用できる`std::exception`として捕捉する方法も、対象環境でコンパイルを確認する。

catch節では、型判別以外の処理を入れない。予想外の例外まで無視する設計を一般化しない。

---

## 7. `<typeinfo>`禁止の意味

禁止されているのはヘッダ`<typeinfo>`のインクルードである。

この課題では次を行う。

- 使用する: `dynamic_cast`
- 使用しない: `typeid`と`std::type_info`を使う設計
- 名前で捕捉しない: `<typeinfo>`で宣言される`std::bad_cast`

`dynamic_cast`は実行時型情報を利用するが、演算子を使うためにソースコードへ`<typeinfo>`を書く必要はない。

コンパイラオプションでRTTIを無効化すると、この課題の実行時検査が成立しない。課題指定の標準的なコンパイル設定を使う。

---

## 8. upcast、downcast、object slicing

### upcast

```text
A* → Base*
```

public継承では暗黙に変換できる。ポインタが指すAオブジェクトは変化しない。

### downcast

```text
Base* → A*
```

実際のオブジェクトがAか、その派生型である必要がある。`dynamic_cast`はこれを実行時に検査する。

### object slicing

```cpp
A derived;
Base base = derived;
```

基底型の値へコピーすると、コピー先にはBase部分だけが含まれる。これがobject slicingである。

過去のレビューコメントにある「upcastで派生部分がデータ落ちする」という説明は、ポインタ・参照には当てはまらない。見えるインターフェースがBaseに限定されても、元のオブジェクト内の派生部分は残る。

---

## 9. テスト

### 9.1 基本

```text
複数回:
    Base* object = generate()
    identify(object)
    identify(*object)
    delete object
```

各回で、ポインタ版と参照版が同じ文字を出すことを確認する。

### 9.2 型を固定したテスト

乱数だけに依存すると、A、B、Cのすべてを一回の実行で確認できる保証がない。

```text
A a
B b
C c
identify(a), identify(b), identify(c)
```

ポインタ版も同様に固定した各型で確認する。

### 9.3 追加

- `identify(static_cast<Base*>(0))`の扱い
- 生成と削除を多数回繰り返し、リークがないか確認
- seedを一度だけ設定した場合と、毎回設定した場合の違いを観察

---

## 10. 実装前チェック

- [ ] 静的型と動的型を説明できる
- [ ] `Base`の仮想デストラクタの役割を二つ説明できる
- [ ] ポインタ版の失敗はnull、参照版の失敗は例外と説明できる
- [ ] 参照版の関数内でポインタを使わない
- [ ] `<typeinfo>`をインクルードしない
- [ ] `srand`を`generate()`のたびに呼ばない
- [ ] A、B、Cを固定したテストも行う
- [ ] `Base*`から`delete`する
- [ ] 4クラスがOCF免除であることを確認する

## 参考資料

- [dynamic_cast（cppreference）](https://en.cppreference.com/w/cpp/language/dynamic_cast)
- [仮想デストラクタ（cppreference）](https://en.cppreference.com/w/cpp/language/virtual)
- [`std::srand`（cppreference）](https://en.cppreference.com/w/cpp/numeric/random/srand)
- [`std::time`（cppreference）](https://en.cppreference.com/w/cpp/chrono/c/time)

次: `CPP06_ex02_事後クイズ.md`
