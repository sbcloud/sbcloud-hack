# aliyuncliのインストール

## 0. Index
 - [インストール手順](#1-インストール手順)
 - [Debian系システムの場合](#2-debian系システムの場合)
 - [Windowsの場合](#3-windowsの場合)
 - [AutoCompleteの設定（オプション）](#4-autocompleteの設定)

## 1. インストール手順

### 1.1 pipのインストール
すでにpipがインストールされている場合はこの手順をスキップして下さい。
<br>
**NOTE:** Windowsユーザは[事前にpythonとpipをインストールして下さい]()
```
$ curl https://bootstrap.pypa.io/get-pip.py | sudo python
```

### 1.2 aliyuncliのインストール
**NOTE:** Debian系システムの場合は[この手順に従って実施して下さい](#2-debian系システムの場合)
```
$ sudo pip install aliyuncli
```

事後確認
```
$ aliyuncli
usage: aliyuncli <command> <operation> [options and parameters]
<aliyuncli> the valid command as follows:
```
実行可能なコマンドは表示されません。ecsやossなど、プロダクト毎のSDKをインストールして、コマンドを追加します。
<br>
以下のSDKをインストールしてしてコマンドを追加します：
- ECS: aliyun-python-sdk-ecs
- RDS: aliyun-python-sdk-rds
- SLB: aliyun-python-sdk-slb
- OSS: aliyun-python-sdk-oss

```
$ sudo pip install aliyun-python-sdk-ecs
$ sudo pip install aliyun-python-sdk-rds
$ sudo pip install aliyun-python-sdk-slb
$ sudo pip install aliyun-python-sdk-oss
```
事後確認
```
$ aliyuncli
usage: aliyuncli <command> <operation> [options and parameters]
<aliyuncli> the valid command as follows:

ecs                                       | oss
rds                                       | slb
```
インストールしたプロダクトごとのSDKが利用可能なコマンドとして表示されます。

## 2. Debian系システムの場合
以下の方法で`aliyuncli`と各SDKをインストールします。
```
$ dist_packages=$( python -c "from distutils.sysconfig import get_python_lib; print(get_python_lib())" )
$ sudo pip install --install-option="--install-purelib=${dist_packages}" aliyuncli
$ sudo pip install --install-option="--install-purelib=${dist_packages}" aliyun-python-sdk-ecs
$ sudo pip install --install-option="--install-purelib=${dist_packages}" aliyun-python-sdk-rds
$ sudo pip install --install-option="--install-purelib=${dist_packages}" aliyun-python-sdk-slb
$ sudo pip install --install-option="--install-purelib=${dist_packages}" aliyun-python-sdk-oss
```
事後確認
```
$ aliyuncli
usage: aliyuncli <command> <operation> [options and parameters]
<aliyuncli> the valid command as follows:

ecs                                       | oss
rds                                       | slb
```

## 3. Windowsの場合

`aliyuncli`はpythonで動いていますので、事前にpythonをインストールする必要があります。インストール済みの場合はこの手順をスキップして下さい。
<br>
[このページ](https://www.python.org/downloads/release/python-2711/)から最新のpythonイントーラを入手し、インストールをして下さい。この時、インストールオプションの中から`pip`ツールが選択されていることを確認して下さい。。
<br>
**NOTE:** 現在は`aliyuncli`が`python3`に対応しておりませんので、`python2`をインストールして下さい。
<br>
pythonのインストール完了後、[1.2 aliyuncliのインストール手順](#12-aliyuncliのインストール)に従って`aliyuncli`と各SDKをインストールして下さい。

## 4. AutoCompleteの設定
Linuxの場合は、AutoCompleteの設定が可能です。
```
complete -C `which aliyun_completer` aliyuncli
```
次回ログイン以降もこの設定を反映させたい場合は`~/.bash_profile`などにこのコマンドを追加して下さい。

## [Getting Started Guide](aliyuncli-getting-started.md)を見る