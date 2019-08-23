#PHP Storm Connect via ssh
ssh-agent -s
ssh-add
ssh-add ~/.ssh/id_rsa

https://www.yougetsignal.com/tools/open-ports/

ufw status verbose
firewall-cmd --state
iptables --line-numbers -vL
iptables -S
root@ubuntu-s-1vcpu-1gb-nyc3-01:~# netstat -plunt
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 0.0.0.0:3306            0.0.0.0:*               LISTEN      3339/mysqld         
tcp        0      0 127.0.0.1:6379          0.0.0.0:*               LISTEN      24035/redis-server  
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      22827/nginx: master 
tcp        0      0 127.0.0.53:53           0.0.0.0:*               LISTEN      646/systemd-resolve 
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      889/sshd            
tcp        0      0 127.0.0.1:5432          0.0.0.0:*               LISTEN      23888/postgres      
tcp        0      0 0.0.0.0:25              0.0.0.0:*               LISTEN      7492/master         
tcp        0      0 0.0.0.0:443             0.0.0.0:*               LISTEN      22827/nginx: master 
tcp        0      0 127.0.0.1:33373         0.0.0.0:*               LISTEN      868/containerd      
tcp6       0      0 ::1:6379                :::*                    LISTEN      24035/redis-server  
tcp6       0      0 :::80                   :::*                    LISTEN      22827/nginx: master 
tcp6       0      0 :::22                   :::*                    LISTEN      889/sshd            
tcp6       0      0 :::25                   :::*                    LISTEN      7492/master         
udp        0      0 127.0.0.53:53           0.0.0.0:*                           646/systemd-resolve 

postgres  127.0.0.1:5432  is not considered open

nmap -sS -Pn -p- -T4 -vv --reason greatlakescode.us

#Allow Postgres Remote
##Create User
su - postgres

createuser --interactive --pwprompt
GRANT CONNECT ON DATABASE mastodon_production TO greatlakescode;
GRANT USAGE ON SCHEMA public TO greatlakescode;
GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO greatlakescode;
GRANT ALL PRIVILEGES ON ALL SEQUENCES IN SCHEMA public TO greatlakescode;

ALTER USER greatlakescode WITH ENCRYPTED PASSWORD 'password';

psql mastodon_production --host=127.0.0.1 --username=greatlakescode


Probably not the best idea. Add password instead.

https://blog.bigbinary.com/2016/01/23/configure-postgresql-to-allow-remote-connection.html

nano /etc/postgresql/10/main/postgresql.conf 
listen_addresses = '*'

service postgresql restart

netstat -plunt

#Add Swap Space
https://linuxize.com/post/how-to-add-swap-space-on-ubuntu-18-04/

sudo fallocate -l 1G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

##Edit fstab
sudo nano /etc/fstab
/swapfile swap swap defaults 0 0


sudo swapon --show

#Node
curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.32.1/install.sh | bash

apt-install nodejs

curl -sL https://deb.nodesource.com/setup_8.x | bash -
curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | apt-key add -

npm cache clean -f
npm install -g n
n stable

error mastodon@: The engine "node" is incompatible with this module. Expected version ">=8.12 <12". Got "8.10.0"


mkdir /www
chown www-data:www-data /www
cd /www
git clone git@github.com:russjohnson09/mastodon.git

chown www-data:www-data -R /www/mastodon

#setup
https://docs.joinmastodon.org/administration/installation/#setting-up-nginx

curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | apt-key add -

apt update
apt install -y \
  imagemagick ffmpeg libpq-dev libxml2-dev libxslt1-dev file git-core \
  g++ libprotobuf-dev protobuf-compiler pkg-config nodejs gcc autoconf \
  bison build-essential libssl-dev libyaml-dev libreadline6-dev \
  zlib1g-dev libncurses5-dev libffi-dev libgdbm5 libgdbm-dev \
  nginx redis-server redis-tools postgresql postgresql-contrib \
  certbot python-certbot-nginx yarn libidn11-dev libicu-dev libjemalloc-dev


