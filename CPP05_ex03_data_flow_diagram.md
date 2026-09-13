# データフロー図 — CPP05 ex03

> 作成日: 2026-09-13  
> 用途: 学習資料（印刷用ペラ1枚）  
> 根拠: `CPP05/ex03` 実装 + `CPP05_ex03_解説.md`

---

## 【設計】書類生成〜実行シーケンス

> 時間軸: いつ、何が起きるか  
> 文字列 → 表照合 → `new` → 署名・実行 → `delete`

```mermaid
sequenceDiagram
    participant Main as main
    participant Intern as Intern
    participant Table as FormInfo表
    participant Form as AForm*
    participant Buro as Bureaucrat

    rect rgb(227, 242, 253)
        Note over Main,Intern: 生成依頼
        Main->>Intern: makeForm("robotomy request", "Bender")
    end

    rect rgb(232, 245, 233)
        Note over Intern,Table: 名前照合
        Intern->>Table: formName を走査
        alt 名前が表にある
            Table-->>Intern: creator 関数
            Intern->>Form: creator(target) / new 具体型
            Intern-->>Main: AForm*
            Note right of Intern: "Intern creates ..."
        else 名前が無い
            Intern-->>Main: NULL
            Note right of Intern: エラーメッセージ
        end
    end

    rect rgb(255, 243, 224)
        Note over Main,Buro: 署名・実行（成功時）
        Main->>Buro: signForm(*form)
        Buro->>Form: beSigned
        Main->>Buro: executeForm(*form)
        Buro->>Form: execute → executeEachForm
    end

    rect rgb(227, 242, 253)
        Note over Main,Form: 所有権の解放
        Main->>Form: delete form
    end
```

---

## 【設計】書類生成フロー（概念）

> 接続: 何が、どこに渡されるか

```mermaid
flowchart LR
    subgraph Caller["呼び出し側"]
        main["main"]
        delete_node["delete"]
    end

    subgraph Creator["生成役"]
        makeForm["Intern::makeForm"]
        table["FormInfo[]\nname + creator"]
        creators["createXxx(target)"]
    end

    subgraph Forms["書類階層"]
        aform["AForm*"]
        concrete["Shrubbery /\nRobotomy /\nPardon"]
    end

    subgraph Actor["官僚"]
        sign["signForm"]
        exec["executeForm"]
    end

    main -->|"formName, target\n(std::string)"| makeForm
    makeForm -->|"照合"| table
    table -->|"FormCreator"| creators
    creators -->|"new"| concrete
    concrete -->|"upcast"| aform
    makeForm -->|"AForm* or NULL"| main

    main -->|"AForm&"| sign
    sign -->|"beSigned"| aform
    main -->|"AForm const&"| exec
    exec -->|"execute"| aform
    main --> delete_node
    delete_node -->|"virtual ~AForm"| aform

    style main fill:#4A90D9,stroke:#2E5A8B,color:#fff
    style delete_node fill:#4A90D9,stroke:#2E5A8B,color:#fff
    style makeForm fill:#50B878,stroke:#3A8A5A,color:#fff
    style table fill:#50B878,stroke:#3A8A5A,color:#fff
    style creators fill:#50B878,stroke:#3A8A5A,color:#fff
    style aform fill:#D9822B,stroke:#9A5A1E,color:#fff
    style concrete fill:#F8D9B0,stroke:#D9822B,color:#3A2A18
    style sign fill:#6C7A89,stroke:#45515C,color:#fff
    style exec fill:#6C7A89,stroke:#45515C,color:#fff
```

---

## 主要データ型


| 境界 | データ型 | 内容 |
| --- | --- | --- |
| `main` → `Intern` | `std::string`, `std::string` | formName（キー）と target |
| `Intern` 内部 | `FormInfo` / `FormCreator` | 名前と `AForm*(*)(target)` の組 |
| `Intern` → `main` | `AForm*` | 成功時はヒープ上の具体型。失敗時は `NULL` |
| `main` → `Bureaucrat` | `AForm&` / `AForm const&` | 署名・実行。所有権は移さない |
| `main` → ヒープ | `delete` | 仮想デストラクタ経由で具体型を破棄 |


---

## 色凡例


| 色 | 意味 |
| --- | --- |
| 🔵 青 | 呼び出し側（`main`） |
| 🟢 緑 | 生成役（`Intern` / 表） |
| 🟠 アンバー | 書類（`AForm*` / 具体型） |
| ⚫ グレー | 官僚（署名・実行） |


---

## 読み方の要点

1. IRC の CommandDispatcher と同じ型の問題である。**文字列から処理を選ぶ。** if/else の森ではなく、表にまとめる。
2. 失敗時に `NULL` を返すなら、呼び出し側の null チェックを忘れるとクラッシュする。
3. 署名・実行は ex02 と同じ経路。ex03 が足すのは **生成の入口だけ** である。
