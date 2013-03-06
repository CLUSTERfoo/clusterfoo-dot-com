---
title: Rails Cheatsheet
kind: article
created_at: 2013-03-01
---
<!-- _. -->

A little cheatsheet of Rails stuff I don't want to forget. Many of these
tips come from 
[Michael Hartl's Rails tutorial](http://ruby.railstutorial.org/){: target="_blank" }.

<!-- _. -->


### Setting Up Environment

#### Install RVM, Ruby, Bundler

Install RVM through instructions on [rvm.io](http://rvm.io){: target="_blank" }.
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
> `$ rvm reinstall 1.9.3-head --with-gcc=clang`


Create project gemset and be in it before installing Rails:

    $ rvm use 1.9.3-head@projectname --create

Project-specific gemsets prevent big headaches. Don't forget to set a 
project `rvmrc` file after Rail app is generated (see below).

#### Install Rails

    $ gem install rails
    
And if on Linux, also run:

    $ sudo apt-get install libxslt-dev libxml2-dev libsqlite3-dev

Install node.js so that rails server can run properly in your development
environment.



### Initialize Rails App

#### Set up Rails App and rvmrc

Skip TestUnit if you're using rspec:

    $ rails new project_name --skip-test-unit
    
Now create an rvmrc inside the project's root directory:

    $ rvm --rvmrc --create 1.9.3-head@projectname

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

Add PostgreSQL gem and production-environment-specific gems. In gem file:

    group :production do
      gem 'pg'
    end

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

