# rpbcopy (remote pbcopy)

## これは何?

sshしたリモートサーバ上での出力や`cat`してpipeで渡した内容を、ローカルMacのクリップボードに送ることが出来ます。

検証環境

 * リモートサーバ CentOS 6.4
 * ローカルMac OS X 10.9.1

## 準備

### Mac側の設定

`2224`ポートで待ち受けて、受け取った内容をクリップボードに渡すLaunchAgentsを用意。
同梱の`pbcopy.plist`を`~/Library/LaunchAgents/pbcopy.plist`に設置して、下記で起動

``` sh
launchctl load ~/Library/LaunchAgents/pbcopy.plist
```

リモートサーバの`2224`ポートをフォワーディングするので、
`~/.ssh/config`に

```
RemoteForward 2224 127.0.0.1:2224
```

を追加。

停止する場合は下記

``` sh
launchctl unload ~/Library/LaunchAgents/pbcopy.plist
```

UTF-8の内容を送って、文字化けが発生する場合は同梱の`.CFUserTextEncoding`を
`~/.CFUserTextEncoding`に設置することで回避出来ると思います。

`~/.CFUserTextEncoding`を編集することで他に影響が及ぶ場合があるので、
必ずバックアップを取り、自己責任で編集お願いします。

### リモートサーバ(CentOS)側の設定

`nc(netcat)`が無いと使えないので`nc`を導入。(`nc`のバージョンが古いと`-C`オプションが存在しない場合があります。)

``` sh
sudo yum -y install nc
```

同梱の`rpbcopy`を`~/bin/rpbcopy`など`PATH`が通っている場所に設置して権限を付与

``` sh
chmod a+x ~/bin/rpbcopy
```

## 使い方

リモートサーバにsshログイン。
手元Macのクリップボードに送りたいファイルやコマンドの出力を、リモートサーバ上で

```
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

## 参考

 * [Sean Coates blogs: Remote pbcopy](http://seancoates.com/blogs/remote-pbcopy)
 * [Remote pbcopy on OS X systems - BrettTerpstra.com](http://brettterpstra.com/2014/02/19/remote-pbcopy-on-os-x-systems/)
