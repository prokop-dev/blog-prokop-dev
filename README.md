# Prokop DEV Blog

My technology blog avaiable [here](https://blog.prokop.dev).

## Hosting on CloudFlare

This blog is hosted on using [CloudFlare Pages](https://pages.cloudflare.com/) techology. I was previously using Netlify. Here is some [comparition](https://blog.logrocket.com/netlify-vs-cloudflare-pages/) between those two platforms.

- My [Original URL](https://blog-prokop-dev.pages.dev/) 
- My [Custom Domain](https://blog.prokop.dev)

### Customization of CloudFlare build.

Use the following [version setting](https://developers.cloudflare.com/pages/how-to/deploy-a-hugo-site#using-a-specific-hugo-version) to use latest hugo.

## Blog stack

### Hugo

Always wanted to have static page produced by [hugo](https://gohugo.io/), so here we go.

### Template

Without too much thinking, I've chosen to use [PaperMod](https://github.com/adityatelange/hugo-PaperMod) theme. It looks great for me and was easy enough to setup.

#### Updating template

It is pretty easy

``` bash
cd themes/PaperMod
git pull
```