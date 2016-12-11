source 'https://rubygems.org'

require 'json'
require 'open-uri'
versions = JSON.parse(open('https://pages.github.com/versions.json').read)

group :test do
  gem 'html-proofer'
end

group :jekyll_plugins do
  gem 'github-pages', versions['github-pages']
  gem 'jekyll-paginate'
  gem 'jekyll-sitemap'
  gem 'jekyll-gist'
  gem 'jekyll-feed'
  gem 'jemoji'
end
