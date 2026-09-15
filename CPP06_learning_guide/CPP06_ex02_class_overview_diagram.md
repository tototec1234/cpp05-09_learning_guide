# クラス関係図 — CPP06 ex02

> スコープ: `Base`、`A`、`B`、`C`の継承関係と、課題指定の関数

## クラス構造

```mermaid
classDiagram
    direction TB

    class Base {
        +~Base()*
    }

    class A {
        <<empty>>
    }

    class B {
        <<empty>>
    }

    class C {
        <<empty>>
    }

    class Functions {
        <<free functions>>
        +generate() Base*
        +identify(Base* p) void
        +identify(Base& p) void
    }

    Base <|-- A : public
    Base <|-- B : public
    Base <|-- C : public
    Functions ..> Base : creates / identifies

    style Base fill:#D9822B,stroke:#9A5A1E,color:#fff
    style A fill:#F8D9B0,stroke:#D9822B,color:#3A2A18
    style B fill:#F8D9B0,stroke:#D9822B,color:#3A2A18
    style C fill:#F8D9B0,stroke:#D9822B,color:#3A2A18
    style Functions fill:#4A90D9,stroke:#2E5A8B,color:#fff
```

`*`は仮想関数を示す。`Base`のpublic仮想デストラクタによって、`Base`はポリモーフィック型になる。

## 静的型と動的型

```mermaid
flowchart LR
    create["new B"] --> object["動的型 B の<br/>オブジェクト"]
    object --> upcast["public upcast"]
    upcast --> basePointer["静的型 Base*"]
    basePointer --> check["dynamic_cast<B*>"]
    check --> success["B* 成功"]

    style object fill:#F8D9B0,stroke:#D9822B,color:#3A2A18
    style basePointer fill:#4A90D9,stroke:#2E5A8B,color:#fff
    style success fill:#50B878,stroke:#3A8A5A,color:#fff
```

upcast後も、動的型Bのオブジェクトは変化しない。`Base*`という式から利用できるインターフェースがBaseに限定される。

## デストラクタの二つの役割

1. `Base*`を通して`delete`したとき、実際の派生型から破棄する
2. `Base`をポリモーフィック型にし、実行時検査付き`dynamic_cast`を使えるようにする

`A`、`B`、`C`は空でよい。型が異なること自体を判別に使う。
