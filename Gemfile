source "https://rubygems.org"

# The github-pages gem pins the exact Jekyll version and plugin set that
# GitHub's servers use to build Pages sites. Depending on it locally means a
# clean local build is a clean deploy -- there is no second toolchain that can
# drift out of sync with production.
#
# It requires Ruby 3.x. Ruby 4 removed Object#tainted?, which the Liquid
# version Jekyll 3.10 depends on still calls, so a Ruby 4 build dies on the
# first template. bin/serve puts the right Ruby on PATH for you.
gem "github-pages", group: :jekyll_plugins

# Ruby 3 dropped webrick from the standard library; `jekyll serve` needs it to
# run the local preview server.
gem "webrick", "~> 1.8"
