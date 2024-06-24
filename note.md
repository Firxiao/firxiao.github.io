```
bundle config mirror.https://rubygems.org https://mirrors.ustc.edu.cn/rubygems/
```


```bash
# arch
yay -S ruby-build rbenv

# mac
brew install rbenv ruby-build

# Load rbenv automatically by appending
# the following to ~/.zshrc:
eval "$(rbenv init -)"

rbenv install 3.3.2
rbenv local
gem install bundler
bundle install
bundle exec jekyll serve

# alpine
sudo sed -i 's#dl-cdn.alpinelinux.org#mirrors.ustc.edu.cn#g' /etc/apk/repositories
apk add ruby ruby-dev ruby-bundler gcc g++ make zlib-dev
bundle config set --local path 'vendor/bundle'
bundle install
bundle exec jekyll serve
```
