Linux仮想環境構築 講習会
=======================

## 目的

- 簡単にLinuxを構築できる方法を提示し、少しでもLinuxを経験できるようにする
- tomcatサーバの構築を行い、サーバ構築の経験を積む

```
Note.
バックエンドのサーバの多くは、Linuxが利用されている
```

## 講習対象者

- Linuxを触ってみたいと考えている人
- ローカル環境で簡単に仮想環境を構築したい人
- tomcatサーバを構築してみたい人

## 講習内容

- vagrantで仮想環境を構築する
- LinuxでWeb/APサーバ(tomcat)の構築を行う
- 自分が作成したソース(Servlet等)を自分が構築したサーバ(仮想環境)上で動かせるようにする


## 検証環境

ホストOS
- Windows10
- Vagrant 2.2.6
- VirtualBox 5.2.34 (※ver6だと動かない可能性あり)

ゲストOS
- Ubuntu 16.04
- Tomcat 9.0.0
- openjdk-8

# Linux仮想環境を構築する

VirtualBox、Vagrantをインストール済み(説明割愛)

## リポジトリをclone

```
git clone http://...
```

## vagrantで仮想環境を構築

`Vagrantfile`があるディレクトリで以下のコマンドを実行(※初回実行は時間がかかります)
```
vagrant up
vagrant provision
```

仮想環境にログイン
```
vagrant ssh
```

## Linux仮想環境にTomcatをインストールする

必要なツールをインストール
```
sudo apt-get update
sudo apt-get -y install curl
sudo apt-get -y install python-software-properties debconf-utils
```

インターネットからTomcatをダウンロード
```
curl -O --progress-bar http://archive.apache.org/dist/tomcat/tomcat-9/v9.0.0.M22/bin/apache-tomcat-9.0.0.M22.tar.gz
```

ダウンロードしたTomcatを解凍
```
sudo mkdir /opt/tomcat
sudo tar xzvf apache-tomcat-9*tar.gz -C /opt/tomcat --strip-components=1
```

`tomcat`実行用のユーザーを作成
```
sudo groupadd tomcat
sudo useradd -s /bin/false -g tomcat -d /opt/tomcat tomcat
```

tomcatのインストールディレクトリのアクセス権限を変更し、所有者を`tomcat`に変更
```
cd /opt/tomcat
sudo chgrp -R tomcat /opt/tomcat
sudo chmod -R g+r conf
sudo chmod g+x conf
sudo chown -R tomcat webapps/ work/ temp/ logs/
```

調査用に`logs`フォルダは`vagrant`ユーザーでも見れるようにしておく
```
sudo chmod -R o+r logs
sudo chmod o+x logs
```

## Open jdk 8をインストール

```
sudo add-apt-repository -y ppa:webupd8team/java && sudo apt-get update
sudo apt-get install -y openjdk-8-jre
```

## tomcatサービスを作成

以下をコマンドを実行し、`tomcat.service`を作成する
```
cat <<-EOF > /home/vagrant/tomcat.service
[Unit]
Description=Apache Tomcat Web Application Container
After=network.target

[Service]
Type=forking

Environment=JAVA_HOME=/usr
Environment=CATALINA_PID=/opt/tomcat/temp/tomcat.pid
Environment=CATALINA_HOME=/opt/tomcat
Environment=CATALINA_BASE=/opt/tomcat
Environment='CATALINA_OPTS=-Xms512M -Xmx1024M -server -XX:+UseParallelGC'
Environment='JAVA_OPTS=-Djava.awt.headless=true -Djava.security.egd=file:/dev/./urandom'

ExecStart=/opt/tomcat/bin/startup.sh
ExecStop=/opt/tomcat/bin/shutdown.sh

User=tomcat
Group=tomcat
UMask=0007
RestartSec=10
Restart=always

[Install]
WantedBy=multi-user.target
EOF
```

`tomcat.service`をコピーし、daemonをリロードする
```
sudo cp /home/vagrant/tomcat.service /etc/systemd/system/
sudo systemctl daemon-reload
```

tomcatサービスを開始する
```
sudo systemctl start tomcat
```

ファイアーウォールの8080番を許可する
```
sudo ufw allow 8080
```

tomcatサービスの自動起動を有効にする
```
sudo systemctl enable tomcat
```

# 動作確認

ホスト側のOSでブラウザを開き、http://localhost:8081 にアクセスする  
tomcatのスタートページが表示されたら無事完了

