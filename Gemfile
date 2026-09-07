source 'https://rubygems.org'

gem "jekyll", "~> 4.3.3"
gem "just-the-docs", "0.8.2"
gem "webrick", "~> 1.8"

group :jekyll_plugins do
  gem "jekyll-feed"
  gem "jekyll-sitemap"
  gem "jekyll-seo-tag"
  gem "jekyll-remote-theme"
  gem "jekyll-redirect-from"
end

# Link checker (run: bundle exec htmlproofer ./_site ...)
gem "html-proofer", "~> 5.0"

platforms :mingw, :x64_mingw, :mswin, :jruby do
  gem "tzinfo", ">= 1", "< 3"
  gem "tzinfo-data"
end

gem "wdm", "~> 0.1.1", :platforms => [:mingw, :x64_mingw, :mswin]
