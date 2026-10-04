# e2e-coverage-architect スキル設計書

- 日付: 2026-10-04
- 状態: 設計承認済み(本書レビュー待ち)
- 配信形態: GitHub public repository / `npx skills add` でインストール可能

## 1. 目的

既存の Web アプリケーションプロジェクトを解析し、ユーザー行動・権限・状態遷移・異常系を洗い出して behavior coverage(実際のユーザー行動がどれだけ E2E テストで検証されているか)をマトリクス化する。既存テストとの重複・不足・false green を判定し、重要度順に Playwright テストを実装・実行・修正し、meaningful な gap がなくなるまで再分析を繰り返す。

一文定義:

> Analyze the existing application, identify meaningful E2E behavior coverage gaps, and systematically implement and verify Playwright tests until critical gaps are exhausted.

完了判定は behavior coverage による。テスト数・route 数は完了基準に使わない。

## 2. 前提・用語・不変条件

### 前提

- 対象はブラウザで動く Web アプリ(frontend + API + 何らかの永続化)である前提で解析する。非 Web プロジェクトと判定された場合は説明して中断する。
- Playwright が未導入のプロジェクトでは、スキルがテスト基盤の bootstrap(devDependencies 追加、`playwright.config.ts`、`tests/e2e/` ディレクトリ、必要な `.gitignore` エントリ)まで行う。
- 言語ポリシー: スキル本体(SKILL.md、references/、README.md)は英語。本設計書は日本語。

### 用語

| 用語 | 定義 |
|---|---|
| behavior coverage | 「ユーザー行動 × 検証観点」のセルが実際のテスト assertions で保証されている割合。route 数やテスト数ではない |
| meaningful gap | ユーザーに見える挙動・データ整合性・セキュリティに影響する未カバーセル。P0/P1/P2 に分類(§6) |
| false green | 緑判定になっているのに実際には挙動を検証できていないテスト(§7 のチェックリストで判定) |
| known-failure | スキルが発見したアプリ本体のバグ。アプリは修正せず findings.md に記録し、対応テストは `test.fixme` で進行する |
| waived | 実装しないことにしたセル。理由を matrix.md に明記する |

### 不変条件(最重要)

**スキルはアプリ本体のソースコードを一切変更しない。** 変更してよいのは (a) テストコードとテスト設定、(b) `docs/e2e-coverage/` 配下の成果物、(c) Playwright bootstrap によるテスト基盤ファイルのみ。新テストの実行でアプリのバグが判明した場合は known-failure として記録するだけで、アプリを直したりユーザーに修正を促して勝手に先へ進んだりしない。

## 3. 配信構成

このリポジトリ自体が配信物である。

```
e2e-coverage-architect/          ← GitHub リポジトリ root
  e2e-coverage-architect/
    SKILL.md                     # スキル本体(簡潔に保つ)
    references/
      inventory.md               # Phase 0: ルート/API/認証/DB/既存テストの調査手順
      behavior-matrix.md         # Phase 1-2: 行動列挙の方法とマトリクスの型
      gap-analysis.md            # Phase 2: 重複・不足・false green 判定チェックリスト
      implementation.md          # Phase 3: Playwright 実装規約と優先順位ルール
      loop-termination.md        # Phase 4-5: 再分析手順と完了基準
  README.md                      # 使い方・要件・インストールコマンド
  docs/superpowers/specs/        # 本設計書
  .gitignore                     # .claude/ と eval/ は除外
```

- `npx skills add` の規約: スキル名は SKILL.md の frontmatter(`name`, `description`)で決まる。追加のメタデータ・ロックファイルは不要。
- インストール時にスキル本体だけがコピーされるよう、スキルは入れ子ディレクトリ `e2e-coverage-architect/` に置く。実装時にローカルで `npx skills add .` を実行して発見されることを確認し、発見されなければルート直下配置に変更する。
- GitHub への push(public)は実行前にユーザーの確認を取る。

### フロントマター案(skill-creator 実装時のたたき台)

```yaml
name: e2e-coverage-architect
description: Analyze an existing web app's routes, auth, permissions, data flows, and current tests; build a behavior-coverage matrix; detect coverage gaps, duplicates, and false-green E2E tests; then implement prioritized Playwright tests until meaningful behavior gaps are exhausted. Use when asked to improve, audit, or extend E2E test coverage, find untested user flows, or verify that existing E2E tests actually assert behavior.
```

