
desc 'Generate tags page'
task :tags do
  puts "Generating tags..."
  require 'rubygems'
  require 'jekyll'
  include Jekyll::Filters

  sidebar = ''
  sidebar << <<-HTML
  <ul><li>
  <h2>Categories</h2>
  <ul>
  HTML

  options = Jekyll.configuration({})
  site = Jekyll::Site.new(options)
  site.read_posts('')
  site.tags.sort.each do |category, posts|

    html = ''
    html << <<-HTML
---
layout: default
title: Postings tagged "#{category}"
---
    <h3 class="title">Posts tagged "#{category}"</h3>
    HTML

    html << '<div class="post"><ul>'
    posts.each do |post|
      post_data = post.to_liquid
      html << <<-HTML
        <li><a href="#{post.url}">#{post_data['title']}</a></li>
      HTML
    end
    html << '</ul></div>'

    FileUtils.mkdir_p "tags/"
    File.open("tags/#{category}.html", 'w+') do |file|
      file.puts html
    end
    sidebar << <<-HTML
      <li><a href="/tags/#{category}.html">#{category}</a></li>
    HTML
  end
  sidebar << "</ul></li></ul>"
  File.open("_includes/sidebar.html", 'w+') do |file|
    file.puts sidebar
  end
  puts 'Done.'
end
