# Steps to deploy a Dockerized Rail app to Heroku
---
## Prerequisites
1. Install Docker
1. Install Ruby
1. `gem install orats`
1. Install Heroku Toolbelt
---
## Build image
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
@[1-4]
@[6-8]
@[10-11]
@[13]
+++
![Waitforit](https://i.pinimg.com/originals/c4/32/6f/c4326fa27456770263a4df5bd9d7a4c3.gif)

[Musics](https://youtu.be/VBlFHuCzPgY)
---
## Get it working locally
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
```
@[1-3]
@[5-6]
@[8]
@[10-11]
---
## Deploy to Heroku
```
# Download Dockerfile.web from this gist
curl -O https://gist.githubusercontent.com/gabrieljoelc/8c04941042e9241a41b840cccf1ad5fb/raw/Dockerfile.web
git add . && git commit -m "Add production Dockfile"
```
+++
```
# Creates a heroku app and adds heroku git remote
heroku create
heroku addons:create heroku-postgresql
```
+++
```
# Manually set Rails/orats environment variables in Heroku app
heroku config:set \
SECRET_TOKEN=`docker-compose exec website rails secret` \
ACTION_CABLE_ALLOWED_REQUEST_ORIGINS=`heroku apps:info | grep "Web URL:" | sed -e 's/^Web URL:        //'` \
RAILS_SERVE_STATIC_FILES=true
```
+++
```
heroku container:login
```
+++
```
# instead of doing `git push master heroku` like usual, we do the following:
heroku container:push -R
```
+++
![Please stand by](https://i.makeagif.com/media/9-03-2015/mPJpu9.gif)

[Fresh Beats](https://youtu.be/G2rLmGdDcUM)
---
## It worky
```
# load schema into Heroku instance
heroku run rails db:schema:load
```
+++
```
heroku open
```
---
## Resources
- https://github.com/nickjj/orats
- https://devcenter.heroku.com/articles/container-registry-and-runtime
