# AIコーディングエージェントによる退行防止の実践原則

- 状態: Draft
- バージョン: 0.1.0
- 対象: AIコーディングエージェントを利用するソフトウェア開発
- 適用範囲: `moukaeritai.work` 配下の各プロジェクト、および同様の開発環境

## 1. 目的

AIコーディングエージェントによる開発では、すべての退行を完全に防止することは現実的ではない。

重要なのは、原則を増やし続けることではなく、次の条件を満たす少数の仕組みを継続して運用することである。

- AIが誤った変更を生成しても検出できる。
- 過去に発生した退行が再発しにくい。
- 検証の追加によって新しい過剰制約が生まれない。
- 一回の作業範囲が限定され、修正しやすい。
- 検証機構自身の変更も一定の検査を受ける。

本書では、実際に退行防止への効果が認められた原則を「中核原則」とし、それ以外を補助的な施策として整理する。

## 2. 基本方針

AIコーディングエージェントに対して、常に完全な判断を要求してはならない。

目標は次の状態である。

> AIが誤ることを前提とし、その誤りを小さな範囲に限定し、テストと検証によって発見し、次の小さな修正で回復できる状態を作る。

したがって、退行防止では次の三点を優先する。

1. 過去の退行を具体的なテストとして残す。
2. 検証ロジックとテストの乖離を防ぐ。
3. 一度に実施する変更を限定する。

## 3. 中核原則

### 原則1: 過去に発生した退行を回帰テストとして固定する

最も重要な原則である。

抽象的な「品質向上」よりも、実際に発生した退行を具体的なテストとして残す方が効果が高い。

例えば、過去に次の問題が起きたとする。

- 有効な動的パスが拒否された。
- 空の説明文が許可された。
- 必須ルートが inactive でも検証を通過した。
- 検証スクリプトの変更で必要なテストが選択されなくなった。

これらは、それぞれ独立したテストとして残す。

```text
allows a valid multi-parameter route
rejects an empty description
rejects an inactive required route
verification-script changes select the conservative test set
```

#### 正常系と異常系を両方固定する

不正な状態を拒否するテストだけでは不十分である。

検証器が過剰に厳格にならないように、許可すべき状態もテストする。

```text
拒否すべき例:
- 空文字列
- 不正な enum
- 非 boolean 値
- 必須項目の欠落

許可すべき例:
- 現在未登録だが仕様上は正当な値
- 動的パラメータを複数含むパス
- 推奨 prefix とは異なるが許可されるパス
```

退行防止テストは次の両方を表現しなければならない。

```text
何を拒否するか
何を拒否してはならないか
```

### 原則2: テストは production 実装を直接使用する

検証ロジックをテスト側へコピーしてはならない。

不適切な構造:

```text
production:
  分類用正規表現

test:
  同じ正規表現のコピー
```

この構造では、production と test が同じ誤りを持ったまま成功する可能性がある。

適切な構造:

```text
production:
  classifyChangedFiles()

test:
  classifyChangedFiles() を import して実行
```

Validator についても同様である。

```js
export function validateCatalog(entries) {
  return errors;
}
```

テストはこの関数を直接呼び出す。

```js
import { validateCatalog } from '../scripts/lib/catalog-validator.mjs';
```

#### 適用対象

この原則は、特に次のロジックへ適用する。

- changed-file classifier
- catalog validator
- schema validator
- version transition 判定
- workspace metadata 同期判定
- verification plan builder

#### 例外

CLI や Git 操作を含む最終スクリプトには integration test が必要になる。

ただし、その内部の判定ロジックは可能な限り純粋関数として分離する。

### 原則3: 検証器は文書化された規則だけを検査する

Validator や hardening script が、新しい設計規則を勝手に作ってはならない。

特に、次のような推測を禁止する。

- 分類値から URL prefix を強制する。
- 現在存在する値だけを許可リストにする。
- 動的パスを閉じた一覧にする。
- access 値から別の role 体系を作る。
- 将来追加される正当な値を禁止する。
- 明示されていない互換性や migration を追加する。

例えば、

```text
surface = app
```

であることから、

```text
path は必ず /app で始まる
```

と推測してはならない。

推奨される命名規則と、機械的に強制する不変条件は別である。

#### 新しい制約を追加できる条件

新しい検証制約を追加する場合は、次のいずれかを必要とする。

1. 既存の authoritative document に根拠がある。
2. 人間が規約変更を明示的に承認している。
3. 実際の障害や退行を防ぐための具体的な根拠がある。

「安全そうだから」「現在の実装がそうだから」という理由だけでは追加しない。

