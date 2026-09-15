# ex00 解説 — スカラー値の分類と変換

## 関連資料

- [モジュール全体](./CPP06_テーマと発展.md)
- [変換フロー図](./CPP06_ex00_conversion_flow_diagram.md)
- [範囲外変換と未定義動作](./CPP06_ex00_範囲外変換と未定義動作.md)

---

## 1. 課題の要求

`ScalarConverter`は、文字列で渡されたC++リテラルを判定し、次の順序で結果を出す。

1. `char`
2. `int`
3. `float`
4. `double`

publicなメンバは、次のstaticメンバ関数だけである。

```cpp
static void convert(std::string const &literal);
```

クラスは利用者がインスタンス化できないようにする。C++98では、コンストラクタ、コピーコンストラクタ、コピー代入演算子をprivateで宣言する方法を使える。

課題書が求める処理順は次のとおりである。

```text
リテラルの型を検出
    ↓
文字列から、その実際の型の値へ変換
    ↓
残り3型へ明示的に変換
    ↓
4型を指定形式で表示
```

---

## 2. 先に入力を分類する

分類候補は次の5群である。

- 1文字の表示可能な`char`
- 符号付き10進`int`
- 小数点または指数部があり、末尾に`f`を持つ`float`
- 小数点または指数部があり、`f`を持たない`double`
- 課題指定の疑似リテラル

課題書はchar以外について10進表記だけを扱うと定めている。16進数、8進数、2進数を独自に追加する必要はない。

### 2.1 判定順序

短い入力から先に確定すると、分岐を整理しやすい。

```text
if 課題指定の疑似リテラル:
    float または double
else if 長さ1で、数字ではない表示可能文字:
    char
else:
    数値として文字列全体を解析
    接尾辞fと、小数点・指数部から型を分類
```

`"0"`は文字コード0を直接表すcharリテラルではなく、整数リテラル0として扱う。`"a"`はcharとして扱う。

課題書はcharの例を`'c'`、`'a'`と表記している。一方、シェルで`./convert 'a'`を実行すると、引用符はシェルが解釈し、プログラムには1文字の`a`が渡る。引用符そのものを含む3文字入力も受理するのか、1文字入力だけを受理するのかを決め、テストと説明を一致させる。

### 2.2 文字列全体を確認する

`atoi`は失敗と0を区別できず、解析終了位置も返さない。C++98で利用できる`strtol`や`strtod`など、終了位置を取得できる関数を使うと、次を確認できる。`strtof`はC99由来なので、使う場合は対象のC++98環境が宣言を提供するか確認する。

- 1文字以上を解析したか
- 解析終了位置が入力末尾か
- floatでは、数値部分の直後に`f`が一つだけあるか
- 範囲エラーが発生したか

課題書は、stringからint、float、doubleへ変換する任意の関数を許可している。関数を呼ぶだけでは入力検証は完了しない。

### 2.3 実装を分ける単位

完成コードを一つの巨大な`convert`へ書くと、分類、変換、表示の条件が混ざる。privateな補助関数へ次の処理を分けられる。

```text
classify(literal)        # 入力種別を返す
parse...(literal)        # 実際の型へ変換する
canConvertToInt(value)   # キャスト前の値域確認
printChar(value)
printInt(value)
printFloat(value)
printDouble(value)
```

補助関数もstaticにすれば、`ScalarConverter`のオブジェクトは不要である。

---

## 3. `static_cast`を書く前に確認する

### 3.1 浮動小数点数から整数

有限の値を整数へ変換すると、小数部分は0方向へ切り捨てられる。

```text
42.9  → 42
-42.9 → -42
```

切り捨て後の値を`int`で表せない場合、動作は未定義になる。NaNと無限大も`int`へキャストしない。

```text
if NaN または無限大:
    int: impossible
else if intの変換可能範囲外:
    int: impossible
else:
    static_cast<int>(value)
```

上限判定では、`INT_MAX`をfloatへ変換した結果が丸められる場合にも注意する。詳細は補足資料を参照する。

### 3.2 数値からchar

charへの処理は2段階で考える。

1. `char`として表せるか
2. 表示可能文字か

```text
表せない       → impossible
表せるが非表示 → Non displayable
表示できる     → 'x'
```

`std::numeric_limits<char>`で範囲を確認し、変換後の文字を`std::isprint`へ渡す。文字分類関数には`unsigned char`へ変換した値を渡す。

```cpp
std::isprint(static_cast<unsigned char>(character))
```

### 3.3 floatとdoubleへの変換

整数をfloatまたはdoubleへ変換できても、すべての整数を正確に表せるとは限らない。たとえば、32-bitの`int`の全値を32-bit IEEE 754 `float`が一対一には表せない。

課題の`impossible`は、変換結果を表現できない場合に使う。精度が落ちることと、変換不能は同じではない。入力値、処理系の型範囲、課題の期待を分けて判断する。

