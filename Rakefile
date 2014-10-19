require 'time'

desc 'Create a new post'
task :post do
  title = ENV['TITLE']
  slug = "#{Date.today}-#{title.downcase.gsub(/[^\w]+/, '-')}"

  file = File.join(
    File.dirname(__FILE__),
    '_posts',
    slug + '.markdown'
  )

  File.open(file, "w") do |f|
    f << <<-EOS.gsub(/^     /, '')
---
layout: post
title: #{title}
---
    EOS
  end

  system ("#{ENV['EDITOR']} #{file}")
end
