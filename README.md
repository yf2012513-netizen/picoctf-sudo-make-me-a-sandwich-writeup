# picoCTF - SUDO MAKE ME A SANDWICH Writeup

## [JP] 概要 / [EN] Overview
- **Challenge**: SUDO MAKE ME A SANDWICH (General Skills)
- **Topic**: Linux Privilege Escalation via Sudo / Sudoを悪用した特権昇格
- **Description**: A technical log demonstrating how to analyze user permissions and escalate privileges to read a restricted file. / ユーザー権限を分析し、権限を昇格させて閲覧制限のあるファイルを読み出す技術的なログです。

---

## [JP] 攻略プロセス / [EN] Walkthrough

### 1. 接続と初期確認 / Connection & Initial Enumeration
[JP] 指定されたSSHコマンドでサーバーに接続し、ファイルを確認します。
[EN] Connect to the target server via SSH and check the available files.

```bash
# [JP] 接続コマンド（アカウント名とIPはダミーに変更しています）
# [EN] Connection command (Account name and IP have been changed to dummies)
ssh -p 59568 ******@***.***.picoctf.net

# [JP] 現在のディレクトリを確認
# [EN] Check the current directory
ls
# Output: flag.txt
```

### 2. 権限拒否の確認 / Confirming Permission Denied
[JP] `flag.txt` をそのまま読み取ろうとしますが、一般ユーザー権限では拒否されます。
[EN] Attempting to read `flag.txt` directly results in a "Permission Denied" error due to standard user restrictions.

```bash
cat flag.txt
# Output: cat: flag.txt: Permission denied
```

### 3. Sudo権限の調査 / Checking Sudo Permissions
[JP] ヒントに従い、`sudo -l` を実行して、自身のアカウントに許可されている管理者権限のコマンドを調査します。
[EN] Following the hints, execute `sudo -l` to inspect the allowed administrative commands for the current user.

```bash
sudo -l
```

**[JP] 出力結果の確認:**
以下のように、パスワードなし（`NOPASSWD`）で `emacs` エディタを最高管理者権限（root）で実行できることが判明しました。
> `(ALL) NOPASSWD: /bin/emacs`

**[EN] Output Analysis:**
The output reveals that the user is permitted to run the `emacs` editor with root privileges without entering a password (`NOPASSWD`).
> `(ALL) NOPASSWD: /bin/emacs`

### 4. 特権昇格とフラグの取得 / Privilege Escalation & Flag Retrieval
[JP] 許可されている `emacs` を `sudo` 経由で起動し、管理者権限で `flag.txt` を強制的に読み込みます。
[EN] Launch the allowed `emacs` editor via `sudo` to force-read `flag.txt` with root privileges.

```bash
sudo emacs flag.txt
```

[JP] 画面がエディタに切り替わり、拒否されることなくファイルの中身（フラグ）が表示されます。
[EN] The terminal switches to the Emacs interface, and the content of the file (the flag) is successfully displayed without any restrictions.

```text
# [JP] エディタ画面に表示されたフラグ（完全に非公開化）
# [EN] Flag displayed inside the editor (completely hidden)
[picoCTF FLAG HIDDEN]
```

[JP] フラグを確認後、`Ctrl + X` に続いて `Ctrl + C` を押し、エディタを終了して元のシェルに戻ります。
[EN] After confirming the flag, press `Ctrl + X` followed by `Ctrl + C` to exit Emacs and return to the original shell.

---

## [JP] まとめ / [EN] Conclusion
[JP] `cat` コマンド自体の実行権限がなくても、管理者権限（Sudo）で実行可能な他のアプリケーション（今回は `emacs`）を中介させることで、目的のシステムファイルを読み出すことができるというLinux特権昇格の基本を学びました。

[EN] Even without direct permission to run `cat` as root, this challenge demonstrates a fundamental Linux privilege escalation technique: utilizing another sudo-allowed application (in this case, `emacs`) to read restricted system files.
