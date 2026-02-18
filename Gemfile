# frozen_string_literal: true

source "https://rubygems.org"

# Core Gems
gem "jekyll", "~> 3.9"
gem "rubyzip", "~> 2.3"
gem "webrick", "~> 1.9"
gem "base64"
gem "jekyll-seo-tag"
gem "jekyll-sitemap"
gem "kramdown-parser-gfm"
gem "bigdecimal"
gem "rouge"

group :jekyll_plugins do
  gem "jekyll-feed", "~> 0.6"
  gem "jekyll-paginate"
  gem "jekyll-gist"
  gem "jekyll-remote-theme"
end

# Dependency for remote themes and SSL
gem "openssl", "~> 3.3.2"
gem "faraday-retry"

# Performance-booster for watching directories on Windows
gem "wdm", "~> 0.1", :platforms => [:windows]

# Lock `http_parser.rb` gem to `v0.6.x` on JRuby builds since newer versions of the gem
# do not have a Java counterpart.
gem "http_parser.rb", "~> 0.6.0", :platforms => [:jruby]
