---
title: Rails Cheatsheet
kind: article
created_at: 2013-06-19
---
<!-- _. -->

**Updated:** Now using vagrant and postgresql.

***

A little cheatsheet of Rails stuff I don't want to forget.

<!-- _. -->


### Setting Up Environment

#### Set Up Vagrant with Ubuntu VM 

(This can be skipped, just set up the same environment in your host machine).
For more complete instructions on setting up Vagrant, visit their site, as their
docs are extremely easy to follow.

Rails uses port 3000 by default. This means that your `Vagrantfile` requires:

    config.vm.network :forwarded_port, host: 4567, guest: 3000

This will make your rails app available through [http://localhost:4567].

Next, (if your vm is ubuntu), you'll need the following packages:

    sudo apt-get install git vim libpq-dev libxslt-dev libxml2-dev \
                         postgresql curl libsqlite3-dev

You'll also need to install `nvm` (nodejs version manager) by following the
instructions on the nvm github and run the latest version of nodejs.

##### Set Up Postgres

Finally, you'll need to actually set up postgres to work with your rails app.
If you've not started the app yet, run:

    $ rails new project_name --database=postgresql

First, in your `config/database.yml`, you want the following settings:

    development:
      adapter: postgresql
      encoding: unicode
      database: database
      host: localhost
      pool: 5
      username: username
      password: password

Next, you need to create the corresponding username and password on postgres:

    $ sudo -u postgres psql
    
Now, to set up user password:

    postgres=# \password

Next create user and password:

    postgres=# create user username with password 'password';

... and a database:

    postgres=# create database database owner username;

All done! Your (vm) environment is all set up. In a vm environment, I don't use
gemsets (though you could), so if you did this on vagrant, you can skip the
gemset instructions.


#### Install RVM, Ruby, Bundler

Install RVM through instructions on [rvm.io](http://rvm.io){: target="_blank" }.
If you're using vagrant, remember to `vagrant ssh` into your vm for this.

<!-- _. -->
RVM now also eliminates the need to use `bundle exec` when used with bundler. 

Install latest ruby. The following OpenSSL option is needed on Ubuntu:

    $ rvm install 1.9.3-head --with-openssl-dir=$HOME/.rvm/


> *Aside:*
> 
> If you ever get the following warning:
> 
> `make: /usr/bin/gcc-4.2: No such file or directory`
> 
> You will also need to run
> 
> `$ rvm reinstall 2.0.0-head --with-gcc=clang`


### Initialize Rails App

#### Set up Rails App Directory

Skip TestUnit if you're using rspec:

    $ rails new project_name --skip-test-unit
    
Now create two files: `.ruby-version` and `.ruby-gemset`. These files will 
ensure that rvm, or whatever ruby envelope you use, know to switch to the 
correct Ruby version (and gemset, in rvm). For example, if you're using Ruby
2.0.0 and a gemset called `my-project`, create a `.ruby-version` file with the
following content:

    2.0.0

And a `.ruby-gemset` file with:

    my-project

Now `cd` in and out of the directory and make sure that rvm is switching to the 
correct gemset with:

    $ rvm gemset list

#### Set up Rspec


Add rspec, guard and spork to development / test environment in Gemfile:
    
    group :development, :test do
      gem 'rspec-rails', '~> 2'

      # If using guard and spork:
      gem 'guard-rspec', '~> 2'
      gem 'guard-spork', '~> 1'
      gem 'spork', '~> 0.9'
    end

Make Rails use rspec instead of TestUnit:

    $ rails g rspec:install

#### Speeding up Tests with Spork and Guard

Follow 
[these instructions](http://ruby.railstutorial.org/chapters/static-pages#sec-guard){: target="_blank" }.
<!-- _. -->


#### Heroku / Deployment

To run bundler without production gems

    $ bundle install --without production

Bundler remembers this setting, no need to use it again.

Initialize Heroku:
    
    $ heroku login
    $ heroku create

Push to Heroku and rename:

    $ git push heroku
    $ heroku rename project_name



### Maintenance

#### After Migrations

After running
    
    $ rake db:migrate

The following come in handy:

Stop the Rails server. This is obvious, but forgetting to restart the server
after a migration is a sure way to waste time wondering why half your tests are
suddenly failing.

Annotate models and migrations (requires the `annotate` gem):

    $ annotate

Tell the test database about the migration too:

    $ rake db:test:prepare

Restart the Rails server.

Sometimes you want to reset your test database:

    $ rake db:reset
    $ rake db:test:prepare

#### Run Console in Sandbox Mode!

Don't break things:

    $ rails console --sandbox

