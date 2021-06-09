https://developer.microsoft.com/en-us/microsoft-edge/tools/vms/から入手できる。

kaliならsudo apt install metasploit

ubuntuの場合metasploit

```
#rubyをインストール
sudo apt install -y build-essential libreadline5 libreadline-dev libssl-dev libpcap-dev libxml2-dev libxslt1-dev libyaml-dev libsqlite3-dev postgresql libpq5 libpq-dev subversion git git-core autoconf curl zlib1g-dev
git clone https://github.com/sstephenson/rbenv.git ~/.rbenv
git clone https://github.com/sstephenson/ruby-build.git ~/.rbenv/plugins/ruby-build
echo 'export PATH="~/.rbenv/bin:$PATH"' >> ~/.bashrc
echo 'eval "$(rbenv init -)"' >> ~/.bashrc
source ~/.bashrc
rbenv -v #versionを確認
rbenv install 2.7.2
rbenv global 2.7.2
```

```
# Postgresql
sudo su postgres
createuser msf -P -S -R -D
createdb -O msf msf
exit
```

```
# metasploit
git clone https://github.com/rapid7/metasploit-framework.git
cd metasploit-framework
sudo apt install ruby
sudo gem install bundler
bundle install #rbenvのバージョンが違うと怒られる。
```

```
#bundle installでエラーが出た時
rbenv uninstall 2.7.2
rbenv install <エラー文に書かれたバージョン>
rbenv global <エラー文に書かれたバージョン>
```

Metasploitデータベースの設定

```
#config/database.yml
production:
  adapter: postgresql
  database: msf
  username: msf
  password: <postpresqlで決めたパスワード>

  host: 127.0.0.1
  port: 5432
  pool: 75
  timeout: 5

```
