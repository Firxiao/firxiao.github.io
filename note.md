```bash
# arch
yay -S ruby-build rbenv
rbenv install 2.7.3
rbenv local
gem install bundler
bundle install
bundle exec jekyll serve
```