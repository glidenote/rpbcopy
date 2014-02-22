# rpbcopy (remote pbcopy)

## これは何?

sshしたリモートサーバ上での出力や`cat`してpipeで渡した内容を、ローカルMacのクリップボードに送ることが出来ます。

動作検証環境

 * リモートサーバ CentOS 6.4
 * ローカルMac OS X 10.9.1

```
+ - - - - - - - - -+                       +- - - - - - - - - - - - - - - - -+
' Remote Server:   '                       ' Mac:                            '
'                  '                       '                                 '
' +--------------+ '  ssh(RemoteForward)   ' +--------------+     +--------+ '
' | rpbcopy(nc)  | ' --------------------> ' | LaunchAgent  | --> | pbcopy | '
' +--------------+ '                       ' +--------------+     +--------+ '
'                  '                       '                                 '
+ - - - - - - - - -+                       +- - - - - - - - - - - - - - - - -+
```

## 準備

### Mac側の設定

#### LaunchAgentを起動

`2224`ポートで待ち受けて、受け取った内容をクリップボードに渡すLaunchAgentを用意。
同梱の`pbcopy.plist`を`~/Library/LaunchAgents/pbcopy.plist`に設置して、下記で起動

``` sh
launchctl load ~/Library/LaunchAgents/pbcopy.plist
```

停止する場合は下記

``` sh
launchctl unload ~/Library/LaunchAgents/pbcopy.plist
```

#### `~/.ssh/config`の設定

リモートサーバの`2224`ポートをフォワーディングするので、
`~/.ssh/config`に

```
RemoteForward 2224 127.0.0.1:2224
```

を追加。

#### `~/.CFUserTextEncoding`の設定

必ず必要な作業ではありませんが、UTF-8の内容を送ると文字化けが発生する場合、転送が上手くされない場合は
同梱の`.CFUserTextEncoding`を`~/.CFUserTextEncoding`に設置することで回避出来ると思います。

`~/.CFUserTextEncoding`を編集することで他に影響が及ぶ場合があるので、
必ずバックアップを取り、自己責任で編集お願いします。

### リモートサーバ(CentOS)側の設定

#### ncの導入

`nc(netcat)`が無いと使えないので`nc`を導入。(`nc`のバージョンが古いと`-C`オプションが存在しない場合があります。)

``` sh
sudo yum -y install nc
```

#### rpbcopyスクリプトの設置

同梱の`rpbcopy`を`~/bin/rpbcopy`など`PATH`が通っている場所に設置して権限を付与

``` sh
chmod a+x ~/bin/rpbcopy
```

## 使い方

リモートサーバにsshログイン。
手元Macのクリップボードに送りたいファイルやコマンドの出力を、リモートサーバ上で

```
# 転送が出来るかテスト
echo "hello" | rpbcopy

# 日本語が転送が出来るかテスト。駄目な場合はMac側の `~/.CFUserTextEncoding`を設定する必要あり
echo "こんにちは" | rpbcopy

# fileの中身を手元Macのクリップボードにコピー
cat fileneme | rpbcopy

# リモートサーバのhistoryの内容を手元Macのクリップボードにコピー
history | rpbcopy
```

という感じでrpbcopyに渡すと、手元のMacのクリップボードに渡されるので
Mac側で`pbpaste`や貼り付けが出来る。

## 補足

`~/.bash_profile`や`~/.zsh_profile`などに

```
alias pbcopy='~/bin/rpbcopy'
```

とaliasを追加して、`pbcopy`でも使えるようにしておけば、
リモートサーバとローカルサーバを意識せずに利用出来る。

## トラブルシューティング

導入の際にハマったところをまとめておく。

### Mac側

#### `launchctl load/unload`を実行すると`Could not open job overrides database at: /private/var/db/launchd.db/com.apple.launchd/overrides.plist: 13: Permission denied`と出る。
 
tmuxを起動している状態で`launchctl load/unload`を実行するとPermissionのエラーが出てしまうことがあるようです。[ChrisJohnsen/tmux-MacOSX-pasteboard](https://github.com/ChrisJohnsen/tmux-MacOSX-pasteboard/)を利用することで回避出来る模様。

#### LaunchAgentは動作しているか、2224ポートは開いているか確認

```
nc -vz 127.0.0.1 2224
```

#### `LaunchAgent` => `pbcopy` とデータが渡っているか。

Macのncの場合オプションは`-c`なので注意

```
echo "hello" | nc -c 127.0.0.1 2224
pbpaste
```

#### `LaunchAgent` => `pbcopy` と日本語のデータが渡っているか。

Macのncの場合オプションは`-c`なので注意.

```
echo "日本語のテスト" | nc -c 127.0.0.1 2224
pbpaste
```

駄目な場合は、`~/.CFUserTextEncoding`を同梱のものに入れ替えるてどうか確認。


### CentOS側

#### ncが入っているか

``` sh
type nc
```

#### 2224ポートと疎通が出来ているか。SSH RemoteForwardが出来ているか

``` sh
nc -vz 127.0.0.1 2224
```

#### SSH RemoteForward経由でMacのpbpasteにデータを渡せているか

```
echo "hello" | nc -c 127.0.0.1 2224
```

mac側で`pbpaste`を実行してみてどうか


## 参考

 * [Sean Coates blogs: Remote pbcopy](http://seancoates.com/blogs/remote-pbcopy)
 * [Remote pbcopy on OS X systems - BrettTerpstra.com](http://brettterpstra.com/2014/02/19/remote-pbcopy-on-os-x-systems/)
