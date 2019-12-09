![Dokku Logo](assets/dokku.png)
# Dokku Rails 6

Deploying Rails 6.0.1 app on Digitalocean VPS using [Dokku 0.19.10](http://dokku.viewdocs.io/dokku/). Following Alan Vardy's [blog post](https://alanvardy.com/posts/6) and [video](https://www.youtube.com/watch?v=xJsJ4paVVA8&lc=) presentation on Youtube. Dokku also uses [dokku-postgres](https://github.com/dokku/dokku-postgres) to handle the database.

_app versions are indicated at the time of this writing (Dec. 9, 2019)_

## Creating a Dokku Droplet on Digitalocean

1. **Obtain a domain name**
For testing I used **Freenom** which is you guessed, free domain hosting. Alan uses Namecheap and is proud about it.

2. **Signup for Digitalocean**
One of the best and cheapest VPS hosting you can find out there.

3. **Create a Droplet on Digitalocean**
You can create a droplet (virtual private server) with Dokku pre-installed! When creating a droplet, select the Marketplace and look for Dokku, [add your SSH keys](https://timleland.com/copy-ssh-key-to-clipboard/) and for testing you can select the lowest and cheapest plan.

4. **Finish Dokku setup**
Navigate to your droplet's IP address which will be listed in digitalocean. You will need to paste in your public ssh key, then make sure to check "Virtual host naming" for your apps. It means that if you create an app called _myapp_, it will be accessible at _myapp.mydomain.com_

## Creating a Dokku app
