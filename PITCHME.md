# Steps to deploy Dockerized Rails app to Heroku

## Prerequisites
1. Install Docker
1. Install Ruby
1. `gem install orats` (see https://github.com/nickjj/orats)
1. Install Heroku Toolbelt

## Steps
```
# Terminal A in ~/dev
orats new herocker
cd herocker
git init && git add . && git commit -m "Scoffald from orats"
# Download puma.rb from this gist
curl -o config/puma.rb https://gist.githubusercontent.com/gabrieljoelc/8c04941042e9241a41b840cccf1ad5fb/raw/puma.rb
git add . && git commit -m "Split BIND_ON from PORT for puma binding"

# move port from BIND_ON to PORT
vi .env

docker-compose up --build
```
https://youtu.be/VBlFHuCzPgY
```
# Terminal B
cd ~/dev/herocker
git add . && git commit -m "Add Gemfile.lock"

# will have error because database isn't setup
open http://localhost:3000

docker-compose exec website rails db:create db:migrate

# should work locally now
open http://localhost:3000

git add . && git commit -m "Add schema file"

# Download Dockerfile.web from this gist
curl -O https://gist.githubusercontent.com/gabrieljoelc/8c04941042e9241a41b840cccf1ad5fb/raw/Dockerfile.web
git add . && git commit -m "Add production Dockfile"

# Creates a heroku app and adds heroku git remote
heroku create
heroku addons:create heroku-postgresql

# Manually set Rails/orats environment variables in Heroku app
heroku config:set \
SECRET_TOKEN=`docker-compose exec website rails secret` \
ACTION_CABLE_ALLOWED_REQUEST_ORIGINS=`heroku apps:info | grep "Web URL:" | sed -e 's/^Web URL:        //'` \
RAILS_SERVE_STATIC_FILES=true

# instead of doing `git push master heroku` like usual, we do the following:
heroku container:login

# This takes a while, so switch to discussing Heroku Docker docs
heroku container:push -R
```
https://youtu.be/VBlFHuCzPgY
```
# load schema into Heroku instance
heroku run rails db:schema:load

heroku open
```
