---
layout: page  
title: Setting Up a Blog on GitHub Pages  
date: 2024-09-22 15:05 +0100  
categories: [Blogging, Tutorial]  
tags: [github-pages, blog, personal blog, jekyll]  
---

## Starting My Blog

After thinking about it for a long time, I finally decided to start a blog! My first post is about how I set up this very blog, which you can do too.

## Why Choose GitHub Pages?

As a programmer, I’ve always wanted a personal website to showcase my projects. I explored platforms like WordPress, Medium, and Substack but settled on GitHub Pages and Jekyll for several reasons:
1. Complete customization control.
2. Free hosting without any extra costs.
3. A fast, lightweight, and easy-to-maintain platform.

If this sounds good to you, here’s a simple guide to help you get started.

---

## Step 1: Picking a Theme

The first step is choosing a Jekyll theme that fits your personal style. There are many places to find themes:

- [Jekyll Themes](https://jekyllthemes.io/)
- [Jekyll Themes](http://jekyllthemes.org/)
- [Jekyll Themes](https://jekyll-themes.com/)
- [Jamstack Themes](https://jamstackthemes.dev/ssg/jekyll/)

I picked the [Chirpy theme](https://github.com/cotes2020/chirpy-starter/), mainly because it has a dark mode that looks great!

---

## Step 2: Activating GitHub Pages

Once you have your theme, it’s time to host it on GitHub Pages. The steps may vary depending on the theme, but for Chirpy, follow these instructions:

1. Use this [template](https://github.com/cotes2020/chirpy-starter/generate) to generate your repository.
   - Name it as `<your-username>.github.io`
   - GitHub will automatically build and deploy the site.
   
2. Clone the repository to your local machine.

3. Install Ruby and Jekyll via the [Jekyll installation guide](https://jekyllrb.com/docs/installation/).

4. Run `bundle install` to get all the necessary dependencies.

5. Customize `_config.yml` by editing:
   - `url`: Your website’s address.
   - `avatar`: Your profile image.
   - `timezone` and `lang`: Adjust these to your preferences.

6. Run `bundle exec jekyll s` to preview your site locally.

For more details, refer to Chirpy’s [Getting Started Guide](https://chirpy.cotes.page/posts/getting-started/).

---

## Step 3: Setting a Custom Domain

If you want a personalized domain name for your blog, you’ll need to purchase it from a registrar. Some options include:

- [GoDaddy](https://www.godaddy.com/)
- [Google Domains](https://domains.google)
- [Namecheap](https://www.namecheap.com/)

### Configuring DNS Records

After buying a domain, you’ll need to configure its DNS settings. In your registrar's domain management, set the following A-type records to point to GitHub Pages’ IP addresses:

| Type  | Data              |
|-------|-------------------|
| A     | 185.199.108.153    |
| A     | 185.199.109.153    |
| A     | 185.199.110.153    |
| A     | 185.199.111.153    |
| CNAME | `<your-username>.github.io` |

These entries link your custom domain to your GitHub Pages site.

### Configuring GitHub Pages

Once your DNS records are set, head to your repository’s settings in GitHub to finalize the setup:

1. Navigate to **Settings**.
2. Scroll down to the **Pages** section.
3. Enter your custom domain in the **Custom Domain** field and save the settings.

Enable **Enforce HTTPS** to ensure your blog is secure with a free SSL certificate from [Let’s Encrypt](https://letsencrypt.org/).

---

By following these steps, you'll have a personalized blog up and running on GitHub Pages. From here, you can customize your site, write posts, and share your projects with the world—all without worrying about hosting fees or complex setup processes.

Happy blogging!
