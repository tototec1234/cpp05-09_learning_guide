# ex01 事前クイズ

> 対象: ex00完了後、`CPP06_ex01_解説.md`へ進む前  
> 目的: ポインタ値の往復と、一般的なserializationの違いを説明する

---

### Q1. この課題の`serialize`は、`Data`の各メンバをバイト列へ書き出すか。

**あなたの回答**:

<details>
<summary>模範回答（クリックで表示）</summary>

書き出さない。`Data*`というポインタ値を`uintptr_t`へ変換するだけである。オブジェクトの内容は同じ場所に残る。

</details>

---

### Q2. 往復後に確認する中心条件を式で書け。

**あなたの回答**:

<details>
<summary>模範回答（クリックで表示）</summary>

```cpp
deserialize(serialize(original)) == original
```

元と同じポインタ型へ戻し、ポインタ値が等しいことを確認する。

</details>

---

### Q3. `uintptr_t`ではなく`unsigned int`へ変換する設計の問題は何か。

**あなたの回答**:

<details>
<summary>模範回答（クリックで表示）</summary>

`unsigned int`がポインタ値を保持できる幅を持つ保証がない。情報が失われると元のポインタへ戻せない。課題はポインタを保持できる型として`uintptr_t`を指定している。

</details>

---

### Q4. `reinterpret_cast`で整数から作った任意のポインタを参照してよいか。

**あなたの回答**:

<details>
<summary>模範回答（クリックで表示）</summary>

よくない。課題で利用する保証は、正しいオブジェクトポインタを十分な大きさの整数へ変換し、その値を元と同じポインタ型へ戻す往復である。任意の整数が有効なオブジェクトを指す保証はない。

</details>

---

### Q5. `Data`オブジェクトを破棄した後、保存していた`uintptr_t`からポインタを戻して使えるか。

**あなたの回答**:

<details>
<summary>模範回答（クリックで表示）</summary>

使えない。数値を戻して同じアドレス表現を得ても、オブジェクトの寿命は復活しない。破棄済みオブジェクトを指すポインタを参照すると不正である。

</details>

---

### Q6. C++98指定と`uintptr_t`の関係を説明せよ。

**あなたの回答**:

<details>
<summary>模範回答（クリックで表示）</summary>

標準C++98には`uintptr_t`がない。C99の`<stdint.h>`で導入され、C++11では`<cstdint>`と`std::uintptr_t`が標準化された。本課題はC++98コンパイルと`uintptr_t`の両方を要求するため、評価環境が提供する`<stdint.h>`のグローバルな`uintptr_t`を使う構成が想定される。

</details>

---

次: `CPP06_ex01_解説.md`
