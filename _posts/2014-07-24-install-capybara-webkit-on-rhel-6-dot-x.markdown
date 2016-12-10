---
title: "Install capybara-webkit on RHEL 6.x"
date: 2014-07-24 17:10:00 +0200
comments: true
published: true
categories: linux
---

Here's how I installed the [thoughtbot/capybara-webkit](https://github.com/thoughtbot/capybara-webkit) gem and configured headless testing on RHEL 6.5 with [jenkinsci/jenkins](https://github.com/jenkinsci/jenkins).

I needed to run our capybara acceptance tests that rely on some javascript to execute in the browser on every Jenkins build. Getting this headless testing to work wasn't exactly trivial and took half a day, so I'm writing down the steps I took to finally run the rspec capybara tests.

First, install Xvfb, a virtual frame buffer X11 server:
```bash
$ yum install xorg-x11-server-Xvfb
```

You may also want to install the [Xvfb Plugin](https://wiki.jenkins-ci.org/display/JENKINS/Xvfb+Plugin) for Jenkins, but this isn't required.

Then install ``qt`` with webkit bindings, a requirement for ``capybara-webkit``:

```bash
$ yum install qt5-qtwebkit-devel
```

Now, we can install ``capybara-webkit``:

```bash
$ gem install capybara-webkit
```

or alternatively by adding it to your Gemfile.

This may result in the following error stating ``qmake`` not being found when building native extensions:

```bash
$ gem install capybara-webkit
Building native extensions.  This could take a while...
ERROR:  Error installing capybara-webkit:
  ERROR: Failed to build gem native extension.

  /home/martin/.rbenv/versions/2.1.1/bin/ruby extconf.rb
Command 'qmake -spec linux-g++ ' not available

Makefile not found

[...]
```

Apprently the ``qmake`` binary can't be run. What I did to fix this was manually defining the ``QMAKE`` variable to the correct binary path:

```bash
$ export QMAKE=/usr/bin/qmake-qt5
```

Then rerunning the gem or bundle install. This should successfully install the ``capybara-webkit`` gem.

Great... Now we have ``capybara-webkit`` installed. Now, add the following to the ``spec_helper.rb`` configure block as stated in the ``capybara-webkit`` documentation to select capybara-webkit as the javascript driver:

```ruby
RSpec.configure do |config|
  config.before(:each) do
    Capybara.javascript_driver = :webkit
  end
end
```

Now, to get Jenkins to run the headless tests, in the "Execute Shell"  part of the configure job page, paste the following to run rspec with Xvfb:

```bash
$ DISPLAY=localhost:1.0 xvfb-run -a bundle exec rspec spec/
```

If, upon running the above this error occurs:

```bash
$ DISPLAY=localhost:1.0 xvfb-run -a bundle exec rspec spec/
...................process 4579: D-Bus library appears to be incorrectly set up; failed to read machine uuid: Failed to open "/var/lib/dbus/machine-id": No such file or directory
See the manual page for dbus-uuidgen to correct this issue.
  D-Bus not built with -rdynamic so unable to print a backtrace
```

run ``$ sudo dbus-uuidgen`` and copy and paste its output into the ``/var/lib/dbus/machine-id`` file. Create it if you have to. For more information about this dbus and xorg related error, see [this](http://www.torkwrench.com/2011/12/16/d-bus-library-appears-to-be-incorrectly-set-up-failed-to-read-machine-uuid-failed-to-open-varlibdbusmachine-id/) blog post.

Now, build and check the console output. Your tests should hopefully have run and Jenkins will have reported whether or not they failed.

Hope this helps!