adduser --disabled-login mastodon
su - mastodon

mastodon@ubuntu-s-1vcpu-1gb-nyc3-01:~$ pwd
/home/mastodon


#Clone Repo and Setup Ruby
su - mastodon
cd ~
pwd
/home/mastodon
git clone git@github.com:russjohnson09/mastodon.git live

git clone https://github.com/rbenv/rbenv.git ~/.rbenv
cd ~/.rbenv && src/configure && make -C src
echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bashrc
echo 'eval "$(rbenv init -)"' >> ~/.bashrc
exec bash
git clone https://github.com/rbenv/ruby-build.git ~/.rbenv/plugins/ruby-build

RUBY_CONFIGURE_OPTS=--with-jemalloc rbenv install 2.6.1
rbenv global 2.6.1

gem update --system

##Install Bundler Used by Repo
gem install bundler --no-document
exit

#postgres
sudo -u postgres psql

CREATE USER mastodon CREATEDB;
\q


#Setup
Setup is done using budle. https://bundler.io/

And yarn installs the lock file for other 

chown mastodon:www-data /www/mastodon -R

su - mastodon

cd /www/mastodon


##Install Ruby Packages
bundle install \
  -j$(getconf _NPROCESSORS_ONLN) \
  --deployment --without development test
  
##Install npm packages
yarn install --pure-lockfile

##Database migrations using ruby rake
RAILS_ENV=production bundle exec rake mastodon:setup

Username: admin
E-mail: russjohnson09@gmail.com
You can login with the password: 3044ab1c0c69a5d4f2c119549b8f5fee


exit

#Recompile If Necessary
RAILS_ENV=production bundle exec rails assets:precompile


#Test Web Service
su - mastodon
cd /home/mastodon/mastodon
Services in 
/mastodon/dist

should have their home directory set properly.

RAILS_ENV=production /home/mastodon/.rbenv/shims/bundle exec puma -C config/puma.rb
curl localhost:3000

#Setup Services
cp /home/mastodon/mastodon/dist/mastodon-*.service /etc/systemd/system/
systemctl start mastodon-web mastodon-sidekiq mastodon-streaming
systemctl enable mastodon-*
systemctl daemon-reload

systemctl restart mastodon-web
systemctl restart mastodon-sidekiq
systemctl restart mastodon-streaming

systemctl status mastodon-web
systemctl status mastodon-sidekiq
systemctl status mastodon-streaming




sudo -u mastodon stat /home/mastodon/.rbenv/shims/bundle

/home/mastodon/.rbenv/shims/bundle is not a directory

service --status-all

RAILS_ENV=production PORT=3000 /home/mastodon/.rbenv/shims/bundle exec puma -C config/puma.rb


#Setup Nginx
cp /home/mastodon/mastodon/dist/nginx.conf /etc/nginx/sites-available/mastodon.conf
ln -s /etc/nginx/sites-available/mastodon.conf /etc/nginx/sites-enabled/mastodon.conf

service nginx restart

curl http://mastodon.greatlakescode.us -Lv


