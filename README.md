# experiment-ssh-connection

このリポジトリは、パスフレーズを設定した SSH 鍵を VS Code で利用できるように `ssh-agent` を設定し、VS Code 上から GitHub にアクセスできるかどうかを確認することを目的としています。

期待する動作をすることを確認できたので、以下にその設定方法を記述します。  
この設定によって、パスフレーズ付きの SSH 鍵を VS Code や Remote Containers の中で利用できるようになります。

## SSH 鍵を作成する

```bash
ssh-keygen -t ed25519 -C "your_email@example.com"
```

## config ファイルを作成する

ログインユーザの指定や秘密鍵のパスの指定をしなくても済むように、`config` ファイルを作成します。

```bash
Host github.com
  User git
  IdentityFile ~/.ssh/<private key>
  IdentitiesOnly yes
  Compression yes
```

## ターミナル起動時に `ssh-agent` が起動するようにする

[GitHub Docs](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/working-with-ssh-key-passphrases#auto-launching-ssh-agent-on-git-for-windows) のコードを参考にして `.bashrc` に以下の内容を記述します。  
`-t` オプションで、鍵の生存期間を指定することができます。

```bash
env=~/.ssh/agent.env

agent_load_env () { test -f "$env" && . "$env" >| /dev/null ; }

agent_start () {
    (umask 077; ssh-agent >| "$env")
    . "$env" >| /dev/null ; }

agent_load_env

# agent_run_state: 0=agent running w/ key; 1=agent w/o key; 2=agent not running
agent_run_state=$(ssh-add -l >| /dev/null 2>&1; echo $?)

if [ ! "$SSH_AUTH_SOCK" ] || [ $agent_run_state = 2 ]; then
    agent_start
    ssh-add -t 3600 ~/.ssh/<private key> # 鍵の生存期間を 1H に設定する
elif [ "$SSH_AUTH_SOCK" ] && [ $agent_run_state = 1 ]; then
    ssh-add -t 3600 ~/.ssh/<private key> # 鍵の生存期間を 1H に設定する
fi

unset env
```

### `.bashrc` を読み込む

ターミナル起動時に `.bashrc` が読み込まれるので、ターミナルを再起動しない場合には、下記のコマンドを実行してください。

```bash
. ~/.bashrc
```