### 原則4: 検証基盤の変更には保守的な検証セットを適用する

Changed-file classifier は高速化に有効である。

しかし、classifier や validator 自身が変更された場合、その変更によって必要なテストが除外される危険がある。

したがって、次のような verification infrastructure が変更された場合は、通常分類とは別に固定された検証セットを選択する。

```text
classifier
verification plan builder
validator
version checker
bootstrap checker
boundary checker
関連テスト
root package manifest
root lockfile
```

最低限の保守的セット:

```text
bootstrap validation
version validation
主要 unit tests
主要 boundary tests
typecheck
build
```

この仕組みにより、検証器の修正が検証そのものを弱める可能性を抑える。

#### 完全な独立性は必須ではない

理想的には、固定検証を classifier の外側へ置く。

ただし、運用コストが高い場合は、production classifier 内で conservative set を選び、その挙動を production function を使ったテストで固定するだけでも一定の効果がある。

完全性より、実際に継続して動く仕組みを優先する。

### 原則5: 一回の作業を小さく限定する

一つの stride へ多くの目的を入れると、意味論的に重要な要求が落ちやすい。

避けるべき作業:

```text
classifier の抽出
validator の修正
CI 変更
version 設計変更
policy 追加
runtime 変更
documentation 再編
```

を一度に行う。

望ましい作業分割:

```text
Stride A:
classifier を単一化する

Stride B:
過去の退行をテスト化する

Stride C:
過剰制約を削除する

Stride D:
最終 gate へ接続する
```

#### プロンプトで明示する内容

各 stride では最低限、次を指定する。

```text
Objective
Allowed files
Forbidden files
削除対象
追加対象
変更してはいけない挙動
Verification commands
Completion criteria
```

特に、削除対象は抽象表現ではなく識別子単位で指定する。

不十分:

```text
過剰な制約を削除する
```

望ましい:

```text
Delete:
- authorizedDynamicPaths
- surfaceMap
- pathPrefixAlignment
- "unauthorized wildcard" を出す分岐
```

#### 機械的 scope enforcement は任意

Scope manifest や diff validator は有効だが、すべてのプロジェクトで必須ではない。

まずはプロンプト上の allowed/forbidden files と、作業後の `git diff --name-only` 確認を徹底する。

それでも scope 逸脱が繰り返される場合に、機械的な scope validation を追加する。

### 原則6: 明示された入力が無効なら fail closed する

ユーザーや CI が明示的に指定した値は、黙って別の値へ置き換えてはならない。

例:

```text
明示された Git base ref
deployment target
environment
schema version
configuration path
```

正しい挙動:

```text
明示値が指定される
    ↓
その値を検証する
    ↓
無効なら失敗する
```

避けるべき挙動:

```text
明示値が無効
    ↓
エラーを無視する
    ↓
別の候補へ fallback する
```

暗黙の fallback は、意図しない対象を検証・変更する原因になる。

## 4. 重要だが補助的な原則

以下は有効だが、すべてのプロジェクトで最初から完全に実装する必要はない。

### 4.1 責務を分離する

次を一つの validator へまとめない。

```text
データ構造検証
runtime 存在確認
authorization
version 判定
migration
deployment
```

望ましい分離:

```text
catalog validator:
  データ構造と documented invariant

runtime boundary checker:
  runtime 登録と guard

version checker:
  version と Git history

classifier:
  changed files から verification plan を構築
```

責務分離は、誤った既存ロジックを丸ごと共通化する危険を減らす。

### 4.2 実際の production data をテストする

Fixture だけではなく、実際の設定や catalog を production module から import して検証する。

```js
import { routes } from '../src/lib/routeCatalog';

expect(validateRouteCatalog(routes)).toEqual([]);
```

これにより、validator の単体仕様と実際の repository state の乖離を検出できる。

### 4.3 Version metadata の同期を検証する

Application version が複数箇所に存在する場合、同期チェックは有効である。

代表的な対象:

```text
root manifest
root lockfile
workspace manifests
workspace lockfiles
application profile
README
agent guidelines
```

ただし、最初から完全な共通 metadata library を作る必要はない。

重要なのは、少なくとも次を自動検出できることである。

```text
一部だけ version が古い
lockfile root version が不一致
packages[""].version が不一致
version が減少した
code change に対して version が変わっていない
```

重複実装が存在しても、直ちに退行防止効果が失われるわけではない。

重複による保守負担が問題になった段階で共通化する。

### 4.4 重要な検証を最終 gate へ接続する

Test script が存在するだけでは、恒常的な退行防止にはならない。

次を確認する。

