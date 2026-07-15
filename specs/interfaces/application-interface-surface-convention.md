# アプリケーション・インターフェース・サーフェス規約

- 状態: Draft
- バージョン: 0.1.0
- 対象: Webアプリケーション、CUIアプリケーション、API、関連文書および開発・検証用インターフェース
- 適用範囲: `moukaeritai.work` 配下の各プロジェクト、および本規約を採用するソフトウェア

## 1. 目的

本規約は、アプリケーションに対して利用者または他のソフトウェアが接触する目的を、共通の語彙によって分類する。

この分類を、Webのパス、CLIのサブコマンド、API文書、ナビゲーション、アクセス方針および開発文書の設計に再利用することで、複数のプロジェクト間における一貫性、予測可能性および保守性を高める。

本規約は、次の六つのアプリケーション・インターフェース・サーフェスを定義する。

```text
public
app
admin
dev
api
test
```

## 2. 規範用語

本書では、次の意味で用語を使用する。

- **必須**: 本規約へ適合するために従わなければならない。
- **禁止**: 実施してはならない。
- **推奨**: 特別な理由がない限り従う。
- **任意**: 必要性と費用を考慮して採用できる。

英語表記では、それぞれ `MUST`、`MUST NOT`、`SHOULD`、`MAY` に対応する。

## 3. サーフェスの定義

アプリケーション・インターフェース・サーフェスとは、次の問いに対する分類である。

> 誰が、何の目的でアプリケーションに接触するか。

サーフェスは、利用者または外部ソフトウェアから観測・操作されるインターフェースの意味上の区分である。

サーフェスは、次の分類とは独立している。

- 成果物種別
- 内部の実装責務
- 規範上の役割
- 実行環境
- デプロイ環境
- 認証方式
- 認可方式
- URLまたはディレクトリの物理配置

したがって、一つのサーフェス分類から、別の分類軸に属する制約を自動的に導出してはならない。

## 4. サーフェス一覧

| サーフェス | 主な利用者 | 主な目的 | 推奨Web名前空間 | 推奨CLI名前空間 |
|---|---|---|---|---|
| `public` | 未認証利用者、初回利用者、一般閲覧者 | 紹介、発見、導入、公開情報 | `/` | トップレベル |
| `app` | 通常利用者 | アプリケーション本来の利用 | `/app` | `app` |
| `admin` | 管理者、運用担当者 | 管理、監査、運用、修復 | `/admin` | `admin` |
| `dev` | 当該アプリケーションの開発者・保守者 | 内部開発、診断、実装文書 | `/dev` | `dev` |
| `api` | 外部アプリケーションの開発者、連携クライアント | 外部統合、契約、機械利用 | `/api` | `api` |
| `test` | 開発者、試験担当者 | 検証、観測、シナリオ実行 | `/test` | `test` |

これらの名前は、概念分類、共通語彙、推奨命名規則、推奨名前空間、CLIサブコマンド体系、文書構成およびテスト分類として再利用できる。

## 5. 共通原則

### 5.1 サーフェスは利用目的を表す

サーフェスは、内部実装レイヤーではなく、外部から見た利用者と目的を表す。

次の同一視は禁止する。

```text
app surface = application layer
dev surface = development environment
test surface = test environment
api surface = network transport layer
```

### 5.2 推奨名前空間と必須制約を区別する

本規約に示すWebパスおよびCLIサブコマンドは、原則として推奨名前空間である。

プロジェクトが別途、特定の名前空間を必須不変条件として採用しない限り、サーフェス分類だけを根拠にパスprefixまたはサブコマンドprefixを強制してはならない。

例えば、次は許容される。

```text
surface = app
path = /settings
```

```text
surface = app
path = /object/:id
```

```text
surface = public
path = /help
```

### 5.3 パス名はセキュリティ境界ではない

`/admin`、`/dev`、`/test`などの名前は目的を伝えるが、それ自体でアクセスを保護しない。

各操作は、必要な認証、認可、環境制約およびサーバー側検証を独立して実装しなければならない。

### 5.4 サーフェスとアクセス方針を分離する

サーフェスとアクセス方針は、別の属性として管理することを推奨する。

例:

```json
{
  "path": "/dev/routing",
  "surface": "dev",
  "access": "admin"
}
```

`dev`であることから、特定のrole名または認証実装を自動的に導出してはならない。

### 5.5 一つの実装が複数のサーフェスを支えてよい

一つのdomain service、application service、データモデルまたはインフラ実装が、複数のサーフェスから利用されてもよい。

サーフェスごとに内部実装を不必要に複製してはならない。

## 6. `public`サーフェス

### 6.1 目的

`public`は、アプリケーションを紹介し、公開情報を提供し、利用開始の入口を提供するサーフェスである。

### 6.2 典型的な内容

- ランディングページ
- 製品・サービス説明
- サインインおよびサインアップへの入口
- 公開ヘルプ
- 利用規約およびプライバシーポリシー
- ステータス情報
- 公開文書
- 認証済みサーフェスへのナビゲーション

