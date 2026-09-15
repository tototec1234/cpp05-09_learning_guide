# ex01 解説 — ポインタと整数の往復

## 関連資料

- [モジュール全体](./CPP06_テーマと発展.md)
- [データフロー図](./CPP06_ex01_data_flow_diagram.md)

---

## 1. 課題の要求

利用者がインスタンス化できない`Serializer`クラスに、次の2関数を実装する。

```cpp
static uintptr_t serialize(Data *ptr);
static Data *deserialize(uintptr_t raw);
```

加えて、一つ以上のデータメンバを持つ`Data`構造体を作る。

テストの中心は次の往復である。

```text
Data object
   ↓ address
Data* original
   ↓ serialize
uintptr_t raw
   ↓ deserialize
Data* restored
```

`restored == original`を確認する。

---

## 2. この課題の「serialization」

一般的なserializationは、オブジェクトの状態を保存・転送できる形式へ変換する。

例:

```text
Data{name: "Alice", score: 42}
        ↓
JSON、バイナリ列、ファイルなど
```

ex01はこれを行わない。

```text
Dataオブジェクトのアドレス
        ↓
同じポインタ値を表す整数
```

したがって、この整数には次の性質がない。

- 別プロセスで復元できる
- プログラム再起動後に復元できる
- `Data`の内容を保持する
- 暗号化されている
- 改ざんを検出できる

過去のレビューコメントには「暗号化に近いのではないか」という疑問があるが、暗号化ではない。型名を知らないことは、暗号学的な秘密性にならない。

---

## 3. `reinterpret_cast`の役割

### 3.1 serialize

```text
Data* ptr
   ↓ reinterpret_cast<uintptr_t>
uintptr_t raw
```

ポインタを、その値を保持できる整数型へ変換する。

### 3.2 deserialize

```text
uintptr_t raw
   ↓ reinterpret_cast<Data*>
Data* ptr
```

整数を元と同じポインタ型へ戻す。

C++の保証は往復方向を区別する。

> ポインタを十分な大きさの整数型へ変換し、その整数を元と同じポインタ型へ戻すと、元のポインタ値を得る。

任意の整数をポインタへ変換し、そのポインタを安全に参照できるという保証ではない。

---

## 4. `uintptr_t`とC++98

`uintptr_t`は、オブジェクトポインタを保持できる符号なし整数型である。ただし、言語バージョンとヘッダに注意する。

- C99: `<stdint.h>`にグローバルな`uintptr_t`
- C++11: `<cstdint>`に`std::uintptr_t`。処理系によってグローバル名も提供される
- 標準C++98: `uintptr_t`自体を標準化していない

課題書は`uintptr_t`というシグネチャと`-std=c++98`の両方を要求する。42の対象環境では、C互換ヘッダ`<stdint.h>`からグローバルな`uintptr_t`を使う構成を確認する。

`std::uintptr_t`を使うために`<cstdint>`を選ぶと、C++11機能へ依存する。提出時は指定コンパイルオプションで確認する。

---

## 5. `Serializer`をインスタンス化させない

`Serializer`は状態を持たず、すべての操作がstaticである。

C++98では、生成やコピーに関係する関数をprivateへ置く。

```cpp
class Serializer {
private:
    Serializer();
    Serializer(Serializer const &other);
    Serializer &operator=(Serializer const &other);
    ~Serializer();

public:
    static uintptr_t serialize(Data *ptr);
    static Data *deserialize(uintptr_t raw);
};
```

これは構造を示すスケルトンである。private関数を実際に呼ばない設計なら、定義しない方法が使える。課題のOCF規則と「初期化できない」という個別要件を両方説明できるようにする。

---

## 6. `Data`とオブジェクト寿命

`Data`は空であってはいけない。

```cpp
struct Data {
    int number;
    std::string text;
};
```

これは構造例であり、メンバ名と型は自由である。

往復中、元の`Data`オブジェクトは生存していなければならない。

```text
作成 → serialize → deserialize → 比較・参照 → 破棄
```

先に破棄すると、戻ったポインタは寿命を終えたオブジェクトの場所を表すだけになる。`reinterpret_cast`はオブジェクトを再作成しない。

スタック上の`Data`でも、ヒープ上の`Data`でもテストできる。`new`を使った場合は`delete`が必要である。

---

## 7. テスト

### 7.1 最小テスト

1. 非空の`Data`を作る
2. 元のポインタを表示する
3. `serialize`して整数を得る
4. `deserialize`してポインタへ戻す
5. 元と戻したポインタを比較する
6. 戻したポインタからメンバを確認する

ポインタとメンバが同じであることには、異なる意味がある。

- ポインタが同じ: 往復保証を確認
- メンバが同じ: 同じ生存中オブジェクトを参照できることを確認

メンバが複製されたわけではない。

### 7.2 追加テスト

- `Data`を2個作り、整数値と復元先を取り違えない
- nullポインタを往復させる
- stack上とheap上のオブジェクトで試す
- `sizeof(uintptr_t)`と`sizeof(Data*)`を観察する

サイズが等しいこと自体は課題の判定条件ではない。重要なのは、対象環境の`uintptr_t`がオブジェクトポインタ値を保持できることである。

---

## 8. レビューコメントの検証

### 正しい点

- `reinterpret_cast`はポインタと整数の低水準変換に使える
- `serialize`と`deserialize`は逆方向の処理である
- 往復後のアドレスを比較する必要がある
- `uintptr_t`はC99由来で、`std::uintptr_t`はC++11で標準化された

### 訂正する点

「reinterpret_castで、それ以外は未定義動作」という一括した説明は不正確である。`reinterpret_cast`には、ポインタ同士、ポインタと整数、関数ポインタなど複数の規定された形式がある。それぞれ保証が異なる。

このexerciseでは範囲を広げず、`Data* → uintptr_t → Data*`だけを扱う。

---

## 9. 実装前チェック

- [ ] 一般的なserializationとの違いを説明できる
- [ ] `reinterpret_cast`を使う理由を説明できる
- [ ] `uintptr_t`と`unsigned int`の違いを説明できる
- [ ] 元と同じポインタ型へ戻す
- [ ] `Data`が非空である
- [ ] `Data`の寿命内に復元ポインタを使う
- [ ] `Serializer`を利用者が生成できない
- [ ] `Data`用ファイルを提出する
- [ ] `-std=c++98`でヘッダと型名を確認する

## 参考資料

- [reinterpret_cast（cppreference）](https://en.cppreference.com/w/cpp/language/reinterpret_cast)
- [固定幅整数型（cppreference）](https://en.cppreference.com/w/cpp/types/integer)

次: `CPP06_ex01_事後クイズ.md`
