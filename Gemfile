source "https://rubygems.org"

# Tema por defecto para nuevos sitios Jekyll
gem "minima", "~> 2.0"

# Plugin para incluir caché
gem "jekyll-include-cache"

# Plugins para Jekyll
group :jekyll_plugins do
  gem "jekyll-feed", "~> 0.6"
end

# Dependencias específicas para Windows
gem "tzinfo-data", platforms: [:mingw, :mswin, :x64_mingw, :jruby]
gem "wdm", "~> 0.1.0" if Gem.win_platform?

# Jekyll compatible con github-pages
gem "jekyll", "3.9.2" # Versión compatible con github-pages 227
gem 'github-pages', '~> 227', group: :jekyll_plugins

# Webrick para servidor local
gem "webrick", "~> 1.8"