## 4. ワークフロー(6 フェーズ)

各フェーズは開始条件・終了条件を持ち、状態は対象プロジェクトの成果物ファイル(§10)に保持する。会話メモリを状態の保存先にしない。

| Phase | 内容 | 開始条件 | 終了条件 |
|---|---|---|---|
| 0 Inventory | スタック検出、Playwright 有無確認(なければ bootstrap)、ルート・API endpoint・認証モデル(session/JWT/ロール)・DB スキーマと seed・既存テスト(fixtures, helpers, storageState 等の認証セットアップ)の目録 | スキル起動 | inventory が matrix.md の所定セクションに記録済み |
| 1 行動列挙 | ユーザー行動単位(例: 「visitor が signup する」「user が post を作成する」「admin が他人の post を削除する」)を行動として列挙し、各行動に適用可能な観点タグを付ける | Phase 0 終了 | 行動リストが matrix.md に記録済み。各行動に観点タグとリスク備考がある |
| 2 ギャップ分析 | 既存テストをセルにマッピング(covered には必ず `file:line` の証跡)、重複クラスタ検出、covered セルへの false green チェックリスト適用、優先度付き gap リスト作成 | Phase 1 終了 | matrix.md の全セルが covered/partial/uncovered/n-a のいずれかになり、gaps.md に優先度付きリストが出力される |
| 3 実装 | 優先度順(P0→P1→P2)にバッチ実装。既存の fixtures・page objects・規約に従う。バッチごとに `playwright test` を実行。失敗のトリアージ: テスト側(locator/wait/データ)ならテストを修正、アプリ側バグなら findings.md に記録し該当テストを `test.fixme`(理由と findings 参照)にして続行 | gaps.md がある | gap リストの全項目が implemented / known-failure / waived のいずれかになり、`test.fixme` 以外の追加テストが緑 |
| 4 再分析ループ | コードから行動を再列挙し取りこぼしを検出、新テストを含めて再マッピング、残 gap 再スキャン。**最大 3 パス** | Phase 3 終了 | 1 パスで新規 P0/P1 gap がゼロになり全セルが終端状態(§8)。3 パス到達で打ち切り、残りは gaps.md に open として記録 |
| 5 最終報告 | before/after マトリクス、追加テスト一覧、発見した false green、known-failure、waived と理由を報告 | Phase 4 終了 | チャットでの報告と成果物のファイナライズ |

## 5. カバレッジマトリクス

- 行 = ユーザー行動、列 = 6 観点: **happy path / validation / 権限 (authorization) / エラー処理 / 状態遷移 / 永続化 (persistence)**
- 全行動に全観点が適用できるとは限らない。適用不能セルは `n/a`(理由)とする。
- セルの解析ステータス: `covered`(証跡 `file:line` 必須)/ `partial`(証跡 + 不足内容)/ `uncovered` / `n/a`
- セルの解決ステータス(Phase 3 以降): `implemented` / `known-failure`(findings.md 参照)/ `waived`(理由)
- matrix.md は markdown テーブル + 凡例で表現し、before/after をセクション分けして保持する。

## 6. 優先度定義

| 優先度 | 対象 |
|---|---|
| P0 | 認証バイパス、権限昇格・横断アクセス、データ損失・破損、セキュリティに直結する挙動、課金 |
| P1 | 主要 happy path、ユーザー影響のある入力 validation、状態遷移、重要エンティティの永続化 |
| P2 | 二次的フロー、表示レベルのエラー処理、エッジケース UX |

## 7. false green 検出チェックリスト

covered と判断した各セルに対して、対応テストが以下に該当しないか確認する(gap-analysis.md に詳細手順として展開):

1. assertion がない、または navigation/wait 系(`waitForURL`, `toHaveURL`)のみで内容・データの検証がない
2. エラーの不在のみを検証している(例: `not.toHaveText(/error/)`、status < 500)— クラッシュしないことしか証明していない
3. `test.skip` / `test.fixme` / 条件付き skip がカバレントシとして扱われている
4. テスト自身が作成していない seed 済みデータを assertion している(そのテストは当該行動を駆動していない)
5. 広すぎる matcher で回帰を隠している(特定テキストを期待すべき箇所の `toHaveCount(0)`、汎用コンテナへの `toBeVisible()` 等)
6. 他テストの共有可変状態に依存し、実行順序が変わると結果が変わる
7. 永続化の検証が UI の初期応答のみで、reload や再取得による再確認がない

