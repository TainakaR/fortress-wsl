# インフラ構築実習レポート：セキュアなWebサーバ環境の構築

## 1. 実習の目的

本実習では、Webサーバー（Apache）の構築に加え、管理者による遠隔操作の安全性を高めるため、SSHのハードニング（要塞化）を実施した。特に「公開鍵認証」の導入と「ファイアウォール」の設定により、外部攻撃に強いセキュアなインフラ構成を理解することを目的とする。

## 2. 構築環境

* **ホストOS:** Windows 11 (PowerShell使用)
* **ゲストOS:** Ubuntu 24.04 LTS (WSL2)
* **主な使用ソフトウェア:**
* Apache2（Webサーバー）
* OpenSSH Server（遠隔管理）
* UFW（ファイアウォール）



## 3. 構築手順と内容

### 3.1 Webサーバー（Apache2）の構築

Webサービスの基盤としてApache2をインストールした。

```bash
sudo apt update
sudo apt install apache2
sudo systemctl start apache2
sudo systemctl enable apache2

```

> **確認事項:** ブラウザで `http://localhost` にアクセスし、デフォルトページが表示されることを確認した。

### 3.2 SSHサーバーの導入と鍵認証の設定

従来のパスワード認証を廃止し、より安全な「公開鍵認証」を導入した。

1. **Windows側（PowerShell）での鍵ペア作成:**
```powershell
ssh-keygen -t ed25519

```


2. **Ubuntu側への公開鍵登録:**
作成した `id_ed25519.pub` の内容を Ubuntu の `~/.ssh/authorized_keys` に追記し、適切な権限を設定した。
```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys

```



### 3.3 SSHサーバーのハードニング（要塞化）

設定ファイル `/etc/ssh/sshd_config` を編集し、セキュリティ設定を厳格化した。

* **主な変更点:**
* `PermitRootLogin no` : rootユーザーの直接ログインを禁止
* `PasswordAuthentication no` : パスワード認証を完全に禁止



```bash
sudo systemctl restart ssh

```

### 3.4 ファイアウォール（UFW）の設定

不要な通信ポートを遮断し、攻撃の入り口を制限した。

```bash
sudo ufw default deny incoming # 基本はすべて拒否
sudo ufw allow 80/tcp         # HTTP（Web）を許可
sudo ufw allow 22/tcp         # SSH（管理用）を許可
sudo ufw enable

```

## 4. 実行結果の確認（エビデンス）

### 4.1 鍵認証によるログイン成功

WindowsのPowerShellからパスワード入力なしでログインできることを確認した。

> **[ここにPowerShellでログインに成功した画面のスクリーンショットを貼付]**
> (実行コマンド: `ssh ユーザー名@localhost`)

### 4.2 サービス稼働状況

ApacheおよびSSHのサービスが正常に稼働していることを確認した。

```bash
sudo systemctl status apache2
sudo systemctl status ssh

```

## 5. 考察とトラブルシューティング

* **トラブルシューティング:** 設定ファイルの編集時に `readonly` エラーや `swapファイル` の警告に遭遇したが、`sudo` の適切な利用と一時ファイルの削除により解決した。この経験から、システム設定における権限管理の重要性を学んだ。
* **セキュリティ効果:** パスワード認証を禁止したことで、総当たり攻撃（ブルートフォース攻撃）のリスクを物理的に排除できた。また、UFWにより必要最小限のポートのみを開放する「最小権限の原則」を実証した。

---

## 次のステップへのアドバイス

このレポートを提出する際、以下のコマンド結果を末尾に添えるとさらに完璧になります。

* `sudo ufw status` の結果（現在の門番のルール一覧）
* `ls -l ~/.ssh/authorized_keys` の結果（権限が `600` になっていることの証明）

Markdownの記述内容について、修正したい部分や追加したい項目はありますか？