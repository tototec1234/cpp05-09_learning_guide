# CPP06 予習教材

対象: C++ Module 06「C++ casts」  
基準: C++98、課題書 Version 8.1  
方針: 完成コードは示さず、判断手順、疑似コード、テスト観点を示す。

## 使い方

1. `CPP06_テーマと発展.md` で、4種類のキャストと3 exerciseの関係を確認する。
2. 各 exercise を **事前クイズ → 解説 → 事後クイズ → 自分で実装** の順で進める。
3. クイズは先に自分で回答し、その後 `<details>` 内の模範回答を確認する。
4. 図は処理の順序や型の関係が分からなくなったときに参照する。

## ファイル一覧

- `CPP06_テーマと発展.md`: モジュール全体の目的、4種類のキャスト、評価対策
- `CPP06_cast_overview_diagram.md`: キャスト選択の全体図
- `CPP06_ex00_事前クイズ.md` / `CPP06_ex00_解説.md` / `CPP06_ex00_事後クイズ.md`
- `CPP06_ex00_conversion_flow_diagram.md`: 文字列判定から4型の表示まで
- `CPP06_ex00_範囲外変換と未定義動作.md`: 変換前に範囲を確認する理由
- `CPP06_ex01_事前クイズ.md` / `CPP06_ex01_解説.md` / `CPP06_ex01_事後クイズ.md`
- `CPP06_ex01_data_flow_diagram.md`: `Data*`から整数を経由して同じポインタへ戻る流れ
- `CPP06_ex02_事前クイズ.md` / `CPP06_ex02_解説.md` / `CPP06_ex02_事後クイズ.md`
- `CPP06_ex02_class_overview_diagram.md`: `Base`、`A`、`B`、`C`の型関係
- `CPP06_ex02_identify_flow_diagram.md`: ポインタ版と参照版の判別手順

## 出典

- 課題書: `/Users/toruinoue/CPP_rote/6CPP/subject_cpp06.pdf`（Version 8.1）
- 過去のレビューコメント: `../../CPP05/review_comment_cpp06.txt`
- 言語仕様の確認: cppreference、C++ Working Draft、CERT C

過去のレビューコメントが誰の提出物を対象にしたものかは不明である。コメント中の説明は技術資料と照合し、条件を補足するか訂正した。

## 表現方針

設計の説明では、処理を「まとめる」「置く」「変換する」「確認する」など、操作が分かる動詞で表す。比喩的な移動動詞や、設計判断を文書解釈のように表す言い方は使わない。
