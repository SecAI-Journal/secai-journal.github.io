source "https://rubygems.org"

gem "jekyll", "~> 4.3"
gem "jekyll-theme-chirpy", "~> 7.2"

group :jekyll_plugins do
  gem "jekyll-sitemap"
  gem "jekyll-seo-tag"
  gem "jekyll-archives"
  gem "jekyll-paginate"
end

platforms :mingw, :x64_mingw, :mswin, :jruby do
  gem "tzinfo", ">= 1", "< 3"
  gem "tzinfo-data"
end

# UPDATED: Using 0.2.0 for Ruby 3.4 compatibility
gem "wdm", ">= 0.2.0", :platforms => [:mingw, :x64_mingw, :mswin]

# Optional: If you encounter an error with 'webrick' (common in Ruby 3.0+)
gem "webrick", "~> 1.8"