# Chris Cundy's Website Guide

## Overview

This is a personal academic website built with Hugo using the Academic theme (now known as Wowchemy). The site follows a simple structure with minimalist design principles for fast loading.

## Key Components

### Site Structure

- **Content**: All content is in `/content/` directory
  - `/content/authors/admin/` - Your profile information and avatar
  - `/content/home/` - Widget configurations for the homepage
  - `/content/post/` - Blog posts
  - `/content/publication/` - Academic publications

### Configuration Files

- **Main Config**: `/config/_default/config.toml` - Core Hugo settings
- **Menu Config**: `/config/_default/menus.toml` - Navigation links
- **Params Config**: `/config/_default/params.toml` - Theme settings, features, appearance

### Custom Files

- **Custom CSS**: `/assets/scss/custom.scss` - Performance optimizations and styling overrides
- **Custom Layout**: `/layouts/_default/baseof.html` - Modified to remove the navbar for better performance
- **CV Redirect**: The CV link at `/cv` redirects to `/files/cv.pdf` via Netlify configuration in `netlify.toml`

## Performance Optimizations

Several optimizations have been made to improve page load speed:

1. **Removed Navbar**: The navigation bar has been completely removed to minimize render-blocking elements.

2. **Disabled CMS**: Netlify CMS is disabled (`netlify_cms = false` in params.toml) to prevent loading unnecessary JavaScript.

3. **Custom CSS**: Performance-focused CSS properties for the avatar image.

4. **WebP Images**: Avatar images are available in WebP format for better compression.

## Content Management

### Adding Blog Posts

Add Markdown files to `/content/post/` with the following front matter:

```yaml
---
title: "Post Title"
date: 2023-05-17
draft: false
---
```

### Adding Publications

Add new publications in `/content/publication/` with a separate folder for each:

```
/content/publication/lastname-year-shortname/
  ├── index.md   # Publication details
  ├── cite.bib   # BibTeX citation
  └── paper.pdf  # Optional PDF
```

## Deployment

The site is deployed via Netlify. Each commit to the master branch triggers an automatic rebuild and deployment.

### Build Command

```
hugo --gc --minify -b $URL
```

## Maintenance Tasks

- **Update Academic Theme**: If needed, run `./update_academic.sh`
- **Update Hugo**: Make sure Hugo version in netlify.toml matches your local version
- **Image Optimization**: Consider converting new images to WebP format for better performance

## Troubleshooting

If you encounter issues with the site:

1. Check the Netlify deploy logs for errors
2. Verify that content files are in the correct format and location
3. Run `hugo server` locally to test changes before pushing

For performance testing, use Google Lighthouse in Chrome DevTools.