# ex00 事後クイズ

> 対象: `CPP06_ex00_解説.md`を確認した後、実装前または実装中  
> 目的: 変換前の検査と課題指定の出力を説明できるか確認する

---

### Q1. 次の処理の問題点を述べよ。

```cpp
int converted = static_cast<int>(value);
if (converted > std::numeric_limits<int>::max())
    std::cout << "impossible" << std::endl;
```

**あなたの回答**:

<details>
<summary>模範回答（クリックで表示）</summary>

範囲外の浮動小数点値をintへキャストした時点で未定義動作になり得る。変換後のintが`INT_MAX`を超えることもないため、比較では検出できない。NaN、無限大、値域をキャスト前に確認する。

</details>

---

### Q2. `strtod("42abc", &end)`が42を返した。何を確認すれば入力を拒否できるか。

**あなたの回答**:

<details>
<summary>模範回答（クリックで表示）</summary>

`end`が入力末尾を指しているか確認する。`a`を指していれば、文字列全体を解析していないため拒否する。1文字も解析していない場合も拒否する。

</details>

---

### Q3. plain `char`がsignedの環境で、値`-1`を`std::isprint`へ直接渡す危険を説明せよ。

**あなたの回答**:

<details>
<summary>模範回答（クリックで表示）</summary>

`-1`が`EOF`と一致する場合を除き、負の値は`unsigned char`で表せる値ではない。文字分類関数の契約に反し、未定義動作になる。char値を`unsigned char`へ変換してから渡す。

</details>

---

### Q4. `static_cast<float>(INT_MAX)`を上限比較に使うと、境界判定を誤る場合があるのはなぜか。

**あなたの回答**:

<details>
<summary>模範回答（クリックで表示）</summary>

floatが`INT_MAX`を正確に表せず、近い値へ丸める場合があるからである。丸められた値が実際の上限より大きいと、intへ変換できないfloatを範囲内と判定する可能性がある。

</details>

---

### Q5. 入力`nanf`に対する4行を書け。

**あなたの回答**:

<details>
<summary>模範回答（クリックで表示）</summary>

```text
char: impossible
int: impossible
float: nanf
double: nan
```

</details>

---

### Q6. `std::fixed << std::setprecision(1)`を全float/double出力へ適用する欠点は何か。

**あなたの回答**:

<details>
<summary>模範回答（クリックで表示）</summary>

必要な精度を持つ値まで小数1桁へ丸める。課題例の`.0`を満たすことと、すべての値を小数1桁へ制限することは別である。整数値か確認して`.0`を補う方法を検討する。

</details>

---

### Q7. staticメンバ関数が、非staticメンバへ「一切アクセスできない」という説明が不正確な理由は何か。

**あなたの回答**:

<details>
<summary>模範回答（クリックで表示）</summary>

staticメンバ関数には暗黙の`this`がないため、自分の非staticメンバを対象なしでは使えない。一方、引数で受け取ったオブジェクトやポインタを通せば、そのオブジェクトのアクセス可能な非staticメンバを使える。

</details>

---

### Q8. 最低限のテスト集合を、正常入力、境界、疑似リテラル、不正入力から2件ずつ挙げよ。

**あなたの回答**:

<details>
<summary>模範回答（クリックで表示）</summary>

例:

- 正常入力: `a`、`42.0f`
- 境界: `INT_MIN`、`INT_MAX`
- 疑似リテラル: `nan`、`+inff`
- 不正入力: `42abc`、`4.2ff`

加えて、0、31、32、126、127付近を使うとcharの表示判定を確認できる。

</details>

---

### Q9. `ScalarConverter`のprivateコンストラクタを宣言するだけでよい場合と、定義が必要になる場合を説明せよ。

**あなたの回答**:

<details>
<summary>模範回答（クリックで表示）</summary>

利用者からのインスタンス化を禁止するだけなら、privateで宣言し、どこからも呼ばないため定義しない構成にできる。クラス自身やfriendから実際に呼ぶ設計なら定義が必要になる。この課題ではインスタンスを作らない。

</details>

---

### Q10. `std::numeric_limits<float>::min()`を、floatの最も小さい負の有限値として使えるか。

**あなたの回答**:

<details>
<summary>模範回答（クリックで表示）</summary>

使えない。`min()`は正規化された正の有限値のうち、0に最も近い値を返す。有限範囲の負側は`-std::numeric_limits<float>::max()`を使って確認する。doubleからfloatへ変換する前に、正負両側の範囲を確認する。

</details>

---

次: `CPP06_ex01_事前クイズ.md`
