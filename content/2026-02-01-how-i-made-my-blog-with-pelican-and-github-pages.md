Title: How I Made My Blog with Pelican and GitHub Pages
Date: 2026-02-01
Category: Tutorial
Tags: pelican, github-pages, python, blog, tutorial
Slug: how-i-made-my-blog-pelican-github-pages

## Introduction

In my first post, I mentioned that setting up GitHub Pages would probably be my next article. Well, here it is! This is a detailed walkthrough of how I set up this blog using Pelican and GitHub Pages, from creating the repository to getting it deployed and live.

I already touched on the basics of Pelican in my "Blog Is Back" post, but this tutorial goes much deeper. I'll show you the complete setup process, including all the configuration steps, GitHub Actions for automated deployment, and the practical decisions I made along the way.

I figured out all these steps with ChatGPT's help, working through each part of the setup until I had a working blog. Now I'm documenting everything in one place so you don't have to go through the same trial and error. This isn't generic documentation. It's exactly how I built this blog, with my actual configuration and the reasoning behind each choice.

**Note:** Throughout this tutorial, I use `ericbsd` as my GitHub username and show my actual configuration. Replace these with your own values as you follow along.

## Creating the GitHub Repository

Create a new GitHub repository named `username.github.io` (replace `username` with your actual GitHub username). In my case, the username is ericbsd.

**Why this specific name?** GitHub Pages uses a special convention: repositories named `username.github.io` are automatically published as user sites at `https://username.github.io`. This is different from project pages, which use a different URL structure.

**Important settings when creating the repository:**
- Set the repository to **public** (GitHub Pages requires public repositories on the free tier)
- Add a `.gitignore` using the **Python** template to automatically ignore Python-specific files

Once created, clone the repository:
```bash
git clone https://github.com/ericbsd/ericbsd.github.io.git
cd ericbsd.github.io
```

## Installing Pelican and Dependencies

### Create requirements.txt

I started by creating a `requirements.txt` file in the project root with the dependencies I needed:

```text
pelican[markdown]
markdown
```

Creating this first made sense because:
- PyCharm can use it to automatically install dependencies when setting up the virtual environment
- GitHub Actions will need it later for automated deployment
- It documents what the project needs upfront

### Set Up Virtual Environment

My PyCharm is configured to use the system Python for GhostBSD tools development, so I manually created a virtual environment for this blog project in PyCharm to keep the dependencies isolated.

In PyCharm, I created a new virtual environment through the project settings. PyCharm then detected the requirements.txt file and prompted me to install the packages.

If you're not using PyCharm, you can create a virtual environment and install it from requirements.txt with:

```bash
python -m venv .venv
source .venv/bin/activate  # On Windows: .venv\Scripts\activate
pip install -r requirements.txt
```

## Initializing the Pelican Project

Pelican provides a `pelican-quickstart` command that guides you through the initial setup with a series of prompts. If you've used Sphinx before, this will feel familiar.

### Run pelican-quickstart

I ran the quickstart command from the root of my repository because I wanted everything in the root directory. This makes it much easier to set up GitHub Actions for automated deployment later.

```bash
pelican-quickstart
```

### Configuration Prompts

The quickstart wizard asked me several questions. Here's how I answered them and my thinking behind each choice:

**1. Where do you want to create your new web site? [.]**
- **I answered:** `.` (I just pressed Enter)
- **My thinking:** I wanted the Pelican files created right in my current directory since I was already in the `ericbsd.github.io` repository. This keeps everything organized in the root.

**2. What will be the title of this web site?**
- **I answered:** `EricBSD`
- **My thinking:** I chose a simple title that represents my online identity. This shows up in the header and browser tabs.

**3. Who will be the author of this web site?**
- **I answered:** `Eric Turgeon`
- **My thinking:** My full name for proper attribution in posts and the site footer.

**4. What will be the default language of this web site? [en]**
- **I answered:** `en` (I pressed Enter for the default)
- **My thinking:** My blog is in English, so the default works. If you're writing in another language, use codes like `fr`, `es`, etc.

