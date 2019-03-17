# SSH

## permission denied (publickey). と出たときの対処法

sshキーの管理をしている`ssh-agent`が使おうとしているキーの場所がわからないためエラーが出ているみたい。
なので、使うキーの場所を教えてあげると解決する。

以下、Githubから引用

```bash
eval "$(ssh-agent -s)"
ssh-add <ssh-path>
```

また余談になるが、ssh-agentのmanドキュメントによれば、`ssh`コマンドでも同じことができるそうだ。

sshを実行する際にオプションに`AddKeysToAgent`を指定してあげるといいみたい。

```bash
ssh -o AddKeysToAgent=yes
```

いちいちオプションに指定するのが面倒な場合は設定ファイルに指定するといい。

以下はsshが使用する設定の優先順位。

1. コマンドライン引数 `ssh -o <options>`
2. ユーザー設定 ~/.ssh/config
3. PC全体の設定 /etc/ssh/ssh_config

`AddKeysToAgent`については`man ssh_config`に書かれていた。

ssh-agentが実行されている限り、設定したキーとそのパスワードは有効になる的なことが書かれている。

man ssh_configから引用
> AddKeysToAgent<br><br>
> Specifies whether keys should be automatically added to a running ssh-agent(1). If this option is set to yes and a key is loaded from a file, the key and its passphrase are added
> to the agent with the default lifetime, as if by ssh-add(1).  If this option is set to ask, ssh(1) will require confirmation using the SSH_ASKPASS program before adding a key (see
> ssh-add(1) for details).  If this option is set to confirm, each use of the key must be confirmed, as if the -c option was specified to ssh-add(1).  If this option is set to no, no
> keys are added to the agent.  The argument must be yes, confirm, ask, or no (the default).