```text
unit test は root test に含まれるか
boundary checker は PR verification に含まれるか
version checker は release verification に含まれるか
classifier tests は classifier 変更時に実行されるか
```

ただし、最初からすべての検証を最終 gate へ追加すると、実行時間や環境依存性が増える。

次の順序で導入する。

1. 過去に退行を検出した検証。
2. 認証、データ破壊、deployment に関係する検証。
3. 検証基盤自身に関係する検証。
4. その他の低リスク検証。

## 5. 必須ではない施策

以下は有効だが、導入コストが高いため、問題が顕在化した場合に追加する。

### 5.1 Scope manifest

```json
{
  "allowedFiles": [],
  "forbiddenFiles": [],
  "requiredCommands": []
}
```

Scope 逸脱が繰り返される場合に導入する。

### 5.2 CODEOWNERS

検証基盤や policy 変更に人間レビューを要求したい場合に利用する。

直接 push の挙動を制御できない環境では補助的な役割となる。

### 5.3 完全に独立した Layer 1 gate

Classifier に依存しない固定検証は理想的である。

しかし、既存の conservative set とその回帰テストで十分な効果が得られている場合、必ずしも直ちに導入する必要はない。

### 5.4 Test-first 実行ログの保存

修正前の test failure を保存することは望ましい。

ただし、ログ保存自体が複雑になる場合は、最終状態の回帰テストが明確であることを優先する。

## 6. AI向け stride prompt の最小構成

AIへの指示は長大にしすぎない。

最低限、次を含める。

### Objective

```text
今回修正する一つの問題
```

### Current defect

```text
現在何が誤っているか
```

### Required regression tests

```text
修正後に通るべき具体例
修正後も失敗すべき具体例
```

### Exact changes

```text
削除する識別子
追加する判定
変更しない部分
```

### Scope

```text
Allowed files
Forbidden files
```

### Verification

```text
focused test
related test
final verification
```

### Completion criteria

```text
観測可能な完了条件
```

詳細な背景説明は、判断に必要な範囲だけにする。

## 7. 優先順位

限られた時間で導入する場合は、次の順序を推奨する。

### 最優先

1. 過去の退行を回帰テストにする。
2. Positive test と negative test を両方作る。
3. Tests が production implementation を直接使う。
4. 一 stride を一目的に限定する。
5. Undocumented constraint を追加しない。

### 次に実施

6. Verification infrastructure 変更時の conservative set。
7. Explicit input の fail-closed 化。
8. Production data を直接検証する test。
9. 重要な boundary check の final gate 接続。

### 必要になった場合に実施

10. Version metadata library の完全な共通化。
11. Scope manifest。
12. CODEOWNERS。
13. 独立した Layer 1 gate。
14. Test-first 実行ログの永続化。

## 8. 適合度の考え方

すべての原則へ完全に適合する必要はない。

退行防止の効果は、原則の数ではなく、次の組み合わせによって生まれる。

```text
過去の退行を表す test
+
production implementation を直接使う test
+
過剰制約を防ぐ positive test
+
小さく限定された stride
+
verification infrastructure 変更時の保守的検証
```

この中核が機能していれば、scope manifest や完全な共通 library が未導入でも、実用上の退行防止効果は期待できる。

適合度は、チェック項目の達成率だけで評価してはならない。

次の問いを重視する。

```text
過去に起きた退行を現在の test は検出できるか
同じ種類の退行が再導入された場合に失敗するか
AIが過剰な制約を追加した場合に positive test が検出するか
検証基盤が変更された場合に十分な test が選択されるか
変更範囲が小さく、次の修正で回復できるか
```

## 9. まとめ

AIコーディングエージェントの退行防止では、完全なガバナンス体系よりも、効果の高い少数の仕組みを継続して使う方が重要である。

中核となるのは次の六原則である。

1. 過去の退行を具体的な回帰テストとして残す。
2. Tests は production implementation を直接使用する。
3. 検証器は文書化された規則だけを検査する。
4. 検証基盤の変更には保守的な検証セットを適用する。
5. 一回の作業を小さく限定する。
6. 明示された無効な入力は fail closed する。

これらが実装されていれば、完全な適合でなくても実用的な効果が得られる。

最終的な目標は、原則を増やすことではない。

> 実際に発生した退行から学び、その退行を検出する最小限の仕組みを、次の作業でも維持できる形で残すことである。

## 10. 変更履歴

### 0.1.0

- 初版。
- 実際に退行抑止への効果が認められた六つの中核原則を中心に整理した。
- Scope manifest、CODEOWNERS、独立した Layer 1 gate などは必要時に導入する補助施策として位置付けた。