![Mastodon](https://i.imgur.com/NhZc40l.png)
========

[![GitHub release](https://img.shields.io/github/release/tootsuite/mastodon.svg)][releases]
[![Build Status](https://img.shields.io/circleci/project/github/tootsuite/mastodon.svg)][circleci]
[![Code Climate](https://img.shields.io/codeclimate/maintainability/tootsuite/mastodon.svg)][code_climate]
[![Crowdin](https://d322cqt584bo4o.cloudfront.net/mastodon/localized.svg)][crowdin]
[![Docker Pulls](https://img.shields.io/docker/pulls/tootsuite/mastodon.svg)][docker]

[releases]: https://github.com/tootsuite/mastodon/releases
[circleci]: https://circleci.com/gh/tootsuite/mastodon
[code_climate]: https://codeclimate.com/github/tootsuite/mastodon
[crowdin]: https://crowdin.com/project/mastodon
[docker]: https://hub.docker.com/r/tootsuite/mastodon/

Mastodon is a **free, open-source social network server** based on ActivityPub. Follow friends and discover new ones. Publish anything you want: links, pictures, text, video. All servers of Mastodon are interoperable as a federated network, i.e. users on one server can seamlessly communicate with users from another one. This includes non-Mastodon software that also implements ActivityPub!

Click below to **learn more** in a video:

[![Screenshot](https://blog.joinmastodon.org/2018/06/why-activitypub-is-the-future/ezgif-2-60f1b00403.gif)][youtube_demo]

[youtube_demo]: https://www.youtube.com/watch?v=IPSbNdBmWKE

## Navigation

- [Project homepage 🐘](https://joinmastodon.org)
- [Support the development via Patreon][patreon]
- [View sponsors](https://joinmastodon.org/sponsors)
- [Blog](https://blog.joinmastodon.org)
- [Documentation](https://docs.joinmastodon.org)
- [Browse Mastodon servers](https://joinmastodon.org/#getting-started)
- [Browse Mastodon apps](https://joinmastodon.org/apps)

[patreon]: https://www.patreon.com/mastodon

## Features

<img src="https://docs.joinmastodon.org/elephant.svg" align="right" width="30%" />

**No vendor lock-in: Fully interoperable with any conforming platform**

It doesn't have to be Mastodon, whatever implements ActivityPub is part of the social network! [Learn more](https://blog.joinmastodon.org/2018/06/why-activitypub-is-the-future/)

**Real-time, chronological timeline updates**

See the updates of people you're following appear in real-time in the UI via WebSockets. There's a firehose view as well!

**Media attachments like images and short videos**

Upload and view images and WebM/MP4 videos attached to the updates. Videos with no audio track are treated like GIFs; normal videos are looped - like vines!

**Safety and moderation tools**

Private posts, locked accounts, phrase filtering, muting, blocking and all sorts of other features, along with a reporting and moderation system. [Learn more](https://blog.joinmastodon.org/2018/07/cage-the-mastodon/)

**OAuth2 and a straightforward REST API**

Mastodon acts as an OAuth2 provider so 3rd party apps can use the REST and Streaming APIs, resulting in a rich app ecosystem with a lot of choice!

## Deployment

**Tech stack:**

- **Ruby on Rails** powers the REST API and other web pages
- **React.js** and Redux are used for the dynamic parts of the interface
- **Node.js** powers the streaming API

**Requirements:**

- **PostgreSQL** 9.5+
- **Redis**
- **Ruby** 2.4+
- **Node.js** 8+

The repository includes deployment configurations for **Docker and docker-compose**, but also a few specific platforms like **Heroku**, **Scalingo**, and **Nanobox**. The [**stand-alone** installation guide](https://docs.joinmastodon.org/administration/installation/) is available in the documentation.

A **Vagrant** configuration is included for development purposes.

## Contributing

Mastodon is **free, open source software** licensed under **AGPLv3**.

You can open issues for bugs you've found or features you think are missing. You can also submit pull requests to this repository, or submit translations using Weblate. To get started, take a look at [CONTRIBUTING.md](CONTRIBUTING.md). If your contributions are accepted into Mastodon, you can request to be paid through [our OpenCollective](https://opencollective.com/mastodon).

**IRC channel**: #mastodon on irc.freenode.net

## License

Copyright (C) 2016-2019 Eugen Rochko & other Mastodon contributors (see [AUTHORS.md](AUTHORS.md))

This program is free software: you can redistribute it and/or modify it under the terms of the GNU Affero General Public License as published by the Free Software Foundation, either version 3 of the License, or (at your option) any later version.

This program is distributed in the hope that it will be useful, but WITHOUT ANY WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the GNU Affero General Public License for more details.

You should have received a copy of the GNU Affero General Public License along with this program. If not, see <https://www.gnu.org/licenses/>.
