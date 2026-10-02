# Jekyll GitHub Site Project

Welcome! This project is set up using [Jekyll](https://jekyllrb.com/) for creating a beautiful static site that can be hosted on **GitHub Pages**.

![Jekyll Logo](https://assets.jekyllrb.com/jekyll-logo-48.png)

## Quick Start Guide

### 1. Install Jekyll (One-time Setup)

```bash
gem install jekyll
# Or use bundler:
bundle install
```

Alternative installation methods:
- **macOS**: `brew install jekyll`
- **Ubuntu/Debian**: `sudo apt-get install jekyll`
- **Ruby gem**: `gem install jekyll`

### 2. Run Development Server

```bash
jekyll serve       # Standard way (serves on localhost:4000)
bundle exec jekyll serve   # Using bundle
```

The site will be available at `http://localhost:4000/`

### 3. Build for Production Deploying to GitHub

```bash
jekyll build        # Builds the production version to _site folder
```

### 4. Deploy to GitHub Pages

1. Push your `_site` folder to a new repository on GitHub (or fork an existing project)
2. Enable GitHub Pages in settings
3. Set **Source branch**: `main` or **Deploy from branch**: `gh-pages`
4. Build locally first to verify: `jekyll build && jekyll serve`

## Project Structure

```
├── _config.yml        # Site configuration
├── README.md          # This file
├── _posts/            # Your blog posts (markdown files)
├── _includes/         # Reusable template includes
├── _layouts/          # Jekyll layouts
├── assets/            # CSS, JS, images
└── gems/              # Dependencies like Bundler config
```

## Configuration File (`_config.yml`)

Change these values in `_config.yml`:
- `title` - Your website's title
- `url` - The root URL of your site
- `base-url` - For subfolders (omit for root)
- `theme` - Theme to use (or null for default)
- Enable/disable analytics, social feeds, etc.

## Gemfile Dependencies

- jekyll (v4+) - Main Jekyll engine
- jekyll-feed - RSS feed support
- jekyll-paginate - Pagination
- jekyll-sitemap - Sitemap generation
- jekyll-watch - Development watch mode
- jemoji - GitHub-style emojis

## Deployment Checklist

✅ Run `jekyll build` locally first  
✅ Test the site at `/http://localhost:4000/`  
✅ Deploy to a private repo (if using for personal project)  
✅ Set up GitHub Pages with proper branch settings  

🎉 Your Jekyll GitHub site is ready to deploy!
