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

