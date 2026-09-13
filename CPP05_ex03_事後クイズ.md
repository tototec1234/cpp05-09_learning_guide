# ex03 事後クイズ

> 対象: `CPP05_ex03_解説.md` のあと。CPP05 の締め

先に自分の回答を書く。そのあと `模範回答（クリックで表示）` をクリックして開き確認すること。

---

### Q1. 次は課題の禁止に当たるか。理由を書け。


```
AForm *Intern::makeForm(...) {
    if (name == "shrubbery creation") return new ShrubberyCreationForm(target);
    if (name == "robotomy request")   return new RobotomyRequestForm(target);
    if (name == "presidential pardon") return new PresidentialPardonForm(target);
    エラー; return NULL;
}
```

`else` が無い。

**あなたの回答**:

<details>
<summary>模範回答（クリックで表示）</summary>

当たる。else の有無ではなく、種類ごとに分岐が並ぶ森である。  
表＋走査に置き換える。
> 関連解説: [課題が拒否する分岐](CPP05_ex03_解説.md#forbidden-if-chain)

</details>

---

### Q2. `std::map<std::string, FormCreator>` で名前と生成関数を結ぶ。C++ としてはきれいである。提出してよいか。


**あなたの回答**:

<details>
<summary>模範回答（クリックで表示）</summary>

いけない。STL コンテナは Module 08/09 まで禁止。評価は -42 と課題書にある。  
C 配列と関数ポインタで同じことをする。
> 関連解説: [Form名と生成関数の対応表](CPP05_ex03_解説.md#form-creator-table)

</details>

---

### Q3. `makeForm` に登録されていないフォーム名を渡すと、エラーメッセージを表示して `NULL` が返された。`main` が戻り値を検査せず、続けて `b.signForm(*form)` を実行すると、プログラムはどうなるか。


**あなたの回答**:

<details>
<summary>模範回答（クリックで表示）</summary>

`NULL` ポインタを `*form` で参照外しするため未定義動作となり、典型的には segmentation fault で異常終了する。  
`makeForm` が `NULL` を返す設計なら、参照外しの前に必ず `form != NULL` を確認する。未知名で例外を送出する設計なら、`NULL` を参照外しする経路自体が生じない。
> 関連解説: [`NULL` を返す場合の検査](CPP05_ex03_解説.md#null-handling)

<details>
<summary>「参照外し」とは何か（クリックで表示）</summary>

参照外し（dereference）とは、ポインタに `*` 演算子を適用し、そのポインタが指しているオブジェクトへアクセスすること。

```cpp
AForm* form = /* AForm オブジェクトのアドレス */;
b.signForm(*form);  // form が指す AForm オブジェクトを引数として渡す
```

`form == NULL` の場合は指しているオブジェクトが存在しない。その状態で `*form` を使用すると、未定義動作になる。

</details>

</details>

---

### Q4. Intern のデストラクタで「今まで作った Form を全部 delete する」設計は、なぜこの課題に合わないか。


**あなたの回答**:

<details>
<summary>模範回答（クリックで表示）</summary>

Intern は作ったポインタを保持していない。戻り値として所有権を渡している。  
保持してデストラクタで消すと、呼び出し側が使っているオブジェクトを破棄する。二重 delete にもなる。
> 関連解説: [`makeForm` が返すポインタの所有権](CPP05_ex03_解説.md#ownership)

</details>

---

### Q5. 評価者が「種類を4つ目に足すならどこを直すか」と聞いた。表方式と if 方式で、答えはどう違うか。


**あなたの回答**:

<details>
<summary>模範回答（クリックで表示）</summary>

表: 生成関数を1つ足し、表に1行足す。`makeForm` の制御流れは変えない。  
if: `makeForm` に分岐を足す。関数本体が伸び続ける。  
課題が表を求める理由を、この質問で確認している。
> 関連解説: [Form名と生成関数の対応表](CPP05_ex03_解説.md#form-creator-table)

</details>

---

### Q6. CPP05 全体。次の文は正しいか。誤りなら直す。

「`GradeTooHighException` は、等級の数値が大きすぎるときに投げる。」


**あなたの回答**:

<details>
<summary>模範回答（クリックで表示）</summary>

誤り。数値が小さすぎる（1 未満）= 権限が高すぎる、ときに投げる。  
数値が 150 超は TooLow。ex00 から ex02 の実行判定まで、この向きが共通。
> 関連解説: [CPP05全体における等級の向き](CPP05_ex03_解説.md#module-review)

</details>

---

### Q7. 自分の提出前チェックリストを、ファイル名 / 例外 / メモリ / Intern の4項目で書け。


**あなたの回答**:

<details>
<summary>模範回答（クリックで表示）</summary>

例:  
ファイル名: `Bureaucrat` / `AForm` / 3 Form / `Intern`。`Bureaucat` ではない。  
例外: `what()` 定義あり。High/Low が逆でない。`catch (std::exception &)` で捕まる。  
メモリ: `makeForm` の戻り値を `delete`。`AForm` デストラクタが virtual。  
Intern: 課題のキー文字列。if の森ではない。未知名でメッセージ。
> 関連解説: [提出前に確認するテスト項目](CPP05_ex03_解説.md#tests)

</details>

---

CPP05 の教材はここまで。実装は exercise 順。ex00 の等級の向きをテストで固定してから先へ進む。