### 6.3 推奨名前空間

Webではルート`/`を中心とする。

CLIでは、次のようなトップレベル操作が対応する。

```text
--help
version
login
init
discover
```

通常、文字どおりの`public`サブコマンドを設ける必要はない。

## 7. `app`サーフェス

### 7.1 目的

`app`は、通常利用者がアプリケーション本来の機能を使用するサーフェスである。

### 7.2 典型的な内容

- ダッシュボード
- データの作成、閲覧、更新
- 検索および一覧
- ワークフロー
- 通常利用者向け設定
- 通知
- ユーザー自身のプロファイル

### 7.3 推奨名前空間

Webでは`/app`を推奨する。

ただし、ユーザビリティまたはdomain vocabularyの観点から適切な場合、次のようなdomain-specific pathを使用してよい。

```text
/settings
/object/:id
/search
/projects/:projectId
```

CLIでは`app`名前空間を使用できるが、頻繁に使用される主要コマンドはトップレベルへ置いてもよい。

## 8. `admin`サーフェス

### 8.1 目的

`admin`は、特権的な管理、運用、監査、修復および制御された保守を行うサーフェスである。

### 8.2 典型的な内容

- ユーザーおよび権限管理
- システム設定
- 監査ログ
- 運用メトリクス
- 制御されたimportおよびexport
- データ修復
- ジョブ管理
- 障害対応
- legacy dataの制限付き閲覧

### 8.3 セキュリティ

管理操作は、サーバー側で明示的な認可を必須とする。

ナビゲーションから非表示にするだけでは不十分である。

### 8.4 推奨名前空間

Webでは`/admin`、CLIでは`admin`を推奨する。

## 9. `dev`サーフェス

### 9.1 目的

`dev`は、そのアプリケーション自体を開発・保守する人のためのサーフェスである。

`dev`は、外部アプリケーション開発者向けの`api`とは異なる。

```text
dev:
  このアプリケーションがどのように設計・実装されているか

api:
  他のアプリケーションがどのように接続・統合するか
```

### 9.2 典型的な内容

- アーキテクチャ文書
- 内部データモデル文書
- 設定確認
- 内部診断
- 開発者向けutility
- コード生成
- 実装固有のトラブルシューティング
- 内部デモ

### 9.3 推奨名前空間

Webでは`/dev`、CLIでは`dev`を推奨する。

### 9.4 本番環境での扱い

読み取り専用の開発文書を本番環境で公開することは、別途認可されていれば許容できる。

一方、任意データ挿入、内部状態の無制限表示、cache消去、job実行などの危険な機能は、本番buildから除外するか、サーバー側環境ポリシーで無効化することを推奨する。

## 10. `api`サーフェス

### 10.1 目的

`api`は、他のアプリケーション、外部開発者および統合クライアントが利用する契約サーフェスである。

### 10.2 典型的な内容

- API文書
- 認証要件
- requestおよびresponse schema
- OpenAPIなどの機械可読定義
- eventおよびwebhook契約
- エラー定義
- rate limit
- 互換性方針
- 外部統合向けplayground

### 10.3 推奨名前空間

Web上の文書rootとして`/api`を推奨する。

機械向けendpointは、原則としてmajor versionを含むversioned subpathを使用する。

```text
/api/v1/...
/api/v2/...
```

`/api`および文書subpathはHTMLを返してよい。一方、`/api/v1/...`以降の機械向けendpointは、通常JSONなどの構造化表現を返す。

### 10.4 バージョニング

後方互換な変更は、同じmajor version内で行ってよい。

外部契約を破壊する変更は、明示的な互換機構がない限り、新しいmajor-version namespaceを必要とする。

## 11. `test`サーフェス

### 11.1 目的

`test`は、開発中の挙動を制御、実行、観測および比較するためのサーフェスである。

中心概念は、一時的なテストページではなく、test harnessである。

### 11.2 典型的な内容

- vertical sliceの実行
- scenario runner
- fixture選択
- mock dependency選択
- fault injection
- clock制御
- identity制御
- event replay
- stateおよびtraceの観測
- 期待結果との比較

### 11.3 推奨名前空間

Webでは`/test`、CLIでは`test`を推奨する。

### 11.4 本番環境での扱い

次のような危険なtest機能は、本番環境から原則として排除する。

- 任意identityへのなりすまし
- 任意時刻へのclock変更
- 任意データ挿入
- fault injection
- fixtureの無制限読み込み
- 内部状態の無制限表示
- 任意job実行

必要な場合は、build-time exclusion、server-side environment policy、強い認可および監査を組み合わせる。

## 12. Webアプリケーションでの適用

推奨される概念構造は次のとおりである。

```text
/        public
/app     app
/admin   admin
/dev     dev
/api     api
/test    test
```

ただし、この対応は意味上の推奨であり、機械的に必須とは限らない。

### 12.1 Route catalog

Route catalogを持つ場合は、少なくとも次を別属性として保持することを推奨する。

