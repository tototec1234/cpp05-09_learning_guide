# クラス関係図 — CPP05 ex03

> 作成日: 2026-09-13  
> 用途: 学習資料（印刷用ペラ1枚）  
> 根拠: `CPP05/ex03` 実装 + `CPP05_ex03_解説.md`

> **スコープ**: クラス間の関係と、ex03 で触る主要 public API のみ。  
> private メンバ、OCF の細部、単純 getter は省略する。

---

## 【実装】ex03 クラス構造図

> 詳細設計: クラスは？関係は？

### クラス構成


| 役割 | クラス | 要点 |
| --- | --- | --- |
| 呼び出し側 | `main` | 文字列を渡し、返ってきた `AForm*` を使う |
| 生成役 | `Intern` | 名前→生成関数の表。`makeForm` だけが入口 |
| 書類（抽象） | `AForm` | 署名・実行の共通チェック。`executeEachForm` は派生 |
| 書類（具体） | 3 Form | `new` の対象。ヘッダは `Intern` が知る |
| 官僚 | `Bureaucrat` | `signForm` / `executeForm`。生成には関わらない |


### 色凡例


| 色 | 意味 |
| --- | --- |
| 🔵 青 | 呼び出し側（`main`） |
| 🟢 緑 | 生成役（`Intern` / 表） |
| 🟠 アンバー | 書類抽象（`AForm`） |
| 🟡 薄いアンバー | 書類具体（3 Form） |
| ⚫ グレー | 官僚（`Bureaucrat`） |


```mermaid
classDiagram
    direction TB

    class main {
        <<Caller>>
        +intern.makeForm(name, target)
        +bureaucrat.signForm(form)
        +bureaucrat.executeForm(form)
        +delete form
    }

    class Intern {
        <<Creator / Dispatcher>>
        +makeForm(formName, target) AForm*
    }

    class FormInfo {
        <<name + creator pair>>
        +name
        +creator FormCreator
    }

    class AForm {
        <<Abstract Form>>
        +beSigned(bureaucrat)
+execute(executor)
        #executeEachForm(executor)*
    }

    class ShrubberyCreationForm {
        <<Concrete Form>>
        +executeEachForm(executor)
    }

    class RobotomyRequestForm {
        <<Concrete Form>>
        +executeEachForm(executor)
    }

    class PresidentialPardonForm {
        <<Concrete Form>>
        +executeEachForm(executor)
    }

    class Bureaucrat {
        <<Actor>>
        +signForm(form)
        +executeForm(form)
    }

    main ..> Intern : uses
    main ..> AForm : owns returned pointer
    main ..> Bureaucrat : uses

    Intern ..> FormInfo : looks up table
    Intern ..> ShrubberyCreationForm : creates
    Intern ..> RobotomyRequestForm : creates
    Intern ..> PresidentialPardonForm : creates
    Intern ..> AForm : returns as AForm*

    AForm <|-- ShrubberyCreationForm
    AForm <|-- RobotomyRequestForm
    AForm <|-- PresidentialPardonForm

    Bureaucrat ..> AForm : signs / executes

    style main fill:#4A90D9,stroke:#2E5A8B,color:#fff
    style Intern fill:#50B878,stroke:#3A8A5A,color:#fff
    style FormInfo fill:#50B878,stroke:#3A8A5A,color:#fff
    style AForm fill:#D9822B,stroke:#9A5A1E,color:#fff
    style ShrubberyCreationForm fill:#F8D9B0,stroke:#D9822B,color:#3A2A18
    style RobotomyRequestForm fill:#F8D9B0,stroke:#D9822B,color:#3A2A18
    style PresidentialPardonForm fill:#F8D9B0,stroke:#D9822B,color:#3A2A18
    style Bureaucrat fill:#6C7A89,stroke:#45515C,color:#fff
```

---

## 読み方の要点

1. **`main` は具体型を直接 `new` しない。** `Intern::makeForm` に文字列を渡す。
2. **具体型のヘッダを `#include` するのは `Intern` 側。** ビルドでは具体型のオブジェクトファイルをリンクする。具体型が消えるわけではない。
3. **生成規則（名前と生成関数の対応）は `Intern` 一箇所にまとめて書く。** 種類を足すとき、直す場所が `Intern` になる。
4. **所有権は呼び出し側。** 返ってきた `AForm*` を `delete` する。`AForm` のデストラクタが virtual である必要がある（ex02）。
