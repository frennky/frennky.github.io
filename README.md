# Intro

This repository contains personal blog. It uses [Jekyll](https://jekyllrb.com/) to generate static files that are published as [Github pages](https://docs.github.com/en/pages). The style is based on default theme [minima](https://github.com/jekyll/minima).

For more details on Jekyll check [docs](https://jekyllrb.com/docs/) and [source](https://github.com/jekyll/jekyll).

# Installing Jekyll

First install all prerequisites:
- Ruby 2.5.0+
- RubyGems
- GCC and Make

```bash
sudo apt install ruby-full build-essential zlib1g-dev
```

Add environment variables to `.bashrc`:

```bash
echo '# Install Ruby Gems to ~/gems' >> ~/.bashrc
echo 'export GEM_HOME="$HOME/gems"' >> ~/.bashrc
echo 'export PATH="$HOME/gems/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```
Install Jekyll:

```bash
gem install jekyll bundler
```

# Running locally

If you have everything setup, just run:

```bash
bundle exec jekyll serve -l
```
