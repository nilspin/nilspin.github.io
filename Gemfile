source "https://rubygems.org"

# This line installs the exact Jekyll version and all plugins that GitHub Pages uses
gem "github-pages", group: :jekyll_plugins

# Since we are using a remote theme, we need this plugin specifically
gem "jekyll-theme-console"

# Add any other Gem dependencies here if needed (e.g., if you install a new plugin)

# The following lines ensure we use the development environment for local testing
group :development do
  gem "jekyll"
end

# Windows does not include zoneinfo files, so bundle the tzinfo-data gem
gem "tzinfo-data", platforms: [:windows]
gem "wdm", ">= 0.1.0" if Gem.win_platform?