# This will ensure that when you run bundle install, you have the correct version 
# of the github-pages gem. If that fails, simplify it:
# ```ruby
# source 'https://rubygems.org'
# 
# gem 'github-pages'
# ```
#
# and be sure to run $ [bundle update] again

source 'https://rubygems.org'

require 'json'
require 'open-uri'
versions = JSON.parse(open('https://pages.github.com/versions.json').read)

gem 'github-pages', versions['github-pages']