**5. Do you want to specify a URL prefix? (Y/n)**
- **I answered:** `y`
- **My thinking:** This is critical for GitHub Pages. Without it, all my links, CSS, and images would break when deployed.

**6. What is your URL prefix?**
- **I answered:** `https://ericbsd.github.io`
- **My thinking:** This is my GitHub Pages URL. Pelican needs this to generate correct absolute URLs.

**7. Do you want to enable article pagination? (Y/n)**
- **I answered:** `y`
- **My thinking:** I didn't want all my blog posts on one endless page. Pagination makes the site more manageable.

**8. How many articles per page do you want? [10]**
- **I answered:** `10` (I kept the default)
- **My thinking:** 10 posts per page feels like a good balance for my blog, enough content without overwhelming readers.

**9. What is your time zone?**
- **I answered:** `America/Moncton`
- **My thinking:** I entered my local timezone so post timestamps display correctly. Make sure to use yours (like `America/New_York`, `Europe/Paris`, etc.).

**10. Do you want to generate a tasks.py/Makefile to automate generation and publishing? (Y/n)**
- **I answered:** `y`
- **My thinking:** You absolutely need this. It creates the build files that let you run commands like `make html` and `make publish`.

**11. Do you want to upload your website using GitHub Pages? (y/N)**
- **I answered:** `y`
- **My thinking:** This configures Pelican specifically for GitHub Pages and adds the right settings to the config files.

**12. Is this your personal page (ericbsd.github.io)? (y/N)**
- **I answered:** `y`
- **My thinking:** This confirms it's a user site (not a project site). User sites deploy from the root URL, which is what I wanted.

### Understanding Generated Files

After running the quickstart, I had several new files and directories in my project. Here's what each one does and how I used them:

- **`pelicanconf.py`** - This is my main configuration file for development. I tweaked this a lot to customize the blog's behavior, add theme settings, and configure social links.

- **`publishconf.py`** - Production configuration that imports `pelicanconf.py` and overrides settings for deployment. The main difference is setting the correct `SITEURL` and enabling RSS/Atom feeds.

- **`content/`** - This is where all my blog posts go. I create Markdown files here for each article.

- **`Makefile` and `tasks.py`** - Build automation scripts. Since I'm on GhostBSD, I use `gmake` commands like `gmake html` to build and `gmake devserver` for local testing.

- **`output/`** - The generated static site. This directory is git-ignored because GitHub Actions builds it automatically during deployment. I never commit this.

## Customizing the Configuration

The quickstart already set up the basics (site name, author, timezone, etc.), but I customized `pelicanconf.py` further to add my theme, social links, and other personal touches.

### Theme Configuration

I used a custom theme instead of the default. I'll cover theme setup in the next section, but I added this to my config:

```python
THEME = 'theme'  # Path to my custom theme directory
```

### Add Copyright Information

I added copyright settings for the footer:

```python
COPYRIGHT_YEAR = 2026
COPYRIGHT_NAME = 'Eric Turgeon'
```

### Dark Mode Support

My theme supports dark mode, so I enabled browser preference detection:

```python
THEME_COLOR_AUTO_DETECT_BROWSER_PREFERENCE = True
THEME_COLOR_ENABLE_USER_OVERRIDE = True
```

### Site Logo

I added my photo to use as the site logo:

```python
STATIC_PATHS = ['images']  # Tell Pelican to copy the images directory
SITELOGO = '/images/photo_2020-03-25_14-00-43.jpg'
```

### Social Links

I added my social media profiles to appear in the site footer:

```python
SOCIAL = (
    ("x-twitter", "https://x.com/ericbsd"),
)
```

### Pages in Menu

I made sure static pages (like About) appear in the navigation menu:

```python
DISPLAY_PAGES_ON_MENU = True
```

### Production Configuration (publishconf.py)

I didn't need to edit `publishconf.py` much. The quickstart already set it up correctly. It imports all settings from `pelicanconf.py` and overrides the SITEURL for production. The main things it does:

- Sets `SITEURL = "https://ericbsd.github.io"` (instead of empty for dev)
- Enables RSS/Atom feeds for production
- Enables `DELETE_OUTPUT_DIRECTORY = True` to clean up before building

## Setting Up the Flex Theme

