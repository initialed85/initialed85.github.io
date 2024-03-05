# initialed85's tech blog

# status: up to date

This is the source for my tech blog; you can view it at [initialed85.cc](https://initialed85.cc/).

## Technologies

-   Static content
    -   [Hugo](https://gohugo.io/)
-   Comments
    -   [Disqus](https://disqus.com/)
-   Continuous deployment
    -   [GitHub Actions](https://github.com/features/actions)
-   Hosting
    -   [GitHub Pages](https://pages.github.com/)
-   Content Delivery Network
    -   [CloudFlare](https://www.cloudflare.com/en-au/)

## Usage

If you're cloning this repo for the first time, ensure to clone the submodules:

```shell
git clone git@github.com:initialed85/initialed85.github.io.git --recurse-submodules
```

To make a new article:

```shell
hugo new posts/some-post-name.md
```

To run Hugo for local article development:

```shell
hugo server --buildDrafts --disableFastRender
```
