# ex03 解説 — Intern と生成の分離

提出物: ex02 まで + `Intern.hpp` / `Intern.cpp`  
官僚は書類の種類を `new` しなくてよくなる。文字列を Intern に渡す。
## 関連図

- [クラス関係図](CPP05_ex03_class_overview_diagram.md) — Intern、AForm、具体Form、Bureaucratの関係
- [データフロー図](CPP05_ex03_data_flow_diagram.md) — Formの生成から署名・実行・破棄までの流れ
- [設計比喩](CPP05_ex03_設計比喩.md) — Intern／Bureaucrat／main の役割分担（結合度・所有権・IRCとの短い対応表）
- [ex02→ex03「28Bじゃなくて28C」](CPP05_ex02_ex03_form28B_not_28C.md) — インターンなしの取り違えと Intern 窓口化の対比

---
---

<a id="intern-role"></a>

## 1.  Intern クラスがあると何が便利？

> **課題書原文**
>
> Since filling out forms all day would be too cruel for our bureaucrats, interns exist to take on this tedious task. In this exercise, you must implement the Intern class. The intern has no name, no grade, and no unique characteristics. The only thing bureaucrats care about is that they do their job.

この段落は、`Intern` が追加された背景と、`Intern` が固有の状態を持たず、仕事を実行するために存在することを説明している。

```
ex02: main が ShrubberyCreationForm f("home"); と型を直接書く
ex03: Intern が "shrubbery creation" から AForm* を返す
```

> **課題書原文**
>
> However, the intern has one key ability: the makeForm() function. This function takes two strings as parameters: the first one represents the name of a form, and the second one represents the target of the form. It returns a pointer to a AForm object (corresponding to the form name passed as a parameter), with its target initialized to the second parameter.

この段落は、`makeForm()` の2つの引数、戻り値、および生成するFormのtargetを定めている。

- Internクラスに仕事を任せれば、呼び出し側（`main`）は、`ShrubberyCreationForm` などの具体型を直接 `new` しなくてよくなる。
	-`Intern` くんに書類名の文字列を渡し、返ってきた `AForm*` を使えばよい。  

- 便利さの核は、隠蔽ではない。
	- **書類名の文字列と、どの具体型を生成するかという対応（生成規則）を、`Intern` 一箇所にまとめて書く**ことで利便性が向上する。
	- 書類の種類を足すとき、呼び出し側の分岐を増やすのではなく、書き足す場所が `Intern` になる。

IRC の B 層で、コマンド名からハンドラを選ぶ dispatcher と同じ問題である。  

---

<a id="forbidden-if-chain"></a>

## 2. 課題が拒否する形

> **課題書原文**
>
> You must avoid unreadable and messy solutions, such as using an excessive if/el-seif/else structure. This kind of approach will not be accepted during the evaluation process. You’re not in the Piscine (pool) anymore.

この段落は、過剰な `if/el-seif/else` を使った読みにくい実装が評価では認められないことを明記している。

if/else の森は、種類が増えるたびに分岐が伸び、読み手が条件を追い切れなくなる。

```
if (name == "shrubbery creation")
    return new ShrubberyCreationForm(target);
else if (name == "robotomy request")
    return new RobotomyRequestForm(target);
else if (name == "presidential pardon")
    return new PresidentialPardonForm(target);
else
    エラー
```

---

<a id="form-creator-table"></a>

## 3. Form名と生成関数を対応させる

書類名の配列と、生成関数の配列を同じ順番で用意する。  
この課題の範囲では、コンテナを使わず、C の配列で書く。

```cpp
型 FormCreator = AForm* (*)(string const & target)

createShrubbery(target):  return new ShrubberyCreationForm(target)
createRobotomy(target):   return new RobotomyRequestForm(target)
createPardon(target):     return new PresidentialPardonForm(target)

文字列 formNames[]:
  "shrubbery creation"
  "robotomy request"
  "presidential pardon"

FormCreator creators[]:
  createShrubbery
  createRobotomy
  createPardon

makeForm(name, target):
    i を 0 から formNames の要素数未満:
        if name == formNames[i]:
            出力 Intern creates <form>
            return creators[i](target)
    明確なエラーメッセージ
    return NULL   # または throw
```

