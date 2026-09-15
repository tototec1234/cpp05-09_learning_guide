# 実型判別フロー — CPP06 ex02

> ポインタ版と参照版では、`dynamic_cast`失敗時の挙動が異なる。

## ポインタ版

```mermaid
flowchart TD
    input["identify(Base* p)"] --> a{"dynamic_cast<A*>(p)<br/>はnullではない？"}
    a -->|はい| printA["A を表示"]
    a -->|いいえ| b{"dynamic_cast<B*>(p)<br/>はnullではない？"}
    b -->|はい| printB["B を表示"]
    b -->|いいえ| c{"dynamic_cast<C*>(p)<br/>はnullではない？"}
    c -->|はい| printC["C を表示"]
    c -->|いいえ| unknown["null または対象外"]

    style input fill:#4A90D9,stroke:#2E5A8B,color:#fff
    style printA fill:#50B878,stroke:#3A8A5A,color:#fff
    style printB fill:#50B878,stroke:#3A8A5A,color:#fff
    style printC fill:#50B878,stroke:#3A8A5A,color:#fff
```

ポインタへのキャストは、失敗時にnullポインタを返す。

## 参照版

```mermaid
flowchart TD
    input["identify(Base& p)"] --> tryA["dynamic_cast<A&>(p) を試す"]
    tryA -->|成功| printA["A を表示して終了"]
    tryA -->|例外| tryB["dynamic_cast<B&>(p) を試す"]
    tryB -->|成功| printB["B を表示して終了"]
    tryB -->|例外| tryC["dynamic_cast<C&>(p) を試す"]
    tryC -->|成功| printC["C を表示"]
    tryC -->|例外| unknown["対象外"]

    style input fill:#D9822B,stroke:#9A5A1E,color:#fff
    style printA fill:#50B878,stroke:#3A8A5A,color:#fff
    style printB fill:#50B878,stroke:#3A8A5A,color:#fff
    style printC fill:#50B878,stroke:#3A8A5A,color:#fff
```

参照へのキャストはnullを返せない。失敗時には`std::bad_cast`に一致する例外を送出する。

## 課題の禁止事項

```text
identify(Base& p)
    └─ identify(&p)  ← ポインタを使うため不可
```

参照版の関数内では、参照への`dynamic_cast`を使う。`<typeinfo>`はインクルードしない。

## generateから削除まで

```mermaid
sequenceDiagram
    participant Main as main
    participant Generate as generate
    participant Object as A / B / C
    participant Identify as identify

    Main->>Generate: generate()
    Generate->>Object: new A / B / C
    Generate-->>Main: Base*
    Main->>Identify: identify(Base*)
    Identify-->>Main: A / B / C
    Main->>Identify: identify(Base&)
    Identify-->>Main: 同じ型名
    Main->>Object: delete Base*
    Note over Object: virtual ~Base()
```
