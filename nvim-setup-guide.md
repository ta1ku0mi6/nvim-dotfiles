# Neovim セットアップ手順書（会社PC向け）

## 前提条件

- Neovim がインストール済みであること
- Git がインストール済みであること
- Java (JDK/JRE) がインストール済みであること（PlantUML プレビューに必要）

## 送付ファイル

3通のメールに分割して送付される：

| メール | 添付ファイル | サイズ |
|---|---|---|
| 1通目 | `nvim-package-part1.dat` | 24 MB |
| 2通目 | `nvim-package-part2.dat` | 24 MB |
| 3通目 | `nvim-package-part3.dat` | 22 MB |

結合すると `nvim-package.zip` になり、中身は以下の通り：

| ファイル | 内容 | 展開先 |
|---|---|---|
| `nvim-config-full.zip` | Neovim 設定ファイル | `%LOCALAPPDATA%\nvim` |
| `nvim-data-full.zip` | プラグイン・LSP・パーサー一式 | `%LOCALAPPDATA%\nvim-data` |
| `tools\plantuml.jar` | PlantUML 実行ファイル | `%USERPROFILE%\tools` |

`nvim-data-full.zip` にはネットワーク経由でダウンロードが必要な以下が全て含まれている：

- **lazy.nvim プラグイン一式** — git clone で取得されるもの（証明書エラー回避）
- **Mason LSPサーバー・ツール** — lua-language-server, shfmt, stylua 等
- **Treesitter パーサー** — c, lua, typescript, python 等のパーサーバイナリ

## 手順

### 1. 分割ファイルを結合

3つの添付ファイルを同じフォルダにダウンロードし、PowerShell で以下を実行：

```powershell
cd ダウンロード先
cmd /c "copy /b nvim-package-part1.dat + nvim-package-part2.dat + nvim-package-part3.dat nvim-package.zip"
```

### 2. nvim-package.zip を展開

```powershell
Expand-Archive -Path "ダウンロード先\nvim-package.zip" -DestinationPath "ダウンロード先\nvim-package" -Force
```

### 3. 必要なツールをインストール

PowerShell を開いて以下を実行：

```powershell
# lazygit
winget install --id=JesseDuffield.lazygit -e

# C コンパイラ（treesitter パーサーの追加ビルドに必要）
winget install --id=BrechtSanders.WinLibs.POSIX.UCRT -e
```

インストール後、PowerShell を再起動する。

### 4. 既存の設定があれば退避

```powershell
Rename-Item "$env:LOCALAPPDATA\nvim" "$env:LOCALAPPDATA\nvim.bak" -ErrorAction SilentlyContinue
Rename-Item "$env:LOCALAPPDATA\nvim-data" "$env:LOCALAPPDATA\nvim-data.bak" -ErrorAction SilentlyContinue
```

### 5. 設定ファイルを展開

```powershell
Expand-Archive -Path "ダウンロード先\nvim-package\nvim-config-full.zip" -DestinationPath "$env:LOCALAPPDATA\nvim" -Force
```

### 6. プラグイン・LSP・パーサーを展開

```powershell
Expand-Archive -Path "ダウンロード先\nvim-package\nvim-data-full.zip" -DestinationPath "$env:LOCALAPPDATA\nvim-data" -Force
```

### 7. PlantUML を配置

```powershell
New-Item -ItemType Directory -Path "$env:USERPROFILE\tools" -Force
Copy-Item "ダウンロード先\nvim-package\tools\plantuml.jar" "$env:USERPROFILE\tools\plantuml.jar"
```

### 8. Neovim を起動

```powershell
nvim
```

全てのプラグイン・LSP・パーサーはローカルに展開済みのため、ネットワークアクセスなしで起動できる。

## 使い方

### lazygit

Neovim 内で `<Space>gg` を押すと lazygit が起動する。`q` で閉じる。

### PlantUML プレビュー

1. `.puml` ファイルを開く
2. `:PlantumlOpen` を実行するとブラウザでプレビューが開く
3. ファイルを編集するとリアルタイムで反映される
4. `:PlantumlSave` で画像ファイルとして保存できる

### よく使うキーマップ

| キー | 動作 |
|---|---|
| `<Space>gg` | lazygit を開く |
| `<Space>ff` | ファイル検索 |
| `<Space>sg` | 全文検索 (grep) |
| `H` | 前のバッファに移動 |
| `L` | 次のバッファに移動 |
| `jj` | インサートモードを抜ける |
| `K` | ホバー情報を表示 |

## プラグイン更新時（ネットワークが必要）

企業プロキシ環境でプラグインを更新する場合：

```powershell
git config --global http.sslVerify false
```

Neovim 内で `:Lazy update` を実行。完了後：

```powershell
git config --global http.sslVerify true
```

## トラブルシューティング

### 起動時にプラグインエラーが出る

nvim-data の展開先が正しいか確認：

```powershell
Test-Path "$env:LOCALAPPDATA\nvim-data\lazy\LazyVim"
```

`True` が返れば正常。`False` なら展開先を確認する。

### PlantUML プレビューが動かない

Java がインストールされているか確認：

```powershell
java -version
```

plantuml.jar が配置されているか確認：

```powershell
Test-Path "$env:USERPROFILE\tools\plantuml.jar"
```

### Mason の LSP が動かない

Mason のパッケージが展開されているか確認：

```powershell
Get-ChildItem "$env:LOCALAPPDATA\nvim-data\mason\packages"
```
