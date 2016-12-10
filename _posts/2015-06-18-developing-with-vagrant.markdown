---
title: "Rails dev with Vagrant and Docker"
date: 2015-06-29 17:03:04 -0700
comments: true
---

This tutorial sets up the same development environment across all platforms (thanks to Vagrant) that is mirrored to the exact same environment you deploy your Ruby on Rails application to (thanks to Docker).

If you are unfamiliar with Docker or Vagrant, or how combining the two work, I highly suggest checking out the Vagrant dev's videos on their Docker provider [here][1].

TLDR: See a base install of Rails with Ruby 2.2 deployed on a Docker container via Vagrant [here][11].

# Preliminaries
This tutorial requires Vagrant >1.6 and uses the Ruby Docker image from the [phusion/passenger-docker](https://github.com/phusion/passenger-docker) Docker image set, but can be followed using their Node.js and full images.

I'm running on OS X 10.10 Yosemite, but you will be able to find the below packages for any OS.

If you have Homebrew installed for Mac, install [VirtualBox][7] with [homebrew-cask][3] if you don't already have it installed.

```bash
$ brew cask install virtualbox
```

Followed by installing Vagrant.

```bash
$ brew cask install vagrant
```

# Guide
Below, I show and describe the 3 files required for deploying your Rails app to Docker via Vagrant: the Vagrantfile required to set up Docker, its Dockerfile to set up the Rails environment, and an Nginx config to serve the Rails app via Docker.

Note that this assumes the current working directory on the host contains your existing Rails app. All of the below has been put together in a working example on GitHub at [{{ site.author.github }}/vagrant-docker-rails][11].

First, we need to create the Vagrantfile to use for bootstrapping the entire process. This takes care of loading Docker and its Dockerfile.

```ruby
Vagrant.configure("2") do |config|
  config.vm.define "app" do |app|
    app.vm.provider :docker do |d|
      d.build_dir = "." # use our own Dockerfile
      d.name      = "rails-app"
    end

    app.vm.network :forwarded_port, guest: 3000, host: 3000
  end
end

```

Our Dockerfile, loaded with `d.build_dir = "."` above, is shown below. It is required, as we are extending the Ruby Docker image from [phusion/baseimage-docker](https://github.com/phusion/baseimage-docker), such as enabling Nginx as the webserver, and copying our Rails app over, along with using the proper user.

```bash
FROM phusion/passenger-ruby22:latest

# Use baseimage-docker's init process.
CMD ["/sbin/my_init"]

# Enable Nginx + Passenger
RUN rm -f /etc/service/nginx/down

# Add Nginx .conf file for our app
ADD webapp.conf /etc/nginx/sites-enabled/webapp.conf

# Run Bundle in a cache efficient way
WORKDIR /tmp
ADD Gemfile /tmp/
ADD Gemfile.lock /tmp/
RUN bundle install

# Add rails app
USER app
RUN mkdir /home/app/webapp
ADD . /home/app/webapp

# Clean up APT when done.
USER root
RUN chown -R app:app /home/app
RUN chmod g+s /home/app
RUN apt-get clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*
```

The passenger-docker imageset supports many versions of Ruby, see the versions they support [here][12].

Finally, as can be seen in the Dockerfile above, we are copying an Nginx config file to Docker containing information relevant to serving our Rails app. It is of course possible to modify this file as if it were any other Nginx site configuration file. See the passenger-docker information on Nginx configuration [here](https://github.com/phusion/passenger-docker#nginx_passenger) for more.

```bash
server {
    listen 3000;
    server_name localhost;
    root /home/app/webapp/public;

    passenger_enabled on;
    passenger_user app;

    passenger_ruby /usr/bin/ruby2.2;
    passenger_friendly_error_pages on;

    passenger_app_env development;

    sendfile off;
}
```

Now, a simple

```bash
$ vagrant up
```

will set up an Nginx + Passenger server serving your Rails app. You can visit now your app at http://127.0.0.1:3000. If the site is not being loaded, see the caveat section below.

This guide is for setting up a development environment, but can be just as easily changed to another environment, by adjusting the Nginx conf file and the forwarded ports.


# Caveats
### Port forwarding
NOTE: If using Mac, and thus using Virtualbox, you must do this!

Open up the VirtualBox GUI and open the settings for the Docker VM. Click on Network, then Port forwarding. Add a new entry with host port being 3000 and the guest port also being 3000. Leave the IP fields blank. This can be fixed by providing a Vagrantfile for the Docker host (Vagrant defaults to the boot2docker Ubuntu install) specifying custom port forwarding for the Virtualbox machine. See [here][10] for more details.

### Shared folders
I can't seem to get Vagrant's shared folders to work, making development seamless. I spent hours pulling my hair out over this. I don't seem to be the [only](http://stackoverflow.com/questions/20240788/shared-volume-in-docker-through-vagrant) one with this problem, whether using Docker volumes or vboxsf. Tweet me [@{{ site.author.twitter }}](https://twitter.com/intent/tweet?screen_name={{ site.author.twitter }})  if you have any idea on how to fix this, please!

# Summing up
Following this setup allows you to host your Rails apps in a contained environment that is independant of the host environment. It is straightforward from here to link to other Docker containers via Vagrant ([details][10]), and even use this in a production environment.

This post has helped me document my experience setting up a simple environment.

Alternatives include [Docker Compose][9]. A similar solution, bi-passing Vagrant entirely.

[1]: http://www.vagrantup.com/blog/feature-preview-vagrant-1-6-docker-dev-environments.html
[2]: https://robots.thoughtbot.com/rails-on-docker
[3]: https://github.com/caskroom/homebrew-cask
[4]: https://docs.docker.com/compose/rails/
[5]: http://docs.vagrantup.com/v2/docker/configuration.html
[6]: https://github.com/phusion/passenger-docker#login_ssh
[7]: https://www.virtualbox.org
[8]: https://github.com/mitchellh/vagrant/wiki/%60vagrant-up%60-hangs-at-%22Waiting-for-VM-to-boot.-This-can-take-a-few-minutes%22
[9]: https://docs.docker.com/compose/
[10]: http://blog.zenika.com/index.php?post/2014/10/07/Setting-up-a-development-environment-using-Docker-and-Vagrant
[11]: https://github.com/{{ site.author.github }}/vagrant-docker-rails
[12]: https://github.com/phusion/passenger-docker#image-variants