```text
path
label
description
surface
access
active state
```

### 12.2 Validator

Validatorは、authoritativeなプロジェクト規約で明示された不変条件だけを検査する。

サーフェス分類から、次を勝手に推論してはならない。

- path-prefix requirement
- role taxonomy
- dynamic-route allowlist
- closed list of future routes
- runtime environment

## 13. CUIアプリケーションでの適用

推奨される概念構造は次のとおりである。

```text
tool
tool app
tool admin
tool dev
tool api
tool test
```

ただし、頻繁に使われる通常操作は、トップレベルへ置いてよい。

例:

```text
tool list
tool get
tool admin users
tool dev inspect-config
tool api schema
tool test run-scenario
```

WebとCLIの名前空間は、文字列として完全一致する必要はない。意味上の対応を維持することを推奨する。

## 14. 他の分類軸との関係

### 14.1 成果物種別

同じサーフェスに、複数の成果物種別が存在できる。

```text
api spec
api docs
api code
api tests
```

成果物種別からサーフェスを決定してはならず、サーフェスから成果物種別を決定してもならない。

### 14.2 実装責務

同じサーフェスは、UI、boundary、application、domain、infrastructureなど複数の実装責務によって実現される。

サーフェス名を内部レイヤー名として直接使用する必要はない。

### 14.3 規範上の役割

サーフェス規約は、規範上の役割が`authoritative`である文書またはcontractから導出する。

実装およびvalidatorは、本規約またはプロジェクト固有のauthoritative sourceに従う。

### 14.4 実行環境

次の組合せはいずれも成立し得る。

```text
app surface in test environment
admin surface in staging environment
dev surface in production environment
test surface in development environment
```

適切性は、サーフェス分類だけでなく、セキュリティおよび環境ポリシーによって判断する。

## 15. 適合例

### 15.1 Domain-specific app path

```json
{
  "path": "/object/:id",
  "surface": "app",
  "access": "authenticated"
}
```

`/app`prefixを持たないが、通常利用者の主要機能であるため適合する。

### 15.2 Restricted developer documentation

```json
{
  "path": "/dev/data-model",
  "surface": "dev",
  "access": "admin"
}
```

サーフェスとアクセス方針を別属性として表現している。

### 15.3 Versioned machine API

```text
/api
  API documentation

/api/v1/objects
  machine-facing endpoint
```

文書rootと機械endpointを区別している。

## 16. 不適合例

### 16.1 Prefixの無根拠な強制

```text
surface = app
    ↓
path must start with /app
```

プロジェクト固有規約に根拠がない限り不適合である。

### 16.2 パス名だけによる認可

```text
/adminだから安全
```

サーバー側認可がなければ不適合である。

### 16.3 `dev`と`api`の混同

外部統合契約を`dev`配下だけに配置し、外部利用者向けの安定性やversioningを定義しない構成は不適合となり得る。

### 16.4 `test`と実行環境の混同

```text
development environmentだから危険なtest operationを無認可で許可する
```

環境名だけを根拠に認可を省略してはならない。

## 17. 採用および移行

本規約は、既存プロジェクト全体を直ちにrenameすることを要求しない。

採用時は、次の順序を推奨する。

1. 共通語彙として六つのサーフェスを採用する。
2. 新しいインターフェースで推奨名前空間を使用する。
3. 既存インターフェースを大きく変更する際に段階的に整合させる。
4. URLまたはCLIを変更する前に、bookmark、外部参照、自動化、認証、互換性およびredirectを評価する。
5. 重大な例外は、プロジェクト固有文書に記録する。

外見上の統一だけを目的とした大規模renameは推奨しない。

## 18. プロジェクト固有の厳格化

プロジェクトは、本規約より厳しい制約を追加してよい。

例えば、次を必須として採用できる。

```text
all admin routes must use /admin
all external machine APIs must use /api/v{major}
all test harness routes must be absent from production builds
```

ただし、厳格化はauthoritativeなプロジェクト文書に明記しなければならない。

Validatorまたはテストだけに厳格化を埋め込んではならない。

## 19. 中核要件

本規約の中核要件は次のとおりである。

1. サーフェスは、誰が何の目的で接触するかを表す。
2. `public`、`app`、`admin`、`dev`、`api`、`test`を共通語彙として使用する。
3. サーフェスを、実装責務、実行環境、アクセス方針または成果物種別と混同しない。
4. 推奨名前空間と必須不変条件を区別する。
5. パス名またはサブコマンド名をセキュリティ境界として扱わない。
6. サーフェス分類から、文書化されていない別軸の制約を推論しない。
7. プロジェクト固有の厳格化は、authoritativeな文書に明記する。

## 20. 変更履歴

### 0.1.0

- 初版。
- 六つのアプリケーション・インターフェース・サーフェスを定義。
- Web、CLI、API、セキュリティ、環境および他の分類軸との関係を整理。
- 推奨名前空間から必須制約を無根拠に推論することを禁止。
