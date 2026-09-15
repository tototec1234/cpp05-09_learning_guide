# ex01 事後クイズ

> 対象: `CPP06_ex01_解説.md`を確認した後  
> 目的: ポインタと整数の往復保証を、適用条件とともに説明する

---

### Q1. `restored->number == original->number`だけで課題の往復を確認したことになるか。

**あなたの回答**:

<details>
<summary>模範回答（クリックで表示）</summary>

十分ではない。同じ値を持つ別オブジェクトの可能性がある。中心条件は`restored == original`というポインタ比較である。メンバ比較は追加確認になる。

</details>

---

### Q2. `serialize`の結果をファイルへ保存し、翌日の別プロセスで`deserialize`すれば元の`Data`を復元できるか。

**あなたの回答**:

<details>
<summary>模範回答（クリックで表示）</summary>

できない。元のオブジェクトは別プロセスには存在せず、同じ数値のアドレスが同じ対象を表す保証もない。この課題の往復は、生存中の同じオブジェクトと同じ実行環境を前提にする。

</details>

---

### Q3. なぜ`reinterpret_cast<unsigned int>(ptr)`ではなく`uintptr_t`を使うのか。

**あなたの回答**:

<details>
<summary>模範回答（クリックで表示）</summary>

`unsigned int`はポインタ値を保持できる幅を持つとは限らない。`uintptr_t`は、提供される処理系では`void*`を保持できる符号なし整数型として定義される。課題もこの型を指定している。

</details>

---

### Q4. 往復後のポインタからメンバを確認できるのは、Dataが整数内へ複製されたからか。

**あなたの回答**:

<details>
<summary>模範回答（クリックで表示）</summary>

違う。元と同じ生存中オブジェクトを、戻したポインタが指すからである。整数が保持するのはポインタ値であり、Dataのメンバではない。

</details>

---

### Q5. 次の順序の問題を説明せよ。

```text
new Data
serialize
delete Data
deserialize
restoredからメンバを参照
```

**あなたの回答**:

<details>
<summary>模範回答（クリックで表示）</summary>

`delete`でDataの寿命が終わっている。`deserialize`してアドレス表現を戻してもオブジェクトは再作成されない。`restored`を参照すると解放後使用になる。

</details>

---

### Q6. `<cstdint>`と`std::uintptr_t`を無条件に使うと、今回の提出条件で問題になる理由は何か。

**あなたの回答**:

<details>
<summary>模範回答（クリックで表示）</summary>

どちらもC++11で標準化された構成であり、課題はC++98コンパイルを要求する。対象環境では`<stdint.h>`とグローバルな`uintptr_t`を確認する。

</details>

---

### Q7. `Serializer`、`Data`、`main`について、提出前に確認する項目を一つずつ挙げよ。

**あなたの回答**:

<details>
<summary>模範回答（クリックで表示）</summary>

例:

- `Serializer`: 利用者がインスタンス化できず、2つのstatic関数を持つ
- `Data`: 一つ以上のデータメンバを持ち、必要なファイルを提出する
- `main`: 往復後のポインタを元のポインタと比較する

</details>

---

次: `CPP06_ex02_事前クイズ.md`
