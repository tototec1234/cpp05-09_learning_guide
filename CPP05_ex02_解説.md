# ex02 解説 — AForm と実行

提出物: `Bureaucrat`、`AForm`、3つの具体 Form、`Makefile`、`main.cpp`  
Form は「紙」から「手続き」になる。チェックは共通、PresidentialPardon,RobotomyRequest,ShrubberyCreationでそれぞれ独自の`side effect` を持つ。

ここでの side effect とは（クリックで表示）

- 薬の副作用（避けたい付随物）ではない。
- 関数が戻り値以外に、外の世界を変えること。ファイルを書く、標準出力に出す、が当たる。  
- 上記の文では、実行( ***execute***)が通ったあとに各 Form が行う本体の動作を指す。
- チェック（署名済みか、等級は足りるか）は3クラス共通であるが、side effect（植樹・ロボトミー・恩赦）は各クラス固有。



---

## 1. ex01 からの変化

ex01 Form   : 署名(***sign***)できるかどうか
ex02 AForm  : 署名(***sign***)したうえで、実行(***execute***)できるかどうか

3つの具体クラス:


| クラス                    | sign | exec | side effect                                        |
| ---------------------- | ---- | ---- | -------------------------------------------------- |
| ShrubberyCreationForm  | 145  | 137  | `<target>_shrubbery` に ASCII ツリーを書く                |
| RobotomyRequestForm    | 72   | 45   | ドリル音のあと、50% 成功 / 50% 失敗と出力                         |
| PresidentialPardonForm | 25   | 5    | `<target> has been pardoned by Zaphod Beeblebrox.` |


コンストラクタは target のみ。等級を引数にしない。

---



## 2. チェックをどこに置くか

課題書 p13: 具体クラスでチェックしても、基底でチェックして別関数を呼んでもよい。

```
Whether you check the requirements in every concrete class or in the base class (and
then call another function to execute the form) is up to you. However, one way is more
elegant than the other.
```

と書かれているが、
共通処理を基底クラスにまとめ、別関数を呼ぶ形を Template Method という、通常はこちらのほうが elegant である。

Template Method:

```cpp
AForm::execute(executor) const:
    if 未署名: throw 未署名例外
    if executor.grade数値 > _executeGrade: throw GradeTooLowException
    executeEachForm(executor)   # 純粋仮想。派生が実装

ShrubberyCreationForm::executeEachForm:
    ファイルを開いてツリーを書く

RobotomyRequestForm::executeEachForm:
    音を出して 50% 判定

PresidentialPardonForm::executeEachForm:
    恩赦の一文を出す
```

派生が***execute***を全部自分で書くと、未署名チェックを1クラス忘れうる。  
基底に寄せると、忘れられない。
なお、ドメイン操作 ***execute*** を純粋仮想にして、各派生クラスの `executeEachForm()` の先頭に同じ if をそれぞれ書く形も、当然機能する。課題書はどちらでもよい、としているが
チェックが3箇所に分かれるので、elegant とは思えない。

スケルトン:

```cpp
class AForm {
public:
    virtual ~AForm();
    void execute(Bureaucrat const &executor) const;
    // beSigned, getters, OCF, 例外クラス
protected:
    virtual void executeEachForm(Bureaucrat const &executor) const = 0;
private:
    const std::string _name;
    bool _isSigned;
    const int _signGrade;
    const int _executeGrade;
};
```

ドメイン操作 ***execute*** を実装するための純粋仮想の関数名 `executeEachForm` はこの実装例での命名であり、チェックを省いた処理本体（side effect）を担う pure virtual であれば名前は自由につけてよい。

`execute()` に `virtual` をつけないのは意図的である。

- `virtual` があると派生クラスが `execute()` をオーバーライドしてチェックを丸ごと省ける。
- Template Method の目的は「チェックを基底に固定し、派生クラスの実装者がチェックを書き忘れられない構造にする」ことなので、テンプレート本体（`execute()`）は非 `virtual` にする。

---



## 3. 例外と多態性が交差する3点

- CPP05 全体の主題は例外処理だが、ex02 の新しさは「新しい例外を増やすこと」ではない。
- 投げる例外の種類は ex01 とほぼ同じで、変わるのは**型階層のどこで投げ、どこで捕まえるか**である。

例外と多態性が実際に噛み合うのは次の3点 3.1  3.2  3.3に限られる。
残りは同じ課題に並んでいるだけで、互いに依存していない。

### 3.1 多態にしない、という判断

- `execute()` を非 `virtual` にするのは、前提条件チェックを派生クラスによる差し替えの対象から外すためである（2節）。
- 派生に開くのは `executeEachForm()` だけで、前提条件チェックの例外を投げる位置は基底に固定される。派生が side effect の実行中に投げる例外（ファイルオープン失敗など）はこれとは別で、3.2 の経路に乗る。
CPP04 が「差し替えられること」を学ぶ課題だったのに対し、ex02 は「差し替えさせない場所を決めること」を学ぶ。



