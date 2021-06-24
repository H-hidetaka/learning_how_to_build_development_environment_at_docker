# learning_how_to_build_development_environment_at_docker

Nginxのインストール
RUN apt-get install -y nginx

Nginxのデーモン実行
CMD nginx -g 'daemon off;'

RUN命令は、Dockerfileの作成では、最も多く使います。
RUN [実行したいコマンド]

Shell形式
RUN apt-get install -y nginx
RUN ["/bin/bash","-c","apt-get install -y nginx"]

CMD　[実行したいコマンド]

Exec形式
CMD ["nginx", "-g", "deamon off;"]

Shell形式
CMD nginx -g 'daemon off;'
http://nginx.org/en/docs/switches.html


環境・ネットワークの設定
1. key value型で指定する場合
ENV [key] [value]

ENV [key]=[value]
ENV value1=value1 \
    value2=value2

ポートの設定（EXPOSE命令）
EXPOSE <ポート番号>

8080ポートを公開するためのEXPOSE命令
EXPOSE 8080

ファイルの設定
Dockerfileでファイルを取り扱うための命令
COPY <ホストのファイルパス> <Dockerイメージのファイルパス>
COPY ["<ホストのファイルパス>" "<Dockerイメージのファイルパス>"]

＜ADD命令はリモートファイルのダウンロードやアーカイブの解凍などの機能を持ちますが、COPY命令は、ホスト上のファイルイメージ内に「コピーする」処理だけを行います。そのため単純に、イメージ内にファイルを配置したいだけの時は、COPY命令を使います。＞

VOLUME ["/マウントポイント"]
設定できる値は、VOLUME ["/var/log/"]のようなJSON配列、もしくはVOLUME /var/logやVOLUME /var/log /var/dbのような複数の引数の文字を指定できます。
永続データはDockerのホストマシン上のボリュームにマウントするか、共有ストレージをボリュームとしてマウント可能です。

Docker Composeを使用したRuby on Railsの開発環境構築
Mac OS
Ruby2.6.3
Rails5.2.3
MySQL5.7

Ruby on RailsのWebアプリケーションサーバとMySQLのデータベースサーバの２つのコンテナを作成します。
mkdir ex5_2
cd ex5_2

Dockerfileを作成
touch Dockerfile
Dockerfile
FROM ruby:2.6.3

RUN apt-get update -qq && \
    apt-get install -y build-essential \
                       libpq-dev \
                       nodejs

RUN mkdir /app_name

##作業ディレクトリ名を環境変数APP_ROOTに割り当てて、以下$APP_ROOTで参照
ENV APP_ROOT /app_name
WORKDIR $APP_ROOT

ADD ./Gemfile $APP_ROOT/Gemfile
ADD ./Gemfile.lock $APP_ROOT/Gemfile.lock

RUN bundle install
ADD . $APP_ROOT

3. Gemfileを作成
touch Gemfile
Gemfile
source 'https://rubygems.org'
gem 'rails', '5.2.3'

4. 空のGemfile.lockを作成
 touch Gemfile.lock

5. docker-compose.ymlの作成
6. touch docker-compose.yml
version: '3' # バージョンを指定
services:
  db: # データベースサーバ用のコンテナの設定を記述
    image: mysql:5.7 # コンテナで使用するイメージ名を記述します。
    environment:
      MYSQL_ROOT_PASSWORD: password # 任意のパスワードを設定
      MYSQL_DATABASE: root # 任意のデータベース名を設定
    ports:
      - "3306:3306" # ホストの3306ポートとコンテナの3306ポートを接続します。

  web: # アプリケーションサーバ用のコンテナの設定を記述
    build: . # docker-compose.ymlと同じ階層にあるDockerfileを使ってイメージをビルドするための記述です。
    command: bundle exec rails s -p 3000 -b '0.0.0.0' # コンテナ立ち上げ時に起動するコマンドです。railsを実行します。

    volumes:
      - .:/app_name # 作業ディレクトリをコンテナ内の/app_nameにマウントします。
    ports:
      - "3000:3000" # ホストの3000ポートとコンテナの3000ポートを接続します。
    links:
      - db # dbコンテナとの接続を記述します。

CLI
docker-compose run web rails new . --force --database=mysql --skip-bundle

7. database.ymlを修正する
rails new で作成されたデータベースの接続設定ファイルにはホスト名とパスワードが設定されていません。railsアプリケーションがデータベースに接続できるようにするため、
ex5_2/config/database.ymlの１２行目から１８行目までの内容を以下のように編集します。
yml
修正前
default: &default
  adapter: mysql2
  encoding: utf8
  pool: <%= ENV.fetch("RAILS_MAX_THREADS") { 5 } %>
  username: root
  password:
  host: localhost
修正後
default: &default
  adapter: mysql2
  encoding: utf8
  pool: <%= ENV.fetch("RAILS_MAX_THREADS") { 5 } %>
  username: root
  password: password # docker-compose.ymlのMYSQL_ROOT_PASSWORD
  host: db # docker-compose.ymlのservice名
8. Docker起動
docker-compose up -d
ここで実行するdocker-composeコマンドは「docker-compose.yml」ファイルを探して実行が進むので、「docker-compose.yml」ファイルがあるディレクトリ上で実行する必要があります。）
docker compose upは、イメージがなければイメージのビルドを行い、コンテナの起動までを行うコマンドです。
（イメージが無ければコンテナが起動できないため、イメージが無い場合はイメージのビルドを行います。）
オプションの-dはバックグラウンドでコンテナを起動するためのオプションです。

9. データベース作成
docker-compose run web rails db:create

bundle install