`formNames[i]` と `creators[i]` は、同じ添字で対応する。  
書類の種類を足すときは、両方の配列に同じ位置で要素を追加する。

過去のレビューコメント（提出者不明）: 配列でも足りるが、構造体で名前と処理を組にした方が分かりやすい。
<details>

<summary>構造体で書いた場合（クリックで表示）</summary>

名前と生成関数を `struct` で一組にし、その構造体を C の配列に並べる。  
この課題の範囲では、コンテナを使わない。

```cpp
型 FormCreator = AForm* (*)(string const & target)

struct FormInfo:
    文字列 name
    FormCreator creator

createShrubbery(target):  return new ShrubberyCreationForm(target)
createRobotomy(target):   return new RobotomyRequestForm(target)
createPardon(target):     return new PresidentialPardonForm(target)

FormInfo forms[]:
  { "shrubbery creation",   createShrubbery }
  { "robotomy request",     createRobotomy }
  { "presidential pardon",  createPardon }

makeForm(name, target):
    i を 0 から forms の要素数未満:
        if name == forms[i].name:
            出力 Intern creates <form>
            return forms[i].creator(target)
    明確なエラーメッセージ
    return NULL   # または throw
```

`FormInfo` の1要素が、書類名と、その書類を作る関数の対応を表す。  
書類の種類を足すときは、生成関数を用意し、`forms` に対応する1行を追加する。
</details>

<a id="unknown-form-name"></a>

> **課題書原文**
>
> It should print something like:  
> `Intern creates <form>`  
> If the provided form name does not exist, print an explicit error message.

この箇所は、生成に成功した場合の出力と、指定されたForm名が存在しない場合のエラー出力を定めている。

---

<a id="form-name-keys"></a>

## 4. キー文字列

> **課題書原文**
>
> For example, the following code creates a RobotomyRequestForm targeted at "Bender":

```cpp
{
Intern someRandomIntern;
AForm* rrf;
rrf = someRandomIntern.makeForm("robotomy request", "Bender");
}
```

この例は、`"robotomy request"` が `RobotomyRequestForm` に対応するForm名であり、`"Bender"` がtargetであることを示している。

キーは次で揃えるのが安全である。

- `"shrubbery creation"`
- `"robotomy request"`
- `"presidential pardon"`

大文字小文字を吸収するかは本文に無い。例はすべて小文字。例どおりでよい。

---

<a id="ownership"></a>

## 5. 所有権

<a id="null-handling"></a>

```
AForm *f = intern.makeForm("robotomy request", "Bender");
if (f != NULL) {
    // 署名・実行
    delete f;
}
```

`AForm` のデストラクタが virtual であること（ex02）が、ここでも必要。  
`makeForm` が失敗で `NULL` を返すなら、呼び出し側の null チェックを忘れるとクラッシュする。  
例外にするなら、成功時だけポインタが返り、失敗時は `catch` する。所有権の説明は例外の方が短い。課題はメッセージ必須で、例外必須ではない。

Intern は状態を持たない。コピーも代入も「何もしない」で OCF を満たす。`(void)other;` で足りる。

---

<a id="tests"></a>

## 6. テスト

> **課題書原文**
>
> As usual, you must test everything to ensure it works as expected.

この文は、実装したすべての動作をテストするよう求めている。

- 例と同じ `"robotomy request", "Bender"`
- 残る2種のキー
- 未知の名前でメッセージが出る
- 返したポインタを署名・実行できる
- `delete` する（リークチェック）
- Intern をコピーしても `makeForm` が同じように動く

---

<a id="module-review"></a>

## 7. モジュール全体の振り返り

```
Bureaucrat : 等級という不変条件。例外で「存在しない」
Form       : 2者間の権限比較。例外は書類、ログは官僚
AForm      : 共通チェック + 派生の `side effect`。抽象と virtual
Intern     : 文字列から具体へ。分岐表。ヒープの所有は呼び出し側
```

ex00 の High/Low の向きが、最後まで署名と実行の判定に残る。  
最初の誤解を残したまま Intern まで進まない。

---

次: `CPP05_ex03_事後クイズ.md`
