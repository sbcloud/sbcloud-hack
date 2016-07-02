# AlibabaCloudでWebサーバーを構築するサンプルのメモ

AlibabaCloudでWebサーバーを立てて、Wordpressでブログサイトを上げるまでの流れの説明です。

- LAMP環境の構築
- AlibabaCloudサービス（Apsara DB / SLB )の利用について

## ECSインスタンスの作成メモ

各項目の簡単な説明と、デモでの選択内容について。

- Subscription：固定期間注文　／　Pay-As-You-Go：1時間毎の利用時間精算：Pay-As-You-Goがオススメ（Subscriptionは、月額がPay-As-You-Goの20日分になります）
- Region：サーバーを作成するデータセンターの場所：どこでも選べます。デモでは「Hongkong」を選びます。
- Zone：Regionによっては複数Zoneで冗長構成ができます：デフォルトまたは「A」ゾーン。冗長性を考えるときは、別々のZoneでECSを上げて負荷分散をかけたほうが良いです。
- instance generation：Generation Ⅰ/Ⅱが選べます。Ⅱのほうが速いです：デモではⅡを選びます。価格的にはⅠのほうが安価です。
- instance type：仮想サーバーのスペックです：デモでは一番安い「ecs.n1.tiny」でOK。必要なスペックを選べます。
- Network type：ClassicかVPCが選べます。VPCを選ぶと、Global無しとか、個別のNW設定ができます。：デモではClassicを選択します。VPCでできることの詳細は、VPCのドキュメントをご参照ください。
- Bandwidth type：bandwidth（帯域固定）かtraffic（流量課金）かを選べます：trafficを選択。
- Bandwidth peak：最大速度、または固定する速度を指定します：trafficを選択した場合、適当で問題無し。bandwidthの場合、値段が変わります。
- Image type：仮想サーバーに導入するOSイメージの選択。独自に保存したイメージも選択可能：デモではPublic imageを選択します。カスタムイメージについては後述します。
- Public image：OSイメージを選べます：デモではCentOS 7/64bitを選択します。
- Storage：Efficient / SSDが選べます。SSDのほうが速いですが、Efficientでも充分実用になります：デモではEfficientを選択します。
- Storage：4本まで追加ディスクが設定できます。デモでは特に指定しません。
- Password：管理者ユーザーのパスワードです。Laterを選ぶと、あとでManagement consoleから設定することになります：何か指定しておきます。あまり容易なパスワードを指定すると不正アクセスされます。
- instance name：サーバーの名前です※ホスト名ではありません：何か指定します。
- Purchase plan：複数台同時にオーダーできます。デモでは1台で行きます。

## LAMP環境構築のコマンドラインメモ

※注意　cenosのイメージのバージョンが上がって（centos7u2_64_40G_cloudinit_20160520.raw）、
このイメージではデフォルトでfirewalldが有効になっています。サービスを停止するか、
正しくfirewalldを設定して使いましょう。そうしないと、外部アクセスがSSH以外つながりません。

    # systemctl stop firewalld
    # systemctl disable firewalld

LAMP環境を構築するメモです（Linux/Apache/MariaDB/PHP）。

    # yum install httpd mariadb-server php php-mysql
    # systemctl status mariadb.service
    # systemctl enable mariadb.service
    # systemctl start mariadb.service
    # mysql_secure_installation
    # mysql -u root -p
    # systemctl status httpd.service
    # systemctl enable httpd.service
    # systemctl start httpd.service

## Wordpress設定メモ

Webサイトのドキュメントルートにwordpressをセットアップして、ブログを書けるようにします。

