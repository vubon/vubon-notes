---
title: 'How to Deploy Your Hugo Blog to GitHub Pages'
date: 2025-12-31 06:45:00
lastmod: 2025-12-31 06:45:00
draft: false
tags:
  - 'Hugo'
  - 'GitHub Pages'
  - 'Static Site'
  - 'Deployment'
categories:
  - 'DevOps'
aliases:
  - '/deploying-hugo-blog-to-github-pages/'
---

If you want to publish a blog without paying for hosting? GitHub Pages is your friend. In this article, I will show you how to deploy a Hugo static site to GitHub Pages. Trust me, once you get it right, it's super smooth.

## Why Hugo + GitHub Pages?

I was looking for a simple way to publish my blog. I don't like to deal with WordPress hosting fees, database backups, or security updates. Hugo generates static HTML files—fast, secure, and free to host on GitHub Pages. What's not to love?

## What You'll Need

Before we start, make sure you have:
1. Hugo installed on your machine (I'm using Hugo Extended v0.138.0)
2. A GitHub account
3. Basic knowledge of Git commands
4. A Hugo site ready to deploy (if you don't have one, run `hugo new site myblog` to create one)

## Step 1: Create Your GitHub Repository

First things first, create a new repository on GitHub. Let's say your username is `yourname` and you want your blog at `https://yourname.github.io/myblog/`.

Create a repo named `myblog`. If you want it at the root (like `https://yourname.github.io/`), name the repo `yourname.github.io` instead.

## Step 2: Install a Hugo Theme

Hugo doesn't come with a default theme, so you need to install one. You can browse themes at [https://themes.gohugo.io/](https://themes.gohugo.io/). For this example, let's use the "Typo" theme (it's the one I'm using).

Navigate to your Hugo site directory and install the theme:

```bash
cd myblog
git init
git clone https://github.com/tomfran/typo.git themes/typo
```

Now update your `hugo.toml` to use the theme:

```toml
theme = 'typo'
```

Build the site locally to make sure everything works:

```bash
hugo server
```

Visit `http://localhost:1313` in your browser. If you see your site, you're good to go. If not, check that the theme files are in `themes/typo/` and the theme name matches in your config.

## Step 3: Add Content to Your Blog

Now let's create some actual content. Hugo makes it easy to generate new posts.

### Create Your First Post

Run this command to create a new blog post:

```bash
hugo new content/posts/my-first-post.md
```

This creates a new markdown file at `content/posts/my-first-post.md`. Open it and you'll see something like this:

```markdown
---
title: 'My First Post'
date: 2025-12-31T10:00:00Z
draft: true
---
```

The stuff between the `---` markers is called "front matter"—it's metadata about your post. The important part here is `draft: true`. Posts marked as drafts won't appear on your published site. When you're ready to publish, change it to `draft: false`.

Now add some content below the front matter:

```markdown
---
title: 'My First Post'
date: 2025-12-31T10:00:00Z
draft: false
tags:
  - 'Hugo'
  - 'Blogging'
categories:
  - 'Technology'
---

## Welcome to My Blog

This is my first post using Hugo and GitHub Pages. Pretty cool, right?

I can write in **Markdown**, add `code snippets`, and even include images.

```python
def hello_world():
    print("Hello from my blog!")
```

Let me know what you think!
```

### Set Up Your Home Page

Create a file at `content/_index.md` for your home page:

```bash
mkdir -p content
touch content/_index.md
```

Add some content:

```markdown
---
title: 'Home'
---

Welcome to my blog! I write about technology, coding, and whatever else interests me.

Check out my latest posts below.
```

### Test Your Content Locally

Run the Hugo server to see your changes:

```bash
hugo server -D
```

The `-D` flag includes drafts, so you can preview posts before publishing them. Visit `http://localhost:1313` and you should see your home page and your first post.

## Step 4: Configure Hugo for GitHub Pages

This is where most people get confused. Your Hugo site needs to know where it will be published. Open your `hugo.toml` (or `config.toml`) and set the `baseURL`:

```toml
baseURL = 'https://yourname.github.io/myblog/'
languageCode = 'en-us'
title = 'My Blog'
theme = 'typo'
```

**Important:** If your site is in a subdirectory (like `/myblog/`), all your links need to include that prefix. The Typo theme uses a custom menu structure under `[[params.menu]]`. Here's how to configure it:

```toml
[params]
  description = 'My blog about tech and coding'

  # Disable breadcrumbs to avoid duplicate navigation on post pages
  [params.breadcrumbs]
    enabled = false

  [[params.menu]]
    name = 'home'
    url = '/myblog/'

  [[params.menu]]
    name = 'blog'
    url = '/myblog/posts/'
```

If you skip this step, your navigation will break. Been there, done that.

**Note:** Different themes handle menus differently. Some use Hugo's standard `[[menus.main]]` structure, but Typo uses `[[params.menu]]`. The breadcrumbs setting prevents duplicate navigation on individual post pages. Always check your theme's documentation.

## Step 5: Set Up GitHub Actions Workflow

GitHub Actions will build and deploy your site automatically every time you push to `main`. Create this file:

`.github/workflows/deploy.yml`

```yaml
name: Deploy Hugo site to GitHub Pages

on:
  push:
    branches:
      - main
  workflow_dispatch:

permissions:
  contents: write

concurrency:
  group: "pages"
  cancel-in-progress: false

jobs:
  deploy:
    runs-on: ubuntu-latest
    env:
      HUGO_VERSION: 0.138.0 # you can use latest version here.
    steps:
      - name: Checkout
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Install Hugo CLI
        run: |
          wget -O ${{ runner.temp }}/hugo.deb https://github.com/gohugoio/hugo/releases/download/v${HUGO_VERSION}/hugo_extended_${HUGO_VERSION}_linux-amd64.deb \
          && sudo dpkg -i ${{ runner.temp }}/hugo.deb

      - name: Build with Hugo
        env:
          HUGO_CACHEDIR: ${{ runner.temp }}/hugo_cache
          HUGO_ENVIRONMENT: production
        run: |
          hugo \
            --gc \
            --minify

      - name: Deploy to gh-pages
        uses: peaceiris/actions-gh-pages@v4
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./public
          publish_branch: gh-pages
          force_orphan: true
          keep_files: false
```

Let me explain what this does:
- **Checkout**: Gets your source code
- **Install Hugo CLI**: Downloads and installs Hugo Extended
- **Build with Hugo**: Generates the static site in `./public`
- **Deploy to gh-pages**: Pushes the built site to a separate `gh-pages` branch

## Step 6: Handle Your Theme Properly

Here's a gotcha—if your theme is a Git submodule, GitHub Actions won't check it out by default. You have two options:

**Option 1: Vendor the theme** (recommended)

```bash
# Remove the submodule
git rm themes/your-theme
rm -rf .git/modules/themes/your-theme

# Copy the theme files directly
cp -r /path/to/theme themes/your-theme
git add themes/your-theme
git commit -m "Vendor theme files"
```

**Option 2: Initialize submodules in the workflow**

Add this step before "Build with Hugo":

```yaml
- name: Initialize theme submodule
  run: git submodule update --init --recursive
```

I prefer Option 1 because it's simpler and you have full control over the theme files.

## Step 7: Enable GitHub Pages

Push your code to GitHub:

```bash
git add .
git commit -m "Initial Hugo site setup"
git push origin main
```

The workflow will run automatically and create the `gh-pages` branch. Once it's done:

1. Go to your repo settings
2. Click "Pages" in the sidebar
3. Under "Source", select the `gh-pages` branch
4. Set the folder to `/ (root)`
5. Click "Save"

Wait a minute or two for GitHub to build your site. Then visit `https://yourname.github.io/myblog/` and boom—your blog is live!

## Common Issues I Ran Into

### Issue 1: 404 Errors Everywhere

If you're getting 404s, check your `baseURL` in `hugo.toml`. It must match your GitHub Pages URL exactly. Also, make sure your menu links include the subdirectory prefix.

### Issue 2: Missing Styles or Broken Layout

This usually means your theme files didn't get deployed. Make sure the theme is either vendored or the submodule is properly initialized in the workflow.

### Issue 3: The Site Builds But Shows RSS Instead of HTML

Check that your theme includes an `index.html` template. Some themes are missing this, and Hugo falls back to RSS output. Either fix the theme or switch to one that's better maintained.

### Issue 4: Images Not Loading

If your images are at `/images/photo.jpg`, they need to be `/myblog/images/photo.jpg` when deployed to a subdirectory. Update all image paths in your markdown:

```markdown
![My photo](/myblog/images/photo.jpg)
```

## Why Use `gh-pages` Branch?

You might wonder, "Why not just deploy from `main`?" Good question. Here's the deal:

- **Separation of concerns**: `main` holds your source files (markdown, templates, config). The `gh-pages` branch holds the generated HTML.
- **Clean history**: Generated files create huge diffs and clutter your Git history. Keeping them on a separate branch (with `force_orphan: true`) means no history at all.
- **Easy rollbacks**: If something breaks, you can always regenerate from source without dealing with conflicting generated files.

## Wrapping Up

That's it! You now have a Hugo blog deployed to GitHub Pages with automatic deployment via GitHub Actions. Every time you push new posts to `main`, GitHub builds and deploys your site automatically.

Here's what we covered:
1. Set up a GitHub repository
2. Install a Hugo theme
3. Add content to your blog (posts and home page)
4. Configure Hugo with the correct `baseURL` and menu links
5. Create a GitHub Actions workflow for automated deployment
6. Handle theme files properly (vendor them or use submodules)
7. Enable GitHub Pages to serve from the `gh-pages` branch

Hope this article saves you from the headaches I went through. If you found it helpful, great! See you in the next one.

## Resources

- [Hugo Documentation](https://gohugo.io/documentation/)
- [GitHub Pages Documentation](https://docs.github.com/en/pages)
- [peaceiris/actions-gh-pages](https://github.com/peaceiris/actions-gh-pages)