巨大な有限`double`を`float`へ変換する場合は、精度低下だけでなくfloatの有限範囲を超える可能性がある。C++98の変換規則では、変換先の浮動小数点型で表せる範囲を外れる変換が未定義動作になり得る。`static_cast<float>`より前に`std::numeric_limits<float>::max()`を使って正負両側を確認する。

`std::numeric_limits<float>::min()`は、最も小さい負の有限値ではない。正規化された正の有限値のうち、0に最も近い値を返す。負側の有限範囲には`-std::numeric_limits<float>::max()`を使う。

---

## 4. 疑似リテラル

必須の6種類は次のとおりである。

```text
float : -inff  +inff  nanf
double: -inf   +inf   nan
```

疑似リテラルでは、charとintは`impossible`になる。floatとdoubleは対応する表記を出す。

例:

```text
入力 nan
char: impossible
int: impossible
float: nanf
double: nan
```

`nan`には正負や大小比較を使った判定が適さない。C++98環境で利用できる数値関数や、`value != value`というNaNの性質を使う方法がある。提出環境でヘッダと関数の利用可否を確認する。

疑似リテラルを先に文字列として判定すれば、NaNや無限大を整数へ誤ってキャストする経路を閉じられる。

---

## 5. 出力形式

課題例は、値が整数であるfloatとdoubleにも小数部を表示する。

```text
float: 42.0f
double: 42.0
```

`std::fixed`を無条件に使うと、`4.2`を`4.200000`のように変える。必要なのは「値に小数部分がない場合に`.0`を補う」処理であり、すべてを固定小数点表記へ変えることではない。

考え方:

```text
if 有限値 かつ 小数部分が0:
    ".0"を補う
floatには最後に"f"を付ける
```

ストリームの書式状態は、その後の出力にも残る。`std::setprecision`や`std::fixed`を使う場合、どこまで影響するかを確認する。

---

## 6. レビューコメントから確認する点

### 6.1 staticメンバ

過去コメントの次の説明は、要点として正しい。

- staticデータメンバは、各インスタンスではなくクラス単位で共有する
- staticメンバ関数には`this`がない

ただし、「staticメンバ関数はstaticデータにしかアクセスできない」は省略がある。オブジェクトへの参照やポインタを引数として受け取れば、そのオブジェクトのpublicメンバへアクセスできる。できないのは、対象オブジェクトを指定せずに非staticメンバへアクセスすることである。

`ScalarConverter::convert`は、クラス自身のインスタンス状態を必要としないためstaticにする。

### 6.2 promotionとupcast

promotionとupcastは別の概念である。

- promotion: `char`から`int`、`float`から`double`など
- upcast: `Derived*`から`Base*`など

ex00の中心はスカラー値変換であり、継承階層のupcastやdowncastではない。

### 6.3 `std::isprint`

レビューコメントの指摘どおり、plain `char`がsignedで負の値になる処理系では、`std::isprint(c)`が未定義動作になる場合がある。

引数を`unsigned char`へ変換してから渡す。結果は現在のC localeにも依存する。

---

## 7. テスト設計

### 7.1 必須例

- `0`
- `nan`
- `42.0f`
- `-inff`、`+inff`、`nanf`
- `-inf`、`+inf`

### 7.2 境界と形式

- 1文字: `a`、`*`
- 非表示: `0`、`31`、`127`
- 符号: `-42`、`+42`
- 小数: `4.2f`、`-4.2`、`42.0`、`-0.0`
- 指数: 対応する設計なら`1e3`、`1e3f`
- int境界: `INT_MIN`、`INT_MAX`とその外側
- float境界: `std::numeric_limits<float>::max()`付近と、それを超えるdouble入力
- char境界: 処理系の`char`範囲付近

### 7.3 不正入力

- 空文字列
- `42abc`
- `4.2ff`
- `f`
- `--42`
- `.`
- 課題指定外の疑似リテラル

引数が一つではない場合のメッセージと終了方法も確認する。

---

## 8. 実装前チェック

- [ ] 分類、解析、表示を別の処理として説明できる
- [ ] 入力全体を解析したか確認できる
- [ ] NaNと無限大を整数・charへキャストしない
- [ ] intとcharの範囲をキャスト前に確認する
- [ ] doubleからfloatへの変換前にfloatの有限範囲を確認する
- [ ] `numeric_limits<float>::min()`を負側境界として使わない
- [ ] `Non displayable`と`impossible`を区別する
- [ ] floatの`f`と、整数値に付ける`.0`を処理する
- [ ] `std::isprint`へ安全な値を渡す
- [ ] `ScalarConverter`をインスタンス化できない
- [ ] `-std=c++98 -Wall -Wextra -Werror`で確認する

次: `CPP06_ex00_事後クイズ.md`
