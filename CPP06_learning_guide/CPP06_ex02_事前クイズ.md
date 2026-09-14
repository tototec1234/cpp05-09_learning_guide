# ex02 事前クイズ

> 対象: ex01完了後、`CPP06_ex02_解説.md`へ進む前  
> 目的: 静的型、動的型、ポインタ版と参照版の失敗を区別する

---

### Q1. 次の`p`の静的型と、指しているオブジェクトの動的型は何か。

```cpp
Base *p = new B;
```

**あなたの回答**:

<details>
<summary>模範回答（クリックで表示）</summary>

`p`の静的型は`Base*`。オブジェクトの動的型は`B`。

</details>

---

### Q2. `Base`にpublicな仮想デストラクタだけを置く理由を二つ挙げよ。

**あなたの回答**:

<details>
<summary>模範回答（クリックで表示）</summary>

1. `Base*`を通した削除で、動的型に対応するデストラクタまで呼ぶため
2. `Base`をポリモーフィック型にし、`dynamic_cast`の実行時型検査を可能にするため

</details>

---

### Q3. `dynamic_cast<A*>(p)`に失敗すると何を返すか。`dynamic_cast<A&>(r)`に失敗すると何が起きるか。

**あなたの回答**:

<details>
<summary>模範回答（クリックで表示）</summary>

- ポインタ版: nullポインタを返す
- 参照版: `std::bad_cast`に一致する例外を送出する

</details>

---

### Q4. `identify(Base& p)`の中で`identify(&p)`を呼ぶ実装は許可されるか。

**あなたの回答**:

<details>
<summary>模範回答（クリックで表示）</summary>

許可されない。課題は参照版の関数内でポインタを使うことを禁止している。参照への`dynamic_cast`と失敗時の例外を使って判別する。

</details>

---

### Q5. `<typeinfo>`が禁止されているなら、`dynamic_cast`も禁止されるか。

**あなたの回答**:

<details>
<summary>模範回答（クリックで表示）</summary>

禁止されない。`dynamic_cast`は言語の演算子であり、このexerciseで使う中心機能である。禁止対象はヘッダ`<typeinfo>`である。

</details>

---

### Q6. ポインタのupcastで、派生オブジェクトのデータは失われるか。

**あなたの回答**:

<details>
<summary>模範回答（クリックで表示）</summary>

失われない。オブジェクトは変化せず、基底型の式から利用できるメンバが限定される。派生オブジェクトを基底オブジェクトの値へコピーするobject slicingとは別である。

</details>

---

次: `CPP06_ex02_解説.md`
