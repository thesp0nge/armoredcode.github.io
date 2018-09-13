---
layout: post
title: "Use the Nexpose API to add a search by IP functionality in your tools"
date: 2012-08-24 09:04
comments: true
published: true
featured: false
categories: breakers builders api nexpose va ruby rapid7 commercial-tool vulnerability-assessment nexty
hn: 
rd: 
---

It's one of my recurrent thoughts. Tools must expose an API allowing people to
customize the tool behaviour to fit their needs.
[Nexpose](http://www.rapid7.com/) it is a commercial tool for vulnerability
assessment exposing an API and I'm happy about it.

<!-- more -->

## Extending your tool

Take two different security specialists and you will see two completely
different working habits and two completely different needs about data
representation and data mining.
How can a commercial security tool can cover this? Exposing an API to ask for
services and to work on results.

One of the most successful feature in open source projects is the way you can
combine more tools or even hack their code to achieve a need.

In closed source you can't hack the code and tools combinations are limited to
how a tool is effective in exporting data. Sometimes they're not so flexible as
I wish.

For such a reason having an API an appsec man can use to ask the tool (he pays
for) for services can be a great deal.

## The need

I do use [Nexpose](http://www.rapid7.com/) for vulnerability assessment and I
want a quick way to pickup a list of ip addresses and extract a CSV report of
the last scan I made. 

Using the GUI it is a cumbersome task.
The [naive ruby gem](https://github.com/rapid7/nexpose-client) either doesn't
allow to quick search for ip addresses and having back the associated device
(using the tool slang to call an host representation).

That's how I solved the issue.

Since I'm lucky enough to have a PDF explaining how to prompt the tool for
requests, I extended the Nexpose::Device class used to represent an host.

Nexpose doesn't allow a direct query against an IP address so I have first to
retrieve the whole list of devices before making a search.

This is not an elegant solution but it's the only way to achieve this goal.

``` ruby
module Nexpose
  class Device
    def self.all(connection)
      @devices = []

      r = connection.execute('<SiteDeviceListingRequest session-id="' + connection.session_id + '"/>')
      if (r.success)
        r.res.elements.each('SiteDeviceListingResponse/SiteDevices') do |rr|
          @sid = rr.attribute("site-id")
          rr.elements.each('device') do |d|
            @devices.push(Nexpose::Device.new(d.attributes['id'], @sid, d.attributes["address"], d.attributes["riskfactor"], d.attributes['riskscore']))
          end
        end
			end
      @devices
    end

    def self.find_by_address(connection, address)
      devices = Nexty::Device.all(connection)
      devices.each do |d|
        if d.address == address
          return d
        end
      end

      return nil
    end
  end
end
``` 

A further more interesting enhancement could be storing the whole device list
in a precomputed cache so you don't have to make the whole XML request each
time.

## Disclaimer

This is not a sponsored post nor I'm affiliated in any manner with Rapid7. I'm
just use their tool and I like the way I can use their public available API
with ruby.

