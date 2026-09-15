# 変換フロー図 — CPP06 ex00

> 入力文字列の分類から、4種類の表示までの処理順序を示す。  
> キャスト前の値域確認を省略しない。

## 全体フロー

```mermaid
flowchart TD
    input["argv[1]: std::string"] --> pseudo{"6種類の<br/>疑似リテラル？"}
    pseudo -->|はい| pseudoValue["float / doubleの<br/>特殊値として処理"]
    pseudo -->|いいえ| charLiteral{"1文字の<br/>char入力？"}
    charLiteral -->|はい| charValue["char値"]
    charLiteral -->|いいえ| numeric["文字列全体を数値解析"]
    numeric --> valid{"全体を解析でき、<br/>範囲エラーなし？"}
    valid -->|いいえ| invalid["失敗理由に応じて<br/>impossibleを表示"]
    valid -->|はい| classify{"末尾f、小数点、<br/>指数部を確認"}
    classify --> intValue["int値"]
    classify --> floatValue["float値"]
    classify --> doubleValue["double値"]

    pseudoValue --> normalized["判定用の共通表現"]
    charValue --> normalized
    intValue --> normalized
    floatValue --> normalized
    doubleValue --> normalized

    normalized --> charCheck{"charの範囲内？"}
    charCheck -->|いいえ| charImpossible["char: impossible"]
    charCheck -->|はい| printable{"表示可能？"}
    printable -->|いいえ| nonDisplay["char: Non displayable"]
    printable -->|はい| charPrint["char: 'c'"]

    normalized --> intCheck{"有限かつ<br/>intへ安全に変換可能？"}
    intCheck -->|いいえ| intImpossible["int: impossible"]
    intCheck -->|はい| intPrint["int: value"]

    normalized --> floatPrint["float: value + f"]
    normalized --> doublePrint["double: value"]

    style input fill:#4A90D9,stroke:#2E5A8B,color:#fff
    style normalized fill:#D9822B,stroke:#9A5A1E,color:#fff
    style charPrint fill:#50B878,stroke:#3A8A5A,color:#fff
    style intPrint fill:#50B878,stroke:#3A8A5A,color:#fff
    style floatPrint fill:#50B878,stroke:#3A8A5A,color:#fff
    style doublePrint fill:#50B878,stroke:#3A8A5A,color:#fff
```

## 各段階の責務

| 段階 | 入力 | 出力 | 失敗条件 |
| --- | --- | --- | --- |
| 分類 | 文字列 | char / int / float / double / special | どの形式にも一致しない |
| 解析 | 文字列 | 分類した型の値 | 全体を解析できない、範囲エラー |
| 変換可否 | 値 | 各型で表示可能か | NaN、無限大、値域外 |
| 表示 | 値と可否 | 課題指定の4行 | `.0`、`f`、文言の不一致 |

## `char`だけ二つの失敗表示がある

```text
数値としてcharへ変換できない
└─ char: impossible

charへ変換できるが印字できない
└─ char: Non displayable
```

`std::isprint`へは`static_cast<unsigned char>(character)`を渡す。
