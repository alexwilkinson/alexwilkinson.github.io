require 'html/proofer'

# rake test
desc "build and test website"

task :test do
  sh "bundle exec jekyll build"
  HTML::Proofer.new("_site", {:href_ignore=> ['http://localhost:4000'], :verbose => true}).run
end

desc "Given a title as an argument, create a new blog post"
task :write, [:title] do |t, args|
  date_stamp = Time.now.strftime('%Y-%m-%d')
  title_param = args.title.gsub(/\s/, '_').downcase
  filename = "#{date_stamp}-#{title_param}.markdown"
  path = File.join("_posts", filename)

  raise RuntimeError.new("Won't clobber #{path}") if File.exist?(path)

  File.open(path, 'w') do |file|
    file.write <<-EOS
---
title: #{args.title}
layout: post
date: #{Time.now.strftime('%Y-%m-%d %k:%M:%S')}
blog: true
---
    EOS
  end

  puts "Created post at #{path}"
end
