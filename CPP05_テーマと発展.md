 # CPP05_テーマと発展

> 前提となるモジュール  
> - CPP03: **Inheritance**（継承）  
> - CPP04: **Subtype Polymorphism, Abstract Classes, and Interfaces**（サブタイプ多態性、抽象クラス、インターフェース）

<a id="toc"></a>
## 目次

- [CPP05\_テーマと発展](#cpp05_テーマと発展)
	- [目次](#目次)
	- [1. この文書の目的と読み方](#1-この文書の目的と読み方)
		- [1.1 対象と目的](#11-対象と目的)
		- [1.2 読み方](#12-読み方)
	- [2. CPP05の全体像](#2-cpp05の全体像)
		- [2.1 Repetition and Exceptions](#21-repetition-and-exceptions)
		- [2.2 新しく学ぶ内容](#22-新しく学ぶ内容)
		- [2.3 ex00からex03への発展](#23-ex00からex03への発展)
	- [3. CPP04までの復習](#3-cpp04までの復習)
		- [3.1 CPP03からCPP04への流れ](#31-cpp03からcpp04への流れ)
		- [3.2 CPP04におけるクラスの継続と拡張](#32-cpp04におけるクラスの継続と拡張)
		- [3.3 virtualと動的多態性](#33-virtualと動的多態性)
		- [3.4 抽象クラス](#34-抽象クラス)
		- [3.5 privateとprotected](#35-privateとprotected)
		- [3.6 仮想デストラクタ](#36-仮想デストラクタ)
	- [4. CPP05の例外処理](#4-cpp05の例外処理)
		- [4.1 Cのエラー処理との違い](#41-cのエラー処理との違い)
		- [4.2 コンストラクタから送出された例外](#42-コンストラクタから送出された例外)
		- [4.3 C++98のthrow()例外仕様](#43-c98のthrow例外仕様)
		- [4.4 発展: 例外安全保証](#44-発展-例外安全保証)
	- [5. CPP05のクラス設計](#5-cpp05のクラス設計)
		- [5.1 クラスごとの責務](#51-クラスごとの責務)
		- [5.2 constデータメンバとOrthodox Canonical Form](#52-constデータメンバとorthodox-canonical-form)
		- [5.3 Template Methodパターン](#53-template-methodパターン)
		- [5.4 AFormが提供する共通インターフェース](#54-aformが提供する共通インターフェース)
		- [5.5 IRCのインターフェース設計との比較](#55-ircのインターフェース設計との比較)
		- [5.6 if/elseの森を避ける生成設計](#56-ifelseの森を避ける生成設計)
	- [6. 官僚機構という題材](#6-官僚機構という題材)
		- [6.1 ソフトウェアの責務分割との対応](#61-ソフトウェアの責務分割との対応)
		- [6.2 『銀河ヒッチハイク・ガイド』との関係](#62-銀河ヒッチハイクガイドとの関係)
	- [7. 課題の制約と評価対策](#7-課題の制約と評価対策)
		- [7.1 CPP05で守る制約](#71-cpp05で守る制約)
		- [7.2 評価で確認されやすい項目](#72-評価で確認されやすい項目)

---

<a id="purpose"></a>
## 1. この文書の目的と読み方

<a id="purpose-target"></a>
### 1.1 対象と目的

この文書は、CPP05の実装前にモジュール全体の構造を把握するための導入資料である。

CPP05「Repetition and Exceptions」で初めて扱う例外処理だけでなく、CPP03「Inheritance」とCPP04「Subtype Polymorphism, Abstract Classes, and Interfaces」で学んだ継承、多態性、抽象クラスなどが、各exerciseでどのように再利用されるかを整理する。

完成コードは示さない。実装を始める前に、次の点を説明できる状態を目標とする。

- ex00からex03へ何が引き継がれ、何が追加されるか
- 例外を送出（`throw`）する側と捕捉（`catch`）する側の責務
- `Bureaucrat`、`AForm`、具体Form、`Intern`の役割
- CPP04で学んだ多態性がCPP05のどこで使われるか

<a id="purpose-reading"></a>
### 1.2 読み方

1. 第2章でCPP05全体の流れを確認する。
2. 第3章でCPP04までの知識を復習する。
3. 第4章と第5章で例外処理とクラス設計を理解する。
4. 第6章で課題の題材と設計の関係を確認する。(IRCに取組済みの場合は、第6章を最初に読むと良い)
5. 第7章で提出時の制約と評価項目を確認する。
6. その後、`CPP05_ex00_事前クイズ.md`へ進む。

[目次へ戻る](#toc)

---

<a id="overview"></a>
## 2. CPP05の全体像

<a id="overview-theme"></a>
### 2.1 Repetition and Exceptions

課題書の題は **Repetition and Exceptions**（反復と例外）である。

新しく学ぶ言語機能の中心は例外処理である。
同時に、CPP03「Inheritance」（継承）とCPP04「Subtype Polymorphism, Abstract Classes, and Interfaces」（サブタイプ多態性、抽象クラス、インターフェース）で学んだクラス設計を、同じ題材を段階的に拡張しながら復習する。

ここでいう「反復」は、同じコードを機械的に書き直すことだけを指さない。前のexerciseで作ったクラスを次のexerciseへ引き継ぎ、機能追加や抽象化を行うことも含む。

<a id="overview-new"></a>
### 2.2 新しく学ぶ内容

| 分類 | 内容 |
| --- | --- |
| 例外処理 | `try`、`throw`、`catch`、`std::exception`、ネストした例外クラス、`what() const throw()` |
| 状態の制約 | 不正な等級を持つオブジェクトを完成させない |
| constデータメンバ | 名前や必要等級を、初期化後に代入で変更できないようにする |
| 責務の分離 | 署名、実行、具体的な処理、生成を別のクラスへ分ける |
| 抽象化 | `Form`を抽象クラス`AForm`へ変更し、具体Formを共通の型で扱う |
| 生成処理 | `Intern`が文字列から具体Formを生成し、`AForm*`として返す |

CPP05で使う多態性、抽象クラス、仮想デストラクタはCPP04の既習事項である。例外処理と組み合わせて利用する点がCPP05での発展になる。

<a id="overview-exercises"></a>
### 2.3 ex00からex03への発展

```text
ex00「Mommy, when I grow up, I want to be a bureaucrat!」
      Bureaucrat
      ├─ 各Bureaucratオブジェクトの名前は生成後に変更できない
      ├─ 等級は1から150
      └─ 範囲外の等級は例外

ex01「Form up, maggots!」
      Bureaucrat + Form
      ├─ BureaucratにsignForm()を追加
      ├─ FormにbeSigned()を追加
      └─ 例外を送出する側と捕捉する側を分ける

ex02「No, you need form 28B, not 28C...」
      Bureaucrat + AForm + 3つの具体Form
      ├─ Formを抽象クラスAFormへ変更
      ├─ BureaucratにexecuteForm()を追加
      ├─ 共通の実行条件はAFormが確認
      └─ 具体的な処理は各派生クラスが実装

ex03「At least this beats coffee-making」
      上記 + Intern
      ├─ Intern::makeForm()を追加
      ├─ 文字列で具体Formの種類を指定
      └─ 生成結果をAForm*として返す
```

`Bureaucrat`クラスはex00からex03まで引き継ぎ、各exerciseで機能を追加する。`Form`クラスはex01で作り、ex02で抽象クラス`AForm`へ変更する。

したがって、ex00で等級の向きを間違えると、ex01の署名判定とex02の実行判定にも同じ誤りが残る。

[目次へ戻る](#toc)

---

<a id="review"></a>
## 3. CPP04までの復習

<a id="review-flow"></a>
### 3.1 CPP03からCPP04への流れ

CPP03「Inheritance」では、主に継承の構造を学んだ。

- is-a関係を型の階層で表す
- 基底クラスから派生クラスを作る
- コンストラクタとデストラクタの呼び出し順を確認する

CPP04「Subtype Polymorphism, Abstract Classes, and Interfaces」では、継承した型を共通の基底型として扱い、実行時に派生クラスの処理を選ぶ動的多態性を学んだ。

- `virtual`による動的な関数選択
- 純粋仮想関数`= 0`による抽象クラス
- 基底クラスポインタを通した派生オブジェクトの操作
- 仮想デストラクタ

<a id="review-cpp04"></a>
### 3.2 CPP04におけるクラスの継続と拡張

CPP05の段階的な拡張は、CPP04「Subtype Polymorphism, Abstract Classes, and Interfaces」のex00からex02の進め方に似ている。

- CPP04 ex00「Polymorphism」: `Animal`、`Dog`、`Cat`を作る
- CPP04 ex01「I don’t want to set the world on fire」: ex00のクラスを引き継ぎ、`Brain`を追加して`Dog`と`Cat`に所有させる
- CPP04 ex02「Abstract class」: 同じクラス群を引き継ぎ、`Animal`をインスタンス化できない抽象クラスへ変更する

	- CPP04 ex02「Abstract class」では、`Animal`を`AAnimal`へ改名してもよいと課題書に書かれていた。
	- CPP05 ex02「No, you need form 28B, not 28C...」では、`Form`を`AForm`へ改名することが課題の要求である。
	- 接頭辞 A は Abstract の慣習。言語仕様ではなく、CPP04 の AAnimal と同じ命名。

<a id="review-virtual"></a>
### 3.3 virtualと動的多態性

基底クラスポインタまたは基底クラス参照を通して仮想関数を呼ぶと、実際のオブジェクト型に対応する関数が実行される。

> 用語の補足: `virtual`を日本語訳の「仮想」だけから「偽物」や「作り物」と理解しないこと。C++でこのキーワードが表す機能と、英語の`virtual`が持つ「実質上の」という語感の関係については、torinoueに確認する。

```cpp
Animal *animal = new Dog();
animal->makeSound(); // Dog::makeSound()
```

CPP05 ex02「No, you need form 28B, not 28C...」では、同じ仕組みを`AForm`と具体Formに適用する。ただし、多態的に選ばれる関数は一段深い位置にある。

```cpp
AForm *form = new RobotomyRequestForm("Bender");
form->execute(bureaucrat); // AForm::execute()が共通の確認を行い、
                           // RobotomyRequestForm::executeEachForm()を呼ぶ
```

`form`の変数型は`AForm*`だが、具体的な処理は`RobotomyRequestForm`側の実装が担当する。`execute()`自体は非`virtual`であり、派生クラスへ委ねるのは`executeEachForm()`だけである（5.3参照）。

<a id="review-abstract"></a>
### 3.4 抽象クラス

純粋仮想関数を一つ以上持つクラスは抽象クラスになり、そのクラス自体のオブジェクトは作れない。

```cpp
class AForm {
protected:
    virtual void executeEachForm() const = 0;
};
```

CPP05 ex02「No, you need form 28B, not 28C...」の「base class Form must be an abstract class」という要求は、この仕組みで実現する。

<a id="review-access"></a>
### 3.5 privateとprotected

| アクセス指定 | 派生クラスから直接アクセス | CPP05での扱い |
| --- | --- | --- |
| `private` | できない | `AForm`の属性は課題指定により`private` |
| `protected` | できる | 派生クラス用の処理関数には使用できる |

派生クラスが`_isSigned`を直接変更できると、署名に必要な等級の確認を回避できる。`AForm`の属性を`private`に保ち、状態変更を`AForm`の公開メンバ関数に限定すると、不変条件を一か所で管理できる。

<a id="review-destructor"></a>
### 3.6 仮想デストラクタ

基底クラスポインタを通して派生オブジェクトを`delete`する場合、基底クラスのデストラクタを`virtual`にする。

```cpp
AForm *form = new RobotomyRequestForm("Bender");
delete form;
```

これはCPP05で初めて学ぶ内容ではない。CPP04 ex01「I don’t want to set the world on fire」では、`Dog`と`Cat`を`Animal*`として保持し、`Animal*`を通して`delete`しても派生クラスのデストラクタが呼ばれるようにする必要があった。

CPP05 ex03「At least this beats coffee-making」では、`Intern::makeForm()`が`AForm*`を返す場面に同じ原則を適用する。CPP04の内容を理解できていれば、CPP05では既習事項の再適用と考えてよい。

[目次へ戻る](#toc)

---

<a id="exceptions"></a>
## 4. CPP05の例外処理

<a id="exceptions-c"></a>
### 4.1 Cのエラー処理との違い

Cの課題では、関数の失敗を戻り値と`errno`で表すことが多かった。

C++のコンストラクタには戻り値がない。CPP05では、不正な引数を受け取ったことを例外で呼び出し側へ伝える。

```cpp
try {
    Bureaucrat alice("Alice", 0);
} catch (const std::exception &e) {
    std::cout << e.what() << std::endl;
}
```

課題書の General rules は、特に指定がない限り出力メッセージを標準出力へ出すと定めている。`subject_cpp05.pdf` 全体を確認しても、標準エラー（`std::cerr`）への出力指定はない。例外やエラーメッセージも、課題書が別ストリームを指定していない限り `std::cout` に出す。

ただし、実務上の落とし穴としては指摘は妥当だ。`catch` で `std::cerr` に出すのは慣習としてよくある。

- `throw`: 例外を送出する
- `try`: 例外が発生する可能性のある処理を囲む
- `catch`: 型が一致する例外を捕捉する

<details>
<summary>参考資料（クリックで表示）</summary>

- [try、throw、catch ステートメント（Microsoft Learn）](https://learn.microsoft.com/ja-jp/cpp/cpp/try-throw-and-catch-statements-cpp?view=msvc-170)

</details>

<a id="exceptions-constructor"></a>
### 4.2 コンストラクタから送出された例外

コンストラクタが例外を送出すると、そのコンストラクタは完了せず、対象のオブジェクトは作られない。

- オブジェクト全体のデストラクタは呼ばれない
- 構築済みの基底クラスとデータメンバは破棄される
- 呼び出し元では、一致する型の`catch`を探す

一致する`catch`が呼び出し履歴上に存在しなければ、`std::terminate()`が呼ばれ、プログラムは異常終了する。

`-Wall -Wextra -Werror -std=c++98`は、実行時に例外が捕捉されないことを通常はコンパイル時に警告しない。`-Werror`は、発生した警告をエラーとして扱うフラグであり、未捕捉例外を静的に検出するフラグではない。

<details>
<summary>参考資料（クリックで表示）</summary>

- [C++での例外とスタックアンワインド（Microsoft Learn）](https://learn.microsoft.com/ja-jp/cpp/cpp/exceptions-and-stack-unwinding-in-cpp?view=msvc-170)
- [Warning Options（GCC公式、英語）](https://gcc.gnu.org/onlinedocs/gcc/Warning-Options.html)

</details>

<a id="exceptions-specification"></a>
### 4.3 C++98のthrow()例外仕様

C++98の`throw()`は、「その関数の外へ例外を送出しない」という動的例外仕様を表す。

```cpp
virtual const char *what() const throw();
```

この約束に反して例外が関数外へ出ると、`std::unexpected()`を経由し、標準の設定では`std::terminate()`に至る。

これは、一致する`catch`が見つからないために`std::terminate()`が呼ばれる場合とは別の経路である。

<details>
<summary>参考資料（クリックで表示）</summary>

- [例外指定（throw、noexcept）（Microsoft Learn）](https://learn.microsoft.com/ja-jp/cpp/cpp/exception-specifications-throw-cpp?view=msvc-170)
- [ILE C/C++解説書（IBM、日本語PDF）](https://www.ibm.com/docs/ja/ssw_ibm_i_75/pdf/sc097852.pdf)

</details>

<a id="exceptions-safety"></a>
### 4.4 発展: 例外安全保証

例外安全保証はCPP05の必須APIではない。ただし、`incrementGrade()`と`decrementGrade()`の安全な処理順序を説明するために役立つ。

「強い保証」に対する正式な分類名は「弱い保証」ではなく、通常はbasic保証である。

| 保証 | 意味 | CPP05での例 |
| --- | --- | --- |
| no-throw / no-fail保証 | 呼び出し側へ例外を送出しない | 文字列リテラルを返すだけの`what() const throw()` |
| strong保証 | 失敗した場合、観測可能な状態が変化しない | 範囲外なら`_grade`を変更せずに例外を送出する |
| basic保証 | リークせず、既存オブジェクトを使用可能な状態に保つ。値は変化していてもよい | 単純な等級変更ではstrong保証を容易に満たせるため、basic保証に限定する理由はない |

`incrementGrade()`で先に`_grade`を変更してから範囲を確認すると、例外送出時に変更前の状態へ戻らない。

```cpp
// strong保証を満たさない処理順序
void Bureaucrat::incrementGrade() {
    --_grade;
    if (_grade < HighestGrade)
        throw GradeTooHighException(); // _gradeは0のまま
}

// strong保証を満たす処理順序
void Bureaucrat::incrementGrade() {
    if (_grade <= HighestGrade)
        throw GradeTooHighException(); // _gradeは変更されていない
    --_grade;
}
```

コンストラクタからの例外は、既存オブジェクトに対するbasic保証の例とは分けて考える。コンストラクタが完了していないため、そのオブジェクト自体が存在しないからである。

<details>
<summary>参考資料（クリックで表示）</summary>

- [例外安全性に対応した設計（Microsoft Learn）](https://learn.microsoft.com/ja-jp/cpp/cpp/how-to-design-for-exception-safety?view=msvc-170)
- [C++での例外とスタックアンワインド（Microsoft Learn）](https://learn.microsoft.com/ja-jp/cpp/cpp/exceptions-and-stack-unwinding-in-cpp?view=msvc-170)
- [noexcept（cpprefjp）](https://cpprefjp.github.io/lang/cpp11/noexcept.html) — C++11の機能。用語の参考資料であり、CPP05の提出コードには使用しない

</details>

[目次へ戻る](#toc)

---

<a id="design"></a>
## 5. CPP05のクラス設計

<a id="design-responsibilities"></a>
### 5.1 クラスごとの責務

| クラス | 担当すること | 担当しないこと |
| --- | --- | --- |
| `Bureaucrat` | 自分の名前と等級を管理し、Formへ署名と実行を依頼する | ツリー作成、ロボトミー、恩赦の具体的な処理 |
| `Form` / `AForm` | 名前、署名状態、必要等級、署名と実行の条件を管理する | 自分を生成したクラスの管理 |
| 具体Form | ファイル作成やメッセージ出力など、Formごとの処理を実行する | 共通する署名条件と実行条件の重複実装 |
| `Intern` | 文字列で指定された種類の具体Formを生成する | Formへの署名と実行 |

<a id="design-const"></a>
### 5.2 constデータメンバとOrthodox Canonical Form

```cpp
class Bureaucrat {
private:
    const std::string _name;
    int _grade;
};
```

- constデータメンバは、コンストラクタのメンバ初期化子リストで初期化する
- コピーコンストラクタでは、新しいオブジェクトの`_name`と`_grade`を初期化できる
- コピー代入演算子では、既に存在するオブジェクトのconstな`_name`へ代入できないため、`_grade`だけをコピーする
- `getName() const`の末尾の`const`は、「この関数がオブジェクトを変更しない」という別の指定である

CPP02「Ad-hoc polymorphism, operator overloading and Orthodox Canonical class form」のex00「My First Class in Orthodox Canonical Form」では、`Fixed`に小数部のビット数を表す`static const`整数が指定されていた。これは全`Fixed`オブジェクトで共有するクラス定数である。

CPP05の`Bureaucrat::_name`は、官僚ごとに値が異なる非staticのconstデータメンバである。この違いを区別する。

<details>
<summary>参考資料（クリックで表示）</summary>

- [コンストラクター（Microsoft Learn）](https://learn.microsoft.com/ja-jp/cpp/cpp/constructors-cpp?view=msvc-170)
- [コピーコンストラクターとコピー代入演算子（Microsoft Learn）](https://learn.microsoft.com/ja-jp/cpp/cpp/copy-constructors-and-copy-assignment-operators-cpp?view=msvc-170)
- [cv型修飾子（cppreference日本語版）](https://ja.cppreference.com/w/cpp/language/cv)

</details>

<a id="design-template-method"></a>
### 5.3 Template Methodパターン

Template Methodはデザインパターン名であり、CPP07「C++ templates」で扱うC++の関数テンプレートとは別概念である。

Template Methodパターンでは、処理の共通の流れを基底クラスに置き、一部の処理を仮想関数として派生クラスへ委ねる。

CPP05 ex02「No, you need form 28B, not 28C...」では、次の分割が該当する。

```text
AForm::execute()
├─ 署名済みか確認する
├─ 実行者の等級を確認する
└─ 具体Form固有の処理を呼ぶ
```

署名と等級の確認を各具体Formへ重複して書くこともできるが、一つの派生クラスだけ確認を忘れる可能性がある。共通処理を`AForm`へ置くと、すべての具体Formへ同じ条件を適用できる。

このとき`AForm::execute()`自体には`virtual`を付けない。付けると派生クラスが`execute()`を上書きして確認を丸ごと省けるため、共通処理を基底へ置いた意味がなくなる。派生クラスへ開くのは、確認のあとに呼ばれる純粋仮想関数だけである。

例外を送出する位置と多態性の関係は、`CPP05_ex02_解説.md`の3節「例外と多態性が交差する3点」で扱う。

<details>
<summary>参考資料（クリックで表示）</summary>

- [Template Method（Refactoring.Guru日本語版）](https://refactoring.guru/ja/design-patterns/template-method)
- [関数テンプレート（cppreference日本語版）](https://ja.cppreference.com/w/cpp/language/function_template)

</details>

<a id="design-aform"></a>
### 5.4 AFormが提供する共通インターフェース

CPP05には、C++コード上のインターフェースが存在する。

- `Bureaucrat`と`Form`の`public`メンバ関数は、各クラスを利用する側に対するインターフェースになる
- `AForm`は、具体Formに共通する操作を提供する抽象基底クラスである
- `Bureaucrat::executeForm(AForm const &form)`は、具体Formではなく`AForm`の公開APIに依存する
- `Intern::makeForm()`は、生成した具体Formを`AForm*`として返す

```cpp
void Bureaucrat::executeForm(AForm const &form) const;
AForm *form = intern.makeForm("robotomy request", "Bender");
```

利用する側は、`form`がどの具体Formかを判定せず、`AForm`が公開する操作を呼べる。

ただし、`AForm`は状態と共通処理も持つ。CPP04 ex03「Interface & recap」の`ICharacter`や`IMateriaSource`のような純粋抽象インターフェースではなく、**共通実装を持つ抽象基底クラス**である。

<a id="design-irc"></a>
### 5.5 IRCのインターフェース設計との比較

42の課題ft_irc「Internet Relay Chat」で採用した設計は、次の三つに分けて説明できる。

- **レイヤードアーキテクチャ**: Network / I/Oを扱うA層、Protocol / Commandを扱うB層、IRC上の状態を扱うC層に分割する
- **契約先行設計（Contract-first）**: 実装前に`interface.md`で層間の入力、出力、公開API、禁止事項を契約として定める
- **実装ではなくインターフェースに対してプログラミングする**: 各層は他層の内部実装を知らず、公開APIと境界オブジェクトだけを使用する

B層はA層の`Server`や`Connection`を直接操作せず、境界オブジェクト`CommandResult`を返す。A層はB層内部のコマンド処理を知らず、返された結果を送信処理へ反映する。

この場合の「インターフェース」は、C++の純粋仮想クラスだけを指さない。層間で合意した公開APIとデータ形式を含む契約を指す。

CPP05にはA/B/C層も、`interface.md`に相当する独立した層間契約書もない。一方、具体Formを`AForm`として利用する設計には、「内部実装ではなく、公開されたインターフェースを通して利用する」という共通点がある。

これは依存性逆転の原則（DIP）と関連するが、同一ではない。DIPは上位方針と下位実装の双方を抽象へ依存させる原則である。IRCもCPP05も、すべての依存関係を抽象基底クラスによって反転させているわけではないため、設計全体を単にDIPと呼ぶのは正確ではない。

<a id="design-dispatch"></a>
### 5.6 if/elseの森を避ける生成設計

文字列から処理を選ぶ課題は、CPP05で初めて登場するわけではない。

- CPP01 ex05「Harl 2.0」では、`Harl::complain(std::string level)`をif/elseの森にせず、メンバ関数ポインタを使うことが要求された
- CPP01 ex06「Harl filter」では、選択したログレベル以降を処理するために`switch`文の使用が必須だった
- CPP04 ex03「Interface & recap」の`MateriaSource::createMateria(std::string const &type)`では、文字列から`Ice`または`Cure`を生成し、`AMateria*`として返した

CPP05 ex03「At least this beats coffee-making」では、`switch`の使用自体は要件ではない。課題書が禁じているのは、過剰で読みにくいif/else構造である。

文字列と生成処理を対応付ける配列や、関数ポインタを使ったディスパッチ表を利用できる。CPP05ではSTLコンテナが禁止されているため、`std::map`は使えない。

[目次へ戻る](#toc)

---

<a id="bureaucracy"></a>
## 6. 官僚機構という題材

<a id="bureaucracy-design"></a>
### 6.1 ソフトウェアの責務分割との対応

CPP05は、権限の階層、書類への署名、手続きの実行、書類を作る担当者という官僚機構の要素を使って、クラスごとの責務を表している。

IRCのA/B/C層も、組織に置き換えれば縦割りの分業に似ている。

| ソフトウェア設計 | 官僚機構に置き換えた場合 |
| --- | --- |
| 各レイヤー | 担当部署 |
| 公開API | 他部署から利用できる窓口 |
| 境界オブジェクト | 部署間で受け渡す申請書 |
| `interface.md` | 部署間で合意した書式と手続規則 |

各部署は担当職務だけを受け持ち、他部署の内部手順には立ち入らず、定められた書式と窓口を通して仕事を依頼する。

ただし、ソフトウェアの責務分割を、否定的な意味の官僚主義と同一視はできない。責務と窓口を明確にすることは、変更の影響を限定し、複数人が並行して実装するために有効である。

窓口が過剰に増える、契約の変更に時間がかかる、層を越えた問題を誰も担当しない、といった状態になったとき、官僚主義や縦割り組織の弊害に近づく。

<a id="bureaucracy-hitchhiker"></a>
### 6.2 『銀河ヒッチハイク・ガイド』との関係

CPP05の固有名詞には、ダグラス・アダムスの『銀河ヒッチハイク・ガイド』への参照がある。

- `PresidentialPardonForm`の文面に登場するZaphod Beeblebroxは、作中の銀河大統領である
- ヴォゴン人は銀河政府で官僚になる種族として描かれている
- 地球は超空間バイパス建設のために破壊される

この背景から、CPP05の題材は単なる役所ではなく、権限と書類手続きが支配する官僚機構として読むことができる。

ただし、作品がCPP05の設計要件の根拠であるとは課題書に明記されていない。この節は、固有名詞と作品設定の対応を説明する補足である。

<details>
<summary>参考資料（クリックで表示）</summary>

- [『銀河ヒッチハイク・ガイド』日本語版書誌（河出書房新社）](https://www.kawade.co.jp/np/isbn/9784309462554/)
- [Zaphod Beeblebrox（BBC、英語）](https://www.bbc.com/cult/hitchhikers/guide/zaphod.shtml)
- [Vogons（BBC、英語）](https://www.bbc.com/cult/hitchhikers/guide/vogon.shtml)

</details>

[目次へ戻る](#toc)

---

<a id="evaluation"></a>
## 7. 課題の制約と評価対策

<a id="evaluation-rules"></a>
### 7.1 CPP05で守る制約

以下のチェックボックス13項目は、[CPP05課題書](../CPP05/subject_cpp05.pdf)の一般規則と各exerciseの要件に基づく。`CPP05/.gitignore`や特定の提出コードは出典として使用していない。

- [ ] 1. `c++`と`-Wall -Wextra -Werror`でコンパイルし、`-std=c++98`を追加してもコンパイルできるようにする
- [ ] 2. C++11以降の機能、Boost、その他の外部ライブラリを使わない
- [ ] 3. `printf`系、`alloc`系、`free`を使わない
- [ ] 4. `using namespace`と`friend`は、課題書で許可された場合を除いて使わない
- [ ] 5. STLのコンテナと`<algorithm>`はModule 08/09まで使わない
- [ ] 6. Module 02以降のクラスはOrthodox Canonical Formに従う。例外クラスは免除される
- [ ] 7. テンプレート以外の関数実装をヘッダへ置かない
- [ ] 8. ヘッダは単独でインクルードできるようにし、インクルードガードを付ける
- [ ] 9. `new`で生成したオブジェクトをリークさせない
- [ ] 10. exerciseのディレクトリ名、必須ファイル名、クラス名を課題書の指定に合わせる
- [ ] 11. クラス名はUpperCamelCaseとし、クラスを定義するファイル名をクラス名に合わせる
- [ ] 12. 特に指定がない限り、出力メッセージ（例外・エラーの表示を含む）の末尾に改行を付け、`std::cout`（標準出力）へ出す。`std::cerr` に出していないか
- [ ] 13. MakefileはC課題と同じ規則に従う

必要なら必須ファイル以外のファイルを追加できる。ただし、必須ファイルはすべて提出する。評価では、理解を確認するために数分で行える小規模な変更を求められる場合がある。

課題書は一部の違反について0点または-42となる条件を明記している。動作だけでなく、使用した言語機能とファイル構成も確認する。

<a id="evaluation-points"></a>
### 7.2 評価で確認されやすい項目

この24項目の一覧は公式evaluation sheetではない。[CPP05課題書](../CPP05/subject_cpp05.pdf)と過去のレビューコメントから再構成した参考項目である。特定の提出コードは出典として使用していない。

角括弧内は、その要件が最初に登場するexerciseを示す。前のexerciseのファイルを引き継ぐため、`[ex00]`の要件はex01以降でも満たす必要がある。

- [ ] 1. **[ex00: `Bureaucrat`／ex01: `Form`]** 等級0と151で、指定された別々の例外を送出するか（ex02以降では`Form`を`AForm`として引き継ぐ）
- [ ] 2. **[ex00: `Bureaucrat`／ex01: `Form`]** 境界値1と150で正常に構築できるか（ex02以降では`Form`を`AForm`として引き継ぐ）
- [ ] 3. **[ex00]** `incrementGrade()`で数値が減り、`decrementGrade()`で数値が増えるか
- [ ] 4. **[ex00]** `catch (std::exception &)`で自作例外を捕捉できるか
- [ ] 5. **[ex00]** constな名前をコピー代入演算子で書き換えようとしていないか
- [ ] 6. **[ex01: `Form`／ex02: `AForm`]** 属性が`private`か
- [ ] 7. **[ex01]** `Form`の出力演算子が、署名状態、署名に必要な等級、実行に必要な等級を取り違えずに出力するか
- [ ] 8. **[ex02]** `ShrubberyCreationForm`の署名等級が145、実行等級が137か
- [ ] 9. **[ex02]** `RobotomyRequestForm`の署名等級が72、実行等級が45か
- [ ] 10. **[ex02]** `PresidentialPardonForm`の署名等級が25、実行等級が5か
- [ ] 11. **[ex02]** 未署名の`AForm`を実行すると例外になるか
- [ ] 12. **[ex02]** 実行者の等級が不足している`AForm`を実行すると例外になるか
- [ ] 13. **[ex02]** `Bureaucrat::executeForm(AForm const &)`が実装されているか
- [ ] 14. **[ex03]** `Intern::makeForm()`が過剰なif/else構造になっていないか
- [ ] 15. **[ex03]** `makeForm("robotomy request", "Bender")`が課題書の例どおり動くか
- [ ] 16. **[ex03]** 存在しないForm名を`Intern::makeForm()`へ渡すと、明示的なエラーメッセージを出すか
- [ ] 17. **[ex03]** Formの生成に成功すると、`Intern creates <form>`に相当するメッセージを出すか
- [ ] 18. **[ex02]** `ShrubberyCreationForm`がファイルを作るか
- [ ] 19. **[ex02]** `RobotomyRequestForm`が成功と失敗の両方を発生させるか
- [ ] 20. **[ex02]** `PresidentialPardonForm`が対象者をZaphod Beeblebroxにより恩赦されたものとして出力するか
- [ ] 21. **[共通]** 各ヘッダが、別のヘッダのインクルード順序に依存せずに使用できるか
- [ ] 22. **[共通]** 各exerciseの`main.cpp`に、引数付きコンストラクタ、正常系、異常系、境界値など、そのexerciseの要件を確認するテストがあるか

`Bureaucrat::GradeTooHighException`と`Bureaucrat::GradeTooLowException`は、課題書が指定した名前であり、学生が自由に命名するものではない。

- [ ] 23. **[ex00]** `grade < 1`: 権限が許容範囲より高いため`GradeTooHighException`
- [ ] 24. **[ex00]** `grade > 150`: 権限が許容範囲より低いため`GradeTooLowException`

High/Lowは整数値の大小ではなく、官僚としての権限の高低を表す。

[目次へ戻る](#toc)

---

次: `CPP05_ex00_事前クイズ.md`