### 3.2 抽象型越しに投げ、抽象型で捕まえる

- `Bureaucrat::executeForm(AForm const &)` は、どの具体 Form を受け取ったかを判定しない。`catch (std::exception &)` で受けるため、投げた側が `AForm` の未署名チェックなのか `ShrubberyCreationForm` のファイルオープン失敗なのかも区別しない。

Form の型階層と、例外クラスの型階層（`what()` が仮想）が、ひとつの catch 節で合流する。

### 3.3 派生コンストラクタの初期化リストからの送出

- 具体 Form のコンストラクタは初期化リストで `AForm(name, sign, exec)` を呼ぶ。
- ここで等級範囲の例外が飛ぶと、派生クラスのコンストラクタ本体は実行されず、オブジェクトは生成されない。

`AForm*` を返す ex03 の `Intern` は、この性質の上に乗っている。

### 交差しない部分

- 例外の種類そのもの（6節の表）は ex01 からの持ち越しで、多態性がなくても同じ内容になる
- 3クラスの side effect（植樹・ロボトミー・恩赦）の**内容そのもの**は多態性だけの話で、例外は絡まない。ただし side effect の実行が失敗して例外を投げる場合（Shrubbery のファイルオープン失敗、5節）は 3.2 の交差点に戻る

---



## 4. Bureaucrat::executeForm

課題:

```
成功: <bureaucrat> executed <form>
失敗: 明確なエラーメッセージ
```

疑似コードは `signForm` と同じ構造。

```
executeForm(form) const:
    try:
        form.execute(*this)
        成功メッセージ
    catch (std::exception &e):
        失敗メッセージ（e.what() を理由にしてよい）
```

過去のレビューコメント（提出者不明）:

> `Bureaucrat::executeForm(AForm const & form)` これだけ足りていなかった

公開の typo 指摘では、課題書本文が `Form`、評価シートが `AForm` と揺れる、とある。  
ex02 以降の引数型は `AForm const &` にする。関数自体を忘れない。

`signForm` は非 const、`executeForm` は課題が `const` を付けている。官僚の状態は実行では変わらない。

---



## 5. 具体クラスでつまずく点



### Shrubbery

- ファイル名は `<target>_shrubbery`
- `<fstream>` はコンテナではない。使ってよい
- 開けないときは例外を投げてよい（課題はファイル失敗を詳しく書いていない。評価で聞かれたら説明できるようにする）
- ASCII ツリーの見た目は指定されていない



### Robotomy

- 50%: `rand()` / `std::rand()` と `time` で種をまく実装が多い
- 種を `execute` のたびに `srand(time(NULL))` すると、1秒以内の連続実行が同じ結果になる
- 種は `main` で一度、または静的フラグで一度



### Presidential

- 出力文面が課題に固定されている

3クラスとも OCF を書く。target をメンバに持つ。target は課題が const とは書いていないが、実行中に変える理由はない。const にしてよい。

具体クラスのコンストラクタから `AForm(name, sign, exec)` を初期化リストで呼ぶ。  
name は `"ShrubberyCreationForm"` のように型名でも、課題の短い名前でも、評価者が `<<` とメッセージで識別できればよい。Intern（ex03）は文字列 `"shrubbery creation"` で生成するので、Form の `getName()` と Intern のキーは別物、と割り切ってよい。

---



## 6. 例外の種類


| 状況                  | 投げうるもの                                       |
| ------------------- | -------------------------------------------- |
| AForm の等級が 1..150 外 | `AForm::GradeTooHigh/LowException`（ex01 と同じ） |
| 署名時に官僚が足りない         | `GradeTooLowException`                       |
| 未署名で execute        | 独自例外が分かりやすい                                  |
| 実行時に官僚が足りない         | `GradeTooLowException`                       |


`GradeTooLowException` が「書類の等級が 150 超」と「官僚が足りない」の両方に使われる。  
`what()` の文言で区別するか、未署名だけ別クラスにする。  
課題はクラスを増やせとは書いていない。説明できることが評価の本体である。

---

j

## 7. テスト

- 3種それぞれ、署名できる官僚とできない官僚
- 署名せずに execute
- 署名したが exec 等級が足りない
- `executeForm` の成功・失敗メッセージ
- shrubbery ファイルができたか
- robotomy を複数回（片方に偏りすぎないこと。厳密 50.000% は要求されない）
- `AForm*` で保持して `delete`（virtual デストラクタの確認）

---



## 8. STL

`<fstream>`、`<ctime>`、`<cstdlib>` はコンテナではない。  
`std::vector` に Form を並べるのはまだ禁止。

---

次: `CPP05_ex02_事後クイズ.md`