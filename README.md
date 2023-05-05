# WARPTRUYEN
## Things you may want to cover: ##

0. **rbenv**

```
- rbenv install [ruby_version]
- rbenv global [ruby_version]
- rbenv rehash
```

1. **Start postgresql:**

```
- pg_lsclusters
- sudo pg_ctlcluster [ver] main start // Ex: sudo pg_ctlcluster 12 main start

* using psql
- sudo -i -u postgres psql
- \l
```
2. **Run project:**

```
- rake db:create
- rake db:migrate
- rails s
- run: localhost:3000
```

3. **Services (job queues, cache servers, search engines, etc.)**
- start Background job
install redis and start redis-server:

```
- sudo service redis-server start/ redis-server
- service --status-all
```
`bundle exec sidekiq`
rails c and run example to see it run:
 `CreateProductWorker.perform_async()`

4. **Install chromium**

```
- wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
- apt install ./google-chrome-stable_current_amd64.deb
```

5. **start cronjob on wsl - ubuntu**

```bash
- whenever --update-crontab
- whenever -f # not sure

# config cronjob for wsl
- sudo service rsyslog start
- sudo service cron start
- sudo usermod -a -G crontab (username)
- grep -i cron /var/log/syslog

- sudo service cron stop
```

6. **create seed file (https://github.com/rroblak/seed_dump)**

```
- rake db:seed:dump # Dump all data directly to db/seeds.rb:*/
- rake db:seed #load seed data
```

## Working with server ##

- Create Ubuntu
- ssh root@[ip]

1. **Create deploy user**

```bash
- apt update && apt upgrade
- adduser deploy
- usermod -aG sudo cardeploy
# optional
- vim /etc/ssh/sshd_config
# CHANGE `PermitRootLogin no`
# CHANGE `PasswordAuthentication no`

- sudo service ssh restart
```
>>> SU deploy user
4. **Install Nginx** `https://www.digitalocean.com/community/tutorials/how-to-install-nginx-on-ubuntu-20-04`

In case got error `deploy user is not in sudoer`, need set chmode for this user

```bash
- su root 
- vim /etc/sudoers # ADD [deploy] ALL=(ALL:ALL) ALL
- cd /home
- chmod o=rx /home/deploy #deploy is user folder
```

```bash
- sudo apt update #su root
- sudo apt install nginx

#List the application configurations that ufw knows how to work with by typing:
- sudo ufw app list
```

3. **Install rbenv and Ruby** `https://www.digitalocean.com/community/tutorials/how-to-install-ruby-on-rails-with-rbenv-on-ubuntu-20-04`

```bash
- sudo apt update
- sudo apt install git curl libssl-dev libreadline-dev zlib1g-dev autoconf bison build-essential libyaml-dev libreadline-dev libncurses5-dev libffi-dev libgdbm-dev
- curl -fsSL https://github.com/rbenv/rbenv-installer/raw/HEAD/bin/rbenv-installer | bash
- echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bashrc
- echo 'eval "$(rbenv init -)"' >> ~/.bashrc
- source ~/.bashrc
- type rbenv
# list all the available versions of Ruby
- rbenv install -l
- rbenv install [3.2.1]
- rbenv global [3.2.1]
- ruby -v
```

3. **Install postgres**

```bash
- sudo apt-get update
- sudo apt-get install postgresql postgresql-contrib libpq-dev

#Create Database User
- sudo -u postgres createuser [deploy] -s
- sudo -u postgres psql

postgres=# \password deploy
postgres=# create database appname;
postgres=# \q
```

2. **Create ssh-key and connect git**

```bash
- ssh-keygen -t rsa -b 4096 / ssh-keygen -t ed25519
- cat ~/.ssh/id_rsa.pub ========= cat ~/.ssh/id_ed25519.pub
# copy ssh-key to github
- ssh -T git@github.com
- touch ~/.ssh/authorized_keys
- chmod 700 ~/.ssh && chmod 600 ~/.ssh/authorized_keys
- copy [~/.ssh/id_rsa.pu] from local into ~/.ssh/authorized_keys
```

5. **Install Node**

```bash
- sudo apt update
- curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash - && sudo apt-get install -y nodejs
- node -v
```

6. **Install yarn**

```bash
- sudo chown -R $USER /usr/lib/node_modules/
- sudo chown -R $USER /usr/bin/
- npm install --global yarn
- yarn --version
- chown root:root /usr/bin/sudo && chmod 4755 /usr/bin/sudo
```

7. **Install Redis**
`https://www.digitalocean.com/community/tutorials/how-to-install-and-secure-redis-on-ubuntu-18-04`

```bash
- sudo apt update
- sudo apt install redis-server
- sudo vim /etc/redis/redis.conf # Change `supervised systemd` and uncomment bind 127.0.0.1 ::1

- sudo systemctl restart redis.service

#  Testing Redis
- sudo systemctl status redis
- redis-cli
-- ping
- sudo systemctl restart redis
```

**Sidekiq**

```bash
- sudo systemctl enable sidekiq # run one time when config sidekiq
- sudo systemctl stop sidekiq
- sudo systemctl start sidekiq
- sudo systemctl restart sidekiq
- systemctl kill -s TSTP sidekiq

- journalctl -u sidekiq -rn 100 #view the last 100 lines of log output.
```

**Capistrano**

`To Prepare code to server`

```bash
# copy maskey from local to remote server
- scp config/master.key deploy@172.104.173.189:/home/deploy/apps/appname/shared/config/
- bundle exec cap production deploy
```

**Production setup Puma & nginx**

```bash
# create puma service
- cd /etc/systemd/system/
- sudo touch puma_[appname]_production.service
# copy code from local puma.rb to file puma_[appname]_production.service
- sudo vim puma_appname_production.service

# local terminal run
- cap production puma:make_dirs

# ssh nginx
- cd /etc/nginx/sites-available/
- sudo touch appname.conf
- sudo vim appname.conf # copy nginx.conf in pj to file
- sudo rm default

- cd /etc/nginx/sites-enabled/
- sudo ln -s /etc/nginx/sites-available/appname.cof /etc/nginx/sites-enabled/
- sudo rm default

# test nginx
- sudo nginx -t

# retstart & check status nginx
- sudo systemctl restart nginx
- sudo ufw app list

-  cd /etc/nginx/sites-enabled/
```

***DEPLOY PRODUCTION***

```bash
# finally run
- cap production deploy

# at ssh term
- /usr/bin/env sudo /bin/systemctl restart puma_appname_production
- /usr/bin/env sudo /bin/systemctl status puma_appname_production  #check status puma
- sudo systemctl restart sidekiq

```