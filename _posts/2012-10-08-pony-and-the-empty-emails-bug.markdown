---
layout: post
title: "Pony and the empty emails bug"
date: 2012-10-08 08:03
comments: true
published: true
featured: false
tags: builders ruby tdd bdd email pony gem rubygem empty-body
thumb:
hn: 
rd: 
---

There were an annoying bug affecting the internal application security self
service platform I deployed on my company.
When a user makes a request the notification email is sent with an empty body.

Sometimes bug hunting is an hard track even if you play with rspec and friends.

<!-- more -->

The notification model of my [padrino](http://www.padrinorb.com) powered web
application is something easy and simple. The core business is made by the send
method that actually call pony rubygem firing up the email.

``` ruby notification.rb - the send method
def send(options={})
  ret = setup(options)
  return ret if ENV['RACK_ENV'] == 'development'

   begin
    Pony.mail(:to=>self.to, :from=>"engage@mycompanydomain", :cc=>self.cc, :subject=>self.subject, :body=>self.body, :html_body=>self.html_body) 
    self.sent=true
   rescue => e
     puts e.message
     self.sent=false
     raise e #useful to let rspec to handle the error condition
   end
  self.save
end
``` 

I don't want to send emails while I'm developing so I made the method to fast
exit if we're not in production or testing stage.

The small rspec code for this method checks if a request to our internal
helpdesk has been correctly issued.

``` ruby notification_spec.rb - checking if we can write to helpdesk
it 'are sent to helpdesk in order to create a working queue' do
  Pony.should_receive(:mail) do |mail|
    mail[:to].should ==  "helpdesk@mycompanydomain"
    mail[:from].should ==  "engage@mycompanydomain"
    mail[:cc].should ==  "requester@mycompanydomain"
    mail[:subject].should == "Please open us a ticket"
    mail[:body].should == 'This is a test'
    mail[:html_body].should be_empty
  end

  notification.kind=:helpdesk
  notification.send({:prj_name => "the test rspec", :body=>"This is a test"})

  notification.sent.should  be_true
end
``` 

{% blockquote %}
Be wise, when something doesn't run the way you expected and you double check
your code and your tests, look at the third party libraries source, often you
will find there the answer.
{% endblockquote %}

From this point everything went well. All tests passed and no problems at all
were reported.
Unfortunately when email arrives in production to our internal helpdesk group,
they are all blanks. No body at all.

It was really weird.

I double checked on my controller and I do put something in the email body
leaving the html body as a feature for the future for better formatting option.

After further unsuccessfull investigations I moved to pony rubygem source code.
Digging for awhile I reached the build_email method that actually it does all
the dirty job.

``` ruby pony.rb - build_mail() the interesting part
if options[:html_body]
  html_part do
    content_type 'text/html; charset=UTF-8'
    body options[:html_body]
  end
end
``` 

Mhm. It seems I found what I was looking for. If I pass to pony a non nil
html_body (actually I was passing an empty string), pony will overwrite the
plain text body with its html counterpart.

I just removed the html_body since it was not used and everything turned to
work.

``` ruby notification.rb - the send method modified
def send(options={})
  ret = setup(options)
  return ret if ENV['RACK_ENV'] == 'development'

   begin
    Pony.mail(:to=>self.to, :from=>"engage@mycompanydomain", :cc=>self.cc, :subject=>self.subject, :body=>self.body)
    self.sent=true
   rescue => e
     puts e.message
     self.sent=false
     raise e #useful to let rspec to handle the error condition
   end
  self.save
end
``` 

When something doesn't run the way you expected and you double check
your code and your tests, look at the third party libraries source, often you
will find there the answer.

Enjoy it!

