---
title: "How'd I Get Here?  GitHub + Hugo + Namecheap"
date: 2022-07-25T09:45:59-04:00
draft: false
toc: false
images:
tags: 
  - blog
---

>A man has no name.
>
>-Jaqen H'ghar 
---
I consider myself a private person, especially my digital persona.  I don't do social media.  The last public internet presence I updated was probably my LinkedIn account some 10 years ago and a short-lived MySpace account almost 20 years ago.  Surprisingly my Flickr account is still active and I'll ask an occassional question on Reddit but that's about it.  Call me old-skool but I prefer to keep my interactions in-person.

However, I do believe in good documentation.  Even if you're the only person who will ever look at your docs, write it down!

Whomever should stumble upon this post, this is how I provisioned this blog.

---

# GitHub Pages is your friend

The Hugo team has a good doc on hosting a blog on GitHub Pages [here](https://gohugo.io/hosting-and-deployment/hosting-on-github/), but this is what I did:

1.  From github.com create two public repositories: blog and codedinsugar.github.io.  The first repo name is arbitrary of course, but the second repo name has to be in the syntax of `<github username>.github.io`.  Obviously for the sake of this tutorial, just substitute codedinsugar for whatever your github username is.
2.  From wherever your typewriter sits, clone both repos above to their own paths:
```bash
cd <arbitrary-path>
git clone https://github.com/codedinsugar/blog.git
git clone https://github.com/codedinsugar/codedinsugar.github.io.git
```
3.  Browse to the local blog repo and create a new hugo site:
```bash
cd blog
hugo new site blog
```
4.  Browse to the local codedinsugar.github.io repo and create a main branch, add a README.md file, commit and push.  This is where the static content will live.
```bash
cd codedinsugar.github.io
git checkout -b main
touch README.md
git add .
git commit -m "Initial commit for static content"
git push origin main
```
6.  Browse back to the local blog repo.  Delete the `blog/public` folder if it already exists (should be empty).  Add the local codedinsugar.github.io repo as a submodule to a new `blog/public` folder:
```bash
cd blog
rm -rf blog/public
git submodule add -b main https://github.com/codedinsugar/codedinsugar.github.io.git public
```
7.  Pick and use a Hugo [theme](https://themes.gohugo.io/).  I went with [Hermit](https://themes.gohugo.io/themes/hermit/) but I'm not pointing to it directly at the `blog/themes` path.  Instead I just cloned the theme outside of the local blog path, looked at the tree structure and copied the relevant files to the local blog repo.  I don't have a `theme="Hermit"` variable in my `config.toml` file either.  This method is cleaner for me because if I'm going to make changes then I don't want to have to depend on an external theme repo.  If you choose to use a theme directly, read about it [here](https://themes.gohugo.io/themes/hermit/)

8.  Start creating that sweet content in the local blog repo:
```bash
cd blog
hugo new posts/foo.md

#file is created at blog/content/posts/foo.md
```
9.  Run the local hugo server to make sure everything builds, renders successfully, and most it passes an initial satisfaction test:
```bash
hugo server -D

#browse to http://localhost:1313
```
10.  Build all binaries which get published to `blog/public`:
```bash
hugo
```
11.  Add, commit, and push the static content to GitHub:
```bash
cd public #remember this is a submodule to the static content repo
git add .
git commit -m "Adding first post"
git push origin main
```
12.  Browse to https://github.com/codedinsugar/codedinsugar.github.io and verify your commits.
13.  Verify HTTPS is enabled for public codedinsugar repo > Settings > Pages > Enforce HTTPS.
14.  Browse to https://codedinsugar.github.io and enjoy your new site.
15.  If you don't intend to use custom domain redirection to point to your site then you're done here, otherwise see next section for Namecheap setup.

---
# Namecheap is also your friend

Namecheap has always been my go to domain registrar for several years.  They're an ICANN accredited registrar and I've never had an issue with their services.  Getting started is pretty simple, create an account and buy a domain name.  Don't worry about buying an SSL certificate from Namecheap since GitHub Pages will be handling [HTTPS](https://docs.github.com/en/pages/getting-started-with-github-pages/securing-your-github-pages-site-with-https).

Namecheap has a [knowledgebase article](https://www.namecheap.com/support/knowledgebase/article.aspx/9645/2208/how-do-i-link-my-domain-to-github-pages/) on how to link your custom domain with GitHub Pages which is exactly what I followed verbatim.

After creating all of the necessary DNS records, head back to your public GitHub Pages repo > Settings > Pages > Custom domain and add your apex domain here e.g. codedinsugar.com.  If the DNS check fails, then you're likely missing a CNAME (case sensitive) file in your public repo.  The value in the CNAME record should be just your apex domain without any http(s):// prefix.

There really isn't much to pointing a custom domain to GitHub Pages, they even have a troubleshooting guide [here](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/troubleshooting-custom-domains-and-github-pages) if you get stuck.  Good luck!