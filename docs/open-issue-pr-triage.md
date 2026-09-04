# go-github Open Issue / PR 調査レポート

> 対象: **google/go-github**（フォーク: `JamBalaya56562/go-github`）／ 調査日: **2026-06-20**（最終更新: **2026-10-10**）
>
> 本レポートは Open な Issue 24件（クローズ済みは掲載対象外）・OPEN な PR 13件を1件ずつ調査し（マージ/クローズ済みは掲載対象外）、本文・全コメントの確認に加えて
> `github/` 配下の実コードを実地確認した上で、根本原因・解決方針・難易度・対応区分をまとめたものです。
> 各 Issue/PR の見出しから GitHub の原文へリンクしています。再調査の手間を省くためのリファレンスです。
>
> **最終更新時点の結果（2026-10-10）**
> - 当フォーク提出の Open PR: **なし**（#4607 が 10-09 にマージされ、#3644 の「同時に1PRのみ」枠が空いた）。
> - 前回更新（2026-08-29）以降に当フォークがマージさせた PR: #4493（`PullRequestComment` 分割）・#4519（`CreateDependencyGraphSnapshotRequest`）・#4540（codespaces 4型）・#4548（secret scanning 2型）・#4606（Teams 3型）・#4607（`CreateForkRequest`/`AddCollaboratorRequest`）（いずれも→#3644）。#4375（metadata `check-schema-fields`）は gmlewis により 09-18 クローズ。
> - 環境の大きな変化: **gmlewis の #4576 `tools/schemafields`（09-20）が lint/CI に組み込まれ、`//meta:schema` 注釈は不要に**（`//meta:operation` と値渡しの `body` 引数から request 型を自動特定）。続く **#4581（09-23, +2874/-542）で値渡し済み request 型の required フィールドが一括で非ポインタ化**、#4592（10-02）で response 型は検査対象外に。認証まわりは **#4564（09-18）で #4363/#4364 を置き換え**て configured origin 以外へ資格情報を送らない設計がマージされたが、stevehipwell の異議で #4365 は再オープン。モジュールは **v92**。
> - 次の着手候補: **#3644 の Projects v2 item 2型（`AddProjectItemOptions`/`UpdateProjectItemOptions`）**。詳細は 2.1 と #3644 の【2026-10-10 更新・残候補】。他コントリビューターの #3644 Open PR は無し（jvm986 の #4551 は 09-16 マージ済み）。

## 1. サマリー

### 1.1 分類別（OPEN Issue 24件）

| 分類 | 件数 |
| --- | ---: |
| メタ/プロセス | 11 |
| 新API対応 | 5 |
| バグ | 2 |
| セキュリティ | 1 |
| CI/ツール | 1 |
| 機能改善 | 1 |
| 質問・サポート | 1 |
| リファクタ | 1 |
| 提案 | 1 |

### 1.2 対応区分別（OPEN Issue 24件）

| 対応区分 | 件数 |
| --- | ---: |
| 🤔 メンテナ判断待ち | 13 |
| 🔀 対応PRあり | 7 |
| ✅ コードで対応可能 | 2 |
| 🚫 対応見送り濃厚 | 2 |

### 1.3 難易度別（OPEN Issue 24件）

| 難易度 | 件数 |
| --- | ---: |
| 難 | 6 |
| 中 | 4 |
| 易 | 7 |
| 些細 | 4 |
| 不明 | 3 |

## 2. 着手の優先順位（おすすめ）

### 2.1 🎯 今すぐ着手を推奨（clean な code-fixable・重複なし）

いずれもメンテナの方向性が固まっており、修正範囲が限定的で、現時点で他者の Open PR や
アサインと競合しないもの。go-github へのコントリビューションとして手戻りが少ない。

> **第一候補: #3644 の値渡しリファクタ follow-up — Projects v2 item 2型。** `AddProjectItemOptions`→`AddProjectItemRequest`／`UpdateProjectItemOptions`→`UpdateProjectItemRequest`（`github/projects.go`・`AddOrganizationProjectItem`/`AddUserProjectItem`/`UpdateOrganizationProjectItem`/`UpdateUserProjectItem` の4メソッド）。OpenAPI 照合で (1) add の `type` が required なのにポインタ、(2) add の oneOf 第2分岐 `owner`/`repo`/`number` が欠落、(3) update の `fields` が required なのに `omitempty`、(4) update の `Archived` がスキーマに無い、の4点が判明（検証済み: `api.github.com.json` を jq で照合）。動詞は docs と一致済みで rename 不要。許可リスト2型（`body-allowed-pointer-types`/`body-allowed-wrong-names` 各2行）を削除できる。`Archived` の扱い（削除するか残すか）は PR 説明で判断根拠を示すかメンテナに確認する。
>
> **第二候補（待機）: #4584 redirect 系リクエストで `http.Client.Timeout` が効かない。** #4585 が gmlewis 指定の修正方式に対応済みだが CLA で停止中。放棄された場合のみ Issue で申し出て引き取る。
>
> **分担**: jvm986 は #4551（`CreateOrgInvitationRequest`, 09-16 マージ）以降 #3644 の Open PR なし。同時に1PRのみ・着手前に相手の Open PR を確認する運用は継続。

### 2.2 🔀 対応 PR が既に存在（重複回避・レビュー対象）

Issue に対応する Open PR が既にある。新規実装ではなく、PR のレビュー／追跡が次の一手。

