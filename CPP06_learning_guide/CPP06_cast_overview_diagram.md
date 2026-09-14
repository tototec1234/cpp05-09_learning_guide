# キャスト選択図 — CPP06

> 用途: 変換の目的から、検討するキャストを選ぶ  
> 注意: キャストを選んだ後も、値域、寿命、型関係を別に確認する

## 選択フロー

```mermaid
flowchart TD
    start["何を変換するか"] --> scalar{"数値・文字などの<br/>値を別の型へ変換？"}
    scalar -->|はい| static["static_cast"]
    scalar -->|いいえ| pointerInt{"ポインタと整数の間など<br/>低水準の表現変換？"}
    pointerInt -->|はい| reinterpret["reinterpret_cast"]
    pointerInt -->|いいえ| hierarchy{"継承階層内で<br/>実際の型を検査？"}
    hierarchy -->|はい| dynamic["dynamic_cast"]
    hierarchy -->|いいえ| cv{"const / volatile<br/>だけを変更？"}
    cv -->|はい| constcast["const_cast"]
    cv -->|いいえ| redesign["キャスト以外の設計を検討"]

    static --> staticCheck["変換先の値域を確認<br/>範囲外ならキャストしない"]
    reinterpret --> reinterpretCheck["整数型の幅・同一プロセス・<br/>オブジェクト寿命を確認"]
    dynamic --> dynamicCheck["基底型がpolymorphicか確認<br/>失敗経路を処理"]
    constcast --> constCheck["元からconstの対象を<br/>書き換えない"]

    style static fill:#4A90D9,stroke:#2E5A8B,color:#fff
    style reinterpret fill:#D9822B,stroke:#9A5A1E,color:#fff
    style dynamic fill:#50B878,stroke:#3A8A5A,color:#fff
    style constcast fill:#8E6BBE,stroke:#62468B,color:#fff
```

## CPP06での対応

| キャスト | exercise | 対象 | キャストが保証しないこと |
| --- | --- | --- | --- |
| `static_cast` | ex00 | スカラー型間の値変換 | 範囲内か、表示可能か |
| `reinterpret_cast` | ex01 | `Data*`と`uintptr_t` | データの複製、永続化、暗号化 |
| `dynamic_cast` | ex02 | `Base*` / `Base&`から派生型 | オブジェクトの寿命、メモリ解放 |
| `const_cast` | 必須処理には不使用 | cv修飾 | 元からconstの対象を書き換える権利 |

## 区別する用語

```text
promotion
└─ 標準変換の一部。例: char → int、float → double

upcast
└─ 派生ポインタ・参照 → 公開基底ポインタ・参照

downcast
└─ 基底ポインタ・参照 → 派生ポインタ・参照
   実際の型が不明なら dynamic_cast で検査する

object slicing
└─ 派生オブジェクトを基底オブジェクトの値へコピーし、
   派生部分がコピー先に含まれない現象
```

ポインタまたは参照のupcastではobject slicingは起きない。