まずは、wordpress（日本語版）をインストールします。

    cd /tmp; wget https://ja.wordpress.org/latest-ja.tar.gz
    cd /var/www/html
    tar zxvf /tmp/latest-ja.tar.gz
    mv wordpress/* .
    chown -R apache:apache .

続いて、DBの設定をします。wordpressからアクセスするためのユーザーを作成する必要があります。

    # mysql -u root -p
    > create database wordpress default charset utf8;
    > grant all on wordpress.* to 'wpuser'@'localhost' identified by 'mywpuserpass';

    # cp /var/www/html/wp-config-sample.php /var/www/html/wp-config.php
    # vi /var/www/html/wp-config.php
    >> define('DB_NAME', 'wordpress');
    >> define('DB_USER', 'wpuser');
    >> define('DB_PASSWORD', 'mywpuserpass');

設定を反映するため、一度apacheを再起動します。

    # systemctl restart httpd.service

これで、wordpressサイトにアクセスできるようになります。

    http://xxx.xxx.xxx.xxx/

サイト名称等の設定を入れると、管理画面（ダッシュボード）に入れるようになるので、サンプル記事を削除して、一般設定からサイトの説明を追記します。
あとは、最初の記事をアップロードして、ちゃんと登録できれば問題無しです。

※IPアドレスは、作成したECSインスタンスのpublic IPアドレスです。デモではサイトURLはIPアドレスのままとします。
※「パーマリンク設定」は”基本”にします。

## Apsara DB作成メモ

Apsara DB作成時のメモです。

- インスタンスを作成します。このとき、RegionはECSと同じRegionを選ぶようにします。
- 容量を10GBにして、性能キャパシティは最低限のものを選択します。

次に、作成されたApsara DBを使うための設定を行います。

- ESCのintranet側のアドレス（10.X）を覚えておく
- whitelistを設定。グループを追加して、ECSのintranetアドレスを記載。
- アカウント管理画面から、アカウントを作る（wpuser）
- データベース（wordpress）を作成し、wpuserでアクセスできるように権限を追加する

以上が完了したら、ECSからアクセスしてみることが可能になります。

     # mysql -u wpuser -p -h rm-j6c1z4q3tpip76a5c.mysql.rds.aliyuncs.com wordpress
     ★コマンドライン例。ホスト名は構築時に変わる。

Apsara DBへの切り替え（データ移行）について。「wordpress」データベースをダンプして、リストアします。

    # mysqldump -u wpuser -p wordpress > /tmp/wordpress.dmp
    # mysql -u wpuser -p -h rm-j6c1z4q3tpip76a5c.mysql.rds.aliyuncs.com wordpress < /tmp/wordpress.dmp

これでDBが移行されました。

続いて、wordpressの参照先DBを変更します。ホスト名を、Apsara DBで作成されたホスト名に変更します。

    # vi /var/www/html/wp-config.php
    >> define('DB_HOST', 'rm-j6c1z4q3tpip76a5c.mysql.rds.aliyuncs.com');

設定を反映するため、apacheをリスタートします。

    # systemctl restart httpd.service

ブラウザでアクセスして、記事が参照できること、管理画面からの操作に問題が無いこと、記事が投稿可能なことを確認します。

再度に、不要となったMariaDBを停止し、アクセスに問題が無いことを確認します。
（Apsara DBが使われていることの確認）

    # systemctl disable mariadb.service
    # systemctl stop mariadb.service

Apsara DBのモニターを設定して、IOPS等の性能グラフが確認できることを確認します。

## SLB作成メモ

SLBを作成するメモです。

- SLBを作る：SLB新規インスタンスを作成します。
- HTTPのリスナーを作る：リスナー設定を追加します。
- バックエンドサーバーにECSを追加：ECSリストから選択して指定することができます。
- ヘルスチェック用ファイルを置く：ヘルスチェックでOKになります。
- ブログサーバーのアドレスをSLBのアドレスに更新：SLB経由でのアクセスが可能になります。

ヘルスチェックファイルはここでは「check.html」としています。

     # touch /var/www/html/check.html

今までECSのグローバルにアクセスしていましたが、SLBのEIPにアクセスすると、サーバーにアクセスできるようになります。
wordpressの「一般設定」で、URLの設定をSLBのIPアドレスをベースとしたアドレスに変更すれば、完了です。

※Apacheのログ上、アクセス元IPが内部アドレスになってしまうため、クライアントIPをログに出力するには、「X-FORWARDED-FOR」ヘッダをログに出力させるなどの対応が必要になります。

ヘルスチェックファイルを削除すると、ヘルスチェックが失敗して、サーバーへのアクセスが閉塞されます。

※メンテナンスサイト構築例を示します。

## セキュリティ設定の確認

「セキュリティグループ」について：一般的なネットワークレベルのファイヤーウォール設定が可能です。必要なサービスへのアクセスのみを開放することが可能です（HTTPやSSH等）。

一般ユーザーを作成して、作業環境を整えてみます。

    # useradd -G wheel -m -c 'Takeshi SHIMADA' shimada
    # passwd shimada

この状態で、作成した「shimada」ユーザーでアクセスしてみます。

一般ユーザーでログインして、管理者権限への昇格が可能であることを確認します。

    # su - ★rootになれることの確認
    # sudo vi /etc/hosts ★sudoコマンドが利用可能なことを確認

ここまで確認できたところで、管理者ユーザー（root）アカウントで直接ログインできないようにsshの設定を変更します。

    # vi /etc/ssh/sshd_config
    >> PermitRootLogin no

設定を反映するために、sshdサービスを再起動します。

    # systemctl restart sshd

※リスタート後、一般ユーザーでのログイン〜管理者権限への昇格が可能かどうかを再度確認し、システム操作が可能であることを確認します。

さらにセキュリティを強化するために、RSA認証の有効化と、SSHポートの変更を実施します（ここではtcp/22→tcp/10022に変更してみます）。

- セキュリティグループの設定を変更し、「tcp/10022」を開放します（tcp/22とtcp/10022が開放されたグループを一時的に作成します）
- ECSインスタンスのセキュリティグループを変更します。

一般ユーザーの公開鍵設定を行います。

    % mkdir .ssh
    % chmod 700 .ssh
    % vi .ssh/authorized_keys
    >>> paste public key of user
    % chmod 600 .ssh/authorized_keys

この状態で、公開鍵認証でログインできることを確認します。確認できたら、sshの設定変更を行います。

    # vi /etc/ssh/sshd_config
    >> Port 10022
    >> PasswordAuthentication no

サービスをリスタートして、設定を反映します。

    # systemctl restart sshd

- 改めて、tcp/10022で公開鍵認証で、ログイン可能なこと、管理者権限に昇格でいることを確認します。
- 最後に、「セキュリティグループ」設定を編集し、「tcp/22」を閉じます。

これで完了です。

## プライベートイメージの作成と、プライベートイメージからのECS作成について

ここまでで構築された環境をプライベートイメージとして保存し、このイメージを利用して、ECSを構成してみます。

- ECSのコンソールからシステムディスクを選択し、「create snapshot」をクリックします。snapshotには名前が付けられますので、判別可能な名前をつけます。
- ECSメニューの「All snapshots」から、snapshotの作成状況が分かります（※結構時間かかります）。
- snapshotが完了したら、snapshotから「Create User-defined Image」をクリックして、カスタムイメージを作成します。

- ECSを作成します。Imageとして、custom imageが選択できるようになっています（同一Regionでのみ可能）。
- IPアドレスやホスト名以外、同じ構成のサーバーができます。

※custom imageが使えるのはシステムディスクだけです。データディスク（追加ディスク）はimage化できません。

## Release setting

最後にこれまで作成したインスタンス情報を削除します。
AlibabaCloudにおけるリソースの削除・開放は「Release setting」（設定の開放）という表記がされていますので、各々このメニューを選択して設定を開放できます。

- ECS：メニューに「Release setting」があります。こちらからリリースできます。
- Apsara DB：管理画面に「Release」ボタンがあります。こちらからリリースできます。
- SLB：メニューに「Release setting」があります。こちらからリリースできます。

## まとめ

基本的なECSの操作と、アプリケーション実行環境作成に係るオペレーションは以上の通りです。
AlibabaCloudには他にも便利な機能がありますので、いろいろ試してみてください。
