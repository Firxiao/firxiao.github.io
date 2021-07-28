```bash
# arch
yay -S ruby-build rbenv
rbenv install 2.7.3
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