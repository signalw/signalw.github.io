---
layout: post
title: "Deploy a Misskey instance on Heroku"
---

Misskey, a decentralized social networking platform, is part of Fediverse. Other well-known platforms are Pleroma, Mastodon, etc. Out of curiosity, I started out and deployed an instance on Heroku, which turns out to be really easy. Here I will document the steps. 

As of right now, the latest version is 12.47.1. If you just want a deployment without installing any development tools, all you need is Git and Heroku command line. 

```bash
$ git clone https://github.com/syuilo/misskey
Cloning into 'misskey'...
remote: Enumerating objects: 132885, done.
remote: Total 132885 (delta 0), reused 0 (delta 0), pack-reused 132885
Receiving objects: 100% (132885/132885), 68.33 MiB | 1.48 MiB/s, done.
Resolving deltas: 100% (99069/99069), done.
Checking out files: 100% (1409/1409), done.

$ cd !$:t
cd misskey

$ git checkout master
Branch 'master' set up to track remote branch 'master' from 'origin'.
Switched to a new branch 'master'

$ export APP_NAME=misskey
$ heroku create $APP_NAME
Creating ⬢ mskey... done
https://mskey.herokuapp.com/ | https://git.heroku.com/mskey.git

$ heroku git:remote -a $APP_NAME
set git remote heroku to https://git.heroku.com/mskey.git

$ heroku buildpacks:set https://github.com/heroku/heroku-buildpack-nodejs
Buildpack set. Next release on mskey will use https://github.com/heroku/heroku-buildpack-nodejs.
Run git push heroku main to create a new release using this buildpack.

$ heroku addons:create heroku-postgresql:hobby-dev
Creating heroku-postgresql:hobby-dev on ⬢ mskey... free
Database has been created and is available
 ! This database is empty. If upgrading, you can transfer
 ! data from another database with pg:copy
Created postgresql-sinuous-86457 as DATABASE_URL
Use heroku addons:docs heroku-postgresql to view documentation

# I tried heroku-redis:hobby-dev, but Misskey exceeds its free tier connection limit
$ heroku addons:create rediscloud:30
Creating rediscloud:30 on ⬢ mskey... free
Created rediscloud-reticulated-52634 as REDISCLOUD_URL
Use heroku addons:docs rediscloud to view documentation

$ heroku config
=== mskey Config Vars
DATABASE_URL: postgres://${DB_USER}:${DB_PASS}@${DB_HOST}:${DB_PORT}/${DB_NAME}
REDIS_URL:    redis://rediscloud:${REDIS_PASS}@${REDIS_HOST}:${REDIS_PORT}

# Misskey expects a default.yml config file
mv .config/example.yml .config/default.yml
vi .config/default.yml
```

Edit `default.yml` file:
```
# Final accessible URL seen by a user.
url: https://$APP_NAME.herokuapp.com

db:
  host: $DB_HOST
  port: $DB_PORT

  # Database name
  db: $DB_NAME

  # Auth
  user: $DB_USER
  pass: $DB_PASS

  # Whether disable Caching queries
  #disableCache: true

  # Extra Connection options
  #extra:
  #  ssl: true

redis:
  host: $REDIS_HOST
  port: $REDIS_PORT
  pass: $REDIS_PASS
  #prefix: example-prefix
  #db: 1
```
Note that port is NOT set since Heroku will dynamically assign one.

Edit `Procfile` file:
```
web: NODE_ENV=production npm start
release: npm run migrate
```
This tells Heroku to run db migrations upon each deployment.

Once the files are updated, just make sure `.config/default.yml` and `Procfile` are not ignored, then push to Heroku.

```bash
$ git diff .gitignore
-!/.config/default.yml
+!/.config/example.yml

$ git add .
$ git commit -m "Initial commit"
[master 57d8388a4] Initial commit
 1 file changed, 8 insertions(+), 8 deletions(-)
$ git push heroku master
```

Wait a couple minutes and voila, Misskey should be up and running on Heroku. 
