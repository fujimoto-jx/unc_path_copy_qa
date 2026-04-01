# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## コミュニケーション

このリポジトリでの作業中は、**やりとりはすべて日本語**で行うこと。

## プロジェクト概要

**UNCパスをコピー** は、macOS のオートメーター（Automator）ワークフローとして実装された Finder のコンテキストメニューサービスです。SMBマウントされたネットワーク共有上のファイルパスを、Mac形式（`smb://host/share/path`）と Windows形式（`\\host\share\path`）の両方に変換してクリップボードにコピーします。日本語 macOS 環境向けに設計されており、パス内のURLエンコードされた非ASCII文字を適切に処理します。

## リポジトリ構成

これは macOS の `.workflow` バンドル（パッケージ形式）であり、通常のソフトウェアプロジェクトではありません。ビルド手順、パッケージマネージャー、テストランナーは存在しません。

```
UNCパスをコピー.workflow/
└── Contents/
    ├── document.wflow      # ワークフロー定義（XML/plist形式）— bashスクリプトを含む
    ├── Info.plist          # macOSサービス登録メタデータ
    └── Resources/
        ├── background.color
        └── QuickLook/      # Finder用プレビュー画像
```

## コアロジック

すべてのロジックは `document.wflow` 内に埋め込まれた bashスクリプト（`ActionBashScript` アクションのテキスト）に記述されています。主要コンポーネント：

1. **`urldecode`** — URLエンコードされたパスセグメントをデコード（例：`%E6%97%A5%E6%9C%AC%E8%AA%9E` → `日本語`）
2. **`notify`** — `osascript` を使って macOS ネイティブ通知を表示
3. **`copy_to_clipboard`** — `osascript` でクリップボードにコピー。AppleScript の文字列リテラル内でバックスラッシュを二重エスケープする必要あり
4. **SMBマウント検出** — `/sbin/mount | grep '(smbfs'` の出力を解析してマウント済みネットワークボリュームを特定
5. **パスマッチング** — `case "$p" in "$mnt"|"$mnt"/*)` で選択された Finder アイテムをマウントポイントと照合
6. **出力形式** — 単一ファイル選択時は Mac + Windows 両形式を出力、複数ファイル選択時は最初のパス＋残り件数を出力

## ワークフローの編集

bashスクリプトは `document.wflow` の XML 内に文字列値として格納されています。編集するには：
- macOS の Automator.app で開く、または
- XMLを直接編集：`<key>ActionBashScript</key>` キーを探し、その直後の `<string>` 値を変更する

クリップボードコピー関数内のパスやエスケープを編集する際は、バックスラッシュが AppleScript の文字列補間を経由するため二重エスケープが必要な点に注意。

## インストール（macOS）

`.workflow` バンドルをダブルクリックして Finder サービスとしてインストールするか、`~/Library/Services/` にコピーする。
