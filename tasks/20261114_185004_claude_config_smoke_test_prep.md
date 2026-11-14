---
date: 2026-11-14
title: claude-config別プロジェクト稼働テストの事前準備
branch: docs/framework-smoke-test-prep
status: PREPARING
---

# claude-config別プロジェクト稼働テストの事前準備

## 目的

`unc_path_copy_qa` のforkを使い、M4をクライアント、M1を現在のローカル推論サーバーとして、claude-configのクロスプロジェクト運用を安全に確認する。M5実機の稼働確認は、M5が準備できるまで対象外とする。

## 作業対象

- upstream: https://github.com/Zappa0420/unc_path_copy_qa
- test fork: https://github.com/fujimoto-jx/unc_path_copy_qa
- 元のclone: ユーザー既存のclone（参照用。未追跡ファイルを含め変更しない）
- fork clone: 元cloneと並ぶ専用の兄弟フォルダ（実パスはローカル環境のみで管理）
- framework repository: `~/claude-config`

## Ollamaの計画提案

- モデル: `qwen3.6:35b-a3b-coding`
- 要約: forkした環境に対し、M1/M5サーバーとM4クライアント構成の稼働テストに先立つ準備を実施する。
- 役割案: 設計=Ollama（分析のみ）、実装=Claude（M4）、検証=Codex（M4）、Read-Onlyレビュー=Codex。
- 初回提案はClaudeをレビュー担当に選びrouter検証で拒否された。独立レビュー担当を利用可能なCodex/Geminiから選ぶと明示した再依頼は成功した。この挙動をrouterの運用確認項目に含める。
- この計画生成ではM1 URLとモデル名をプロセス環境に一時設定した。URLや資格情報はリポジトリへ保存しない。

## 事前準備タスク

### A. Frameworkの使用版を固定する

- [ ] `~/claude-config` の現在の作業ブランチと、最新のマージ済みframework変更を比較する。
- [ ] 現在のcloneは `feat/cross-project-pipeline-runner`、PR #20はPR #18のbase branchにマージ済みである。PR #18を含むどのcommitをテスト対象にするか決める。
- [ ] Codexの共通ルール入口、`multi-model-pipeline` skill、Ollama router、`agy` / Geminiの扱いが同一revisionに揃っていることを確認する。
- [ ] 選定したrevisionのglobal setupをM4で確認する。既存のCLI設定を上書きせず、reset可能な管理手順を使う。

### B. M1/M4接続とCLI環境を整える

- [x] M1 Ollamaの `/api/tags` が応答し、`qwen3.6:35b-a3b-coding` が一覧にあることを確認。
- [x] M1 Ollama routerで有効な計画提案を取得。
- [ ] 稼働テスト用M4ターミナルで `OLLAMA_BASE_URL` と `OLLAMA_MODEL` を設定し、routerから同じモデルへ計画要求できることを確認する。値はshell環境に置き、repoへ記録しない。
- [ ] テストは信頼できるLAN内のみで実施する。Tailscale、port forwarding、`0.0.0.0` bindは使わない。
- [ ] 利用可能なM4 providerを再検出する。今回 `codex` は利用可能、Gemini CLIは検出されなかった。`agy` とGemini CLIは別製品として扱う。
- [ ] M5は未準備のため、本テストはM1/M4のPhase Aとして範囲を限定する。M5での移行・モデル性能確認を完了条件に混ぜない。

### C. 対象プロジェクトの安全な確認条件を決める

- [ ] 作業開始時にfork cloneがcleanで、branchとoriginがforkを指すことを確認する。upstreamへpushしない。
- [ ] `CLAUDE.md` にあるテンプレート表記の扱いを決める。元プロジェクトの指示を無断で書き換えない。
- [x] 複数選択時はworkflow本体とREADMEが全選択パスを出力することを確認。`CLAUDE.md` の「先頭パス＋残り件数」という記述をテスト前に整合させる。
- [ ] Finder、Automator、クリップボード、通知の手動確認に使うテスト用SMB共有と、ASCII/日本語パスのfixtureを用意する。実データの共有では試さない。
- [ ] このrepoには自動テストランナーがない。実装変更を伴う場合はmacOS上のFinder/Automator手動確認を検証条件として記録する。

## 稼働テスト開始条件

- framework revisionとM4 global setupの対象が固定されている。
- fork cloneのoriginが `fujimoto-jx/unc_path_copy_qa` で、upstreamが参照専用である。
- M1 API、M4 shell環境、Codex Read-Onlyレビュー経路が確認済みである。
- SMBテストfixtureと期待するクリップボード出力を決めている。
- M5を対象外とすることを含め、テスト範囲をユーザーが確認している。

## 稼働テストの想定フロー

1. ClaudeターミナルからM1 Ollamaへ依頼し、役割案と受入条件を作る。
2. fork cloneで専用branchと実装タスクを作成する。
3. ClaudeがM4上で低リスクのタスクを実装し、M4で検証する。
4. CodexへRead-Onlyレビューを依頼し、成果物だけ `tasks/reviews/` に記録する。
5. 報告、commit、forkへのpush、PR作成、ユーザー確認までを記録する。mergeは別途明示承認後に行う。

## 現在の状態

- forkと隔離cloneは作成済み。
- M1のモデル一覧とOllama planner応答を確認済み。
- Codexの独立レビューは自動審査に拒否されたため未実施。対象taskファイルをCodexサービスへ送る明示許可が必要。
- 上記の事前準備が完了するまで、対象アプリの実装・Finder操作を伴う稼働テストは開始しない。
