# キャスト選択図 — CPP06

> 用途: 変換の目的から、検討するキャストを選ぶ （一般的な用途でなく、CPP06を解くことに特化しています）

## 選択フロー
```mermaid
flowchart TD
    start["何を変換したいか？"] --> isCV{"1. const / volatile の<br/>脱着のみが目的か？"}
    
    isCV -->|はい| constcast["const_cast<br/>【cv修飾の変更】"]
    isCV -->|いいえ| isValue{"2. 対象はスカラー値そのものか？<br/>(char, int, float, double 等)"}
    
    isValue -->|はい| staticScalar["static_cast<br/>【ex00: 値表現の変換・昇格】"]
    isValue -->|いいえ| isDynamic{"3. 多態的（仮想関数を持つ）型の<br/>ダウンキャスト/実行時型検査か？"}
    
    isDynamic -->|はい| dynamic["dynamic_cast<br/>【ex02: 安全な型識別】"]
    isDynamic -->|いいえ| isLowLevel{"4. ポインタ ↔ 整数、または<br/>無関係なポインタ同士のビット再解釈か？"}
    
    isLowLevel -->|はい| reinterpret["reinterpret_cast<br/>【ex01: アドレスの整数化・再解釈】"]
    isLowLevel -->|いいえ| isSafePtr{"5. 言語仕様で保証された安全なポインタ変換か？<br/>(void* ↔ T*, 明示的アップキャスト等)"}
    
    isSafePtr -->|はい| staticPtr["static_cast<br/>【[発展] 安全なポインタ操作】"]
    isSafePtr -->|いいえ| redesign["キャスト以外の設計を検討<br/>(暗黙変換、インターフェース再設計)"]

    %% 検証ステップ
    staticScalar --> checkStatic["【事前/事後確認】<br/>値域・オーバーフローの検査<br/>(ex00: impossible / nan / inf)"]
    dynamic --> checkDynamic["【事前/事後確認】<br/>基底クラスに仮想デストラクタがあるか<br/>ポインタ: NULL検査 / 参照: bad_cast捕捉"]
    reinterpret --> checkReinterpret["【事前/事後確認】<br/>整数型の幅 (uintptr_t)<br/>アライメント保証・オブジェクト寿命"]
    constcast --> checkConst["【事前/事後確認】<br/>元オブジェクトがconst定義なら<br/>書き込み厳禁（未定義動作防止）"]

    style staticScalar fill:#4A90D9,stroke:#2E5A8B,color:#fff
    style reinterpret fill:#D9822B,stroke:#9A5A1E,color:#fff
    style dynamic fill:#50B878,stroke:#3A8A5A,color:#fff
    style constcast fill:#8E6BBE,stroke:#62468B,color:#fff
    style staticPtr fill:#4A90D9,stroke:#2E5A8B,color:#fff
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

