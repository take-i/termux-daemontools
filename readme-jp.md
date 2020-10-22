# daemontoolsとは？
daemontools とは、デーモンを監視するツールのことで、qmail の作者 D.J.B. によるツールの事。メリットは、daemontools によって監視させておけば、 自動的に再起動してくれます。注意事項は以下。
* バックグラウンドになるデーモンは管理できない。
* この為、run から動作させるプロセスは、 & を付けてバックグラウンドにしないこと。

詳細は、オフィシャルサイトを参照。https://cr.yp.to/daemontools.html
このリポジトリでは、Termuxでdaemontoolsを動作させるようにした修正しました。

# インストール

daemontoolsは、展開したソースディレクトリにコマンドがシンボリック・リンクされます。

このインストール例では、ホームディレクトリに `bin` を作成し、そこに展開してインストールします。

```$ cd
$ mkdir bin
$ cd bin

$ git clone https://github.com/take-i/termux-daemontools.git
$ cd termux-daemontools/admin/daemontools-0.76/
$ package/install
```

# コマンドはどこにあるの?

コマンドは、termux配下の `$PREFIX/bin` ディレクトリ以下にありますが、これはシンボリック・リンクです。

```
$ which svscan
/data/data/com.termux/files/usr/bin/svscan
```

また、このシンボリック・リンク先は、`$PREFIX/command/` になります。

```
$ ls -l /data/data/com.termux/files/usr/bin/svscan
lrwxrwxrwx 1 u0_a257 u0_a257 46 Oct 21 15:10 /data/data/com.termux/files/usr/bin/svscan -> /data/data/com.termux/files/usr/command/svscan
```
つまり、コマンドの本体はビルドしたディレクトリ以下にあります。ビルドしたディレクトリを削除するときは注意しましょう。

```
$ tree $PREFIX/command/
├── envdir -> /data/data/com.termux/files/home/bin/termux-daemontools/admin/daemontools/command/envdir
├── envuidgid -> /data/data/com.termux/files/home/bin/termux-daemontools/admin/daemontools/command/envuidgid
├── fghack -> /data/data/com.termux/files/home/bin/termux-daemontools/admin/daemontools/command/fghack
├── multilog -> /data/data/com.termux/files/home/bin/termux-daemontools/admin/daemontools/command/multilog
├── pgrphack -> /data/data/com.termux/files/home/bin/termux-daemontools/admin/daemontools/command/pgrphack
├── readproctitle -> /data/data/com.termux/files/home/bin/termux-daemontools/admin/daemontools/command/readproctitle
├── setlock -> /data/data/com.termux/files/home/bin/termux-daemontools/admin/daemontools/command/setlock
├── setuidgid -> /data/data/com.termux/files/home/bin/termux-daemontools/admin/daemontools/command/setuidgid
├── softlimit -> /data/data/com.termux/files/home/bin/termux-daemontools/admin/daemontools/command/softlimit
├── supervise -> /data/data/com.termux/files/home/bin/termux-daemontools/admin/daemontools/command/supervise
├── svc -> /data/data/com.termux/files/home/bin/termux-daemontools/admin/daemontools/command/svc
├── svok -> /data/data/com.termux/files/home/bin/termux-daemontools/admin/daemontools/command/svok
├── svscan -> /data/data/com.termux/files/home/bin/termux-daemontools/admin/daemontools/command/svscan
├── svscanboot -> /data/data/com.termux/files/home/bin/termux-daemontools/admin/daemontools/command/svscanboot
├── svstat -> /data/data/com.termux/files/home/bin/termux-daemontools/admin/daemontools/command/svstat
├── tai64n -> /data/data/com.termux/files/home/bin/termux-daemontools/admin/daemontools/command/tai64n
└── tai64nlocal -> /data/data/com.termux/files/home/bin/termux-daemontools/admin/daemontools/command/tai64nlocal
```
# daemontoolsの起動

必須ではありませんが、明示的にPATHを記載したい場合は以下のように追記しておきます。

~/.bashrc
```
::
#djb tool daemontools
export PATH=$PATH:$PREFIX/command
```
Daemontools の起動は、スマートフォンのroot化を行なっている場合はroot で起動できます。その場合は、