For the theme, I wanted something I could customize over time rather than using a pre-installed theme. Some people use `pelican-themes` to install themes system-wide, but I preferred to have the theme directly in my project where I could modify it freely.

### My Approach: Clone and Customize Flex

I chose the [Flex theme](https://github.com/alexandrevicenzi/Flex) because it's clean, modern, and supports dark mode. Instead of installing it with `pelican-themes`, I cloned it and copied it into my project:

```bash
# Clone the Flex theme repository
git clone https://github.com/alexandrevicenzi/Flex.git

# Copy the theme to my project
cp -r Flex/* theme/

# Clean up the cloned repo
rm -rf Flex/
```

Now I had the theme as `theme/` in my project root, which I could modify as needed.

### Customizations I Made

With Claude Code's help, I made several customizations to make the theme fit my preferences:

**1. Updated Social Icons**

The theme's icon set was missing the X (formerly Twitter) icon, so I had to download a newer set of icons that included it. This ensured my social links would display properly.

**2. Changed Title Colors**

I changed some title colors from the default orange to blue to better match my personal style preferences. These kinds of tweaks are easy when you have the theme files in your project.

**3. Other Modifications**

Having the theme in my repository means I can continue tweaking templates, styles, and behavior whenever I want.

In `pelicanconf.py`, I set:

```python
THEME = 'theme'
```

This tells Pelican to use my local `theme/` directory instead of a system-installed theme.

### Why This Approach?

Some people prefer using `pelican-themes` to install themes globally, which works fine if you want a theme as-is. But I knew I'd want to customize things over time, so having the theme in my project gave me full control to modify templates, styles, and behavior whenever I wanted.

## Creating Pages and Blog Posts

Pelican handles two types of content: **pages** (static content like About) and **posts** (blog articles). Here's how I set up both.

### Create a Static Page (About)

Static pages go in `content/pages/`. I created my About page as `content/pages/About.md`:

```markdown
Title: About
Date: 2026-01-10

# About the Blog

This blog is about BSD, Linux, NAS, Open Source, programming, automation, and occasionally ranting about Open Source and tech industry trends.

# About Me

My username is **ericbsd** for everything related to BSD, open source, and programming, but my real name is **Eric Turgeon**.
...
```

**Key points about pages:**
- Go in `content/pages/` directory
- Only require `Title` metadata (Date is optional)
- No `Category`, `Tags`, or `Slug` needed
- Automatically appear in navigation menu (if `DISPLAY_PAGES_ON_MENU = True`)
- Good for: About, Contact, Projects, etc.

### Create a Blog Post

Blog posts go directly in `content/`. I created my first post as `content/2026-01-09-blog-is-back.md`:

```markdown
Title: My Blog Is Back
Date: 2026-01-10
Category: Announcement
Tags: announcement
Slug: my-blog-is-back

## Why the Blog Is Back

A few years ago, I had a blog, but I was not actively writing...
```

**Key points about blog posts:**
- Go directly in `content/` directory
- Require `Title`, `Date`, `Category`, `Tags`, and `Slug`
- I name files with date prefix (YYYY-MM-DD-title.md) for organization
- Posts appear in chronological order on the homepage
- Support pagination (remember we set 10 posts per page)

### Metadata Comparison

**Pages need:**
- `Title` (required)
- `Date` (optional, I included it but you don't have to)

**Posts need:**
- `Title`
- `Date`
- `Category` (like Announcement, Tutorial, Opinion)
- `Tags` (comma-separated keywords)
- `Slug` (URL-friendly version of the title)

## Testing Changes Locally

Throughout the setup, I constantly tested my changes locally before pushing to GitHub. Since I'm on GhostBSD, I have to use `gmake` instead of `make`.

### Running the Development Server

I used the development server to preview changes:

```bash
gmake devserver
```

This starts a local server at `http://localhost:8000`. The devserver automatically rebuilds the site whenever I save changes to files, but I have to manually refresh my browser to see the updates.

My workflow was simple:
1. Start `gmake devserver` once
2. Open `http://localhost:8000` in my browser
3. Make changes and save files
4. Refresh the browser to see the updates

To stop the server, I just pressed `Ctrl+C` in the terminal.

**Tip:** Testing locally makes it much easier to catch issues before deployment. I used `gmake devserver` throughout the entire setup process.

## Setting Up Automated Deployment

GitHub Actions handles the building and deployment of my blog automatically every time I push changes. I set up a workflow file that does all the work.

### Creating the Workflow File

I created `.github/workflows/static.yml` in my repository:

```yaml
name: Deploy Pelican site to Pages

on:
  push:
    branches: ["master"]
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: "pages"
  cancel-in-progress: false

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.x'

      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install -r requirements.txt

      - name: Build with Pelican
        run: make publish

      - name: Setup Pages
        uses: actions/configure-pages@v5

      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: 'output'

  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

### How It Works

This workflow does everything automatically:

1. **Triggers on push to master** - Every time I push changes, the workflow runs
2. **Sets up Python environment** - GitHub Actions creates a clean Ubuntu environment with Python
3. **Installs dependencies** - Uses my `requirements.txt` to install Pelican and Markdown
4. **Builds the site** - Runs `make publish` (not `gmake`; GitHub Actions uses standard GNU make on Linux)
5. **Uploads to GitHub Pages** - Takes the `output/` directory and deploys it

The workflow also supports `workflow_dispatch`, which means I can manually trigger a deployment from the GitHub Actions tab if needed.

**Important:** The workflow uses `make publish` (not `make html`) because it needs to build with production settings from `publishconf.py`, which sets the correct SITEURL and enables feeds.

## Configuring GitHub Pages

After setting up the GitHub Actions workflow, I needed to configure GitHub Pages to use it.

### Enable GitHub Pages

1. Go to your repository on GitHub
2. Click **Settings → Pages**
3. Under "Build and deployment":
   - **Source**: Select **GitHub Actions**
4. Save

This tells GitHub Pages to use your workflow file instead of the old gh-pages branch method.

### Custom Domain (Optional)

I set up a custom domain (`ericbsd.com`) for my blog. If you want to use a custom domain instead of `username.github.io`:

1. In the same Pages settings, under "Custom domain"
2. Enter your domain name (e.g., `yourdomain.com`)
3. Save
4. Configure your DNS provider to point to GitHub Pages (see [GitHub's custom domain documentation](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site))
5. Check **Enforce HTTPS** once DNS is configured

**Note:** If you use a custom domain, update `SITEURL` in `publishconf.py` to match your custom domain.

### Verify .gitignore

The Python `.gitignore` template we added when creating the repository should already cover what we need, but make sure it includes:
```
output/
*.pyc
__pycache__/
*.pid
.DS_Store
*.swp
*.swo
idea
```

The `output/` directory is especially important. You never want to commit the generated site since GitHub Actions builds it automatically.

## Deploying the Site

With GitHub Actions configured, deployment happens automatically whenever you push to the master branch.

### How It Works

When you commit and push changes to master:

1. **GitHub Actions triggers** - The workflow file (`.github/workflows/static.yml`) detects the push
2. **Build job runs** - GitHub creates an Ubuntu environment, installs dependencies from `requirements.txt`, and runs `make publish`
3. **Pelican builds the site** - Using `publishconf.py` settings, it generates the static site in `output/`
4. **Upload to Pages** - The workflow uploads the `output/` directory
5. **Deployment** - GitHub Pages makes the site live

The entire process takes 30-60 seconds. You can monitor progress in the **Actions** tab of your repository. Once the workflow completes (green checkmark), your changes are live.

## Conclusion

That's how I set up this Pelican blog on GitHub Pages. Since I was already familiar with Sphinx, the Pelican setup felt natural and came together quickly with ChatGPT's help.

For writing posts, I don't have a formal workflow yet, but my approach is simple. I write my ideas in Obsidian, and since both Obsidian and Pelican use Markdown, I can copy the text to a file in `content/` and add the Pelican metadata. Then I have Claude Code review the syntax and suggest text improvements before publishing.

The beauty of this setup is that I can focus on writing without worrying about hosting, building, or deploying. Everything just works.

If you're setting up your own Pelican blog, I hope this walkthrough saves you some time.