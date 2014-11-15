# jbinto/jbinto.ca-ansible-playbook

**Forked from [jbinto/ansible-ubuntu-rails-server](https://github.com/jbinto/ansible-ubuntu-rails-server)**

This playbook is used to build `jbinto.ca`, which hosts several of my sites/applications.

## Software

* Ubuntu 14.04.1 (this is the only pre-requisite)
* Postgresql 9.3
* rbenv
* ruby 2.1.5
* [Phusion Passenger](https://www.phusionpassenger.com/) + nginx from Phusion's apt repo
* ~~[Papertrail logging](https://papertrailapp.com/)~~ (TODO)

## Sites/web applications

* Prepares multiple nginx vhosts for Rails apps, ready for deployment with `cap`:
  * staging.416.bike
  * 416.bike
  * ttc.uniquename.org (for now, until we get a real domain)
* Prepares some static nginx vhosts as well:
  * jbinto.ca (redirect)
  * www.jbinto.ca (redirect)
  * www.416.bike (redirect)

## Cheatsheet

See detailed instructions below.

### Upgrading

Staging:

```
ansible-playbook build-server.yml -i hosts-digitalocean -u staging0 -K -vvvv
```

### Adding a new site

See commit 973cddc for more info. (TODO: make this easier)

### Environments

An example `site-bikeways.env.yml` file is provided in `vars/`. All configuration for the application is stored here, including but not limited to secrets.

Change it as necessary and rename to `site-bikeways.env.yml` before attempting to run the `build-server` playbook.

The `site-bikeways.env.yml` file itself is under gitignore, as it contains sensitive credentials and API keys.

## Usage (locally via Vagrant)

[Vagrant](http://www.vagrantup.com/downloads.html) must be installed from the website.

I'm currently retrieving Ansible from git, as well as the `dopy` module for DigitalOcean.

```
git clone https://github.com/jbinto/jbinto.ca-ansible-playbook.git
cd jbinto.ca-ansible-playbook
sudo pip install -r requirements.txt
```

Generate a crypted password, and put it in `vars/default.yml`.

```
python support/generate-crypted-password.py
```

The following command will:

* Use Vagrant to create a new Ubuntu virtual machine.
* Boot that machine with Virtualbox.
* Ask you for the `deploy` sudo password (the one you just crypted).
* Use our Ansible playbook to provision everything needed for the Rails server.

```
vagrant up
```

**BUG:** Sometimes, `vagrant up` times out before Ansible gets a chance to connect. I haven't figured this out yet. If this happens, run `vagrant provision` to continue the Ansible playbook.

To run individual roles (e.g. only install nginx), try the following. You can replace `nginx` with any role name, since they're all tagged in `build-server.yml`.

```
ansible-playbook build-server.yml -i hosts --tags nginx
```

Once the provision is complete, continue to [jbinto/rails4_sample_app_capistrano](https://github.com/jbinto/rails4_sample_app_capistrano) to deploy a Rails application using Capistrano.

## Usage (in the cloud on DigitalOcean)

Set up environemnt variables `DO_CLIENT_ID` and `DO_API_KEY`, or hardcode them in `./hosts/digital_ocean.ini`.

Install `tugboat`:

```
gem install tugboat
tugboat authorize
```

Edit `./vars/digitalocean.yml`. 

* Note that `hostname` must be a real FQDN you own, and the DNS must be pointing to DigitalOcean.
* You can use `tugboat` to acquire the magic numbers needed for region/image/size IDs.

Now you can provision the DigitalOcean droplet:

```
ansible-playbook -i local provision-digitalocean.yml
```

This will spin up a new DigitalOcean VPS **which costs real money**. Since you set up the SSH keys with DigitalOcean, you already have passwordless `root` access.

At the end, it should tell you the new IP address of your server (doesn't seem to be doing this anymore c. 10-2014).

* **Manually** add the IP address to `./hosts-digitalocean`.
* **Manually** set your DNS as necessary with this new IP address. (**TODO use digital_ocean_domain module**)
* Generate a crypted password, if not already done, and put it in `vars/default.yml`. This will be the password for your `deploy` user.

```
python support/generate-crypted-password.py
```

Now, run the playbook as usual. Good luck!

```
ansible-playbook build-server.yml -i hosts-digitalocean -u root -K -vvvv
```

Note that after the first run, `root` will no longer be able to log in. To run the playbook again, replace `root` with the `deploy` user as set in `vars/defaults.yml`.