該当した場合は partial 扱いに格下げし、gaps.md に記録する。

## 8. 完了判定

両方満たした場合のみ完了とする:

1. マトリクスの全セル(ただし `n/a` を除く)が `implemented` / `known-failure` / `waived` のいずれかの終端状態にある
2. 直近の再分析パスで新規の P0/P1 gap がゼロ

安全性のための硬い上限: 再分析は最大 3 パス。到達時に残項目があれば、gaps.md に open として残し「未完了」と明示して報告する。完了主張は matrix.md / gaps.md の状態から客観的に検証可能でなければならない。

## 9. エッジケースと縮退運用

| 状況 | 挙動 |
|---|---|
| Playwright 未導入 | bootstrap を実行(追加ファイルを報告)して Phase 0 を続行 |
| アプリの起動方法が不明 | `playwright.config.ts` の `webServer`、package.json scripts、docker-compose を探索し、不明ならユーザーに質問。起動不能なら解析のみレポート(matrix + gap リスト)を出力して実装フェーズに入らず終了 |
| 非 Web プロジェクト | 説明して中断 |
| 巨大リポジトリ | Phase 0 の inventory をサブエージェントに並列分散する戦術を許可(成果物の一貫性はメインスレッドで検証) |
| flaky なテスト | 最大 2 回 retry し、再現する場合は findings.md に quarantine 記録。削除はしない |
| 既存テストとの重複 | 検出して報告するのみ。削除・統合はユーザーの判断 |

## 10. 成果物仕様(対象プロジェクト内 `docs/e2e-coverage/`)

| ファイル | 内容 |
|---|---|
| `matrix.md` | inventory セクション、行動リスト、行動×観点マトリクス(before/after)、凡例 |
| `gaps.md` | 優先度付き gap リストと解決状況(implemented / known-failure / waived / open)。waived は理由必須 |
| `findings.md` | 発見したアプリバグ(known-failure)・false green・flaky quarantine の記録。各項目に再現条件と該当テスト参照 |

これらは完了の証跡であると同時に、作業再開時の状態復元を兼ねる。

## 11. 検証計画(eval)

リポジトリ直下の gitignore 済み `eval/` に検証専用のダミー Web アプリを作成する(リポジトリには同梱しない)。

### ダミーアプリ要件

- 小規模 Web アプリ: ログイン(session)、2 ロール(admin / user)、CRUD 対象エンティティ(post 等)、入力 validation、エラー表示、DB 永続化
- 実行可能な既存 Playwright テストを少数同梱

### 仕込み(正解データ)

- 未カバー領域: admin 権限系フロー、validation エラー、ログアウト後の状態遷移、永続化の再確認 など
- false green 2〜3 件: 例 — assertion が navigation のみのテスト、skip 済みテストが coverage に数えられている、seed データを検証するだけのテスト
- 実バグ 1 件: 例 — post 編集で title の変更が保存されない(スキルはこれを known-failure として記録し、アプリを修正しない)

### 合格基準

1. 仕込んだ未カバー領域を gap として検出できる
2. 仕込んだ false green を検出できる
3. 実バグを findings.md に記録し、該当テストが `test.fixme` になり、アプリコードが無変更である
4. 完了判定が §8 の基準どおり客観的に出せる
5. 追加テストが実行されて緑になる

### 合格しなかった場合

スキル側(SKILL.md / references/)を修正して再評価する。ダミーアプリ側を正解に合わせて改変しない。

## 12. スコープ外

- アプリ本体のバグ修正(一切行わない)
- Playwright 以外のテストフレームワーク(Cypress, Selenium 等)
- Unit / Integration テスト
- 既存テストの削除・統合(報告のみ)
- 負荷・パフォーマンス・アクセシビリティテスト

## 13. 実装工程

1. **skill-creator** で SKILL.md + references/ を実装(本書 §3〜§10 が入力)
2. `eval/` にダミーアプリを作成し、§11 に基づき評価・改善
3. README.md 整備、ローカルで `npx skills add .` の発見確認
4. GitHub public リポジトリ作成・push(実行前にユーザー確認)
5. クリーンなディレクトリで `npx skills add <owner>/e2e-coverage-architect` を実行し、導入と動作を確認
