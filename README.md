# UNCパスをコピー

Finder のコンテキストメニューから SMB ネットワーク共有上のファイルパスを Mac 形式・Windows 形式の両方でクリップボードにコピーする macOS サービスです。

## 機能

- SMB マウント済みネットワーク共有上のファイル・フォルダを右クリックして起動
- Mac 形式（`smb://host/share/path`）と Windows 形式（`\\host\share\path`）の両方をクリップボードにコピー
- 日本語などの非 ASCII 文字を含むパスに対応（URL エンコードを自動デコード）
- 複数ファイル選択時は全パスをまとめてコピー
- macOS ネイティブ通知でコピー結果を表示

## 出力形式

**単一ファイル選択時：**
```
Mac: smb://server/share/フォルダ/ファイル.txt
Win: \\server\share\フォルダ\ファイル.txt
```

**複数ファイル選択時：**
```
Mac: smb://server/share/ファイル1.txt
Win: \\server\share\ファイル1.txt
Mac: smb://server/share/ファイル2.txt
Win: \\server\share\ファイル2.txt
```

## インストール

1. `UNCパスをコピー.workflow` をダブルクリック
2. 「インストール」をクリック

または手動でコピー：

```bash
cp -r "UNCパスをコピー.workflow" ~/Library/Services/
```

インストール後、Finder でファイルを右クリック →「サービス」→「UNCパスをコピー」から使用できます。

## 動作環境

- macOS（Automator サービス対応バージョン）
- SMB マウント済みのネットワーク共有

## リポジトリ構成

```
UNCパスをコピー.workflow/
└── Contents/
    ├── document.wflow      # ワークフロー定義（bashスクリプトを含む XML/plist）
    ├── Info.plist          # macOS サービス登録メタデータ
    └── Resources/
        ├── background.color
        └── QuickLook/      # Finder 用プレビュー画像
```

## スクリプトの編集

bashスクリプトは `document.wflow` 内に埋め込まれています。編集方法：

- **Automator.app で開く**（推奨）
- **直接編集**：`<key>ActionBashScript</key>` の直後にある `<string>` 要素を変更

> **注意**：クリップボードコピー処理内のパス文字列は AppleScript 経由で渡すため、バックスラッシュを二重エスケープする必要があります。
