# Deploying a Docker Rails image to Heroku

Note:
Deploying a Rails app to Heroku is dead simple. However, using Docker images as our production deployment artifacts makes it easier to migrate these apps to a different cloud host. And it's the new hotness.

---

## Prerequisites
1. Install Docker
1. Install Ruby
1. `gem install orats`
1. Install Heroku Toolbelt

---

## Build image
```
cd ~/dev
orats new herocker
cd herocker
git init && git add . && git commit -m "Scoffald from orats"

curl -o config/puma.rb https://gist.githubusercontent.com/gabrieljoelc/8c04941042e9241a41b840cccf1ad5fb/raw/puma.rb
git add . && git commit -m "Split BIND_ON from PORT for puma binding"

vi .env

docker-compose up --build
```
@[1-4](Assume we're in terminal A)
@[6-7](Download puma.rb from this gist)
@[9](We need to move the port from BIND_ON to it's own variable (PORT))
@[11](Build the image)

+++

![Waitforit](https://i.pinimg.com/originals/c4/32/6f/c4326fa27456770263a4df5bd9d7a4c3.gif)

[Musics](https://youtu.be/VBlFHuCzPgY)

Note:
- Checkout https://github.com/nickjj/orats/tree/master/lib/orats/templates/base
- Compare https://github.com/nickjj/orats/blob/master/lib/orats/templates/base/config/puma.rb#L3 with https://gist.github.com/gabrieljoelc/8c04941042e9241a41b840cccf1ad5fb#file-puma-rb-L3
- https://devcenter.heroku.com/articles/runtime-principles#web-servers

---

## Get it working locally
```
cd ~/dev/herocker
git add . && git commit -m "Add Gemfile.lock"

open http://localhost:3000

docker-compose exec website rails db:create db:migrate

open http://localhost:3000

git add . && git commit -m "Add schema file"
```
@[1-2](Assume we are in terminal B)
@[4](This will have an error because database isn't setup)
@[6](Update local database)
@[8](It should work locally now)
@[10](Git stuff)

---

## Deploy to Heroku
```
curl -O https://gist.githubusercontent.com/gabrieljoelc/8c04941042e9241a41b840cccf1ad5fb/raw/Dockerfile.web
git add . && git commit -m "Add production Dockfile"

heroku create
heroku addons:create heroku-postgresql

heroku config:set \
SECRET_TOKEN=`docker-compose exec website rails secret` \
ACTION_CABLE_ALLOWED_REQUEST_ORIGINS=`heroku apps:info | grep "Web URL:" | sed -e 's/^Web URL:        //'` \
RAILS_SERVE_STATIC_FILES=true
```
@[1-2](Download Dockerfile.web from this gist)
@[4-5](Create a heroku app)
@[7-10](Manually set Rails/orats environment variables in Heroku app)

+++

### Heroku Container Registry
```
heroku container:login

heroku container:push -R
```
@[1](Log into the container registry)
@[3](This instead of `git push master heroku` like usual)

+++

![Please stand by](https://i.makeagif.com/media/9-03-2015/mPJpu9.gif)

[Fresh Beats](https://youtu.be/G2rLmGdDcUM)

Note:
- Checkout https://dashboard.heroku.com/apps
- Checkout https://devcenter.heroku.com/articles/container-registry-and-runtime
- Switches for `heroku container:push` and `Dockerfile` extensions

---

## It worky
```
heroku run rails db:schema:load

heroku open
```
@[1](Load schema into Heroku app)
@[3](Boom)

---

## Resources
- https://github.com/nickjj/orats
- https://devcenter.heroku.com/articles/container-registry-and-runtime
