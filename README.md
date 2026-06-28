### Serves GitHub Pages on https://alex-d-bondarev.github.io/SashaTests/

### How to debug locally

```shell
ruby-install ruby 3.2.11 
# restart session
chruby ruby-3.2.11
ruby --version 
bundle clean
bundle install
bundle update github-pages
bundle exec jekyll serve
```
