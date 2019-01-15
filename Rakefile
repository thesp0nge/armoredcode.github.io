#require "rubygems"
require "bundler/setup"
require "stringex"
## -- Config -- ##

ssh_user       = "thesp0nge@goliath.armoredcode.com"
ssh_port       = "22"
document_root  =  "/var/www/armoredcode.com"
rsync_delete   = true
deploy_default = "rsync"
public_dir      = "_site"    # compiled site directory

deploy_dir      = "_site"
posts_dir       = "_posts"    # directory for blog files
news_dir        = "_news"
new_post_ext    = "markdown"  # default new post file extension when using the new_post task
new_page_ext    = "markdown"  # default new page file extension when using the new_page task

deploy_dir      = "_site"
source_dir      = "_source"
draft_dir       = "_drafts"

css_output_dir  = "#{deploy_dir}/stylesheets"
less_dir        = "#{source_dir}/assets/less"
image_dir       = "#{source_dir}/images"

#############################
# Create a new Post or Page #
#############################

namespace :new do

desc "Create a new draft in #{draft_dir}"
task :draft, :title do |t, args|
  if args.title
    title = args.title
  else
    title = get_stdin("Enter a title for your post: ")
  end
  filename = "#{draft_dir}/#{title.to_url}.#{new_post_ext}"
  if File.exist?(filename)
    abort("rake aborted!") if ask("#{filename} already exists. Do you want to overwrite?", ['y', 'n']) == 'n'
  end
  tags = get_stdin("Enter tags to classify your post (comma separated): ")
  create_post(filename, title, "post", tags)
end

desc "Create a new news in #{news_dir}"
task :news, :title do |t, args|
  if args.title
    title = args.title
  else
    title = get_stdin("Enter the news title: ")
  end
  filename = "#{news_dir}/#{Time.now.strftime('%Y-%m-%d')}-#{title.to_url}.#{new_post_ext}"
  if File.exist?(filename)
    abort("rake aborted!") if ask("#{filename} already exists. Do you want to overwrite?", ['y', 'n']) == 'n'
  end
  tags = get_stdin("Enter tags to classify your news (comma separated): ")
  create_post(filename, title, "news", tags)
end

# usage rake new_post
desc "Create a new post in #{posts_dir}"
task :post, :title do |t, args|
  if args.title
    title = args.title
  else
    title = get_stdin("Enter a title for your post: ")
  end
  filename = "#{posts_dir}/#{Time.now.strftime('%Y-%m-%d')}-#{title.to_url}.#{new_post_ext}"
  if File.exist?(filename)
    abort("rake aborted!") if ask("#{filename} already exists. Do you want to overwrite?", ['y', 'n']) == 'n'
  end
  tags = get_stdin("Enter tags to classify your post (comma separated): ")
  create_post(filename, title, "post", tags)
end

# usage rake new_page
desc "Create a new page"
task :page, :title do |t, args|
  if args.title
    title = args.title
  else
    title = get_stdin("Enter a title for your page: ")
  end
  filename = "#{title.to_url}.#{new_page_ext}"
  if File.exist?(filename)
    abort("rake aborted!") if ask("#{filename} already exists. Do you want to overwrite?", ['y', 'n']) == 'n'
  end
  tags = get_stdin("Enter tags to classify your page (comma separated): ")
  puts "Creating new page: #{filename}"
  open(filename, 'w') do |page|
    page.puts "---"
    page.puts "layout: page"
    page.puts "permalink: /#{title.to_url}/"
    page.puts "title: \"#{title}\""
    page.puts "modified: #{Time.now.strftime('%Y-%m-%d %H:%M')}"
    page.puts "tags: [#{tags}]"
    page.puts "image:"
    page.puts "  feature:"
    page.puts "  credit:"
    page.puts "  creditlink:"
    page.puts "share: true"
    page.puts "---"
  end
end
end

###############################################################################
# Notification tasks
# Taken as is from https://github.com/mmistakes/made-mistakes-jekyll
#
# This is the best Jekyll based blog seen from far. Tons of kudos
###############################################################################

# Usage: rake notify
task :notify => ["notify:pingomatic", "notify:google", "notify:bing"]
desc "Notify various services that the site has been updated"
namespace :notify do

  desc "Notify Ping-O-Matic"
  task :pingomatic do
    begin
      require 'xmlrpc/client'
      puts "* Notifying Ping-O-Matic that the site has updated"
      XMLRPC::Client.new('rpc.pingomatic.com', '/').call('weblogUpdates.extendedPing', 'codiceinsicuro.it' , 'https://codiceinsicuro.it', 'https://codiceinsicuro.it/atom.xml')
    rescue LoadError
      puts "! Could not ping ping-o-matic, because XMLRPC::Client could not be found."
    end
  end

  desc "Notify Google of updated sitemap"
  task :google do
    begin
      require 'net/http'
      require 'uri'
      puts "* Notifying Google that the site has updated"
      Net::HTTP.get('www.google.com', '/webmasters/tools/ping?sitemap=' + URI.escape('https://codiceinsicuro.it/sitemap.xml'))
    rescue LoadError
      puts "! Could not ping Google about our sitemap, because Net::HTTP or URI could not be found."
    end
  end

  desc "Notify Bing of updated sitemap"
  task :bing do
    begin
      require 'net/http'
      require 'uri'
      puts '* Notifying Bing that the site has updated'
      Net::HTTP.get('www.bing.com', '/webmaster/ping.aspx?siteMap=' + URI.escape('https://codiceinsicuro.it/sitemap.xml'))
    rescue LoadError
      puts "! Could not ping Bing about our sitemap, because Net::HTTP or URI could not be found."
    end
  end
end


def create_post(filename, title, category, tags)
  puts "Creating new post: #{filename}"
  open(filename, 'w') do |post|
    post.puts "---"
    post.puts "layout: #{category}"
    post.puts "title: \"#{title.gsub(/&/,'&amp;')}\""
    post.puts "image: "
    post.puts "bootstrap: true"
    post.puts "tags: [#{tags}]"
    post.puts "categories: [ ]"
    post.puts "author: thesp0nge"

    post.puts "---"
  end
end
def get_stdin(message)
  print message
  STDIN.gets.chomp
end

def ask(message, valid_options)
  if valid_options
    answer = get_stdin("#{message} #{valid_options.to_s.gsub(/"/, '').gsub(/, /,'/')} ") while !valid_options.include?(answer)
  else
    answer = get_stdin(message)
  end
  answer
end

namespace :blog do
  desc "Deploy website via rsync"
  task :rsync do
    exclude = ""
    if File.exists?('./rsync-exclude')
      exclude = "--exclude-from '#{File.expand_path('./rsync-exclude')}'"
    end
    puts "## Deploying website via Rsync"
    system("rsync -avze 'ssh -p #{ssh_port}' #{exclude} #{"--delete" unless rsync_delete == false} #{deploy_dir}/ #{ssh_user}:#{document_root}")
  end

  desc "Generate jekyll site"
  task :generate do
    puts "## Generating Site with Jekyll"
    system "JEKYLL_ENV=production jekyll build"
  end
end
