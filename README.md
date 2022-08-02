# A Trick A Day

A blog to list a tons of code tricks.

[Using this jekyll theme](https://github.com/zerostaticthemes/jekyll-atlantic-theme)

## Installation

### Installing Ruby & Jekyll

If this is your first time using Jekyll, please follow the [Jekyll docs](https://jekyllrb.com/docs/installation/) and make sure your local environment (including Ruby) is setup correctly.

### Installing Theme

Download or clone the theme.

To run the theme locally, navigate to the theme directory and run:

To use Tailwind and PostCSS:

```bash
npm install
```

```bash
bundle install
bundle add webrick
```

To start the Jekyll local development server.

```bash
bundle exec jekyll serve
```

To build the theme.

```bash
bundle exec jekyll build
```

To build the search algolia data.

```bash
ALGOLIA_API_KEY='your_admin_api_key' bundle exec jekyll algolia
```