`$ nohup sudo svscan $PREFIX/service/ &`

一般ユーザーで起動する場合は、

`$ nohup svscan $PREFIX/service/ &`

どちらかの方法で起動してください。

# 設定のサンプル

例えば、crondをdaemontoolsで起動させるサンプルです。ホームディレクトリにbootディレクトリを作成して後から $PREFIX/service/ にシンボリック・リンクを張って起動させます。実態のディレクトリは$PREFIX/var/boot/ とか好きなところにしてください。

```
$ cd
$ mkdir -p boot/crond
$ cd boot/crond

$ vi run
```

```
#!/bin/sh

export PATH=$PATH:$HOME/bin
export PATH=$PATH:$PREFIX/command
export PREFIX=/data/data/com.termux/files/usr

exec 2>&1
exec > /dev/null

# start up script example.
# need pkg install cronie
exec crond -n
```
デーモン起動しないように、crond を -n オプションでフォアグランドで起動しています。

```
$ cd ../
$ chmod +t crond
```

# 起動

シンボリック・リンクを張ると起動します。

`$ ln -s $HOME/boot/crond $PREFIX/service/`

# 起動確認

`$ svstat $PREFIX/service/crond`

```
$ ps axu | grep crond
u0_a257  20259  0.2  0.0  33936  2692 ?        S<    1970   0:14 supervise crond
u0_a257  32339  0.1  0.0  34852  2984 ?        S<    1970   0:00 crond -n
```
# サービスの一時的な停止

`$ svc -d $PREFIX/service/crond`

# サービスの再開

`$ svc -u $PREFIX/service/crond`

# 永続的なサービスの削除

監視ディレクトリからシンボリック・リンクを削除します。

```
$ cd $PREFIX/service/
$ unlink crond
```

# 注意事項

setuidgidなど、一般ユーザ（termuxをインストールしたユーザ）では動作しないと思います。また、すべてのコマンドをテストしていませんので、動作無保証です。

|    | Command       | Overview                                    | Tested | Works as root | Works as user | 備考                                  |
|----|---------------|---------------------------------------------|--------|---------------|---------------|-------------------------------------|
| 1  | envdir        | 指定したDirの環境変数を読み込んでからコマンドを起動                 | No     |               |               |                                     |
| 2  | envuidgid     | 指定したアカウントの uid と gidでコマンド起動                 | No     |               |               |                                     |
| 3  | fghack        | バックグランドに移るのを防ぐ                              | No     |               |               |                                     |
| 4  | multilog      | tai64n 形式のログを作成                             | No     |               |               |                                     |
| 5  | pgrphack      | 別のプロセスグループでプログラムを実行                         | No     |               |               |                                     |
| 6  | readproctitle | psにreadproctitleというエラーメッセージを出す              | No     |               |               |                                     |
| 7  | setlock       | ファイルをロックして別のプログラムを起動                        | No     |               |               | 多重起動を防止するのに使える                      |
| 8  | setuidgid     | 指定したアカウントの uid と gid で別のプログラムを起動            | Yes    | O             | X             | root以外では使えない仕様                      |
| 9  | softlimit     | リソース制限をかけて別プログラムを起動                         | No     |               |               |                                     |
| 10 | supervise     | サービスを開始させ、監視するプログラム。svscan により起動する          | Yes    | O             | O             |                                     |
| 11 | svc           | supervise により監視されているサービスを制御                 | Yes    | O             | O             |                                     |
| 12 | svok          | supervise が起動しているかを調べる                      | Yes    | O             | O             | `svok $PREFIX/service/test ; echo $?` |
| 13 | svscan        | サービスの集まりを開始させ、監視する                          | Yes    | O             | O             |                                     |
| 14 | svscanboot    | /serviceディレクトリでsvscanを開始し、readproctitleにパイプ | X      |               |               |                                     |
| 15 | svstat        | supervise により監視されているサービスの状態を出力              | Yes    | O             | O             |                                     |
| 16 | tai64n        | TAI64N 形式の正確なタイムスタンプをつける                    | No     |               |               |                                     |
| 17 | tai64nlocal   | TAI64N 形式のタイムスタンプを人間が読める形式に変換               | No     |               |               |                                     |