| Issue | 関連 PR | PR の状況 |
| --- | --- | --- |
| [#4077](https://github.com/google/go-github/issues/4077) | #4550 | [#4550](https://github.com/google/go-github/pull/4550): `Attestation.BundleURL`（チェックリストの1項目）。コミット作者の identity 混在で CLA 失敗・停滞 |
| [#4408](https://github.com/google/go-github/issues/4408) | #4565 | [#4565](https://github.com/google/go-github/pull/4565): Draft・企業 CCLA で停滞。**既存 `GetBudget` と完全に重複**（未指摘） |
| [#4457](https://github.com/google/go-github/issues/4457) | #4456 | [#4456](https://github.com/google/go-github/pull/4456): CLA 待ちで 08-28 以降停滞 |
| [#4520](https://github.com/google/go-github/issues/4520) | #4523 | [#4523](https://github.com/google/go-github/pull/4523): 非アサイン者の PR・CONFLICTING・gmlewis はアサイン者を待つ方針 |
| [#4524](https://github.com/google/go-github/issues/4524) | #4525 | [#4525](https://github.com/google/go-github/pull/4525): gmlewis APPROVED・2人目待ち・CI 緑 |
| [#4526](https://github.com/google/go-github/issues/4526) | #4527 | [#4527](https://github.com/google/go-github/pull/4527): alexandear CHANGES_REQUESTED に約1か月未対応 |
| [#4584](https://github.com/google/go-github/issues/4584) | #4585 | [#4585](https://github.com/google/go-github/pull/4585): 指摘対応済みだが CLA 失敗（作者メール2種）で停止 |

### 2.3 🤔 メンテナの方針判断が必要

方向性・命名・破壊的変更の可否などメンテナの合意が前提で、独力で完結しにくいもの。

| Issue | 分類 | 難易度 | タイトル |
| --- | --- | --- | --- |
| [#98](https://github.com/google/go-github/issues/98) | メタ/プロセス | 不明 | Add integration tests |
| [#310](https://github.com/google/go-github/issues/310) | CI/ツール | 中 | Automatically run integration tests |
| [#999](https://github.com/google/go-github/issues/999) | 機能改善 | 難 | Proposal: Make preview features configurable |
| [#3037](https://github.com/google/go-github/issues/3037) | メタ/プロセス | 些細 | FYI: New official alpha Go SDK released based on K |
| [#3067](https://github.com/google/go-github/issues/3067) | メタ/プロセス | 難 | Proposal/Question: Release frequency |
| [#3366](https://github.com/google/go-github/issues/3366) | 質問・サポート | 些細 | How to ensure that a repository has been synced to |
| [#3644](https://github.com/google/go-github/issues/3644) | リファクタ | 中 | Refactor codebase to use value parameters instead  |
| [#3658](https://github.com/google/go-github/issues/3658) | メタ/プロセス | 些細 | Feature Request: configurable telemetry |
| [#3761](https://github.com/google/go-github/issues/3761) | メタ/プロセス | 難 | Consistent naming for enterprise & organization sc |
| [#3894](https://github.com/google/go-github/issues/3894) | メタ/プロセス | 些細 | Four billings endpoints have suddenly disappeared  |
| [#4237](https://github.com/google/go-github/issues/4237) | 提案 | 難 | Move compatibility for non github.com targets to github.With<THING> options |
| [#4334](https://github.com/google/go-github/issues/4334) | メタ/プロセス | 不明 | Metadata question |
| [#4365](https://github.com/google/go-github/issues/4365) | セキュリティ | 中 | Auth transports add credentials after cross-origin redirects |

## 3. Issue 詳細（OPEN 24件・番号順）

### Issue [#98](https://github.com/google/go-github/issues/98) — Add integration tests

分類: **メタ/プロセス** / 難易度: **不明** / 対応区分: **🤔 メンテナ判断待ち**

**概要**: 2014年に作成された統合テスト整備のための包括的トラッキングIssue。ライブのGitHub APIに対して実行する統合テスト群をライブラリ全体に追加していくことが目的。コメント欄では、ライブラリ側の破壊的なAPI変更後に統合テストが追従できずビルドが壊れていた(branch.Protection / EditBranch / RequiredStatusChecks.EnforcementLevel が存在しない等)という派生問題が報告され、メンテナ(dmitshur)が別Issue化を依頼した。

**根本原因**: 根本原因は「統合テストが手動実行のみで、CIで自動チェックされていなかったため、ライブラリ本体のAPI変更にテストコードが追従せずビルドが破綻した」こと。報告当時(2017)のtests/integration/repos_test.go は古いAPI(branch.Protection, client.Repositories.EditBranch, RequiredStatusChecks.EnforcementLevel)を参照していたためコンパイルできなかった。現在のリポジトリを確認したところ、この破綻は既に解消済み。test/integration/repos_test.go(現在のパスは test/ 配下に移動)は v88 モジュールパス・github.ProtectionRequest/github.Protection・client.Repositories.UpdateBranchProtection など現行APIに更新済みで、mise exec go -- go test -tags=integration -run=^$ ./test/integration は "ok ... [no tests to run]" で正常にビルドが通った。さらに .github/workflows/tests.yml の61-63行に「Ensure integration tests build」ステップ(go test -v -tags=integration -run=^$ ./test/integration)が追加され、push/PR毎にビルド健全性がCIで保証されるようになっている。

**解決方針**: このIssueはコード上の具体的バグではなく「統合テストを拡充し続ける」という終わりのない作業項目(ongoing work item)である。コメントで報告された具体的なビルド破綻(派生問題)は既に修正済みで、dmitshur が依頼した別Issue化はもはや不要。残る価値があるのは(1)未カバーのサービス(現状 activity, audit_log, authorizations, issues, licences, misc, pagination, projects, pulls, repos, users の約13ファイル/25テスト関数のみ)に統合テストを追加していくこと、(2)関連するOpen Issue #310「Automatically run integration tests」(ライブAPIに対して実際に走らせる自動化)で議論を集約すること。実コード変更を伴う新規貢献としては、特定サービスファイル(例: test/integration/ に新規 *_test.go を //go:build integration タグ付きで追加)に対する統合テスト追加が現実的な着手点となる。

**関連ファイル**: `test/integration/repos_test.go`, `test/integration/github_test.go`, `test/integration/doc.go`, `test/README.md`, `.github/workflows/tests.yml`

**現状**: 最新コメントは2021-08-06のgmlewisによるもので、関連Issue #310 へのポインタのみ。実質的な議論は2017年で停止している。当時dmitshurは「統合テストは手動実行のため破壊的API変更後に陳腐化した。新Issueを立ててPRで直してよい」と回答(派生のビルド破綻に対する見解)。2026年現在、その破綻は解消済みで、CIにビルド健全性チェックも追加されている。本Issue自体はオープンのままだが、トラッキングIssueとして放置されている状態。関連: PR #506(2017マージ済み、READMEのリンク修正のみ)、Issue #310「Automatically run integration tests」(オープン)。

**推奨アクション**: このユーザー(新規コントリビューター)が「Issue #98 全体を解決する」ために着手するのは推奨しない。これは2014年からのオープンエンドなトラッキングIssueであり、単一PRでクローズできる性質ではない。コメントで報告された具体的なビルド破綻は既に解消済み(現在ビルドは通り、CIの「Ensure integration tests build」ステップで保護されている)なので、その派生問題に取り組む余地もない。メンテナにはこのIssueをクローズし #310 に集約するか、「特定サービスの統合テスト追加」という粒度のIssueに分解することを推奨。コントリビューターとして手を動かしたい場合は、test/integration/ に未カバーのサービス向け統合テストを1ファイル単位で追加するのが現実的(ただしライブAPIトークンが必要で、メンテナとの事前合意が望ましい)。

**関連 PR/リンク**: https://github.com/google/go-github/issues/310 (関連 Open Issue)

---

### Issue [#310](https://github.com/google/go-github/issues/310) — Automatically run integration tests

分類: **CI/ツール** / 難易度: **中** / 対応区分: **🤔 メンテナ判断待ち**

**概要**: go-github の test/integration 配下の統合テストは、ライブの GitHub API を叩き認証トークンを要するため通常の CI では実行されていない。このIssueは、それらを定期的に(当初案ではGCEインスタンス、後にGitHub Actionsのスケジュール実行で)自動実行し、恒常的な失敗を検知・報告できるようにしたいという要望。fields ツールの定期実行も併せて検討対象として挙げられている。

**根本原因**: 技術的バグではなくCI運用上の課題。統合テストはライブAPIに対し実データを変更し、未認証レート上限(60req/h)を即座に使い切るため(test/README.md 23-28行・56-58行で明記)、専用テストアカウントと GITHUB_AUTH_TOKEN 等のシークレットが必須。現状 .github/workflows/tests.yml の61-63行は "Ensure integration tests build" として go test -v -tags=integration -run=^$ ./test/integration を実行しているが、-run=^$ で全テストにマッチさせず「ビルド確認のみ・実行はしない」(62行コメント "don't actually run tests since they hit live GitHub API")。.github/ 内に統合テストを走らせる schedule/cron ワークフローは存在しない(schedule: の一致は dependabot.yml のみ)。よって本Issueの本質的要望(ライブAPIに対する定期自動実行と報告)は未実装のまま。

**解決方針**: 専用のGitHubテストアカウント(またはbot)を用意し、そのPAT(またはGitHub App)をリポジトリシークレットに登録した上で、.github/workflows/ に新規のスケジュール実行ワークフロー(on: schedule: cron、例: 日次)を追加する。ステップで GITHUB_AUTH_TOKEN: ${{ secrets.INTEGRATION_TOKEN }} を渡し go test -v -tags=integration ./test/integration を実行。データ変更系テストの副作用を許容できる隔離アカウントを使い、レート上限と既知のフレーキーさ(README 19-21行)を踏まえ失敗時はIssue自動起票やSlack/メール通知で報告する。fields ツールは本文どおり「完全クリーンにはならない」性質上、別途ヒント用途として情報提供のみに留める設計が無難。実装自体はワークフローYAML追加が主だが、シークレット/専用アカウントの用意というメンテナ側の体制判断が前提。

**関連ファイル**: `.github/workflows/tests.yml`, `test/README.md`, `test/integration`, `.github/dependabot.yml`

**現状**: 最新コメント(2021-08-06)時点で、gmlewisが「企業の専任投資・支援なしには実現困難」として一度クローズしたが、willnorrisが「Issue起票当時はGitHub Actions/リポジトリシークレットが存在しなかった。現在はActionsのスケジュール実行で比較的容易に実装できる。実装を見てみるかもしれない」として再オープン。gmlewisが関連Issueとして #98 を挙げて謝意を示した。以降2021年から進展なし。ブロッカーは専用テストアカウント・シークレットの用意とデータ変更系テストの安全な隔離。

**推奨アクション**: 外部コントリビューターが単独で完結させるのは難しい。ワークフローYAMLの追加自体は容易だが、専用テストアカウントの作成とリポジトリシークレット登録、失敗報告の運用方針はメンテナ(google org)の判断・権限が必須なため、まずIssue上でメンテナにシークレット用意と専用アカウントの可否を確認するのが次の一手。willnorrisが「実装してみるかも」と表明したまま2021年以降停滞しているので、進捗確認のコメントで掘り起こすのが現実的。着手するなら関連の #98 "Add integration tests" と合わせて整理するとよい。

---

### Issue [#999](https://github.com/google/go-github/issues/999) — Proposal: Make preview features configurable

分類: **機能改善** / 難易度: **難** / 対応区分: **🤔 メンテナ判断待ち**

**概要**: go-github はライブラリ全体にわたり、多くのエンドポイントで GitHub の preview API を有効化する Accept ヘッダ (例: mediaTypeCodesOfConductPreview) をデフォルトで無条件に付与している。報告者 (gauntface) は、リポジトリ一覧取得時にこの codes-of-conduct preview ヘッダが原因で GitHub が Server Error を返す事象に遭遇し、ライブラリ側でこのヘッダを除去する手段が無いことを問題提起した。提案は、preview ヘッダをデフォルト付与ではなく利用者が制御できるようにすること(RequestOptions 案、グローバルスイッチ案など)。

**根本原因**: 各サービスメソッドが req.Header.Set("Accept", mediaType...Preview) を直接呼び、preview ヘッダを無条件付与している点が根本原因。github/codesofconduct.go:39 と :69 で List/Get が mediaTypeCodesOfConductPreview を必ずセットしており、利用者が抑止する手段がない。preview media type 定数は github/github.go:84-159 付近に約26種定義され、現行 v88 でも31ファイルが preview を参照、repos.go だけでも複数箇所 (例 repos.go:461,610,669-672,699) で付与している。当時はこれを上書きする公開 API が存在しなかった。なお github/github.go:66-81 のコメント(PR #2125/#2188 由来)が、preview ヘッダは主に旧 GitHub Enterprise Server 互換のため残しており「GitHub Cloud では非機能 preview ヘッダは副作用を生まない」というメンテナの現行見解を示している。

**解決方針**: スレッド最終盤 (2021) で gmlewis・willnorris・gauntface が合意した方向は context.WithValue を用いた preview 有効/無効の切替であり、github.EnablePreviewFeatures(ctx, ...) / github.DisablePreviewFeatures(ctx, ...) の形が提示され全員が好意的(👍)。重要なのは、この context ベースのリクエスト設定パターンが現在の go-github には既に実装済みである点。github/github.go:1151-1160 に requestContext 型と BypassRateLimitCheck / SleepUntilPrimaryRateLimitResetWhenRateLimited という context キーが定義され、bareDo/Do (github.go:1206, 1281, 1418) で ctx.Value() を参照して挙動を分岐させている。これは #999 で提案された手法そのもの。具体的実装案: (1) requestContext に preview 有効/無効を表すキーを追加、または preview 種別の集合を持つ context ヘルパー (EnablePreviewFeatures/DisablePreviewFeatures) を新設する。(2) 各サービスメソッドの req.Header.Set("Accept", mediaType...Preview) を、ctx の指定を確認してから付与する内部ヘルパー (例 setPreviewHeader(ctx, req, preview)) 経由に置換する。(3) デフォルトは現行どおり全 preview 有効を維持し後方互換を保つ。ただし対象エンドポイントが数十に及ぶため機械的な一括置換と網羅的テスト更新が必要。実装前に、現在も「副作用は無い」とするメンテナ見解(github.go:66-81)が残る中で本対応を進めるか、メンテナの方針確認が必要。

**関連ファイル**: `github/github.go`, `github/codesofconduct.go`, `github/repos.go`

**現状**: 最終コメントは2021-10-29。gmlewis が context.WithValue 方式を提案 → willnorris が EnablePreviewFeatures/DisablePreviewFeatures の利用イメージを提示 → gauntface が「この案が好き」と賛同(👍2)で合意に至るも、PR 提出には至らず4年以上停滞。当初の代替案(RequestOptions に AcceptHeaders フィールド追加、グローバルスイッチ、利用者側で client.Do を使った独自実装)は出尽くしている。ブロッカーは PR の実装者不在と、API 側修正により問題が解消している可能性の未検証。関連 #1037 (curl repro) はクローズ済み。label は NeedsInvestigation のまま。

**推奨アクション**: 着手は推奨するが、まずメンテナ(gmlewis)への方針確認から始めるべき。理由: (1) スレッドは2021年で停止しており、その後 github.go:66-81 のコメントで「preview ヘッダは主に旧 GHES 互換のため残置、Cloud では副作用なし」というスタンスが明文化されたため、報告された Server Error が現在も再現するか不明(linked #1037 はクローズ済み、API 側で修正された可能性)。(2) 一方で、合意済みの context.WithValue 方式は BypassRateLimitCheck として既に前例があり、設計的ハードルは低い。次の一手: 現在 codes-of-conduct preview などで実害が出るか curl で再現確認し、再現するなら EnablePreviewFeatures/DisablePreviewFeatures を提案・PR 化、再現しないなら本 issue は wontfix/クローズ候補としてメンテナに確認する。コード量は大きく初心者向けではない。

---

### Issue [#3005](https://github.com/google/go-github/issues/3005) — Add first party support for Github App

分類: **メタ/プロセス** / 難易度: **不明** / 対応区分: **🚫 対応見送り濃厚**

**概要**: GitHub App を使う際、JWT 生成・installation token の取得/自動リフレッシュ・複数クレデンシャル(App本体/installation/user)の使い分けを、ghinstallation 等のサードパーティに頼らず go-github 本体でサポートしてほしいという要望。提案者は (1) JWT生成と token自動リフレッシュのコードを本体に取り込む、(2) クレデンシャルを切り替えやすくする API 変更、の2点を望んでいる。短命トークンの使い分けで複数の github.Client を作る必要があり、ハンドリングが煩雑である点が動機。

**根本原因**: 該当なし(コードのバグではなく方針・設計の議論)。技術的背景としては、go-github 本体には認証ヘルパーを持たない設計方針があり、App認証は外部に委ねている。README.md の128-215行に「As a GitHub App」節があり、JWT認証/installation認証の使い分けと、ghinstallation・go-githubauth の利用例が記載されている。実コードでは example/newfilewithappauth/main.go が ghinstallation.NewAppsTransport で JWT 認証→Apps.CreateInstallationToken→トークンで再クライアント生成、という現行の2クライアント方式を示している。Apps.CreateInstallationToken は github/apps_installation.go 等に実装済みでエンドポイント自体は存在する。要望の核心は「本体にJWT/oauth2依存を持ち込むこと」であり、これは依存最小化方針と衝突する。

**解決方針**: 提案 (1)(JWT・token自動リフレッシュの本体取り込み)はメンテナ gmlewis が明確に却下しており、外部ヘルパーで実装し README から参照する方針が確立済み。実際、姉妹issue #3178 から外部ヘルパー jferrl/go-githubauth が生まれ、README更新PR #3180 がマージされ、ghinstallation と go-githubauth の両方が README に記載された。よって (1) に対する go-github 側の対応は「READMEへの導線整備」で完了している。残るのは提案 (2)(クレデンシャル切り替えを容易にする API 変更)だが、WillAbides が ashi009 に具体的なAPI変更案を求めたものの提示されず、alvaroaleman が org ごとにラウンドトリッパで透過認証する Prow の実装(app_auth_roundtripper.go)を参考案として挙げたにとどまる。対応としては、本Issueを「外部ヘルパーで解決済み」としてクローズするか、(2)を独立した設計提案Issueとして切り出すのが妥当。本体にoauth2/JWT依存を追加する方向は方針上採用されない。

**関連ファイル**: `README.md`, `example/newfilewithappauth/main.go`, `github/apps.go`, `github/apps_installation.go`

**現状**: 2024-09-01最終更新でOPENのまま。メンテナ gmlewis は「依存最小化のため本体にJWT操作コードは入れたくない、ghinstallation もこの議論から生まれた外部ヘルパー、必要機能は外部リポで実装しREADMEから参照したい」と明言。WillAbides は方針に同意しつつ「クレデンシャル切替を楽にするAPI案」に興味を示し ashi009 に具体案を求めたが回答なし。jferrl は同種issue #3178 から外部ヘルパー go-githubauth を作成し、README更新(両ヘルパー併記)を実施。alvaroaleman は Prow の透過認証ラウンドトリッパを参考案として提示。実質的に (1) は外部ヘルパー方式で決着済み、(2) は具体提案待ちのまま停滞しているのが現状。

**推奨アクション**: このコントリビューターが本体実装に着手するのは非推奨。提案(1)(JWT/oauth2を本体へ)はメンテナ方針で却下済みで、PRを出しても受理されない可能性が高い。現実的な次の一手は、(a) 本Issueにコメントし「外部ヘルパー(ghinstallation / go-githubauth)+ README導線で (1) は対応済み」と整理してメンテナにクローズ可否を確認する、または (b) 提案(2)「クレデンシャル切り替えを容易にする API」だけを、具体的なインターフェース案(例: Prow の app_auth_roundtripper のような org 推論型 RoundTripper、あるいは TokenSource 差し替えAPI)を添えた新規設計提案Issueとして切り出すこと。コード変更を伴う着手より、まず方向性の合意形成が先。

---

### Issue [#3037](https://github.com/google/go-github/issues/3037) — FYI: New official alpha Go SDK released based on Kiota

分類: **メタ/プロセス** / 難易度: **些細** / 対応区分: **🤔 メンテナ判断待ち**

**概要**: GitHubが公式にKiotaで自動生成したalpha版のGo SDK (octokit/go-sdk) をリリースしたことを、go-githubのメンテナへ周知 (FYI) するためのIssue。本文はGitHub公式ブログ記事 (https://github.blog/2024-01-03-our-move-to-generated-sdks/) と新SDKリポジトリ (https://github.com/octokit/go-sdk) へのリンクの2行のみ。具体的なバグ報告・機能要望・API追加要望ではなく、go-githubの将来 (公式SDKとの関係性・棲み分け) に関する情報提供および議論のきっかけを目的としている。

**根本原因**: 該当なし。コード上の不具合や欠落ではなく、外部 (GitHub公式) の動向に関する周知が目的のため、根本原因と呼べる技術的問題は存在しない。go-github側のコード (github/ 配下) に修正を要する主張は本文・コメントに一切含まれていない (コメントは0件)。

**解決方針**: コード変更は不要。対応方針としては、(1) メンテナがコメントで見解 (例: 「go-githubは引き続き手書き/半自動生成のクライアントとして維持する」「公式alpha SDKはまだ実用段階ではない」など) を述べてクローズする、(2) 情報提供への謝意を示しつつ「メンテナ間で認識済み」としてクローズする、のいずれか。必要であればREADMEでGraphQL向けshurcooL/githubv4を案内しているのと同様に、公式生成SDKへの言及を1行追加する選択肢もあるが、alpha段階であり推奨にはあたらないため現時点では不要と判断する。コントリビューター (ユーザー) 側で着手すべきコードタスクは存在しない。

**関連ファイル**: `README.md`

**現状**: 2024-01-04の作成以降、コメントは0件・ラベルなし・stateはOPENのまま (updatedAtも作成日と同一) で、メンテナからの公式見解は記録されていない。関連PRも存在しない (検索 "3037"・"kiota"・"octokit SDK" でこのIssue自身しかヒットせず)。実質的に放置されているFYI Issueであり、メンテナの方針表明待ちの状態。

**推奨アクション**: このユーザー (コントリビューター) が着手すべきコードタスクは無いため、実装目的での着手は非推奨。次の一手はメンテナによる判断: go-githubと公式生成SDKの棲み分けについて一言コメントを残してクローズするのが妥当。もしコントリビューターとして関与するなら、READMEへの公式SDK言及追加 (alphaのため任意) を提案するくらいだが、メンテナの方針確認が先決。情報提供Issueとしての役目は果たしており、クローズ候補。

---

### Issue [#3067](https://github.com/google/go-github/issues/3067) — Proposal/Question: Release frequency

分類: **メタ/プロセス** / 難易度: **難** / 対応区分: **🤔 メンテナ判断待ち**

**概要**: コントリビュータ(chapurlatn)が、PRのマージは速いがリリース頻度が遅く、変更が必要な利用者は当面スナップショット(疑似コミット参照)に頼らざるを得ないと指摘。各マージごとの自動リリース、あるいはPR種別(fix/feature/breaking)に応じたpatch/minor/majorの自動採番を提案している。後続コメントでは himazawa らが release-please + Conventional Commits + SemVer の採用を支持している。コード変更を伴わない、リリース方針・運用プロセスの議論。

**根本原因**: 該当なし(機能バグではなくプロセス論)。技術的背景として、リポジトリは現在も手動リリース運用である点を確認した。CONTRIBUTING.md の342行目に「リリース作成時は github.go の Version 定数を手動更新せよ」と明記されており、.github/workflows/ には linter.yml と tests.yml のみでリリース自動化ワークフローは存在しない。release-please-config.json や .release-please-manifest.json も存在しない。一方、Conventional Commits 風のPRタイトル接頭辞(feat:/fix:/feat!: 等)は CONTRIBUTING.md 108-111行目で既に導入済みで、最近のマージ済みPRタイトルでも実際に使われている。ただし go-github は GitHub API のフィールド追加も破壊的変更と扱うため毎回メジャーを上げており(v81→v88、2026年1月〜5月でほぼ月次)、release-please の SemVer 自動採番(minor/patch)とは構造的に相性が良くない。

**解決方針**: 該当なし

**関連ファイル**: `CONTRIBUTING.md`, `github/github.go`, `.github/workflows/tests.yml`, `.github/workflows/linter.yml`

**現状**: 最終コメント(2024-02-16, himazawa)時点で、提案者・himazawa はSemVer+Conventional Commits+release-please を支持。メンテナ gmlewis は「release-please に2票入った。ただスター10k・フォーク2.2kの規模なので、この非自明な大作業に着手する前にもう少し合意を待ちたい」と表明し保留状態。以後コメントは付いておらず、Issueはオープンのまま停滞。関連PRは存在しない(release-please導入PRは作られていない)。実リポジトリ状態でも自動化は未導入で、CONTRIBUTING.md 342行目の手動Version更新運用が継続中。リリース頻度は2026年でほぼ月次(v81→v88)。

**推奨アクション**: このコントリビュータ(調査担当者)は着手すべきでない。本件はコード課題ではなくメンテナの方針決定待ちであり、gmlewis 自身が「合意がもっと集まるまで待つ」「release-please 導入は非自明な大作業」と明言している。現状(2026年)を見ると、提案の一部であるConventional Commits風のPRタイトル接頭辞は既にCONTRIBUTING.mdに取り込まれており部分的に前進している一方、release-please自体は未導入で手動リリースが続いている。次の一手としては、(a) 利用者は急ぎなら疑似バージョンで該当コミットを直接 go get できる旨をIssueに案内する、(b) 毎回メジャー版を上げる本repoの慣行とSemVer自動採番の不整合を論点として明確化し、メンテナに現状維持かツール導入かの判断を仰ぐ、のいずれか。新規コントリビューションとして自発的に実装するには合意とメンテナの了承が前提。

---

### Issue [#3127](https://github.com/google/go-github/issues/3127) — Use enums for the action field in GitHub Webhooks

分類: **メタ/プロセス** / 難易度: **難** / 対応区分: **🚫 対応見送り濃厚**

**概要**: 多くのGitHub Webhookペイロードが持つ action フィールドは列挙的な値(例: check_run なら "created", "completed", "requested_action", "rerequested")を取るが、go-github では現在すべて free-form の *string として表現されている。報告者は、これらを型付きの enum(string 派生型 + const 定数)にすれば値の候補が自己文書化され、ユーザーにとってより使いやすくなると提案している。

**根本原因**: 設計上の選択であり技術的バグではない。github/event_types.go では各イベント構造体の Action が *string として定義されている(`Action *string json:"action,omitempty"` が同ファイル内に18箇所)。例えば CheckRunEvent(event_types.go:49-62)では line 51-52 のコメントで "created"/"completed"/"rerequested"/"requested_action" という候補を明記しているものの、型としては単なる *string であり、コンパイル時の値制約や補完支援は無い。よって主張(action が enum 化されていない)は事実。

**解決方針**: 技術的には、各イベントの action 専用の string 派生型(例: type CheckRunAction string)と候補値の const 群を定義し、構造体フィールドの型を *string から *CheckRunAction 等へ置き換える。Issue 本文の Go Playground 例の通り encoding/json は string 派生型へ問題なく unmarshal できるため実装自体は容易。ただしこれは全 webhook イベントにまたがる大規模な破壊的変更(*string を返す既存利用コードがすべてコンパイルエラーになる)。実際、対応 PR #3136 は +690/-1105 行・4ファイルの大改修だった。結論として、メンテナ判断によりこの方針は不採用となっており、新規 PR を出しても受理される見込みは無い。対応するなら方針転換の再交渉(または完全な後方互換を保つ別アプローチの提示)が前提。

**関連ファイル**: `github/event_types.go`

**現状**: 対応 PR #3136 (prnvbn 作成) は 2025-01-22 に未マージのままクローズされた。経緯: 当初 gmlewis は「安全そう」と賛同したが、PR の規模(全イベントにわたる破壊的変更)を見て willnorris の意見を求めた。2025-01-22、willnorris が「個人的には反対。ライブラリの表面積は既に巨大で、それをさらに増やす類の変更には慎重になる」と明言。これを受け gmlewis が「この変更は進めない」として PR をクローズした。つまりメンテナによる方針的な不採用が確定している。なお Issue #3127 自体は OPEN のまま残っている(クローズ漏れ)。

**推奨アクション**: 着手非推奨。これはコードの問題ではなくメンテナ方針(破壊的変更 + API 表面積増加への忌避)による不採用案件で、対応 PR #3136 も同理由でクローズ済み。コントリビューターとして取り組む価値は低い。むしろ go-github メンテナに対しては、追跡上クローズ漏れになっている本 Issue #3127 を「wontfix」として正式クローズし PR #3136 のクローズ理由へリンクするよう提案するのが妥当な次の一手。どうしても enum 的な利便性が欲しい場合は、既存 *string を変更せず別途定数(const string)や検証ヘルパーを追加する後方互換アプローチを別 Issue として提案する余地はあるが、これも受理は不透明。

---

### Issue [#3366](https://github.com/google/go-github/issues/3366) — How to ensure that a repository has been synced to the Github servers?

分類: **質問・サポート** / 難易度: **些細** / 対応区分: **🤔 メンテナ判断待ち**

**概要**: リポジトリを Repositories.Create で作成した直後に Repositories.Edit で GitHub Advanced Security / Secret Scanning を有効化すると、API レスポンスは 200 で SecurityAndAnalysis のステータスも "enabled" を返すが、github.com 上では実際には無効のままになる、という相談。Create と Edit の間で数秒 sleep すると github.com 上にも反映される。Create のドキュメント通り Get でリトライ(指数バックオフ)しても、Get が 200 を返すようになってもまだ Edit が実効しないため、確実に「同期完了」を判定する方法が知りたい、という質問。

**根本原因**: go-github 側のコードのバグではなく、GitHub サーバ側のリポジトリ作成伝播(eventual consistency)に起因する。github/repos.go:563 の Create は単発の POST を送って即座に返すだけで、伝播完了は待たない(doc comment repos.go:552-555 が明示的に警告)。github/repos.go:736 の Edit も単発の PATCH を送り、サーバが echo した SecurityAndAnalysis をそのまま返すだけ。GitHub のバックエンドは、新規リポジトリの作成直後でセキュリティ機能のプロビジョニングが完了する前でも、PATCH のリクエスト値をそのまま 200 で echo するため、go-github のレスポンス上は "enabled" に見えても実際には反映されていない。さらに Get(repos.go:660)が 200 を返すこと自体もセキュリティ機能設定の準備完了を保証しないため、Get リトライだけでは不十分。いずれも go-github のクライアント層で検出・解決できる問題ではない。

**解決方針**: コード修正は不要(GitHub サーバ側の伝播タイミングの問題のため go-github では解決不可)。メンテナ(gmlewis)が既に案内している通り、確実な readiness 判定は GitHub 公式サポートへの問い合わせが本筋。実用的なワークアラウンドとして次を回答すれば本 Issue はクローズ可能: (1) Create 後に「Get が 200」ではなく「目的のセキュリティ機能が実際に反映されたか」を確認するポーリングにする。すなわち Edit 実行後に Repositories.Get で SecurityAndAnalysis.AdvancedSecurity.Status / SecretScanning.Status が "enabled" になるまで指数バックオフでリトライし、ならなければ再度 Edit する。(2) Edit のレスポンス(サーバの echo)を信用せず、別の Get での再確認を真実とみなす。任意の go-github 改善としては、repos.go の Create doc comment に「Get が 200 を返しても後続の設定変更が即座に反映されるとは限らない(リポジトリのプロビジョニングは非同期)」旨を追記する程度。これはドキュメントのみの軽微な変更。

**関連ファイル**: `github/repos.go`

**現状**: ラベルは [waiting for reply]。最新コメント(2025-07-13, af-ilya)は別ユーザーが同一現象を報告し「Get が True を返した後もセキュリティ設定の構成にはまだ時間が必要。sleep 以外の方法はあるか?」と質問しているが、未回答のまま。メンテナ gmlewis の唯一のコメント(2024-12-04)は「GitHub 公式サポートに問い合わせ、回答をここに報告してほしい」というもの。原報告者(herzrasen)からの公式サポート回答の報告は無い。対応 PR は存在しない(open/all とも 3366 紐付け PR なし)。ブロッカーは GitHub 側の挙動であり go-github 内では解決手段が無いこと。

**推奨アクション**: このユーザー(コントリビューター)が着手すべきコード作業はほぼ無い。本質は GitHub サーバ側の eventual consistency であり go-github では修正不能なため、コードを書く価値は低い。次の一手としては、Issue にワークアラウンド(Edit 後に Get で SecurityAndAnalysis のステータスが実際に "enabled" になるまでポーリングする、Edit の echo レスポンスを信用しない)を回答し、根本解決は GitHub サポート案件である旨を明記してクローズ提案するのが妥当。もし何かコントリビュートするなら、repos.go:552-555 の Create doc comment に「作成直後の後続設定変更は非同期に反映され得る」注意書きを足す docs-only PR が唯一の候補(trivial、good first issue になり得る)。

---

### Issue [#3644](https://github.com/google/go-github/issues/3644) — Refactor codebase to use value parameters instead of pointers where appropriate

分類: **リファクタ** / 難易度: **中** / 対応区分: **🤔 メンテナ判断待ち** / **good first issue**

**概要**: 多くの exported なサービスメソッド（例: CreateRelease(ctx, owner, repo string, release *RepositoryRelease)）が、関数内でミューテートしない必須の構造体引数をポインタ（*Struct）で受け取っている。これは慣習やコピペの産物で、性能上の理由ではない。これを値（Struct）で受け取るよう変更することで「この引数は必須・nil不可」であることをコンパイル時に明示し、API設計の一貫性と安全性を高めるのが目的。Breaking changeのため複数PRに分割して段階的に進める方針。

**根本原因**: 設計上の慣習が根本原因であり、バグではない。github/repos_releases.go:208 の CreateRelease は依然 release *RepositoryRelease を受け取り、line 209-211 で `if release == nil { return ... errors.New("release must be provided") }` という実行時nilチェックを行っている。EditRelease(repos_releases.go:249) も同様。値渡しに変えればこの実行時チェックが型シグネチャによる強制に置き換わる。一方で actions_hosted_runners.go の CreateHostedRunner(..., body CreateHostedRunnerRequest) や UpdateActionsPermissions(..., body ActionsPermissions) など、既に値渡しに移行済みのメソッドも混在しており、移行は service 単位で部分的に進行中。確立された変換パターンは、シグネチャの *Struct を Struct に変え、NewRequest には &status を渡し、テストを value 初期化に直し、accessor を再生成するだけのシンプルな形（新リクエスト構造体やラッパーは作らない）。

**解決方針**: 確立された最小パターンで 1ファイル/PR に分割する。具体的には (1) 対象 exported メソッドの引数を *Struct から Struct に変更し関数内の実行時nilチェックを削除、(2) NewRequest 呼び出しを &param に変更、(3) 対応するテストの input を値初期化(&X→X)に修正し cmp.Equal の比較を調整、(4) script/generate.sh 相当で github-accessors.go と同テストを再生成、(5) example/ 配下の呼び出し箇所も追従修正。後方互換ラッパーは導入しない（clean break）。Edit系メソッドはこの機会に Update へ改名する案もメンテナ間で支持されている（実際 #4340 で `Set*`、#4360 で `EditHookConfiguration`→`UpdateHookConfiguration` を実施）。旗艦例 repos_releases.go の CreateRelease/EditRelease、actions_runners.go の JITConfig、actions_oidc.go（#4340）、HookConfig（Repos/Orgs/Apps 共用, #4360）は当フォークで変換完了済み。残る着手候補は `.golangci.yml` 許可リストに残る他の型（ConfigSettings 等）。dependabot secrets も stevehipwell の #4348 でマージ済み（2026-07-07）。着手前に必ず assignee である quinqu とメンテナに調整し重複作業を避けること。

**関連ファイル**: `github/repos_releases.go`, `github/actions_oidc.go`, `github/actions_runners.go`, `github/gists.go`, `github/gists_comments.go`, `github/github-accessors.go`, `CONTRIBUTING.md`, `.golangci.yml`

**現状**: Issue は OPEN（reopened・未アサイン）。**当フォークが継続的に 1型/1PR で消化中**。前回更新以降にマージされた当フォークの PR: #4372（`RepositoryMergeRequest`+`RepoMergeUpstreamRequest`）・#4378（`TemplateRepoRequest`）・#4382（`DeploymentBranchPolicy` を Create/Update に分割）・#4394（`SarifAnalysis`+`validate` 追加）・#4395（`NewPullRequest`→`CreatePullRequest` 改名＋値渡し）・#4399（`AutolinkOptions`→`CreateAutolinkRequest`＋`AddAutolink`→`CreateAutolink`）・#4401（`CreateOrUpdateCustomRepoRoleOptions` を Create/Update に分割）・#4406（pulls レビュー3endpoint）。**【2026-08-29 更新】** さらに #4425（`UpdateConnectedExternalGroupRequest`）・#4432（LDAP mapping）・#4434（`UpdatePreReceiveHookRequest`）・#4438（`Milestone` 分割）・#4444（`IssueCommentRequest` 共用型）・#4477（`Key` 3分割）がマージ済み。現在 OPEN は #4493（`PullRequestComment` 分割＋`EditComment`→`UpdateComment`・gmlewis APPROVED・マージ待ち）。jvm986 も #4462（`CreateUserImpersonationRequest`）・#4475・#4476（stale allowlist 掃除）を消化して復帰し、stevehipwell も #4446（teams create/update patterns）で参加。なお #4481 の `Ptr(x)`→`new(expr)` 一斉変換により、以後の変換 PR のテストは `new(expr)` スタイルで書く。

**⚠️ 分担運用（2026-07-18 合意）**: 新規コントリビューター **jvm986** が本 Issue の引き取りを申し出たのに対し、gmlewis の仲介で「単独アサインでなく分担」に決着（[issuecomment-5011093547](https://github.com/google/go-github/issues/3644#issuecomment-5011093547)）。**運用ルール: 各自 同時に1PRのみ・新しい型/サービスに着手する前に相手の Open PR を確認する**。jvm986 は **Issues サービス担当**（#4396 `IssueRequest` 分割・#4400 `Label` 分割＋`EditLabel`→`UpdateLabel` をマージ済み）なので、**当フォークは `IssueComment`/`Milestone`/`LockIssueOptions`/`IssueImportRequest` など Issues 配下の型を回避**する。

**確立済みの変換パターン**: ミューテート系 body 引数を `*T`→値 `T`、required フィールドは非ポインタ＋`omitempty` 除去、**optional なスライスは `omitempty` でなく `omitzero`**（空配列を明示送信できるようにするため。#4401 で gmlewis 指摘）、`Edit`→`Update` 等 docs 操作名に合わせた改名、`Options`→`Request` 改名、非推奨ラッパー無し（clean break）、`.golangci.yml` 許可リストから対象型を除去、生成物再生成、custom-gcl(paramcheck) 0 issues を確認。**レスポンス型を request body に流用している場合は専用 request 型を新設**（#4406 `PullRequestSubmitReviewRequest`・#4425 `UpdateConnectedExternalGroupRequest`）。yongzhang の hugeParam(gocritic) 懸念は lint 設定上 disable のため非ブロッカー。

**推奨アクション**: 当フォークの主力タスク。**候補選定の基準**（実績から確立）: (1) required なのにポインタのフィールドを持つ型 ＞ param-flip のみの型（後者は「価値が薄い」と見られやすい）、(2) 単一メソッド専用 ＞ 共用型（共用型は全メソッド一括変換が必要）、(3) 共用型でも create/update でスキーマが非対称なら**最初から分割**して提出（#4382/#4401 の教訓。後から alexandear に要求される）、(4) レスポンス型の body 流用は専用型新設、(5) `example/`（別モジュール）と `test/integration/`（`//go:build integration`・root モジュール）への波及を事前確認（後者は通常の `go build ./...` では検出できず `go vet -tags integration` が必要）。**回避すべき候補**: `PagesUpdate`（`CNAME` の omitempty 無しが「空でカスタムドメイン削除」の意味を担う）、`AddProjectItemOptions`（docs が oneOf）、`Repository`/`User`/`Organization`（波及が数百箇所）。

**【2026-10-10 更新】** Issue 本体への新規コメントはないが、cross-reference が続いている。@JamBalaya56562 が #4519(09-10)・#4540(09-12)・#4548(10-03)・#4606(10-07)・#4607(10-09) を、@jvm986 が #4551（`CreateOrgInvitationOptions`→`CreateOrgInvitationRequest`＋値渡し＋`TeamID`→`TeamIDs`, 09-14 提出・09-16 マージ）をマージ済み。メンテナ側では @gmlewis が #4375 をクローズし（09-18）、代わりに #4576 `tools/schemafields`（`//meta:operation` と値渡しの `body` 引数から request struct を自動特定するので注釈は不要。lint/CI に組み込み済み, 09-20 マージ）を作り、さらに #4581 `fix!: Match request body fields to the OpenAPI schemas`（`-fix` を一括適用し、required な request フィールドを多数非ポインタ化, +2874/-542/59 ファイル, 09-23 マージ）と #4592（response struct をチェック対象外にする, 10-02 マージ）をマージした。現時点で #3644 関連の open PR はない。

**【2026-10-10 更新・残候補】** 当フォークの #4607（`CreateForkRequest`/`AddCollaboratorRequest`）が 10-09 にマージされた（merge `40091e21`）。`.golangci.yml` の `body-allowed-pointer-types` 残りは **37 型**（`body-allowed-wrong-names` は 16 型）。OpenAPI（`api.github.com.json`）と照合した次の候補は **Projects v2 の item 2型（`AddProjectItemOptions`/`UpdateProjectItemOptions`・`github/projects.go`・org/user で計4メソッド）**。理由: (1) `AddProjectItemOptions.Type` は required なのに `*ProjectV2ItemContentType`、(2) add の oneOf 第2分岐（`owner`/`repo`/`number`）が Go 側に欠落、(3) update は `fields` が required なのに `omitempty`、(4) update の `Archived` はスキーマ（properties=`fields` のみ）に存在しない。他の小型候補（`LockIssueOptions`/`PullRequestBranchUpdateOptions`/`UserSuspendOptions`/`CreateWorkflowDispatchEventRequest`）は param-flip＋rename のみで価値が低い。

---

### Issue [#3658](https://github.com/google/go-github/issues/3658) — Feature Request: configurable telemetry

分類: **メタ/プロセス** / 難易度: **些細** / 対応区分: **🤔 メンテナ判断待ち**

**概要**: Webサービスで数万件規模のGitHub API呼び出しを行う際、ホットパスや過剰なAPI呼び出しを特定するための計測手段が欲しいという要望。報告者は既存の3つの回避策(各API呼び出しのラップ、client取得時のラップ、struct全体のラップ)はいずれも冗長/迂回容易/ページング非対応だと指摘。簡易案として Client.bareDo 内の caller.Do() 直前に呼ばれる Client.OnDo func() / WithCallback() の追加を提案している。

**根本原因**: 該当なし(バグではなく機能/方針の議論)。技術的背景としては、報告者の3案がいずれも github/ レイヤーでラップする方式のためページングで発生する個別HTTPリクエストを捕捉できない点が本質的な不満。一方、go-github は http.Client / http.RoundTripper を差し替え可能な設計(github/github.go:387-398 の WithTransport、c.client.Transport)であり、transport層で計測すればページングを含む全リクエストを自然に捕捉できる。実際 otel/transport.go の RoundTrip(otel/transport.go:54-109)が各リクエストにspanを張り、rate-limitヘッダ・request-id・status_codeを記録している。

**解決方針**: 既にリポジトリには native な /otel モジュール(github.com/google/go-github/v88/otel)が存在し、PR #3938「feat(otel): Add native OpenTelemetry Transport module」(2026-02-04 merge、本Issue停滞後)で追加済み。otel.NewTransport(base, opts...) が OpenTelemetry 計装済みの http.RoundTripper を返し、github.WithTransport(...) で差し込める。これは transport層計装なのでページングを含む全API呼び出しを計測でき、本Issueで挙がった3案の「ページング非対応」という共通欠点を解消する。コメントで stevehipwell が提案した「client transport上のOpenTelemetry」そのものでもある。したがって追加実装はほぼ不要で、メンテナ判断としては (a) 本Issueを /otel モジュールへ誘導してクローズ、または (b) OTel非依存の素朴なコールバック(WithCallback)が別途必要かを報告者 @aThorp96 に確認、のいずれか。なお PR #3938 は closingIssuesReferences が空で本Issueを自動クローズしていない。

**関連ファイル**: `otel/transport.go`, `github/github.go`

**現状**: ラベルは waiting for reply。最新の実質コメントでメンテナ gmlewis が、PR #3681(Not-Dhananjay-Mishra による WithCallback 実装)はこの要望を完全には満たさず、かつ報告者 @aThorp96 への確認(issuecomment-3167600730: 「client transport上のOpenTelemetryで足りない点は?」)が未回答のためPR作成は時期尚早、として #3681 をクローズし議論を保留している。報告者からの返信が約束のブロッカー。ただし調査の結果、保留中に PR #3938 で /otel transport モジュールが本体に追加されており、要望の中核は事実上満たされている可能性が高い。

**推奨アクション**: このユーザー(コントリビューター)は新規コードに着手しないことを推奨。理由は (1) ラベルが waiting for reply で原作者の回答待ちのブロッカーがある、(2) 既に /otel モジュール(PR #3938)が要望をほぼ充足しており重複実装になりかねない、(3) PR #3681 が同方向の提案で既にクローズ済み。次の一手としてはコード変更ではなく、Issueに「v88で /otel モジュール + github.WithTransport により transport層でのOTel計装が可能になりページングも捕捉できる。これで要望は満たされるか? OTel非依存の軽量コールバックが別途必要か?」というコメントを残し、原作者の回答を促してメンテナのクローズ判断を後押しするのが有効。

---

### Issue [#3761](https://github.com/google/go-github/issues/3761) — Consistent naming for enterprise & organization scoped functions

分類: **メタ/プロセス** / 難易度: **難** / 対応区分: **🤔 メンテナ判断待ち**

**概要**: enterprise / organization スコープを持つメソッドの命名規則が不統一であるという指摘。提案者(stevehipwell)は `<VERB><SCOPE><SUBJECT>` パターン(例 `GetOrganizationThing`)への統一を要望している。本文では greenfield なら `client.organization.GetThing` のような構造的設計もあり得たと触れているが「現状そうではない」と明記。具体的なバグではなく、ライブラリ全体の API 命名ポリシーに関する方針議論。

**根本原因**: 該当なし(バグではない)。ただし指摘されている不統一は実在することをコードで確認した。github/billing.go では同一ファイル内に `GetOrganizationPackagesBilling` / `GetOrganizationStorageBilling`(Organization 表記)と `GetOrgAICreditUsage`(Org 表記)が混在。さらにスコープ語の位置も不統一で、前置型(`GetOrgPublicKey`, `ListOrgSecrets` in actions_secrets.go)、`ForOrg`/`ForEnterprise` 後置型(`ListAlertsForOrg` in code_scanning, `GetTotalCacheUsageForEnterprise` in actions)、`InOrg` 後置型(`ListEnabledReposInOrg`, `ListInOrg` in codespaces)、`InEnterprise` 型(`AddEnabledOrgInEnterprise` in actions_permissions_enterprise.go)が併存する。原因は、サービスファイルが GitHub API ドキュメントの構成に沿って個別に追加され(CONTRIBUTING.md 「Other notes on code organization」)、命名規則が文書化されていない(CONTRIBUTING.md に naming 節が存在しない)ため、各 PR ごとに表記が分かれたことにある。

**解決方針**: 既述の通り。

**関連ファイル**: `CONTRIBUTING.md`, `github/billing.go`, `github/actions_secrets.go`, `github/actions_variables.go`, `github/actions_permissions_enterprise.go`, `github/codespaces_orgs.go`, `github/copilot.go`, `github/github-iterators.go`

**現状**: 2025-10-08 の最新コメント時点で、提案者 stevehipwell が greenfield 案(`client.Organization.SomeService.GetThing`)は現実には採れないと明確化し、現状では「スコープと subject の語順を決めれば十分」という方向に収束しつつある。ただしメンテナ(gmlewis 等)からの公式見解・合意はまだ無く、規約の最終決定はブロック状態。labels 未付与、対応中の PR も無し。zyfy29 との元の議論は別の PR レビューに由来すると推測される(Issue 本文には明示リンク無し)。

**推奨アクション**: このコントリビューターは着手すべきでない(現段階では)。これはコード実装タスクではなく、ライブラリ全体に波及する API 命名ポリシーの方針決定であり、まずメンテナ(gmlewis/willnorris)の合意が必須。本 Issue は labels なし・コメント1件・メンテナ未関与で、規約自体(Org vs Organization、語順、許容スコープ)が未確定。建設的な次の一手としては、「(a) Org か Organization か、(b) スコープ前置か ForX 後置か」の二択に絞った具体的な規約案を Issue にコメントしてメンテナの判断を仰ぐこと。合意が得られたら、まず CONTRIBUTING.md への規約追記(低リスク・小PR)から始め、既存メソッドのリネームは Deprecated ラッパ方式で段階的に行うのが現実的。今すぐ大規模リネーム PR を出すのは互換性破壊リスクが高く非推奨。

---

### Issue [#3894](https://github.com/google/go-github/issues/3894) — Four billings endpoints have suddenly disappeared from the GitHub v3 API

分類: **メタ/プロセス** / 難易度: **些細** / 対応区分: **🤔 メンテナ判断待ち** / **good first issue**

**概要**: go-github のメンテナ(gmlewis)が openapi_operations.yaml 更新作業中に、4つの billing 関連エンドポイント(GET /orgs/{org}/settings/billing/packages、/orgs/{org}/settings/billing/shared-storage、/users/{username}/settings/billing/packages、/users/{username}/settings/billing/shared-storage)が OpenAPI メタデータと GitHub v3 API 公式ドキュメントの両方から突然消えたことを発見した。これは GitHub の「product-specific billing APIs are closing down」(2025-09-26 changelog)というアナウンスに対応する。外部ユーザのバグ報告ではなく、メンテナ自身が立てた追跡/方針決定用の Issue である。残る論点は「GitHub Enterprise(オンプレ旧バージョン)ユーザのために、これら廃止エンドポイントをいつまでコードに残すか/Deprecated マークを付けるか」というプロセス・方針の議論。

**根本原因**: 技術的なバグではない。根本原因は GitHub 側でこれら product-specific billing API が廃止された(closing down)ことであり、それに伴い公式 OpenAPI 記述とドキュメントから当該エンドポイントが削除された。go-github 側では、メタデータ検証ツール(tools/metadata)が openapi_operations.yaml に存在しないエンドポイントを各サービスメソッドの //meta:operation 注釈と突き合わせて検証するため、注釈が残ったままだと CI が失敗する。確認したコード: github/billing.go の該当4関数(GetOrganizationPackagesBilling 178行、GetOrganizationStorageBilling 199行、GetPackagesBilling 245行、GetStorageBilling 266行)は //meta:operation 注釈が外され、代わりに「This endpoint appears to have disappeared ... See issue #3894」というコメントが付いている。tools/metadata/metadata.go には issue #3894 を参照する skipServiceMethod マップが追加され、上記4メソッドをメタデータ検証から除外している。これらは MERGED 済みの PR #3895 (「Relates to: #3894」)で実装済み。

**解決方針**: コードによる主要対応(エンドポイントを残しつつ openapi 自動ツールから除外する stop-gap)は PR #3895 で既に実施・マージ済みのため、追加の必須コード変更はない。残課題への対応として、(1) 最新コメントで elminster-aom が提案している通り、4関数の doc コメントに Go 標準の「// Deprecated:」マーカーを追加し、廃止 changelog URL(github.blog/changelog/2025-09-26-product-specific-billing-apis-are-closing-down)を明記する。これは github/billing.go の該当4関数(178/199/245/266行付近)のコメント編集のみで完結する小規模変更で、good first issue 向き。(2) Enterprise 向けにいつ完全削除するかは、メンテナ(gmlewis)の方針決定待ちであり、コントリビュータ側で判断・実装すべきではない。現状コメントには Issue リンクは付くが Deprecated マークは未付与であることを確認済み。

**関連ファイル**: `github/billing.go`, `tools/metadata/metadata.go`, `openapi_operations.yaml`

**現状**: **elminster-aom が求めていた「コード上で deprecated とマークする」対応は、当フォークの PR [#4362](https://github.com/google/go-github/pull/4362)（`chore: Mark removed billing endpoints as deprecated`）で実装され 2026-07-04 にマージ済み**。#4358（stevehipwell）が入れた deprecated 自動注入機構に乗せ、消滅した billing 4エンドポイント（`GetOrganization`/`GetPackages`/`GetStorage` 系）を `openapi_operations.yaml` の手動 `operations:` に `deprecated: true` ＋元 dotcom URL 付きで登録し、`//meta:operation` を復元して `script/metadata.sh update-go` に canonical な `// Deprecated:` コメントを自動挿入させた。ad-hoc な「appears to have disappeared」コメントも削除。**残る未決事項は「完全削除の可否・時期」だけ**で、これは Enterprise 互換方針としてメンテナ判断待ち（PR は `Updates #3894` に留め、クローズ判断を gmlewis に委ねている）。主要 stop-gap の PR #3895(MERGED, 2025-12-29) は従来どおり skipServiceMethod でツール除外を継続。

**推奨アクション**: このユーザ(コントリビュータ)が着手するなら、「方針決定が要る完全削除」ではなく「Deprecated マーカー追加」の小タスクに限定するのが安全。具体的には github/billing.go の4関数(GetOrganizationPackagesBilling/GetOrganizationStorageBilling/GetPackagesBilling/GetStorageBilling)の doc コメントに「// Deprecated: ...」を追記し廃止 changelog を引用する PR を、本 Issue の elminster-aom 提案を根拠に出す。ただし着手前に Issue 上で gmlewis に「Deprecated マーク追加 PR を出してよいか」を一言確認するのが無難(完全削除時期はメンテナ判断のため触らない)。それ以外の本体コード対応は #3895 で完了済みなので不要。

---

### Issue [#3939](https://github.com/google/go-github/issues/3939) — Before contributing AI slop PRs to this repo...

分類: **メタ/プロセス** / 難易度: **易** / 対応区分: **✅ コードで対応可能** / **good first issue**

**概要**: 低品質な「AI slop」PR(投稿者がGitHub UI上で自身のPRを確認すらせずレビュー依頼するもの)が急増し、メンテナとレビュアーの時間を浪費しているという問題提起。コード変更ではなく、CONTRIBUTING.md やPRテンプレートなどプロセス/ドキュメント面での対策を議論するメタIssue。当初の「Tipsセクション追加」は既にPR #3940 でマージ済み。

**根本原因**: 該当なし(コードのバグではない)。CONTRIBUTING.md を確認したところ、本Issueから派生したPR #3940 により「Tips」セクション(CONTRIBUTING.md の135-153行)が既に追加されている。内容は「reviewerと同じUIで自己レビュー」「短いPRタイトル/説明」「バグ修正には再現テストを付ける」「小さくフォーカスしたPR」をカバー。一方、コメントで議論された .github/pull_request_template.md は現状リポジトリに存在しない(.github/ 配下は dependabot.yml と workflows/ のみ)。

**解決方針**: スレッドの議論を踏まえた残タスクは主に1点: Caddyリポジトリを参考にした .github/pull_request_template.md の追加。テンプレートには Description セクションと「Assistance Disclosure(AI/LLM利用の開示)」セクションを設け、AI利用は開示すれば許容する旨を明記する(alexandearが2026-02-24に具体例を提示済み)。なお、alexandearが提案したAnthropic magic string案は本人が「2026-06-02時点でもう機能しない(deprecated)」と更新しており、採用すべきではない。「AIレビューコメントを抑制するガイダンス追加」案はgmlewisが「CONTRIBUTING.mdが肥大化しすぎる懸念」を示しており、PRテンプレート方式に集約する方が合意を得やすい。

**関連ファイル**: `CONTRIBUTING.md`, `.github/pull_request_template.md`

**現状**: 最新コメント(2026-05-16, gmlewis)は「Feel free to write a PR and we can all take a look.」でPRテンプレート/AIレビューコメント方針のPR提出を歓迎する姿勢。当初の主目的(CONTRIBUTING.mdへのTips追記)はPR #3940 で2026-01-26にマージ済みのため、Issueは「ほぼ対応済みだが派生タスク(PRテンプレート)が未着手」の状態でOpen継続中。vouchやmagic stringはgmlewisが見送り(magic stringは本人が後にdeprecatedと訂正)。CONTRIBUTING.md肥大化への懸念が一貫したブロッカー。

**推奨アクション**: 着手推奨だが、主目的はPR #3940 で既に達成済みである点に注意。新規コントリビューターが取り組むなら、残った具体的アクションとして「.github/pull_request_template.md の新規作成(Description + Assistance Disclosure セクション)」が最適。alexandearが提示したCaddy由来のテンプレート例とgmlewisの『簡潔さ重視』の意向を尊重し、短く保つこと。実装前に該当コメントスレッドで意図を一言述べ、magic string案は採用しないこと。難易度はMarkdownファイル1枚の追加でeasy、ラベルもgood first issue。

---

### Issue [#4077](https://github.com/google/go-github/issues/4077) — New API Version

分類: **メタ/プロセス** / 難易度: **難** / 対応区分: **🔀 対応PRあり（#4550・一部項目）**

**概要**: GitHubが新しいREST APIバージョン 2026-03-10 をリリースしたことを受け、go-githubとしてどう対応するか(両バージョン併存サポートか、新バージョン採用を待つか)を議論するために立てられたIssue。メンテナ(gmlewis)は「以前から対応予定でREADMEにも方針記載済み」と回答し、対応を歓迎。その後コントリビューター(maishivamhoo123)が、breaking changes ドキュメントに基づく25項目のエンドポイント対応チェックリストを作成し、これが事実上の追跡(トラッキング)Issueとなった。

**根本原因**: 該当なし(単一バグではなくプロセス/方針議論+追跡)。技術的背景としてgithub/github.go:43-44 に api20221128="2022-11-28" と api20260310="2026-03-10" の定数があり、github.go:570-572 で apiVersionDefault=2022-11-28 のまま、apiVersionMax=2026-03-10 に設定されている。つまり新バージョンはper-methodオーバーライドの上限としてサポート済みだが、デフォルトは旧バージョンのまま。README.md:521-534 がこの移行戦略を明文化している(breaking changesのある各メソッドを先にper-methodヘッダーで上書き→全完了後に最後の1コミットでデフォルトを上げてメジャーバージョンbump)。チェックリストの未完了項目に対応する非推奨フィールドはコード上に現存(例: github/repos.go:74 UseSquashPRTitleAsDefault、:93 HasDownloads)し、チェック済み項目はコードに反映済み(rate_limit.go:90-139 はresources.coreから読む、repos_contents.go:181-182 は type=="submodule" を処理、github.go:1912-1913 にSARIFパス分岐)。

**解決方針**: このIssue自体は1つのPRでクローズするものではなく、25項目チェックリストの追跡Issueとして維持する。コントリビューターは個別チェックボックス(例: HasDownloads削除、UseSquashPRTitleAsDefault→SquashMergeCommitTitle置換、code-scanning default-setup の javascript/typescript→javascript-typescript統合、Security Advisories の cvss→cvss_severities など)を1つずつ取り上げ、READMEの戦略どおり該当メソッドにper-method APIバージョンオーバーライドを適用するPRを出す。全項目完了後に最後のPRで github.go:570 の apiVersionDefault を api20260310 に変更し、per-methodオーバーライドを除去、README互換表を更新してメジャーバージョンbumpする。

**関連ファイル**: `README.md`, `github/github.go`, `github/repos.go`, `github/rate_limit.go`, `github/repos_contents.go`, `github/code_scanning.go`, `github/security_advisories.go`

**現状**: 最新コメント(gmlewis)時点で、25項目チェックリストが追跡用に確立され、メンテナは各チェックボックスへの貢献を歓迎・重複防止のため事前コメントを依頼している状態。チェックリスト4項目(rate_limit, teams permission, repo contents submodule, SARIF content-type)は完了済みとマークされ、コードでも確認できた。残り21項目は未着手。基盤となるクライアントAPIバージョンサポート(#4246)とAPIバージョン定数(#4236)は既にMERGED。デフォルトAPIバージョンは依然 2022-11-28 のまま(github.go:570)で、移行は進行中・未完了。本Issue番号を直接参照するオープンPRは検出されず。

**推奨アクション**: このIssueは「親(追跡)Issue」であり、ここで一括対応すべきではない。新規コントリビューターには、チェックリストの未完了項目のうち独立性が高く小さいもの(例: HasDownloads の has_downloads 削除、Gists の history/forks 除去、API root の authorizations_url/hub_url 削除)を1つ選び、READMEのper-method移行方針(README.md:521-534)に従って単一PRで対応することを勧める。着手前に必ず本Issueにコメントして重複を防ぐこと。デフォルトAPIバージョンのbump(github.go:570)は全項目完了後にメンテナ主導で行うべき最終ステップなので、個人で勝手に上げない。基盤(api20260310定数、apiVersionMax、ErrUnsupportedAPIVersion検証、per-methodオーバーライド機構)はPR #4246/#4236で既に整備済みなので、各項目PRはその上に乗せる形になる。

**【2026-10-10 更新】** @keeltrace が 09-07〜09-08 にチェックリスト 3 項目の担当を宣言した。#4528（単数 `Assignee` の deprecate, 09-08 マージ）と #4529（app installation 削除で 202 を受け付ける, 09-08 マージ）はマージ済み。Attestations の `bundle_url` 対応 #4550 は OPEN のまま（CLA 未署名で `cla/google` が FAILURE、@Not-Dhananjay-Mishra の COMMENTED レビューあり）。

---

### Issue [#4237](https://github.com/google/go-github/issues/4237) — Move compatibility for non github.com targets to github.With<THING> options

分類: **提案** / 難易度: **難** / 対応区分: **🤔 メンテナ判断待ち（傘的提案・残作業は分割PR）**

**概要**: 非github.comターゲット(GHES/GHEC/GHAS)向けの互換性処理をデフォルトのクライアント構成から切り離し、すべて github.With<THING> オプション経由で有効化する設計提案。具体的には、デフォルトAPIバージョンを最新(2026-03-10)にし、WithAPIVersion でバージョンを明示設定、WithAdvancedServer(GHAS)/WithEnterpriseCloud(GHEC)を追加、WithEnterpriseURLs を非推奨化する。デフォルトのクライアント表面積を単純化し、特定バージョンが必要なエンドポイントはクライアントバージョンに対して検証してエラーを返す方式にしたい、という内容。

**根本原因**: 該当なし(バグではなく設計提案)。現状コードを確認した結果、課題の出発点は実在する。github/github.go:476 の WithEnterpriseURLs が非github.com向け設定の唯一の入口であり、サービスメソッドは特定バージョンが必要な場合に生の WithVersion(api20260310) を直接埋め込んでいる(github/private_registries.go:247,268,289,311,334,351 で6箇所確認)。この方式では、対象サーバがそのAPIバージョンを持たない場合に明示的なエラーを返せず、クライアントレベルのバージョン状態も持っていなかった、という提案者の指摘は正しい。

**解決方針**: 傘的な設計提案で、複数の小さなPRに分割して段階的に実装する方針（デフォルト最新版移行の #4077 とは分離）。残作業: (1)公開オプション `WithAPIVersion` 追加、(2)`WithAdvancedServer`(GHAS)、(3)`WithEnterpriseCloud`(GHEC)、(4)`WithEnterpriseURLs`(github.go:476) の非推奨化、(5)`private_registries.go` 等の生 `WithVersion` 直書きを再利用可能な `checkVersion` ヘルパ経由へ移行。クライアントレベルのバージョン状態＋`checkVersion` プリミティブを既存挙動を変えずに導入する第1段は上流で実装済み。着手前に設計スライス（エラー挙動・デフォルト版の扱い）をIssueでメンテナ合意。

**関連ファイル**: `github/github.go`, `github/github_test.go`, `github/private_registries.go`

**現状**: メンテナ gmlewis は概ね賛同（特定バージョンが必要な endpoint はクライアント側で明示し、利用者がバージョニングを意識せず済むようにすべき、という認識）。alexandear/danyalahmed1995/stevehipwell は #4077（デフォルト最新版移行＝大量作業）と enterprise オプション再設計を分離し、小さなPRに割る方針で収束。第1段（クライアントレベルのバージョン状態＋`checkVersion` プリミティブ、挙動不変）は上流で実装済み。`WithAdvancedServer` 等の後続オプションの公開PRは未提出。

**推奨アクション**: 着手の価値は高いが大物。Issue自体はオープンのまま(複数PRにまたがる傘的提案)。第1段(クライアントバージョン状態のプリミティブ)は上流で実装済みなので、次の現実的な一手は提案者 stevehipwell が示唆した「WithAdvancedServer を追加し、URL接尾辞補正と max版を 2022-11-28 に設定する」小さなPR。実装する場合は、(a)公開 WithAPIVersion オプションの追加、(b)private_registries.go の WithVersion(api20260310) 直書きを checkVersion ヘルパに置換、のいずれかから着手するのが分割の単位として適切。ただし設計方針(エラー挙動・デフォルト版の扱い)はメンテナ確定待ちの論点が残るため、コーディング前にIssueで該当スライスの合意を取ること。新規コントリビューターには不向き、go-github の client 構成とバージョニングに精通した中〜上級者向け。

---

### Issue [#4238](https://github.com/google/go-github/issues/4238) — Add support for Copilot Spaces

分類: **新API対応** / 難易度: **中** / 対応区分: **✅ コードで対応可能（#4379 で org CRUD はマージ済み・残エンドポイント）**

**概要**: GitHubが2026-05-18にGA化したCopilot Spaces APIをgo-githubでサポートしてほしい、という新規エンドポイント追加要望。Copilot Spacesは、組織またはユーザー単位でCopilotの「スペース」(コンテキスト共有領域)を作成・管理し、コラボレーターやリソースを紐付けるためのREST API。現状go-githubには該当する実装が一切存在しない。

**根本原因**: バグではなく未実装の機能要望。リポジトリ内を確認したところ、github/ 配下に copilot.go / copilot_cloud_agent.go は存在するが、copilot-spaces を扱う型・メソッドは皆無(github/ 全体を grep しても CopilotSpace / copilot-spaces のヒットは0件)。一方で openapi_operations.yaml には Copilot Spaces の全エンドポイント定義が既に取り込まれており(2856行目以降の /orgs/{org}/copilot-spaces 系、8488行目以降の /users/{username}/copilot-spaces 系)、対応する //meta:operation 注釈付き Go メソッドが無い「未カバー」状態になっている。つまり go-github のメタデータ同期ツールは API の存在を既に検知済みで、実装だけが欠けている。

**解決方針**: 既存の CopilotService(github/copilot.go の CopilotService、または新規 copilot_spaces.go ファイル)に対し、openapi_operations.yaml に列挙された全28エンドポイントを実装する。エンドポイントは org 用と user 用で完全に対をなし、3グループに分かれる: (1) Spaces本体 — List/Create/Get/Set(PUT)/Delete (GET,POST,GET,PUT,DELETE × {orgs/{org}, users/{username}})、(2) Collaborators — List/Add(POST)/Set(PUT)/Remove(DELETE) で actor_type と actor_identifier をパスに取る、(3) Resources — List/Create/Get/Set(PUT)/Delete で space_resource_id を取る。実装パターンは既存メソッド(例 copilot.go:311 ListCopilotSeats、copilot_cloud_agent.go:36 GetCloudAgentConfiguration)に倣い、各メソッドに GitHub API docs コメントと //meta:operation 注釈を付ける。CopilotSpace, CopilotSpaceCollaborator, CopilotSpaceResource などの構造体を新設し、対応する _test.go と、script/metadata.sh / generate.sh で生成される github-accessors.go・metadata.yaml を更新する必要がある。空間番号は space_number(数値)、コラボレーター削除/設定は actor_type+actor_identifier の複合パスである点に注意。

**関連ファイル**: `github/copilot.go`, `github/copilot_cloud_agent.go`, `openapi_operations.yaml`

**現状**: OPEN・ラベル無し。担当の変遷: 当初 @ibrahimkarimeddin が立候補しアサイン → 約1ヶ月音沙汰なし → @maishivamhoo123 が再依頼されアサイン。**その後 2026-07-13 に @Ackberry が PR [#4379](https://github.com/google/go-github/pull/4379)（`feat: Add simple CRUD endpoints for GitHub Copilot Spaces`, +1453/-0・6ファイル）を提出**し、`reviewDecision=APPROVED` まで到達。ただし PR 本文が明記するとおり **`Solves #4238 partially`** ＝ org スコープの5エンドポイント（List/Get/Create/Update/Delete）のみで、user スコープ等は未対応のため Issue は open のまま残る見込み。alexandear が 2026-07-15/20/22/24 と複数回インラインレビューし、Not-Dhananjay-Mishra も 2026-07-31 にコメント。→ 対応区分を「✅ コードで対応可能」から **「🔀 対応PRあり」** に変更。

**推奨アクション**: 着手の前提として、このIssueは既に @maishivamhoo123 にアサイン済み(メンテナ gmlewis が「the issue is now yours」とコメント、前担当 @ibrahimkarimeddin から引き継ぎ)である点に注意。新たに着手するなら、まずアサイン状況と進捗をコメントで確認すべき。技術的には実装難易度は中程度: API設計自体は明快で既存 CopilotService のパターンに完全に倣えるが、28エンドポイント(org/user 対称 × spaces/collaborators/resources)とそれぞれの構造体・テスト・生成物更新があるため分量が多い。PRを分割(例: spaces本体 / collaborators / resources、あるいは org系 / user系)するとレビューしやすい。openapi_operations.yaml に定義が既存なので、generate.sh のチェックを実装完了の指標として使える。

**【2026-10-10 更新】** #4379（@Ackberry、組織の Copilot Spaces CRUD 5 エンドポイント）は 09-08 にレビュー指摘へ対応した。@gmlewis（09-09）と @alexandear（09-11）が APPROVED を出し、09-11 にマージされた。ただし PR 本文は「Solves #4238 partially」で closing 参照もないため、Issue は OPEN のまま。残りのエンドポイント（ユーザースコープなど）に取り組む PR は見当たらない（`gh search prs "Copilot Space"` の結果は #4379 のみ）。

---

### Issue [#4334](https://github.com/google/go-github/issues/4334) — Metadata question

分類: **メタ/プロセス** / 難易度: **不明** / 対応区分: **🤔 メンテナ判断待ち**

**概要**: stevehipwell が受け入れテストの再実装中に、Actions secret 系関数（`GetEnvPublicKey`/`ListEnvSecrets`/`GetEnvSecret`/`CreateOrUpdateEnvSecret`/`DeleteEnvSecret`）のシグネチャが、公式 OpenAPI にもう存在しない GHES v3.7 由来の旧 API を追っていることに気づいた（この個別ケースの修正は PR #4335 で提案済み）。これを起点に、サポート終了した API/エンドポイントを go-github のメタデータでどう扱うかという方針を問うている。

**根本原因**: バグではなく方針・プロセスの相談。openapi_operations.yaml には旧 GHES バージョンの未使用エンドポイント（projects v1 系など）や、GitHub 側でサポート終了したエンドポイントが deprecated マークされないまま残存し、それに対応する Go メソッドも旧シグネチャのまま残っている。「サポート終了エンドポイントの扱い」を定めるポリシーが未確立なのが背景。

**関連ファイル**: `openapi_operations.yaml`, `tools/metadata/`, `github/actions_secrets.go`

**現状**: 2026-06-26 起票（stevehipwell）・ラベル無し・OPEN・コメント11件。**gmlewis が3つの質問すべてに回答済みで、方針はほぼ決着している**: (1) **削除しない** —「Enterprise ユーザーが想像以上に多く、削除すると『まだ GHES X.Y.Z を使っている』『でも新版のバグ修正も要る』の板挟みになる。長年の結論として deprecated エンドポイントは消さない」、(2) **deprecated マークはすべき**（最近そうしている）、(3) **`tools/metadata` が Go 関数の deprecated 化を自動で行う**。さらに「**DotCom / GHEC / 最新 GHES のどのスキーマにも無いエンドポイントは deprecated にすべき**」で合意し、stevehipwell が該当する **46 エンドポイント**を `yq` で特定・一括更新用コマンドも共有済み（ただしまだ PR 化されていない）。個別ケースの PR #4335 は 2026-07-01 マージ済み。**残る話題は stevehipwell の新提案（2026-07-03）**＝「`//meta:operation` コメントの URL と関数内の実際の URL 宣言をトークン化して突き合わせる検証」で、gmlewis は「別ツールを新規に書くのではなく、`meta:operation` 行と URL を実際に生成・検証している `tools/metadata` に乗せて既存の非自明なロジックを活用せよ」と steer。stevehipwell も「まさにそれを聞きたかった。組み込み方が分かった」と応答。

**当フォークとの関係**: 当フォークの Open PR **#4375**（`tools/metadata` に `check-schema-fields` サブコマンドを追加）は、まさに gmlewis がここで示した「別ツールでなく `tools/metadata` に統合せよ」という方針と同じ形をとっている。#4375 の本丸課題（マッチング対象が 4/1142 に留まる問題を `//meta:schema` 明示注釈へ差し替える再設計）を提案する際は、**この #4334 のやり取りを根拠として引用できる**。

**推奨アクション**: コード着手ではなくメンテナ（gmlewis 他）の方針決定が前提。建設的な一手は、(a) 個別ケースの PR #4335 のレビュー/取り込みを進める、(b)「サポート終了エンドポイントの削除 vs deprecated マーク」の運用方針を Issue 上で整理し合意を得る、の2点。方針が固まれば、metadata ツール（tools/metadata）に deprecated 検出/警告を組み込む拡張が次段階として考えられる。エンドポイント削除は破壊的になり得るため慎重な合意が必要。

---

### Issue [#4365](https://github.com/google/go-github/issues/4365) — Auth transports add credentials after cross-origin redirects

分類: **セキュリティ** / 難易度: **中** / 対応区分: **🤔 メンテナ判断待ち（2026-10-10 変更: #4564 マージ後に再オープン・設計議論中）**

**概要**: go-github が同梱する2つの認証用 `http.RoundTripper` ― `BasicAuthTransport`（Basic 認証: username/password + 2FA の `OTP`）と `UnauthenticatedRateLimitedTransport`（OAuth アプリの `ClientID`:`ClientSecret` を Basic ヘッダとして付与しレート上限を引き上げる）― が、`RoundTrip` の中で通過するすべての送信リクエストに無条件で認証情報を付与している。Go 標準 `net/http` の `Client` はリダイレクト（3xx）を追跡する際、同じ `Transport` を使って新しいリクエストを再送するため、リダイレクト先が別オリジン（別スキーム/ホスト/ポート）であっても、その cross-origin リクエストに Basic 認証情報が再付与されて送信されてしまう。報告者（huynhtrungcsc）は「これらのトランスポートが付与する認証情報は元のオリジンにのみ送られるべきで、オリジンを跨ぐリダイレクト先には付与されずに転送されるべき」と主張し、2つの `httptest` サーバ（1台目がリダイレクトを返し、2台目が Authorization ヘッダの有無を記録）で再現可能とする。攻撃者（またはリダイレクト先の第三者ホスト）が認証情報を窃取できる資格情報漏洩（credential leakage）である。

**根本原因**: `github/github.go` の `setCredentialsAsHeaders`（1967行）が `req.Header` をディープコピーした上で `SetBasicAuth(id, secret)` を呼び、`UnauthenticatedRateLimitedTransport.RoundTrip`（2016行）と `BasicAuthTransport.RoundTrip`（2057行）がリクエスト毎に無条件でこれを適用する構造にある。ポイントは Go 標準ライブラリのリダイレクト時ヘッダ保護との相互作用にある。`net/http.Client` は cross-domain リダイレクトの際、機微ヘッダ（`Authorization`/`Www-Authenticate`/`Cookie`/`Cookie2`）を剥がす保護（`shouldCopyHeaderOnRedirect`、Go 1.8+）を持つが、それが対象とするのは**元リクエスト（`Client.Do` に渡された `*http.Request`）に載っていたヘッダ**だけである。本件では認証情報が `RoundTrip` の**内側**、すなわち `Client` がリダイレクト時のヘッダコピー判定を終えた**後**に注入される。したがって標準ライブラリの剥がし処理はこれらの資格情報を一度も観測せず、トランスポートが各ホップ（cross-origin リダイレクトのホップを含む）で新鮮に付け直してしまう。結果として、`api.github.com` 等の信頼ホストが返した 302 を辿って別オリジン（攻撃者サーバ）へ飛ぶと、username/password あるいは OAuth の `ClientID:ClientSecret` がその別オリジンに漏れる。

**解決方針**: PR #4364 は、リダイレクト由来のリクエストがオリジンを跨ぐ場合に限って資格情報の付与をスキップし、素の下位トランスポートへそのまま転送する。判定は新規ヘルパ `isCrossOriginRedirect(req)` が担い、`net/http` がリダイレクト追跡時にセットする `req.Response`（＝リダイレクトを返したレスポンス）と `req.Response.Request.URL`（＝そのレスポンスを受け取った直前リクエストの URL）を利用する。初回リクエストでは `req.Response == nil` なので「cross-origin ではない」と判定し従来どおり資格情報を付与、リダイレクト追跡時のみ直前オリジンと今回のオリジンを比較する。オリジン比較 `sameRedirectOrigin` はスキーム・ホスト名を `strings.EqualFold`（大文字小文字非依存）で、ポートを `defaultPort`（明示ポートが無ければ http→80 / https→443 に正規化）で突き合わせる。cross-origin なら `t.transport().RoundTrip(req)` に素通しし、`BasicAuthTransport` 側では `OTP` ヘッダも付与しない。両 `RoundTrip` の先頭に3行のガードを挿入するのみの最小変更で、`+38` 行（本体）＋テスト `+242` 行。

**関連ファイル**: `github/github.go`（`BasicAuthTransport.RoundTrip`:2057 / `UnauthenticatedRateLimitedTransport.RoundTrip`:2016 / `setCredentialsAsHeaders`:1967 / 新規 `isCrossOriginRedirect`・`sameRedirectOrigin`・`defaultPort`）, `github/github_test.go`（各ヘルパの表駆動テスト＋2トランスポートの cross-origin リダイレクト回帰テスト）

**現状**: 報告者本人が同時に修正 PR #4364（`fix: Avoid auth on cross-origin redirects`）を提出済み。経緯は、alexandear が「先に Issue を立てて問題を確認してからレビューを進めたい」とコメント → 報告者が本 Issue #4365 を起票、という順。レビュー状況は複雑で、gmlewis は一度 APPROVED（「#4363 と同じコメントで LGTM。他コントリビューターの2人目の LGTM+Approval を待つ」）を出したものの、その後 `defaultPort` 追加でカバレッジが下がった点を CHANGES_REQUESTED（「この関数は本当に必要か？残すなら専用ユニットテストを付けよ」）→ 報告者が `TestDefaultPort` を追加（commit `c5be7ac`）しカバレッジ 100%（`isCrossOriginRedirect`/`sameRedirectOrigin`/`defaultPort` とも）に回復。さらに stevehipwell が CHANGES_REQUESTED で「#4363 と同じ懸念。急ぎの PR ではなく、実際のコンテキストを巡るより広い議論が必要」と指摘。これを受け報告者は 2026-07-06 に「同意する。トランスポート挙動を変える前に #4363 と同じ広い議論に属する。実装変更は一旦保留し、#4365 で期待されるリダイレクト/認証境界を先に明確化する」と表明し、実装作業を一時停止している。CI は全緑（check-changes/check-generated/check-openapi/golangci-lint/test(stable・oldstable, ubuntu・windows)/scan-pr/cla/google/codecov すべて SUCCESS、カバレッジ 97.51% 維持）、`mergeable=MERGEABLE`、`reviewDecision=CHANGES_REQUESTED`。ラベルは付与なし（内容上のセキュリティ分類は本レポートの判断）。姉妹案件として #4363（`fix: Limit token auth to configured hosts`, Fixes #4366）が併走しており、こちらは `WithAuthToken` のトークンが絶対 URL 経由の cross-host リクエストへ漏れる同系統の問題。#4364/#4365 と #4363/#4366 は「リダイレクト／別ホストにおける認証境界をライブラリがどこまで責務として扱うか」という同一の設計論として束ねる方向で合意されつつある。

**【2026-08-01 更新】設計合意が成立し、PR は作り直された。** #4366 上での議論（詳細は #4366 のエントリ参照）で gmlewis が「非設定ホストにトークンを届かせないのは explicit goal であり後方互換は NON-GOAL」と明確化したのを受け、**#4364 は `fix!: Scope auth transports to allowed origins` に改題され破壊的変更（`fix!:`）として再構成された**。実装も hop 単位（`isCrossOriginRedirect`＝直前ホップとの比較）から**オリジン単位**へ転換し、`BasicAuthTransport` と `UnauthenticatedRateLimitedTransport` の**両方に公開フィールド `AllowedOrigins []*url.URL` を追加**、判定は新ヘルパ `isAllowedAuthOrigin(u, allowedOrigins)` が担い、**未設定（空）の場合は `defaultAuthOrigins`（dotcom オリジン）にフォールバック**する設計になった。これは旧実装の穴（`req.Response == nil` の直接リクエストではガードが発火せず、絶対 URL 指定で資格情報が漏れる）を塞ぐもの。レビューは **gmlewis が 2026-07-19 に APPROVED**、一方 **stevehipwell は 2026-07-22 に再び CHANGES_REQUESTED**（「以前のコメントのとおり、きちんとした議論なしにこの PR を取り込むべきではない」）＝技術面ではなく**プロセス面**での保留。

**推奨アクション**: このユーザー（当フォーク）が新規に実装へ着手する必要はない。修正 PR #4364 が既に存在し、コード自体は最小・クリーンで CI 全緑・カバレッジ 100%、gmlewis の LGTM も一度は付いている。ボトルネックはコードではなく設計方針の合意で、(1) 標準 `net/http` が既に元リクエストのヘッダを cross-domain リダイレクトで剥がす以上、トランスポート注入分まで剥がす責務をライブラリが負うべきか、(2)「オリジン境界」の定義（スキーム/ホスト/ポートで十分か、GHES のサブドメイン間や upload host との関係をどう扱うか）、(3) #4363（トークン auth の cross-host 制限）と統一した挙動・API 面にするか、をメンテナ（gmlewis/stevehipwell/alexandear）が先に固める必要がある。次の一手は、報告者の宣言どおり #4365 上でこの境界仕様を確定させ、その結論に沿って #4364 を更新（もしくはクローズ）すること。当フォークとしては #4363/#4366 と併せてセキュリティ系の議論を追跡し、方針確定後にレビュー協力する程度で十分。重複 PR の新規作成は避ける。Issue は #4364 マージ時に自動クローズ（PR 本文に `Fixes #4365`）。

**関連 PR/リンク**: #4364（修正 PR・OPEN・CHANGES_REQUESTED・保留中）, #4363 / #4366（姉妹案件: `WithAuthToken` の cross-host トークン漏洩）

**【2026-10-10 更新】** @gmlewis が 09-17 に方針を示し、#4564（`fix!: Send credentials only to configured origins`）を 09-18 にマージして本 Issue をクローズした。これに対し @stevehipwell が「レビューの時間がないまま押し通された」「redirect/認証の仕様整理が先」と反論した（09-18）。@gmlewis は謝罪し、同日 13:44 に議論継続のため再オープンした。関連して、redirect 周りの open PR が 2 件ある: #4585（@sb123sb123、redirect 時に `http.Client.Timeout` を効かせる）と #4605（@DorianChn、相対 Location の解決）。

---

### Issue [#4408](https://github.com/google/go-github/issues/4408) — Add additional functionality for enterprise budgets

分類: **新API対応** / 難易度: **易** / 対応区分: **🔀 対応PRあり（#4565・ただし既存 `GetBudget` と重複）**

**概要**: 報告者 wtneff から、enterprise budgets API について2点の指摘。(1) 「Get all budgets」エンドポイントはページネーションをサポートしているのに `ListBudgets` にその機能がない、(2) 「Get a budget by ID」エンドポイントに対応する関数が存在しない。「自分で実装するつもり」と表明している。

**根本原因**: **実コードで検証したところ、2つの主張のうち成立するのは (1) のみ。**
- (1) **成立**: `github/enterprise_budgets.go:95` の `ListBudgets(ctx context.Context, enterprise string)` には `opts *ListOptions` 引数が無く、`addOptions` も呼んでいない。`github-iterators.go` にも `ListBudgets` 用のイテレータは生成されていないため、ページネーションの手段が一切ない。
- (2) **不成立（事実誤認）**: `github/enterprise_budgets.go:139` に `GetBudget(ctx, enterprise, budgetID string)` が既に存在し、`//meta:operation GET /enterprises/{enterprise}/settings/billing/budgets/{budget_id}`・doc アンカー `#get-a-budget-by-id` と、報告者が挙げたエンドポイントそのものに対応している。同ファイルの履歴を確認すると初出は #4069（2026-03-11）で、**Issue 起票（2026-07-24）の4ヶ月以上前から実装済み**。報告者が見落としたと考えられる。

**解決方針**: (1) のみ対応すればよい。`ListBudgets` のシグネチャに `opts *ListOptions` を追加し、`addOptions(u, opts)` を通す（go-github の List 系メソッドの定型）。破壊的変更になるためコミットは `feat!:` 相当の扱いとし、併せて `gen-iterators.go` の再生成でイテレータが生えるかを確認する。(2) は「既に `GetBudget` が存在する」旨を Issue にコメントすれば足りる。

**関連ファイル**: `github/enterprise_budgets.go`（`ListBudgets`:95 / `GetBudget`:139）, `github/github-iterators.go`

**現状**: 2026-07-24 起票（wtneff）・**ラベルなし・コメント0件**・OPEN。報告者が「自分で実装する」と明言しているが、起票から日が経っても PR は出ていない（本更新時点で enterprise budgets 関連の Open PR は無し）。メンテナからの反応もまだない。

**推奨アクション**: 当フォークが横取りする形での実装着手は避けるべき（報告者が実装表明済み）。**最も価値のある一手は、(2) が事実誤認である点を根拠付きでコメントすること**＝`GetBudget` が `enterprise_budgets.go:139` に既に存在し #4069 で追加済みだと示せば、報告者の作業スコープが (1) のページネーション追加だけに絞られ、無駄な重複実装を防げる。当フォークの #3000 トリアージと同じ「実コード照合でスコープを正す」パターン。実装自体は `opts *ListOptions` 追加のみで易しいので、報告者が動かない場合は改めて着手を検討する余地がある。**【2026-08-29 更新】** 起票から1ヶ月経過してもコメント0件・対応 PR 無しの状態が継続（報告者は動いていない）。

**【2026-10-10 更新】** #4501（@devaniketh、`ListBudgets` のページネーションと `GetUserStatesForBudget`）が 09-04 にマージされて一度クローズされた。しかし @wtneff が `GetBudgetByID` が未実装だと指摘し、@gmlewis が同日再オープンした。@wtneff は #4563 を出したあと、author 履歴を整理した #4565 に出し直した（09-17）。#4565 は CLA 未署名で、`NeedsReview` と `waiting for signed CLA` のラベルが付いている。
なお #4565 の `GetBudgetByID` は、master の `EnterpriseService.GetBudget`（`github/enterprise_budgets.go:212`・同一 endpoint／anchor `#get-a-budget-by-id`）と重複する（grep で確認済み）。前回（08-01）指摘した「起票者の事実誤認」が PR 段階まで持ち越されている。

---

### Issue [#4457](https://github.com/google/go-github/issues/4457) — Including an enterprise team as a reviewer for an environment results an unmarshal error

分類: **バグ** / 難易度: **易** / 対応区分: **🔀 対応PRあり（#4456）**

**概要**: Enterprise Team を environment の deployment reviewer に設定すると、API が返す reviewer の `type` が `Team` ではなく **`BusinessTeam`** になるため、go-github の reviewer type 判定が未知の型として unmarshal エラーになるバグ報告。起票者 JWilkinsonMB 自身が修正 PR #4456 を同時提出している（本文に「Potential fix in #4456」）。

**現状**: PR [#4456](https://github.com/google/go-github/pull/4456)（+18/-1・2ファイル）が `BusinessTeam` を通常の `Team` として扱う形で修正。レビューは alexandear の APPROVED のみで reviewDecision は未成立＝メンテナ（gmlewis）のレビュー待ち。Issue 側はコメント0件・ラベルなし。

**推奨アクション**: 当フォークの着手は不要（作者自身の修正 PR が存在）。実バグの unmarshal エラーなので優先度は高め。マージを見守る。

**【2026-10-10 更新】** 変化なし。対応 PR #4456 は CLA 待ちで 08-28 以降停滞。

---

### Issue [#4520](https://github.com/google/go-github/issues/4520) — Repository ruleset rule type `code_quality` is not modelled

分類: **新API対応** / 難易度: **易** / 対応区分: **🔀 対応PRあり（#4523）**

**概要**: ブランチ ruleset の「Require code quality results」（2026-07-20 GA）に対応する rule type `code_quality`（パラメータ `severity`）が `RepositoryRulesetRules` / `BranchRules` に無く、PUT で ruleset を往復させると当該ルールが黙って消える（terraform-provider-github で実害）。master の `github/rules.go` に `code_quality` / `CodeQuality` が存在しないことを grep で確認済み（`github/code_quality.go` の `CodeQualityFinding*` は別機能）。

**現状**: gmlewis が「`errors` しか観測されていないなら `severity` は plain `string` で良い」と回答。Issue は起票者 @etifontaine にアサイン済み。非アサインの @tvost2 が #4523 を提出したが、gmlewis は「アサイン者の応答を待つ」と保留。

**推奨アクション**: 当フォークは関与不要。#4523（非アサイン・typed severity・CONFLICTING）と起票者本人の PR のどちらが採られるかを見守るだけで良い。

---

### Issue [#4524](https://github.com/google/go-github/issues/4524) — Add Copilot custom agents enterprise endpoints

分類: **新API対応** / 難易度: **易** / 対応区分: **🔀 対応PRあり（#4525）**

**概要**: Enterprise の Copilot custom agents API（`GET .../copilot/custom-agents`、`GET/PUT/DELETE .../copilot/custom-agents/source`）の 4 endpoint が未対応。master の `github/` に `custom-agents` / `CustomAgent` は存在しない（grep 確認済み）。

**現状**: 起票者 @Tens1des にアサイン済みで、同日に #4525 を提出。gmlewis APPROVED、2 人目の承認待ち。

**推奨アクション**: 当フォークは関与不要。#4525 のマージ待ち。

---

### Issue [#4526](https://github.com/google/go-github/issues/4526) — Add Copilot content exclusion set/get enterprise and org set endpoints

分類: **新API対応** / 難易度: **易** / 対応区分: **🔀 対応PRあり（#4527）**

**概要**: Copilot content exclusion API のうち、master にあるのは `GetOrganizationContentExclusionDetails`（`GET /orgs/{org}/copilot/content_exclusion`）だけで、enterprise の GET/PUT と org の PUT が無い（`github/copilot.go:622` 周辺を grep で確認済み）。

**現状**: 起票者 @Tens1des にアサイン済み。#4527 は alexandear の CHANGES_REQUESTED（09-08）に未対応のまま約 1 か月止まっている。

**推奨アクション**: 当フォークは関与不要。#4527 の作者が alexandear の指摘に対応するのを待つ。

---

### Issue [#4584](https://github.com/google/go-github/issues/4584) — http.Client.Timeout is ignored by redirect-aware requests

分類: **バグ** / 難易度: **易** / 対応区分: **🔀 対応PRあり（#4585）**

**概要**: `roundTripWithOptionalFollowRedirect`（`github/github.go:2380`）は `c.client.Transport.RoundTrip(req)` を直接呼び、`http.Client.Do` を経由しない。そのため `WithTimeout` / `http.Client.Timeout` がワークフローログ、artifact、archive link などのダウンロードで効かない。コードは master で読んで確認したが、再現テストは実行していない（未検証）。

**現状**: 未アサイン。修正 PR #4585 で gmlewis は Bug の存在を認めた（「the missing timeout needs to be addressed」）うえで、修正方法として ctx に `context.WithTimeout` を被せる方式を指示し、作者はその方式で修正済み。ただし CLA 未署名で、コミット作者のメールアドレスも 2 種類混在しているため止まっている。

**推奨アクション**: 当フォークは当面待機。#4585 が CLA 問題で放棄された場合は、gmlewis 指定の修正（数行）と synctest のテストで引き取る余地がある（Issue で引取りを申し出てから着手する）。

---

## PR

---

## 4. Open PR 詳細（OPEN 13件・番号順）

> マージ/クローズ済みの PR は掲載対象外（追跡不要のため）。**当フォーク提出の Open PR は無し**。前回更新以降、当フォークの #4493/#4519/#4540/#4548/#4606/#4607（いずれも→#3644）はマージ済み、#4375 はクローズ（#4576 `tools/schemafields` に置換）。他者の #4363/#4364 は #4564 に置き換えられてクローズ、#4379（Copilot Spaces org CRUD）・#4436（stacked PR）・#4488/#4490（Copilot metrics）・#4491（`MergeAsync`）・#4551（jvm986・#3644）はマージ済み。

| PR | 作者 | 状況 | CI | 関連Issue | 概要 |
| --- | --- | --- | --- | --- | --- |
| [#4456](https://github.com/google/go-github/pull/4456) | @JWilkinsonMB | CLA 待ち（08-28 以降停滞） | ❌ cla/google のみ | #4457 | `BusinessTeam` を Environment reviewer の `Team` として扱う |
| [#4461](https://github.com/google/go-github/pull/4461) | @aoright | CLA 待ち（停滞） | ❌ cla/google | — | doc コメントの文言・句読点修正 |
| [#4482](https://github.com/google/go-github/pull/4482) | @ralucas | 作者応答待ち（alexandear CHANGES_REQUESTED 未対応・08-28 以降停滞） | ✅ | — | org rule-suites API 追加 |
| [#4492](https://github.com/google/go-github/pull/4492) | @n-g | CLA 待ち（08-28 以降停滞） | ❌ cla/google・codecov | — | 文字列値のエラーレスポンスを処理 |
| [#4523](https://github.com/google/go-github/pull/4523) | @tvost2 | OPEN・CONFLICTING・メンテナ保留（アサイン者別） | codecov/patch ❌（他は ✅） | #4520 | ruleset `code_quality` rule を追加（typed severity） |
| [#4525](https://github.com/google/go-github/pull/4525) | @Tens1des | OPEN・gmlewis APPROVED・2 人目待ち | ✅ | #4524 | Copilot custom agents enterprise 4 endpoint（body 値渡し） |
| [#4527](https://github.com/google/go-github/pull/4527) | @Tens1des | OPEN・alexandear CHANGES_REQUESTED 未対応 | ✅ | #4526 | Copilot content exclusion の set と enterprise get/set（型 rename の BREAKING） |
| [#4550](https://github.com/google/go-github/pull/4550) | @keeltrace | OPEN・CLA 作者混在で停滞 | cla ❌ | #4077（Part of） | `Attestation.BundleURL` 追加と `Bundle` の Deprecated 化 |
| [#4565](https://github.com/google/go-github/pull/4565) | @wtneff | OPEN・Draft・企業 CCLA で停滞 | cla ❌ | #4408 | `GetBudgetByID` 追加（既存の `GetBudget` と完全に重複） |
| [#4577](https://github.com/google/go-github/pull/4577) | @sundaypaulfz3-cell | OPEN・レビューなし | lint ❌・cla ❌ | なし | 長い installation token のテストのみ（#4598 と重複） |
| [#4585](https://github.com/google/go-github/pull/4585) | @sb123sb123 | OPEN・CHANGES_REQUESTED（対応済み）・CLA 停滞 | lint ❌・cla ❌ | #4584 | redirect 系リクエストに `http.Client.Timeout` を適用 |
| [#4598](https://github.com/google/go-github/pull/4598) | @rajveer002 | OPEN・lint 修正待ち | lint ❌ | なし | stateless token 形状のテストのみ（#4577 と重複） |
| [#4605](https://github.com/google/go-github/pull/4605) | @DorianChn | OPEN・メンテナが必要性を質問中 | codecov/patch ❌ | なし | 相対 redirect Location をリクエスト URL 基準で解決 |

### PR [#4456](https://github.com/google/go-github/pull/4456) — fix: Treat `BusinessTeam` as a regular `Team` for Environment reviewers

作者: **@JWilkinsonMB** / 状況: **OPEN・alexandear APPROVED のみ（reviewDecision 未成立）** / 関連Issue: #4457（同一作者の自己修正） / 規模: 2ファイル +18/-1

**概要**: environment の reviewer に Enterprise Team を設定すると API の reviewer `type` が `BusinessTeam` になり unmarshal エラーになるバグ（#4457）の修正。`BusinessTeam` を `Team` と同様に扱う小さな変更。

**推奨アクション**: 実バグ修正で規模も小さく、メンテナ（gmlewis）のレビューが付けば早期マージが見込める。当フォークからのアクションは不要。

**【2026-10-10 更新】** 08-28 の @JWilkinsonMB のコメント（「CLA は社内で確認中」）以降、動きはない。`cla/google` のみ FAILURE。

---

### PR [#4461](https://github.com/google/go-github/pull/4461) — docs: Fix phrasing and punctuation in doc comments

作者: **@aoright** / 状況: **OPEN・alexandear APPROVED のみ（reviewDecision 未成立）** / 関連Issue: なし / 規模: 1ファイル +3/-3

**概要**: `github/github.go` の doc コメント微修正（`on-premise`→`on-premises`・`Client` フィールドコメントの末尾ピリオド追加）のみの docs PR。

**推奨アクション**: 当フォークからのアクションは不要。trivial なのでメンテナが気づけば即マージされる類。

**【2026-10-10 更新】** 変化なし（最終更新 08-20）。

---

### PR [#4482](https://github.com/google/go-github/pull/4482) — feat: Add org rule-suites APIs

作者: **@ralucas** / 状況: **OPEN・gmlewis APPROVED（alexandear の CHANGES_REQUESTED は decision に算入されず）** / 関連Issue: なし（直接の起票 Issue なし） / 規模: 6ファイル +929/-0

**概要**: org レベルの rule suites 2エンドポイント（`GET /orgs/{org}/rulesets/rule-suites`・`GET .../rule-suites/{rule_suite_id}`）を追加。repo レベルの既存実装の org 版。

**推奨アクション**: 当フォークからのアクションは不要。alexandear のインライン指摘が残ったまま decision は APPROVED 表示なので、マージ前に対応されるかだけ観察。

**【2026-10-10 更新】** @alexandear の CHANGES_REQUESTED（08-28）と `waiting for reply` ラベル（08-28 @gmlewis）以降、作者からの push もコメントもない。reviewDecision は APPROVED と表示されるが、これは @gmlewis の 08-25 APPROVED だけが算入されているため。

---

### PR [#4492](https://github.com/google/go-github/pull/4492) — fix: Handle string-valued error responses

作者: **@n-g** / 状況: **OPEN・レビュー未着手（reviews 0件）** / 関連Issue: なし / 規模: 2ファイル +55/-0

**概要**: GitHub がエラーレスポンスの `errors` を「オブジェクト配列」でなく**単なる文字列**で返すケースがあり、現行の unmarshal が対応できない問題への修正。`ErrorResponse` というコアのエラーハンドリング挙動に触れるため、レビューは慎重になる可能性がある。

**推奨アクション**: 当フォークからのアクションは不要。`Client.Do` 周辺（#4494 で当フォークが触った領域）に隣接するため、マージされたら挙動変化を把握しておく。

**【2026-10-10 更新】** @gmlewis が 08-28 に「CLA が解決したら進める」とコメントして以降、動きはない。`cla/google`・`codecov/patch`・`codecov/project` が FAILURE。

---

### PR [#4523](https://github.com/google/go-github/pull/4523) — feat: Add repository ruleset code quality support

作者: **@tvost2** / 状況: **OPEN・レビューなし・CONFLICTING・メンテナ保留（Issue のアサイン者は別人）** / 関連Issue: #4520 / 規模: 4ファイル +147/-0

**概要**: `RulesetRuleTypeCodeQuality`、`CodeQualityRuleParameters` / `CodeQualityBranchRule`、型付きの `CodeQualitySeverity`（`errors` / `errors_and_warnings` 定数）を追加し、marshal/unmarshal を実装している。gmlewis は「`severity` は plain `string` で良い」と言っているが、この PR は型付きで、`errors_and_warnings` は Issue で観測されていない値。gmlewis は「#4520 は別の人にアサイン済みなので、その人の応答を待つ。着手前にサインアップを」とコメントしている。

**推奨アクション**: 当フォークは関与不要。放置またはクローズされる見込み。

---

### PR [#4525](https://github.com/google/go-github/pull/4525) — feat: Add Copilot custom agents enterprise endpoints

作者: **@Tens1des** / 状況: **OPEN・gmlewis APPROVED・2 人目待ち・MERGEABLE・CI 緑** / 関連Issue: #4524 / 規模: 6ファイル +862/-0

**概要**: Enterprise Copilot custom agents の 4 endpoint を新ファイル `github/copilot_custom_agents.go` に追加し、iterator も生成している。リクエスト型 `SetCopilotCustomAgentsSourceRequest` は**値渡し**で、required の `OrganizationID` は非ポインタの `int64` になっており、#3644 の方針に沿っている（diff 確認済み）。alexandear は戻り値型を `EnterpriseCustomAgents` から `CopilotEnterpriseCustomAgents` に変える suggestion を出したが、未反映のまま 09-07 以降コミットがない。

**推奨アクション**: 当フォークは関与不要。ポインタ渡しの新しい body 型は追加しないので、#3644 の許可リストへの影響はない。

---

### PR [#4527](https://github.com/google/go-github/pull/4527) — feat!: Add Copilot content exclusion set and enterprise endpoints

作者: **@Tens1des** / 状況: **OPEN・gmlewis APPROVED＋alexandear CHANGES_REQUESTED（未対応・約 1 か月停滞）・MERGEABLE・CI 緑** / 関連Issue: #4526 / 規模: 4ファイル +226/-7

**概要**: org の PUT と enterprise の GET/PUT を追加し、`CopilotOrganizationContentExclusionDetails` を `CopilotContentExclusionDetails` に rename する BREAKING 変更を含む（gmlewis が型エイリアスを残さず rename するよう求めた結果）。body は map 型の値渡し。alexandear は、不要行の削除、専用リクエスト型 `CopilotSetContentExclusionDetailsRequest` とレスポンス型 `CopilotSetContentExclusionResponse` への改名、doc の `NOTE Breaking API change` 削除（breaking 情報は PR 説明に書くのが慣例）を求めている。

**推奨アクション**: 当フォークは関与不要。body は値渡しなので #3644 には抵触しない。

---

### PR [#4550](https://github.com/google/go-github/pull/4550) — feat: Support attestation bundle URLs

作者: **@keeltrace** / 状況: **OPEN・レビュー承認なし・MERGEABLE・CLA 失敗（コミット作者 2 種混在）** / 関連Issue: #4077（Part of。Closes なし） / 規模: 5ファイル +55/-4

**概要**: API version 2026-03-10 で attestation が `bundle` の埋め込みではなく `bundle_url` を返すようになったため、`Attestation.BundleURL *string` を追加して `Bundle` を Deprecated にする。あわせて `example/verifyartifact` を URL 取得方式に変え、旧形式へのフォールバックも残している。master の `Attestation` に `BundleURL` が無いことは確認済み（`github/attestations.go`）。lint 指摘には対応済みだが、5f60e58e の作者が `@218190424`（CLA 未署名の別 identity）になっており、gmlewis が force-push での作り直しか PR の作り直しを求めている。

**推奨アクション**: 当フォークは関与不要。#4077 の移行チェックリストのうち 1 項目に当たる。

---

### PR [#4565](https://github.com/google/go-github/pull/4565) — feat: Add `GetBudgetByID` function for enterprise budgets

作者: **@wtneff** / 状況: **OPEN・Draft・レビューなし・CLA 失敗（企業 CCLA の処理が止まっている）** / 関連Issue: #4408 / 規模: 2ファイル +70/-0

**概要**: `GET /enterprises/{enterprise}/settings/billing/budgets/{budget_id}` 用に `EnterpriseService.GetBudgetByID` を追加する PR。ただし master には同じ endpoint・同じ docs anchor（`#get-a-budget-by-id`）・同じ実装の `GetBudget` が既に存在する（`github/enterprise_budgets.go:207-228`、#4069 で 2026-03-11 に追加）。この PR は**完全に重複するメソッド**を足すことになる（diff と master を突き合わせて確認済み）。CLA は会社の CCLA が Google 側で処理されず、gmlewis も打つ手なしの状態。

**推奨アクション**: 当フォークは関与不要。ただし「既存の `GetBudget` と重複している」とメンテナや作者が気づいていない可能性がある。指摘する場合は反論系ではなく事実指摘のコメントになるので、ユーザーに確認してから投稿する。

---

### PR [#4577](https://github.com/google/go-github/pull/4577) — test: Support stateless GitHub App installation tokens

作者: **@sundaypaulfz3-cell** / 状況: **OPEN・レビューなし・golangci-lint 失敗・CLA 失敗** / 関連Issue: なし / 規模: 2ファイル +44/-0

**概要**: `ghs_` に 256 文字を足した長いトークンについて、`CreateInstallationToken` のデシリアライズと `WithAuthToken` の Bearer ヘッダー送出が切り詰めなしで行われることを確かめるテストだけを足す。プロダクションコードは変更しない。#4598 とほぼ同じ趣旨の重複 PR。

**推奨アクション**: 当フォークは関与不要。放置またはクローズされる見込み。

---

### PR [#4585](https://github.com/google/go-github/pull/4585) — fix: Honor HTTP client timeouts in redirects

作者: **@sb123sb123** / 状況: **OPEN・gmlewis CHANGES_REQUESTED（指摘には対応済み・再レビュー前）・golangci-lint 失敗・CLA 失敗（作者メール 2 種）** / 関連Issue: #4584 / 規模: 2ファイル +57/-0

**概要**: 当初は `http.Client.Do` を経由させる案だったが、gmlewis が 3 つの副作用（`*url.Error` によるラップ、CookieJar の付与、Location のパースが `checkRedirectHost` より先に走る）を指摘した。現在は指示どおり、`roundTripWithOptionalFollowRedirect` の冒頭で `c.client.Timeout > 0` のときに `context.WithTimeout` を被せる 5 行の修正と、`github_test.go` に synctest のテストを置く形になっている。CLA が通らない限り先に進めない。

**推奨アクション**: 当フォークは待機。CLA 問題で放棄された場合の引取り候補（Issue #4584 の推奨アクションを参照）。

---

### PR [#4598](https://github.com/google/go-github/pull/4598) — test: Cover stateless GitHub App token format

作者: **@rajveer002** / 状況: **OPEN・レビューなし・golangci-lint 失敗（gmlewis が修正を依頼済み）・CLA 通過** / 関連Issue: なし / 規模: 1ファイル +26/-0

**概要**: `ghs_` に `.` と `_` で区切った長いセグメントを続けた「stateless token 形状」で、`CreateInstallationToken` がトークンを変えずに保持することを確かめるテストだけを足す。#4577 とほぼ重複する。PR 本文は「既存の長トークンテストがある」と書いているが、master の `github/*_test.go` に long/stateless トークンのテストは無い（grep 確認済み）。

**推奨アクション**: 当フォークは関与不要。

---

### PR [#4605](https://github.com/google/go-github/pull/4605) — fix: Resolve relative redirect locations

作者: **@DorianChn** / 状況: **OPEN・レビュー承認なし・MERGEABLE・codecov/patch 失敗（1 行未カバー）・メンテナが必要性に懐疑的** / 関連Issue: なし / 規模: 7ファイル +144/-24

**概要**: 相対 `Location` を `c.baseURL` ではなく、レスポンスを返したリクエストの URL を基準に解決するよう変更する（`resp.Location()` の利用、`checkRedirectHost` を `resolveAndCheckRedirect` に置き換えてパストラバーサル検査を追加）。対象は 301 の追従と artifact / workflow log / archive のダウンロードリンク。gmlewis は「実際に観測された問題か？GitHub が相対 URL を返すことがあるのか？」と問い返しており、作者は回答していない。

**推奨アクション**: 当フォークは関与不要。仮説上の問題として見送られる可能性が高い。

---

## 5. 付録: コントリビューション規約メモ

- **CLA 必須**: Google CLA への署名が必要（未署名の PR は CLA チェックでブロックされる）。
- **コミット/PR タイトル**: 命令形・先頭大文字・末尾ピリオドなし・50字以内。`feat:`/`fix:`/`docs:`/`refactor:` 等の接頭辞、破壊的変更は `!`（例 `refactor!:`）。
- **テスト必須**: codecov がカバレッジ低下を検知。バグ修正は再現テストを同梱。
- **コード生成**: 新 API メソッドは `//meta:operation` ディレクティブ付きで追加し、`script/generate.sh` で生成物を更新。`openapi_operations.yaml` は手編集しない。
- **検証スクリプト**: `script/fmt.sh`（整形）/ `script/test.sh`（テスト）/ `script/lint.sh`（lint・生成物チェック）。
- **ファイル構成**: `github/{service}_{api}.go`。GitHub REST API ドキュメントの構成に対応。
- **force-push 禁止**: レビュー差分の追跡のため。マージは squash-merge。
- **Windows 注意**: Git Bash/WSL 推奨。`core.autocrlf=false` / `core.eol=lf`。

## 6. 調査メソドロジー

**初回調査（2026-06-20）**: 1 Issue / 1 PR ごとに調査エージェントを並列展開（計43エージェント）。各エージェントは
`gh issue view --comments` / `gh pr view --comments` / `gh pr diff` で本文・全コメント・差分を取得し、
`github/` 配下の該当コードを Read/Grep で実地確認した上で、主張（例「フィールドが常に nil」「エンドポイントが消えた」）の
真偽を検証して構造化出力した結果を集約しています。`file:line` 参照は調査時点の実コードに基づきます。

**更新（reconcile）の運用**: 更新時は `gh issue list --state open` / `gh pr list --state open` の実データと突き合わせ、
**クローズ済み Issue・マージ/クローズ済み PR は削除**（active な OPEN のみ掲載）、新規分は初回と同じ粒度で追記します。
既存エントリの分析本文は履歴として残し、進展は「**【YYYY-MM-DD 更新】**」を付けて追記する方針（上書きしない）。
新規エントリでは**報告内容を実コードで検証**するのが要点で、実際に 2026-08-01 の更新では
#4408 の「Get a budget by ID に対応する関数が無い」という主張が、`github/enterprise_budgets.go:139` の `GetBudget`
（#4069 / 2026-03-11 追加＝起票の4ヶ月以上前から存在）により**事実誤認**であると判明しています。

**更新履歴**: 2026-06-20 初版（Issue 37 / PR 6）→ 2026-07-03（Issue 20 / PR 4）→ 2026-07-04（Issue 21）→ 2026-07-08（Issue 23 / PR 6）→ 2026-08-01（Issue 22 / PR 7）→ 2026-08-29（Issue 25 / PR 13）→ **2026-10-10（Issue 24 / PR 13）**。

