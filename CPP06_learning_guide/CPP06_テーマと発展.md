# CPP06_テーマと発展

> 前提となるモジュール  
> - CPP00〜CPP05: クラス、継承、多態性、例外  
> - C: 文字列解析、整数型、メモリ上のアドレス

<a id="toc"></a>
## 目次

- [1. この文書の目的](#1-この文書の目的)
- [2. CPP06の全体像](#2-cpp06の全体像)
- [3. 4種類の名前付きキャスト](#3-4種類の名前付きキャスト)
- [4. ex00からex02への発展](#4-ex00からex02への発展)
- [5. 値、表現、実際の型](#5-値表現実際の型)
- [6. レビューコメントの検証](#6-レビューコメントの検証)
- [7. 課題の制約と評価対策](#7-課題の制約と評価対策)
- [8. 参考資料](#8-参考資料)

---

## 1. この文書の目的

CPP06の題は **C++ casts** である。このモジュールでは、構文だけでなく「何を変換し、どの保証が必要か」に応じてキャストを選ぶ。

実装前に、次を説明できる状態を目標とする。

- 暗黙変換と明示的変換の違い
- `static_cast`、`reinterpret_cast`、`dynamic_cast`、`const_cast`の担当範囲
- 値の変換と、ビット表現やアドレスの扱いの違い
- ダウンキャストで実行時確認が必要な理由
- キャスト式を書く前に、範囲や前提条件を確認する理由

完成コードは示さない。各exerciseの解説では、判断手順、疑似コード、テスト項目を示す。

[目次へ戻る](#toc)

---

## 2. CPP06の全体像

CPP06は、異なる3種類の「変換」を扱う。

1. **ex00 — scalar conversion**  
   文字列が表す値を判定し、`char`、`int`、`float`、`double`の間で値を変換する。
2. **ex01 — serialization**  
   `Data*`を、そのポインタ値を保持できる符号なし整数型へ変換し、同じポインタ型へ戻す。
3. **ex02 — real type identification**  
   `Base*`または`Base&`が参照するオブジェクトの実際の型を、実行時検査によって判別する。

3 exerciseは同じ操作を繰り返しているわけではない。

- ex00は、数値として意味のある**値変換**
- ex01は、オブジェクトへのアドレスを整数で保持する**表現上の変換**
- ex02は、継承階層内で実際の型を確認する**実行時検査付き変換**

この違いが、使用するキャストの違いになる。

```text
ex00: 文字列 → 値 → 別のスカラー型
                    static_cast

ex01: Data* → uintptr_t → Data*
                    reinterpret_cast

ex02: Base* / Base& → A・B・C の候補
                    dynamic_cast
```

`const_cast`は課題の主要処理には登場しない。ただし、4種類の名前付きキャストを区別するために学ぶ。

[キャスト選択の全体図](./CPP06_cast_overview_diagram.md)

[目次へ戻る](#toc)

---

## 3. 4種類の名前付きキャスト

### 3.1 `static_cast`

`static_cast`は、コンパイル時に変換規則を適用できる型変換に使う。
CPP06 ex00では、文字列を検出した型の値にしたあと、残りのスカラー型へ `static_cast` で変換する場面で使う。
```cpp
double value = 42.8;
int converted = static_cast<int>(value); // 42
```

浮動小数点数から整数への変換では、小数部分を0方向へ切り捨てる。ただし、切り捨て後の値を変換先の整数型で表せない場合、動作は未定義になる。`static_cast`は範囲を自動確認しない。

継承での用法は補足を参照。

<details>
<summary>補足（ex00では不要）: static_cast と継承階層</summary>

`static_cast`は継承階層でも使える。

- アップキャスト: 派生型から基底型へ変換する。安全な方向であり、明示しなくても暗黙変換できる場合が多い
- ダウンキャスト: 基底型から派生型へ変換する。コンパイルは通ることがあるが、実行時に実際の型を確認しない

ダウンキャストでオブジェクトの実際の型を呼び出し側が証明できない場合、`static_cast`では安全を保証できない。実行時検査が必要なダウンキャストは、ex02の`dynamic_cast`が担当する。

</details>

### 3.2 `reinterpret_cast`

`reinterpret_cast`は、ポインタと十分な大きさを持つ整数型の間など、低水準の変換に使う。

CPP06 ex01では次の往復だけを行う。

```text
Data* → uintptr_t → Data*
```

十分な大きさを持つ整数型へポインタを変換し、その整数を元と同じポインタ型へ戻すと、元のポインタ値が得られる。

この操作は、`Data`オブジェクトの中身をバイト列へ変換しない。データ圧縮、暗号化、永続化、ネットワーク送信も行わない。整数値を別のプロセスへ渡して復元できるという意味でもない。

### 3.3 `dynamic_cast`

`dynamic_cast`は、継承階層内のポインタまたは参照を変換し、必要な場合は実行時に型を確認する。

CPP06 ex02では、`Base`に仮想デストラクタがあるため、`Base`はポリモーフィック型になる。これにより、`Base*`や`Base&`から`A`、`B`、`C`へのダウンキャストを検査できる。

- ポインタへの変換に失敗: nullポインタを返す
- 参照への変換に失敗: `std::bad_cast`に一致する例外を送出する

課題では`<typeinfo>`が禁止されている。これは`dynamic_cast`の禁止ではない。参照版では`std::bad_cast`を名前で捕捉せず、`std::exception`または`...`で失敗を処理できる。

### 3.4 `const_cast`

`const_cast`は、`const`または`volatile`というcv修飾を変更する。

```cpp
const int *source = 0;
int *destination = const_cast<int *>(source);
```

キャスト式がコンパイルできても、元からconstであるオブジェクトを書き換えると動作は未定義になる。`const_cast`は書き換え可能性を作る機能ではない。

CPP06の必須処理には不要である。過去のレビューコメントが述べる「今回登場しなかった4種類目」という位置付けは正しい。

### 3.5 C形式キャストとの違い

C形式キャストは短いが、どの変換規則を使ったかがコード上で分かりにくい。

```cpp
int result = (int)value;
```

名前付きキャストは、実装者が意図した変換の種類を明示する。課題の追加規則は、exerciseごとに適切なキャストを選び、その理由を評価で説明できることを求めている。

[目次へ戻る](#toc)

---

## 4. ex00からex02への発展

### 4.1 ex00: 値の種類と表現可能範囲

入力はC++リテラルの文字列表現である。処理は次の4段階に分ける。

1. 引数の個数を確認する
2. 文字列全体を検査し、文字・整数・float・double・疑似リテラルを分類する
3. 分類した型の値へ変換する
4. 残り3型への変換可否を確認し、指定形式で表示する

難所はキャスト式ではない。次の判定である。

- 文字列の途中までしか解析できていない場合を拒否する
- `nan`と無限大を整数や文字へ変換しない
- `int`や`char`の範囲を超える値をキャストしない
- doubleがfloatの有限範囲を超える場合は、floatへキャストしない
- 表示できない`char`と、変換できない`char`を区別する
- `float`では末尾の`f`を扱う
- 整数値のfloat/doubleにも`.0`を付ける

[ex00変換フロー](./CPP06_ex00_conversion_flow_diagram.md)  
[範囲外変換と未定義動作](./CPP06_ex00_範囲外変換と未定義動作.md)

### 4.2 ex01: ポインタ値の往復

`Serializer`は状態を持たず、インスタンス化できないクラスである。公開するのは2つのstaticメンバ関数だけである。

```text
serialize(Data*)      → uintptr_t
deserialize(uintptr_t) → Data*
```

確認すべき結果は、`deserialize(serialize(original)) == original`である。

戻ったポインタが同じオブジェクトを指すため、メンバも同じ値として観測できる。ただし、これはメンバを複製した結果ではない。同じオブジェクトへ再びアクセスしている。

課題書はC++98を指定する一方、`uintptr_t`を要求している。`std::uintptr_t`と`<cstdint>`はC++11の標準ライブラリである。C++98の提出では、評価環境が提供する`<stdint.h>`のグローバルな`uintptr_t`を使う構成が想定される。提出環境で`-std=c++98`によるコンパイルを確認する。

[ex01データフロー図](./CPP06_ex01_data_flow_diagram.md)

### 4.3 ex02: 静的型と動的型

```cpp
Base *p = new A;
```

- 式`p`の静的型: `Base*`
- 指しているオブジェクトの動的型: `A`

`dynamic_cast<A*>(p)`は動的型を検査する。成功すれば`A*`、失敗すればnullポインタになる。

参照版`identify(Base&)`では、関数内でポインタを使うことが禁止されている。`&p`をポインタ版へ渡す実装は要件違反になる。`dynamic_cast<A&>(p)`の成功と例外を利用して判別する。

`Base`のpublicな仮想デストラクタは、次の2つの役割を持つ。

- `Base*`を通した削除で、動的型に対応するデストラクタまで呼ぶ
- `Base`をポリモーフィック型にし、`dynamic_cast`の実行時検査を可能にする

[ex02クラス関係図](./CPP06_ex02_class_overview_diagram.md)  
[ex02判別フロー](./CPP06_ex02_identify_flow_diagram.md)

[目次へ戻る](#toc)

---

## 5. 値、表現、実際の型

3 exerciseを区別する軸は次のとおりである。

| exercise | 守りたいもの | 変換前後 | 主な危険 |
| --- | --- | --- | --- |
| ex00 | 数値としての意味 | スカラー値 → 別のスカラー値 | 範囲外変換、解析漏れ、表示形式 |
| ex01 | ポインタ値 | オブジェクトポインタ → 整数 → 同じポインタ型 | 整数型の幅、用途の誤解 |
| ex02 | 継承階層内の型関係 | 基底ポインタ・参照 → 派生ポインタ・参照 | 検査なしのダウンキャスト |

### 5.1 promotionとconversion

promotionは、狭い型の値を、値を保てる広い型へ変換する標準変換の一部である。例は`char`から`int`への整数昇格や、`float`から`double`への浮動小数点昇格である。

upcastは、派生クラスのポインタまたは参照を、公開基底クラスのポインタまたは参照へ変換することである。promotionとは別の分類である。

### 5.2 upcastとdowncast

- upcast: `A*`から`Base*`。公開継承なら暗黙変換できる
- downcast: `Base*`から`A*`。実際のオブジェクト型を確認する必要がある

「upcastで派生部分のデータが失われる」という説明は、ポインタ・参照の変換には当てはまらない。オブジェクト自体は変化しない。基底型の式から派生クラス固有のメンバへ直接アクセスできなくなるだけである。

派生オブジェクトを基底クラスの**値**へコピーする操作では、object slicingが起きる。これはポインタ・参照のupcastと分けて説明する。

### 5.3 キャストは検証処理の代わりではない

名前付きキャストは、変換の意図を表す。すべてのキャストが安全性を保証するわけではない。

- `static_cast`: 数値範囲を確認しない
- `reinterpret_cast`: 整数が有効なオブジェクトを指すことを証明しない
- `dynamic_cast`: 継承階層の実行時検査を行う
- `const_cast`: 元からconstのオブジェクトを書き換え可能にはしない

各キャストの前後で、値域、寿命、型関係、所有権を別に確認する。

[目次へ戻る](#toc)

---

## 6. レビューコメントの検証

過去のコメントは、評価で問われやすい論点を知る材料になる。ただし、次の補足と訂正が必要である。

### 6.1 正しい指摘

- staticデータメンバはクラス単位で一つの実体を共有する
- staticメンバ関数には`this`がなく、非staticメンバへオブジェクトなしではアクセスできない
- `static_cast`は変換先で値を安全に表せるか自動確認しない
- `reinterpret_cast`によるポインタと整数の変換は低水準であり、用途を限定する
- `dynamic_cast`は検査付きダウンキャストに使用できる
- `<typeinfo>`は禁止されている
- `std::isprint`などへ負の`char`値を直接渡すと未定義動作になり得る
- 擬似乱数のseed設定は、最初の`rand()`より前に一度行う

### 6.2 訂正が必要な指摘

**「upcastすると派生クラス独自部分がデータ落ちする」**  
ポインタまたは参照のupcastでは、オブジェクトのデータは失われない。基底型の式を通して見えるメンバが限定される。データが失われるobject slicingは、派生オブジェクトを基底オブジェクトの値へコピーした場合に起きる。

**「ex01のserializeは暗号化に近い」**  
暗号化ではない。整数値を知っていても、同じプロセス内で対象オブジェクトが生存しているなどの前提が必要である。秘密性、改ざん検出、永続化の保証はない。

**「ポインタと整数以外のreinterpret_castは未定義動作」**  
一括した説明として不正確である。`reinterpret_cast`には複数の許可された変換があり、結果の保証や利用可能な操作が変換ごとに異なる。ex01では、課題指定のポインタと`uintptr_t`の往復だけを説明対象にする。

**「downcastは中身がないため不正アクセスになる」**  
問題は「中身が追加される」ことではない。実際のオブジェクトが変換先の派生型ではないのに、その型として利用することが危険である。`dynamic_cast`は継承階層と動的型を検査する。

### 6.3 条件を付ける指摘

**`std::time()`は秒単位**  
`std::time`が返す`std::time_t`の表現と分解能は処理系定義であり、C++規格が秒単位の整数と保証するわけではない。多くの環境では秒単位として使われる。短時間に同じ値でseedを設定すると同じ列が生じやすいため、`srand`を各`generate()`呼び出しで実行しない。

**`uintptr_t`はC由来、`std::uintptr_t`はC++11から**  
`uintptr_t`はC99の`<stdint.h>`で導入された。C++11は`<cstdint>`と`std::uintptr_t`を標準化した。C++98そのものには標準型として存在しないが、本課題は名前を明示的に要求している。

[目次へ戻る](#toc)

---

## 7. 課題の制約と評価対策

### 7.1 モジュール共通

- [ ] `c++ -Wall -Wextra -Werror`でビルドでき、`-std=c++98`を加えても成功する
- [ ] C++11以降、Boost、外部ライブラリを使わない
- [ ] `printf`系、`alloc`系、`free`を使わない
- [ ] 許可がない`using namespace`と`friend`を使わない
- [ ] STLコンテナと`<algorithm>`を使わない
- [ ] ヘッダにインクルードガードがあり、単独でインクルードできる
- [ ] テンプレート以外の関数実装をヘッダへ置かない
- [ ] 課題が指定した例外を除き、クラスをOrthodox Canonical Formで設計する
- [ ] exerciseごとに選んだキャストと理由を説明できる
- [ ] 数分で行える小規模な変更に対応できる

### 7.2 ex00

- [ ] `ScalarConverter`を利用者がインスタンス化できない
- [ ] `convert(std::string)`が唯一のpublic staticメンバ関数である
- [ ] char、int、float、double、6種類の疑似リテラルを扱う
- [ ] 入力文字列全体を検証し、末尾の不要文字を受理しない
- [ ] 非表示文字では`Non displayable`、変換不能では`impossible`を出す
- [ ] 範囲外、NaN、無限大を整数や文字へキャストしない
- [ ] doubleからfloatへの変換前に、floatの有限範囲を確認する
- [ ] `numeric_limits<float>::min()`を負側の有限範囲として使わない
- [ ] 課題例に合う`.0`と`f`を表示する

### 7.3 ex01

- [ ] `Serializer`を利用者がインスタンス化できない
- [ ] `Data`は一つ以上のデータメンバを持つ
- [ ] `serialize`と`deserialize`で`reinterpret_cast`を使う
- [ ] 往復後のポインタが元のポインタと等しい
- [ ] `Data`オブジェクトが生存している間に戻ったポインタを使う
- [ ] `Data`用のファイルを提出する

### 7.4 ex02

- [ ] `Base`はpublicな仮想デストラクタだけを持つ
- [ ] `A`、`B`、`C`は空で、`Base`をpublic継承する
- [ ] `generate()`がA、B、Cを選んで`Base*`として返す
- [ ] `identify(Base*)`がポインタ版`dynamic_cast`で判別する
- [ ] `identify(Base&)`が関数内でポインタを使わない
- [ ] `<typeinfo>`をインクルードしない
- [ ] 生成したオブジェクトを`Base*`から安全に削除する
- [ ] A、B、Cが複数回の実行で選ばれることをテストする

この一覧は公式evaluation sheetではない。課題書と過去のレビューコメントから作った確認項目である。

[目次へ戻る](#toc)

---

## 8. 参考資料

- [static_cast（cppreference）](https://en.cppreference.com/w/cpp/language/static_cast)
- [暗黙変換・浮動小数点数と整数の変換（cppreference）](https://en.cppreference.com/w/cpp/language/implicit_cast)
- [reinterpret_cast（cppreference）](https://en.cppreference.com/w/cpp/language/reinterpret_cast)
- [dynamic_cast（cppreference）](https://en.cppreference.com/w/cpp/language/dynamic_cast)
- [const_cast（cppreference）](https://en.cppreference.com/w/cpp/language/const_cast)
- [固定幅整数型と`uintptr_t`（cppreference）](https://en.cppreference.com/w/cpp/types/integer)
- [`std::isprint`（cppreference）](https://en.cppreference.com/w/cpp/string/byte/isprint)
- [文字分類関数の引数（CERT STR37-C）](https://wiki.sei.cmu.edu/confluence/display/c/STR37-C.+Arguments+to+character-handling+functions+must+be+representable+as+an+unsigned+char)

次: `CPP06_ex00_事前クイズ.md`
