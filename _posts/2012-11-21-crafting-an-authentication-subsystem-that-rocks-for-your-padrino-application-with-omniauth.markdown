---
layout: post
title: "Crafting an authentication subsystem that rocks for your Padrino application with Omniauth"
date: 2012-11-21 08:26
comments: true
published: true
featured: false
categories: ruby padrino builders authentication breakers omniauth howto h4f
thumb: authentication.jpg
level:
hn: http://news.ycombinator.com/item?id=4813592
rd: http://redd.it/13k6my
---

Next time you point your browser to a _/login_ url wait a minute before
submitting your credentials. There is a complex system you're going to use when
you submit that form and it must be honored in some way.

You're a software craftman and you want to get the job done. Users have to
register to your app and they must have the chance to login. 

Attackers will hit this code first and they will try to bypass it or rather to
exploit it. It's a very critical subsystem that you have to design with care.

<!-- more -->

## Don't reinvent the wheel unless you're forced to

Unless you **do** want to cook an authentication subsystem yourself, and you
have some hard-to-fight constraints that force this decision, I suggest to you
using a third-party opensource library implementing OAuth2 over well known
identity providers like Google, Facebook, Github, OpenID.

To my taste, [omniauth](https://github.com/intridea/omniauth) is one of best
and easy to use ruby library implementing OAuth authentication.

Let's see how easy is to let your users to authenticate to your applications
using their [github.com](https://github.com) account.

## Omniauth and Github.com integration

The followings are real code snippets from the
[codesake.com](http://codesake.com) source code. Since I need to bind my users
with their github account, using [omniauth for github](https://github.com/intridea/omniauth-github) 
was the number 1 choice.

### Register your application to github

Before going further, you must [register your web application](https://github.com/settings/applications/new) to github. You will
get a client identifier and a secret that you **must not** share.

While in development it's not a good idea to register your web application
using real urls since the code will be likely running on localhost.

When I registered codesake.com, I gave [localhost:3000](http://localhost:3000)
as base URL and
[/auth/github/callback](http://localhost:3000/auth/github/callback) as callback
URL.

The callback URL is the one used by the identity provider (github in this
example) to redirect user after he authorizes codesake.com to use his
github.com account informations.

Specifiying the ID provider in the callback url can be a clever choice if in a
future you want to add other ID providers. 

### Tell bundler about your choice

Now, it's time for you to te tell bundler you will use omniauth for github gem.

``` ruby A Gemfile snippet
...
gem 'omniauth-github', :git => 'git://github.com/intridea/omniauth-github.git'
...
```

Run _bundler update_ now and you're ready to tell your application the secrets
it will share with github.com website.

Place this code into your main Padrino::Application, let's say you have the default app/app.rb in your code:

``` ruby app/app.rb
 use OmniAuth::Builder do
    provider :github, "your client id", "your client secret"
  end
```

Now, you can start padrino and OAuth workflow is working out-of-the-box right
now. Just point your browser to the following url:
[/auth/github](http://localhost:3000/auth/github)

You will be redirect to a github.com page asking you to authorized your website to access your data. If you grant it, the call back url will be called and... an exception will be thrown unless you already created the auth controlled as described... now!

### Create the auth controller

Create a controller to handle authentication:

``` ruby Create a Padrino controller
$ bundle exec padrino g controller auth
``` 

Add some code in case of callback (authentication succeeded) and authentication failed. 

``` ruby Codesake.com auth controller snippet
get :github_callback, :map => "/auth/github/callback" do
    
    omniauth = request.env["omniauth.auth"]

    @user = User.find_uid(omniauth["uid"]) 
    @user = User.new_from_omniauth(omniauth) if @user.nil?
    
    # save @user into your session to say he's authenticated
  end

  get :github_callback_failed, :map => "/auth/failure" do
    flash[:error] = "Error logging with github.com #{params[:message]}"

    redirect url("/")
  end
``` 

In the environement it's placed the omniauth.auth hash object containing all github users informationss.
Now it's time to make your users informations to be persistent.

### Create a user model

I use [datamapper](http://datamapper.org) as ORM but the only differnt thing
here is that the migrate command changes [accordingly](http://datamapper.org)
with the ORM you like most.
 
``` ruby Create the User model and run migration
$ bundle exec padrino g model user uid:integer name:string email:string created_at:datetime update_at:datetime
$ bundle exec padrino rake dm:migrate
```

To my personal taste, I always default the created_at and update_at model properties to be equal to Time.now. 

``` ruby My User model
  property :created_at, DateTime, :default=>Time.now
  property :update_at, DateTime, :default=>Time.now
```

I rather enforce the presence of a valid email, also if this should be there in
the user's github account informations.

I use [factory girl rubygem](https://github.com/thoughtbot/factory_girl) to
mock model instances so I added some rubygems in my Gemfile for the testing
environment.

``` ruby Gemfile for testing
gem 'rspec', :group => "test"
gem 'capybara', :group => "test"
gem 'cucumber', :group => "test"
gem 'rack-test', :require => "rack/test", :group => "test"
gem 'webmock', :group=>"test"
gem 'factory_girl', :group => "test"
``` 

FactoryGirl must be initialized in the spec_helper.rb file in order to have
rspec to look for factories.

``` ruby spec_helper.rb snippet
require 'factory_girl'

require 'webmock/rspec'

FactoryGirl.definition_file_paths = [
    File.join(Padrino.root, 'factories'),
    File.join(Padrino.root, 'test', 'factories'),
    File.join(Padrino.root, 'spec', 'factories')
]

FactoryGirl.find_definitions
``` 

And now... let's write a trivial test for a user with empty email field that it must be not valid.

``` ruby user_spec.rb
  it "should have an email" do
    user = FactoryGirl.create(:user)
    user.email = ""
    user.valid?.should  be_false
  end
``` 

Run our test and we should see it to fail:

``` ruby 
$ bundle exec padrino rake spec:models
``` 

Add the model validation constraint and now the rspec test passes.
Great.

``` ruby My User model
...
  validates_presence_of, :email
...
``` 

Now you may want to add to your user model some helper to search users from
github uid field and to create a new user if not present in your db.

``` ruby My User model
def self.find_uid(uid)
  User.first(:uid=>uid)
end

def self.new_from_omniauth(omniauth)
  user = User.new 
  user.uid = omniauth["uid"]
  user.name= omniauth["info"]["nickname"]
  user.email= omniauth["info"]["email"]

  user.save!
  user
end
```

Now your callback handler, when called it search for the user in the database
and if not present the user is created.

## Off by one

With the post I gave you some hints about using github as identity provider for your web application.

From the security point of view you:

* don't have to care about password complexity rules, password aging, password
  reset issues. That means less code for you to write and less chances to
  introduce security issues.
* don't have to be compliant to laws about sensitive data storage (unless your
  application doesn't handle PCI, SOX or something like that data)
* rely on github.com security that means that a break-in can also affect your
  web application. Fortunately you can reset your webapplication secrets so
  people will have to grant your application access again. 

And you? Which is your opinion about using a third party identity provider to
manager authentication in your application?
