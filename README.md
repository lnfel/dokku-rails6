![Dokku Logo](assets/dokku.png)

[![Build Status](https://img.shields.io/circleci/project/github/dokku/dokku.svg?style=flat-square "Build Status")](https://circleci.com/gh/lnfel/workflows/dokku-rails6)

# Dokku Rails 6

Deploying Rails 6.0.1 app on Digitalocean VPS using [Dokku 0.19.10](http://dokku.viewdocs.io/dokku/). Following Alan Vardy's [blog post](https://alanvardy.com/posts/6) and [video](https://www.youtube.com/watch?v=xJsJ4paVVA8&lc=) presentation on Youtube. Dokku also uses [dokku-postgres](https://github.com/dokku/dokku-postgres) to handle the database.

_app versions are indicated at the time of this writing (Dec. 9, 2019)_

## Creating a Dokku Droplet on Digitalocean

#### 1. **Obtain a domain name**
For testing I used **Freenom** which is you guessed, free domain hosting. Alan uses Namecheap and is proud about it.

#### 2. **Signup for Digitalocean**
One of the best and cheapest VPS hosting you can find out there.

#### 3. **Create a Droplet on Digitalocean**
You can create a droplet (virtual private server) with Dokku pre-installed! When creating a droplet, select the Marketplace and look for Dokku, [add your SSH keys](https://timleland.com/copy-ssh-key-to-clipboard/) and for testing you can select the lowest and cheapest plan.

**Note:**
If working on Cloud IDE like Cloud9 do your ssh-keygen on your Cloud9 project terminal then copy your id_rsa.pub file contents, this will serve as your IDE's ssh key to access the droplet.
It is usually located on root/.ssh/id_rsa.pub, you can try using: 

```console
cat /.ssh/id_rsa.pub
```

#### 4. **Go to your server's IP and follow the web installer**
Navigate to your droplet's IP address which will be listed in digitalocean. You will need to paste in your public ssh key, then make sure to check "Virtual host naming" for your apps. It means that if you create an app called _myapp_, it will be accessible at _myapp.mydomain.com_

#### 5. **Put your domain on Hostname field**
This will be important as dokku will generate url based on what you put on Hostname field such as _myapp.**mydomain.com**_ where **mydomain.com** is the hostname. It is possible to change hostnames after installation but I would just recommend to rebuild the app completely if you are changing hostnames. _TIP:_ you can also use IP address instead of domain. If you decide to use a domain name, Dokku will generate url base on the domain but if you choose to put IP on Hostname field it will only generate using the IP. This is good if you are expecting to change domain name sooner or later.

## Creating a Dokku app

#### 5. **SSH onto your server**

```console
ssh root@your.droplet.ip.address
```

#### 6. **Update everything** (Digital Ocean's droplets will not be completely up to date, and Dokku can easily be a few versions behind)

```console
apt update && apt upgrade
```

#### 7. **Create the app**

```console
dokku apps:create yourappname
```

#### 8. **Install Postgres, create and link database**

```console
dokku plugin:install https://github.com/dokku/dokku-postgres.git postgres
dokku postgres:create yourdbname
dokku postgres:link yourdbname yourappname
```

#### 9. **Create a swap file** to help out on the ram front. You will only see output after the 3rd line.

```console
fallocate -l 2G /swapfile
chmod 600 /swapfile
mkswap /swapfile
swapon /swapfile
```

#### 10. **Open fstab with Vim**

```console
vi /etc/fstab
```

#### 11. **Add this line at the bottom**

```console
/swapfile none swap sw 0 0
```

* Press "i" to enter edit mode.
* Press "esc" after finish adding the line to exit edit mode
* Press "shift+:" then type "wq" to write file and quit editor

#### 12. **Configure Digitalocean DNS**
Click Add domain on digitalocean then add the domain you have got freely on freenom or bought on namecheap.
Create 'A' record for your domain:

Type | Hostname | Value
---- | -------- | -----
A | yourdomain.com | (select your droplet)

## Rails app

#### 1. **Creating the app**

```ruby
rails new yourrailsapp --database=postgresql
```

#### 2. **Navigate to your app directory and generate a controller**

```ruby
cd awesomeapp
rails generate controller Static index
```

* Here we got a controller name Static with index method
* Add something to that index page in app/views
* then add the route to config/routes.rb

```ruby
root 'static#index'
```

#### 3. **Automatic Migrations**
This one just runs `rails db:migrate` automatically.
Create `app.json` in the root directory of your app

```json
{
  "name": "awesomeapp",
  "description": "My awesome Rails app, running on Dokku!",
  "keywords": [
    "dokku",
    "rails"
  ],
  "scripts": {
    "dokku": {
      "postdeploy": "bundle exec rails db:migrate"
    }
  }
}
```

#### 4. **Add checks**
This feature is rather nice, it makes Dokku check to make sure that your freshly uploaded code actually starts up before switching over to it!
Create a file called `CHECKS` in the root of your project directory

```ruby
# CHECKS

WAIT=10  
ATTEMPTS=6  
/check.txt it_works
```

* Add the following route to config/routes.rb

```ruby
get '/check.txt', to: proc {[200, {}, ['it_works']]}
```

* And it will make a call to that route when it starts up your new code, thereby ensuring that the new server actually started.

#### 5. **Add your secrets file**

```ruby
# config/secrets.yml
production:
  secret_key_base: <%= ENV["SECRET_KEY_BASE"] %>
```

* on Rails 6 we now also have encryption for the secret key base file in the form of the master.key file which decrypts the file called credentials.yml.enc that holds your secret_key_base
* the 2 file will be generated upon executing `rails new` command at the start of creating your rails app.

```ruby
# config/database.yml
production:
  adapter: postgresql
  secret_key_base: <%= Rails.application.credentials.dig(:secret_key_base) %>
  rails_master_key: <%= ENV['RAILS_MASTER_KEY'] %>
  encoding: unicode
  pool: 5 
```

* It is recommended not to include your master.key when pushing to repo. By default it is included on .gitignorefile
* Without the master.key, the application won't run
* Locate ythe master key on same directory as credentials.yml.enc `config/master.key` then copy the hash string
* Since it will not be included when pushing to our dokku vps, we will save it on ENV variable instead and access it from the variable. ssh to your droplet then run the following dokku command:

```console
dokku config:set yourappname RAILS_MASTER_KEY=thehashstringfromyourmasterkeyfile
```

#### 6. **Set up Puma correctly**
If you are using Rails 6, [take a look at this post](https://www.alanvardy.com/posts/38) to save yourself some headache in getting Puma up and running.

#### 7. **Add remote repository**
Navigate to your yourrailsapp project directory and add the repository

```git
git remote add dokku dokku@your.droplet.ip.address:yourappname
```

#### 8. **You can now push your code with**

```git
git add .
git commit -m 'Initialize repo'
git push dokku master
```

## Free SSL with Let's Encrypt

[Let's Encrypt](https://letsencrypt.org/) provides free SSL certificates. You can find more complete instructions and explanations here.

#### 1. **SSH onto your server**

```console
ssh root@your.droplet.ip.address
```

#### 2. **Install Let's Encrypt**

```console
dokku plugin:install https://github.com/dokku/dokku-letsencrypt.git
```

* Update if it was already installed

```console
dokku plugin:update letsencrypt
```

#### 3. **Set your email address** (note that you need to change MYAPP and ME@MYEMAIL.COM)

```console
dokku config:set --no-restart awesomeapp DOKKU_LETSENCRYPT_EMAIL=ME@MYEMAIL.COM
```

#### 4. **Turn it on**

```console
dokku letsencrypt yourappname
```

#### 5. **Set up auto-renewal with a cronjob**

```console
dokku letsencrypt:cron-job --add
```
