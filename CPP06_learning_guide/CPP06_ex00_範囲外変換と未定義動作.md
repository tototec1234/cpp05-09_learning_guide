# ex00補足 — 範囲外変換と未定義動作

## 1. `static_cast`は値域を確認しない

`static_cast<int>(value)`がコンパイルできることと、その実行が定義済みであることは別である。

浮動小数点数から整数への変換では小数部分を0方向へ切り捨てる。切り捨てた値を変換先の整数型で表せない場合、動作は未定義になる。

```text
42.9   → 42     定義済み
-42.9  → -42    定義済み
+inf   → int    未定義
nan    → int    未定義
範囲外 → int    未定義
```

課題が`impossible`の表示を求める場合、危険なキャストを実行してから結果を調べることはできない。先に条件を確認する。

---

## 2. 上限比較の落とし穴

次の形は一見すると値域を確認している。

```cpp
value <= static_cast<float>(std::numeric_limits<int>::max())
```

しかし、`INT_MAX`を`float`へ変換した値が丸められ、実際の`INT_MAX`より大きい値になる処理系がある。32-bit IEEE 754 floatでは、`2147483647`を正確に表せず、近い値へ丸める。

その丸められた境界値を通過したfloatをintへキャストすると、intの範囲外になる可能性がある。

### 現実的な方針

- 判定を、入力値を正確に含められる広い型で行う
- 上限について、変換先で表せる最後の区間を意識する
- NaNと無限大を値域比較より前に除外する
- 境界付近を実際のコンパイラと実行環境でテストする

この補足の目的は、完成した境界判定式を提示することではない。「キャスト前に確認する」という処理にも、型変換が含まれる点を理解することである。

---

## 3. NaNは比較で範囲内にならない

NaNとの大小比較はfalseになる。

```text
nan < min  → false
nan > max  → false
nan == nan → false
```

次の条件だけでは、NaNを安全な値として扱ってしまう。

```text
if value < min or value > max:
    impossible
else:
    cast
```

NaNと無限大を独立して判定する。課題指定の疑似リテラルを文字列段階で処理することも有効である。

---

## 4. `char`の範囲は処理系に依存する

plain `char`がsignedかunsignedかは処理系定義である。

```cpp
std::numeric_limits<char>::min()
std::numeric_limits<char>::max()
```

数値からcharへ変換する前に、この範囲を確認する。範囲外の整数をcharへ変換した結果を、ASCIIの下位8 bitとして扱えるとは限らない。

さらに、charへ変換できても表示可能とは限らない。

```text
値域確認 → static_cast<char> → isprintの確認
```

`std::isprint`は、引数値が`EOF`または`unsigned char`で表せる値でない場合に未定義動作となる。plain `char`を使う場合は次の形にする。

```cpp
std::isprint(static_cast<unsigned char>(character))
```

---

## 5. doubleからfloatへの範囲外変換

有限のdoubleでも、floatの有限範囲を超える値がある。C++98では、その値をfloatへ変換する処理が未定義動作になり得る。

```text
if value < -numeric_limits<float>::max()
   または value > numeric_limits<float>::max():
    float: impossible
else:
    floatへ変換
```

`numeric_limits<float>::min()`は負側の下限ではない。正規化された正の値のうち、0に最も近い値を表す。負側の有限範囲を確認するには`-numeric_limits<float>::max()`を使う。

---

## 6. 未定義・処理系定義・精度低下を分ける

### 未定義動作

規格が結果を定めない。浮動小数点値から整数への範囲外変換が該当する。実行して結果を利用してはいけない。

### 処理系定義または処理系依存の差

処理系が選択し、文書化する性質がある。plain `char`のsignednessなどが該当する。

### 精度低下

変換は成立するが、元の値を正確に表せない。大きいintからfloatへの変換などが該当する。変換不能と同じ意味ではない。

---

## 7. テスト候補

- `INT_MIN`、`INT_MAX`
- その直前と直後を表せる入力
- `CHAR_MIN`、`CHAR_MAX`
- 0、31、32、126、127
- `nan`、`nanf`
- `+inf`、`-inf`、`+inff`、`-inff`
- `std::numeric_limits<float>::max()`付近と、それを超えるdouble入力
- 非常に大きい指数表記
- 変換関数が途中まで解析できる`42abc`

AddressSanitizerやUndefinedBehaviorSanitizerは補助になるが、すべての未定義動作を必ず検出するわけではない。仕様に基づく事前条件の確認が必要である。

## 参考資料

- [Implicit conversions（cppreference）](https://en.cppreference.com/w/cpp/language/implicit_cast)
- [static_cast（cppreference）](https://en.cppreference.com/w/cpp/language/static_cast)
- [`std::numeric_limits`（cppreference）](https://en.cppreference.com/w/cpp/types/numeric_limits)
- [`std::isprint`（cppreference）](https://en.cppreference.com/w/cpp/string/byte/isprint)
