# jbinto/jbinto.ca-ansible-playbook

**Forked from [jbinto/ansible-ubuntu-rails-server](https://github.com/jbinto/ansible-ubuntu-rails-server)**

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

### Upgrading

Production:

```
ansible-playbook build-server.yml -i hosts-digitalocean -u production0 -K -vvvv
```

### Adding a new site

See commit 973cddc for more info. (TODO: make this easier)

### Environments

An example `site-bikeways.env.yml` file is provided in `vars/`. All configuration for the application is stored here, including but not limited to secrets.

Change it as necessary and rename to `site-bikeways.env.yml` before attempting to run the `build-server` playbook.

The `site-bikeways.env.yml` file itself is under gitignore, as it contains sensitive credentials and API keys.
