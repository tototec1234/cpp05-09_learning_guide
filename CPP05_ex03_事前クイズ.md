# ex03 事前クイズ

> 対象: ex02 が動いたあと、`CPP05_ex03_解説.md` の前

先に自分の回答を書く。そのあと `模範回答（クリックで表示）` をクリックして開き確認すること。

---

### Q1. Intern が持たないものは何か。唯一の仕事は何か。戻り値の型は何か。


**あなたの回答**:

<details>
<summary>模範回答（クリックで表示）</summary>

名前も等級も個性もない。  
`makeForm(formName, target)` で具体 Form をヒープに作り、`AForm*` を返す。

> 関連解説: [Intern の役割と `makeForm` の戻り値](CPP05_ex03_解説.md#intern-role)
</details>

---

### Q2. 課題書の例は `"robotomy request"` である。クラス名 `RobotomyRequestForm` をキーにすると何が起きるか。


**あなたの回答**:

<details>
<summary>模範回答（クリックで表示）</summary>

例のコードが動かない。評価は例の文字列を使う。キーは課題の短い名前（`"robotomy request"` など）。
> 関連解説: [課題書で使われているキー文字列](CPP05_ex03_解説.md#form-name-keys)

</details>

---

### Q3. 「if / else if / else の森は評価で受け付けない」と課題が書く。禁止されるのは読みにくさである。許可されうる代替を2つ挙げよ。STL の `std::map` は使えるか。

**あなたの回答**:

<details>
<summary>模範回答（クリックで表示）</summary>

名前の配列と関数ポインタの配列を並行に走査する。名前配列で index を決め、`switch` は index に対して1回。  
`std::map` はコンテナなので Module 05 では禁止（-42）。

> 関連解説: [課題が拒否する分岐](CPP05_ex03_解説.md#forbidden-if-chain) / [Form名と生成関数の対応表](CPP05_ex03_解説.md#form-creator-table)

</details>

---

### Q4. `makeForm` が `new` したオブジェクトは、誰が `delete` するか。Intern のデストラクタか。


**あなたの回答**:

<details>
<summary>模範回答（クリックで表示）</summary>

呼び出し側。Intern はポインタを返すだけで所有しない。Intern のデストラクタで delete すると、返したあとのオブジェクトまで壊す。
> 関連解説: [`makeForm` が返すポインタの所有権](CPP05_ex03_解説.md#ownership)

</details>

---

### Q5. 未知のフォーム名のとき、課題が要求するのは何か。例外は必須か。


**あなたの回答**:

<details>
<summary>模範回答（クリックで表示）</summary>

明確なエラーメッセージを出すこと。例外にするかは本文に必須と書いていない。  
返すなら `NULL` とメッセージ、または例外。`NULL` なら呼び出し側がチェックする。例外なら `catch` する。どちらでも、メッセージなしの沈黙は不可。
> 関連解説: [存在しないForm名への対応](CPP05_ex03_解説.md#unknown-form-name)

</details>

---

次: `CPP05_ex03_解説.md`
