source "https://rubygems.org"

gem "github-pages", group: :jekyll_plugins
gem "jekyll-include-cache", group: :jekyll_plugins
gem "minimal-mistakes-jekyll"

# Override liquid to 4.0.4+ to fix `tainted?` NoMethodError on Ruby 3.3+
gem "liquid", ">= 4.0.4"

group :jekyll_plugins do
end

# Ruby 3.4+ no longer bundles these stdlib gems.
# Keep them here so local builds work, but GitHub Actions
# (both built-in and custom workflows using ruby/setup-ruby)
# install them via bundle install, not via github-pages gem.
gem "csv"
gem "bigdecimal"
gem "base64"
gem "logger"
gem "mutex_m"
gem "abbrev"
gem "kramdown-parser-gfm"
gem "webrick"
