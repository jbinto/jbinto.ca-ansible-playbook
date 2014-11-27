# jbinto/jbinto.ca-ansible-playbook

**Forked from [jbinto/ansible-ubuntu-rails-server](https://github.com/jbinto/ansible-ubuntu-rails-server)**. See there for more detailed (but perhaps outdated) docs.

This playbook is used to build `jbinto.ca`, which hosts several of my sites/applications.

## Current state

**BROKEN/WIP.** I'm tackling a couple of things at once.

* **1) Being able to deploy to multiple environments (staging/production).**
  * This requires factoring out everything environment-specific into `host_vars`.
  * Originally I tried to do `production` and `staging` git branches, but that doesn't make sense for this application. Instead, what I'll probably do is `master`, `live-staging`, and `live-production`. Do work on master, as it is applied, merge into the other branches.
* **2) Being able to deploy multiple Rails apps.**
  * Many of the variables were named naïvely, assuming there would only be one instance. I've prefixed everything.

## Software

* Ubuntu 14.04.1 (this is the only pre-requisite)
* Postgresql 9.3
* rbenv
* ruby 2.1.5
* [Phusion Passenger](https://www.phusionpassenger.com/) + nginx from Phusion's apt repo
* ~~[Papertrail logging](https://papertrailapp.com/)~~ (TODO)

## Sites/web applications

* Prepares multiple nginx vhosts for Rails apps, ready for deployment with `cap`:
  * 416.bike
  * ttchotline.uniquename.org (for now, until we get a real domain)

## Cheatsheet

See detailed instructions below.

### Pre-requisites

* Install `pip` via homebrew
* Install required Python modules (`ansible`, `dopy`, `passlib`, etc):

```
sudo pip install -r requirements.txt
```

### Spin up a new DigitalOcean droplet

`dopy` (python, ansible) and `tugboat` (ruby, for command-line support only) currently use DigitalOcean APIv1. This will have to be updated to use APIv2 at a later date. 

Get your legacy API key at http://cloud.digitalocean.com/api_access and set magic environment variables `DO_CLIENT_ID` and `DO_API_KEY`.

Configuration (droplet size, region, etc.) available in `vars/digitalocean.yml`. You will be interactively prompted to enter the hostname.

To provision, run:

```
ansible-playbook provision-digitalocean.yml -i local 
```

### Droplet post-requisites

* A new DNS zone is created for each droplet you create, and is handled by DigitalOcean's DNS. This may not be exactly what you want. Configure your DNS manually if necessary.

* Generate a crypted Linux/root password for your environment:

```
python support/generate-crypted-password.py
```

Paste the crypted password into the `password` field (in `host_vars/{{HOST}}/defaults.yml`)

* Add the IP address of your new droplet to `hosts-production` or `hosts-staging`. You can find it at: https://cloud.digitalocean.com/droplets

### Building (or upgrading) server on DigitalOcean

```
ansible-playbook build-server.yml -i $INVENTORY -u $USER -K -vvvv
```

Where `$INVENTORY` is one of: `hosts-production`, `hosts-staging`

And `$USER` is one of: `root` (first run only), `staging0`, `production0`.

To only run a specific tag, add `--tags`, e.g.:

```
ansible-playbook build-server.yml -i $INVENTORY -u $USER -K -vvvv --tags site-bikeways
```

### Deploying a Rails site

This playbook only prepares the execution environment (e.g. Passenger, nginx, Postgres, dotenv).

From here, it's time to continue with a Capistrano deployment, e.g. [rails4-sample-app-capistrano](https://github.com/jbinto/rails4-sample-app-capistrano). 

It should be as sample as `cap staging deploy`.


### Adding a new Rails site

TODO, but basically clone `roles/site-bikeways` and add the new role to `build-server.yml`

### Configuration

Hosts are defined in two separate inventory files: `hosts-production` and `hosts-staging`.

All roles contain content that is common between execution environments. Elements that differ should be factored out into Ansible variables.

Host-specific configuration is available in `host_vars/{{HOST}}/*`. All files within the host directory are loaded.

### Environments

An example `site-bikeways.env.yml` file is provided in `vars/`. All configuration for the application is stored here, including but not limited to secrets.

Change it as necessary and rename to `site-bikeways.env.yml` before attempting to run the `build-server` playbook.

The `site-bikeways.env.yml` file itself is under gitignore, as it contains sensitive credentials and API keys.
