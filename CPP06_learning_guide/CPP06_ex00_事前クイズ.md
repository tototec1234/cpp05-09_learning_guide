# ex00 事前クイズ

> 対象: `CPP06_テーマと発展.md`を確認した後、`CPP06_ex00_解説.md`へ進む前  
> 目的: 値の分類、変換可能性、表示要件を言葉で説明する

先に自分の回答を書く。分からなければ`?`と書き、その後に模範回答を確認する。

---

### Q1. `static_cast<int>(42.9)`の結果は何か。`static_cast<int>(nan)`にも同じ規則を適用してよいか。

**あなたの回答**:

<details>
<summary>模範回答（クリックで表示）</summary>

`42`。浮動小数点数から整数への変換は小数部分を0方向へ切り捨てる。  
NaNは整数で表せないため、整数へキャストしてはいけない。表現不能な浮動小数点値から整数への変換は未定義動作になる。

</details>

---

### Q2. `char: Non displayable`と`char: impossible`は何が違うか。

**あなたの回答**:

<details>
<summary>模範回答（クリックで表示）</summary>

- `Non displayable`: `char`へ変換できるが、表示可能文字ではない。例: 数値0
- `impossible`: NaN、無限大、または`char`の範囲外など、意味のある変換結果を作れない

</details>

---

### Q3. 文字列`"42abc"`を数値変換関数が42まで解析できた。入力42として受理してよいか。

**あなたの回答**:

<details>
<summary>模範回答（クリックで表示）</summary>

受理しない。入力文字列全体が一つのリテラル形式に一致する必要がある。終了位置を取得できる変換関数を使う場合、解析後の位置が文字列末尾か確認する。

</details>

---

### Q4. `42.0f`と`42.0`をどの型のリテラルとして分類するか。末尾の`f`は何を表すか。

**あなたの回答**:

<details>
<summary>模範回答（クリックで表示）</summary>

`42.0f`は`float`、`42.0`は`double`。末尾の`f`は浮動小数点リテラルを`float`型にする接尾辞である。

</details>

---

### Q5. 課題書が指定する疑似リテラルを、float用とdouble用に分けてすべて挙げよ。

**あなたの回答**:

<details>
<summary>模範回答（クリックで表示）</summary>

- float: `-inff`、`+inff`、`nanf`
- double: `-inf`、`+inf`、`nan`

</details>

---

### Q6. `std::isprint(c)`へplain `char`を直接渡すと問題になる場合がある。どのように渡すか。

**あなたの回答**:

<details>
<summary>模範回答（クリックで表示）</summary>

`static_cast<unsigned char>(c)`を経由して渡す。

```cpp
std::isprint(static_cast<unsigned char>(c))
```

文字分類関数の引数は、`EOF`または`unsigned char`で表せる値でなければならない。

</details>

---

### Q7. `ScalarConverter`をインスタンス化できないようにする理由と、C++98で使える方法を述べよ。

**あなたの回答**:

<details>
<summary>模範回答（クリックで表示）</summary>

このクラスは状態を持たず、publicな機能がstaticな`convert`だけだからである。コンストラクタ、コピーコンストラクタ、コピー代入演算子をprivateで宣言し、利用者から呼べないようにする。C++11の`= delete`は使わない。

</details>

---

### Q8. 入力が`0`のとき、課題例どおりの4行を書け。

**あなたの回答**:

<details>
<summary>模範回答（クリックで表示）</summary>

```text
char: Non displayable
int: 0
float: 0.0f
double: 0.0
```

</details>

---

次: `CPP06_ex00_解説.md`
