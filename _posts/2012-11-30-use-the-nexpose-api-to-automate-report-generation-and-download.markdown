---
layout: post
title: "Use the Nexpose API to automate report generation and download"
date: 2012-11-30 07:53
comments: true
published: true
featured: false
categories: breakers builders api nexpose va ruby rapid7 commercial-tool vulnerability-assessment nexty
thumb: nexty.png
level:
hn: 
rd: 
---

In [a previous post](http://armoredcode.com/blog/use-the-nexpose-api-to-add-a-search-by-ip-functionality-in-your-tools/)
I talked about [Rapid7 Nexpose)](http://www.rapid7.com) vulnerability assessment
tool and how you can write some ruby code to search a server by IP address.

Today I want to show you something I added to a rubygem I'm working on,
[nexty](https://github.com/thesp0nge/nexty). The idea is to give a command line
alternative to some GUI tasks if you need fresh data you want to grep, plot or
whatever.

<!-- more -->

## Something you can't play well with the GUI: reports

Nexpose is good and I'm pretty happy using it. However the web GUI reporting
functionality doesn't satisfy me that much.

To fulfil my goals, I created a report template with the information I need to
estract and I created a scheduled report using that template to create a CSV
file every month.

I observed that:
* it seems report template information was lost during the time, so my
  scheduled report will generated with a basic default template after a
  successful save;
* if I add a Nexpose site I have to manually add to the report information.
  This manual task is something annoying that it can bring to errors and so on.
  I need to automate the whole process.

## Nexty::Report API

This time I don't cook any raw request using API documentation. I'll create an
API on top to Nexpose native APIs.

If you look bin/nexty ruby command line utility in the
[nexty](https://github.com/thesp0nge/nexty) repository, you'll find there is a
'--report' command line flag that it will generate a report from a list of
Nexpose sites.

``` ruby nexty binary command
...
  when '--report'
    fn = Nexty::Report.generate_from_a_list_of_sites(arg, nsc)
    puts "Report saved: #{fn}".color(:white)
...
``` 

The idea is to extract all Nexpose sites, saving them in a text file and then
use them to fill our report.

``` ruby Nexty::Report.generate_from_a_list_of_sites
def self.generate_from_a_list_of_sites(site_list=nil, nsc)
  sites=Nexty::Sites.load_from_file(site_list)
  s = []
  sites.each do |site|
    s << nsc.find_site_by_name(site) 
  end
  result = Nexty::Report.generate(nsc, s, {:template=>"my-default-template", :format=>'csv', :filename=>nil, :scan_to_include=>4})
  Nexty::Report.download(result[:url], result[:filename], nsc)
end
``` 

The Nexty::Report.generate is the routing handling all the dirty job. It
creates a report using a given template (a fruther improvement it will be
letting the user to specify this using a flag) loading a number of scans in the
scan history for all the sites loaded from the list.

``` ruby Nexty::Report.generate
def self.generate(nsc, list, options={:template=>'', :filename=> '', :format=>"csv", :scan_to_include=>1})

  options[:filename] = "export_#{Time.now.strftime("%Y%m%d%H%M%s")}.csv" if options[:filename].nil? or options[:filename].empty?
  report = Nexpose::ReportConfig.new(nsc)
  report.set_name(options[:filename])
  report.set_template_id(options[:template])
  report.set_format(options[:format])

  list.each do |item|
    site_config = Nexpose::SiteConfig.new
    site_config.getSiteConfig(nsc, item[:site_id])
    scan_history = nsc.site_scan_history(item[:site_id])
    scan_history.sort! { |a,b| b[:start_time] <=> a[:start_time]}
    scan_history.take(options[:scan_to_include]).each do |scan|
      report.addFilter('scan', scan[:scan_id])
    end
  end

  report.saveReport

  url = nil
  while not url
    url = nsc.report_last(report.config_id)
    select(nil, nil, nil, 10)
  end

  full_url="https://#{nsc.host}:#{nsc.port}#{url}"

  {:url=>full_url, :filename=>options[:filename]}
end
```

As return value, the method prompts back the absolute URL you can grab the copy
of your report using either a browser or our gem with the --download option.

``` ruby Nexty::Report.download
def self.download(url, filename, nsc)
  return nil if url.nil? or url.empty?
  uri = URI.parse(url)
  http = Net::HTTP.new(uri.host, uri.port)
  http.use_ssl = true
  http.verify_mode = OpenSSL::SSL::VERIFY_NONE # XXX: security issue
  headers = {'Cookie' => "nexposeCCSessionID=#{nsc.session_id}"}
  resp = http.get(uri.path, headers)

  file = File.open(filename, "w")
  file.write(resp.body)
  file.close

  filename
end
``` 

I excluded SSL certificate check so, I introduced a potentially security issue
here. You may want to implement a full SSL certificate check to avoid [man in the middle](http://en.wikipedia.org/wiki/Man-in-the-middle_attack) attacks.

## Off by one

Extending a commercial tool with custom ruby code it is great. It gives me a
lot of freedom in terms of automating my daily job tasks.

Nexty can be improved in a lot of ways, and I will implement the Nexpose API
following their PDF documentation improving functionality and tests further
more.
