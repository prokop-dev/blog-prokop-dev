# My Blog

This is repository for my blog [available here](https://blog.prokop.dev).

## Hosting on CloudFlare

This blog is hosted on using [CloudFlare Pages](https://pages.cloudflare.com/) technology. I was previously using Netlify. Here is some [comparition](https://blog.logrocket.com/netlify-vs-cloudflare-pages/) between those two platforms.

- Cloudflare Pages [Original URL](https://blog-prokop-dev.pages.dev/).
- My custom domain [URL](https://blog.prokop.dev).

### Customization of CloudFlare build.

Use the following [version setting](https://developers.cloudflare.com/pages/how-to/deploy-a-hugo-site#using-a-specific-hugo-version) to use latest hugo.
This was necessary as I was unable to properly run hugo with template I have chosen.

## Blog stack

### Hugo

Always wanted to have static page produced by [hugo](https://gohugo.io/), so here we go.

### Template

Without too much thinking, I've chosen to use [PaperMod](https://github.com/adityatelange/hugo-PaperMod) theme. It looks great for me and was easy enough to setup.

#### Updating template

It is pretty easy and needs to be done from time to time to keep things tidy.

``` bash
cd themes/PaperMod
git pull
```

#### Template customization

Started working off example site from [here](https://github.com/adityatelange/hugo-PaperMod/tree/exampleSite):

- config.yml - update baseURL, title and googleAnalytics.

### Hosting static assets

