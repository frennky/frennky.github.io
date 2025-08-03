# Setup

This repository contains personal blog. It uses [Hugo](https://gohugo.io/) to generate static files that are published as [Github pages](https://docs.github.com/en/pages). The style is based on [PaperMod](https://themes.gohugo.io/themes/hugo-papermod/) theme.

For more details on Hugo check [docs](https://gohugo.io/getting-started/quick-start/) and for more details on theme check [wiki](https://github.com/adityatelange/hugo-PaperMod/wiki).

## Running locally

```bash
go install github.com/gohugoio/hugo@latest
git clone git@github.com:frennky/frennky.github.io.git
cd frennky.github.io/
git clone https://github.com/adityatelange/hugo-PaperMod themes/PaperMod --depth=1
hugo server --disableFastRender
```
