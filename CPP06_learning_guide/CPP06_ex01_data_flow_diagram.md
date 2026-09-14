# データフロー図 — CPP06 ex01

> `Data*`を整数へ変換し、同じポインタ値へ戻す流れを示す。

## 往復シーケンス

```mermaid
sequenceDiagram
    participant Main as main
    participant Data as Data object
    participant Serializer as Serializer

    Main->>Data: non-empty Dataを作成
    Data-->>Main: Data* original
    Main->>Serializer: serialize(original)
    Note over Serializer: reinterpret_cast<uintptr_t>
    Serializer-->>Main: uintptr_t raw
    Main->>Serializer: deserialize(raw)
    Note over Serializer: reinterpret_cast<Data*>
    Serializer-->>Main: Data* restored
    Main->>Main: restored == original を確認
    Main->>Data: restoredからメンバを確認
```

## データとポインタ値を分ける

```mermaid
flowchart LR
    subgraph Object["オブジェクト"]
        data["Data<br/>number / text"]
    end

    original["Data* original"] -->|"指す"| data
    original -->|"reinterpret_cast"| raw["uintptr_t raw"]
    raw -->|"reinterpret_cast"| restored["Data* restored"]
    restored -->|"同じオブジェクトを指す"| data

    style data fill:#F8D9B0,stroke:#D9822B,color:#3A2A18
    style original fill:#4A90D9,stroke:#2E5A8B,color:#fff
    style raw fill:#D9822B,stroke:#9A5A1E,color:#fff
    style restored fill:#50B878,stroke:#3A8A5A,color:#fff
```

`raw`が保持するのはポインタ値である。`Data`のメンバは`raw`の中へコピーされない。

## 保証が成立する範囲

```text
正しい往復:
生存中のData* → 十分な大きさの整数型 → 元と同じData*

保証されない使い方:
任意の整数 → Data* → 参照
破棄済みData* → 整数 → Data* → 参照
別プロセスへ整数を送信 → Data*として復元
```

戻ったポインタを使う時点まで、元の`Data`オブジェクトを生存させる。
