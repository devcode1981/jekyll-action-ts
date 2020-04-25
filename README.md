# jekyll-action-ts
A GitHub Action to build and publish Jekyll sites to GitHub Pages

Out-of-the-box Jekyll with GitHub Pages allows you to leverage a limited, white-listed, set of gems. Complex sites requiring custom ones or non white-listed ones (AsciiDoc for intstance) used to require a continuous integration build in order to pre-process the site.

## About this version
Originated from https://github.com/helaili/jekyll-action, however this has been converted from a Docker action to a typescript/js action to cut down on the Docker initialisation time, as well as to use https://github.com/ruby/setup-ruby to automatically select bundler version. 

## Official jekyll tutorial
If you prefer to follow the official [jekyll docs](https://jekyllrb.com/docs/continuous-integration/github-actions/) (or [here](https://deploy-preview-8119--jekyllrb.netlify.app/docs/continuous-integration/github-actions/) if that link does not work yet), just replace "helaili/jekyll-action@2.0" with "limjh16/jekyll-action-ts@v1". Also need to add a "setup-ruby" action before this. For an example, scroll down to see the [sample workflow file](#Create-a-Gemfile).

(optional) Can also add caching to decrease build time, scroll down to the [sample workflow file](#Create-a-Gemfile) for an example.

## Usage

### Create a Jekyll site
If you repo doesn't already have one, create a new Jekyll site:  `jekyll new sample-site`. See [the Jekyll website](https://jekyllrb.com/) for more information. In this repo, we have created a site within a `sample_site` folder within the repository because the repository's main goal is not to be a website. If it was the case, we would have created the site at the root of the repository.

### Create a `Gemfile`
As you are using this action to leverage specific Gems, well, you need to declare them! In the sample below we are using [the Jekyll AsciiDoc plugin](https://github.com/asciidoctor/jekyll-asciidoc)

```Ruby
source 'https://rubygems.org'

gem 'jekyll', '~> 3.8.5'
gem 'coderay', '~> 1.1.0'

group :jekyll_plugins do
  gem 'jekyll-asciidoc', '~> 2.1.1'
end

```

### Configure your Jekyll site
Edit the configuration file of your Jekyll site (`_config.yml`) to leverage these plugins. In our sample, we want to leverage AsciiDoc so we added the following section:

```yaml
asciidoc: {}
asciidoctor:
  base_dir: :docdir
  safe: unsafe
  attributes:
    - idseparator=_
    - source-highlighter=coderay
    - icons=font
```

Note that we also renamed `index.html` to `index.adoc` and modified this file accordingly in order to leverage AsciiDoc.

### Use the action
Use the `limjh16/jekyll-action-ts@v1` and `ruby/setup-ruby@v1` action in your workflow file. It needs access to a `JEKYLL_PAT` secret set with a Personal Access Token. The directory where the Jekyll site lives will be detected (based on the location of `_config.yml`) but you can also explicitly set this directory by setting the `jekyll_src` parameter (`sample_site` for us). The `SRC` environment variable is also supported for backward compatibilty but it is deprecated.

Use the `actions/cache` action in the workflow as well, to shorten build times and decrease load on GitHub's servers

```yaml
name: Testing the GitHub Pages publication

on:
  push
    
jobs:
  jekyll:
    runs-on: ubuntu-16.04
    steps:
    - uses: actions/checkout@v2

    # Use GitHub Actions' cache to shorten build times and decrease load on servers
    - uses: actions/cache@v1
      with:
        path: vendor/bundle
        key: ${{ runner.os }}-gems-${{ hashFiles('**/Gemfile.lock') }}
        restore-keys: |
          ${{ runner.os }}-gems-

    - uses: ruby/setup-ruby@v1
      with:
        ruby-version: 2.7

    - uses:  limjh16/jekyll-action-ts@v1
      env:
        JEKYLL_PAT: ${{ secrets.JEKYLL_PAT }}
	# Only add below to specify the Jekyll source location as a parameter
	# By default, the directory will be detected automatically so this is not needed
      with:
        jekyll_src: 'sample_site'
```

Upon successful execution, the GitHub Pages publishing will happen automatically and will be listed on the *_environment_* tab of your repository. 

![image](https://user-images.githubusercontent.com/2787414/51083469-31e29700-171b-11e9-8f10-8c02dd485f83.png)

Just click on the *_View deployment_* button of the `github-pages` environment to navigate to your GitHub Pages site.

![image](https://user-images.githubusercontent.com/2787414/51083411-188d1b00-171a-11e9-9a25-f8b06f33053e.png)

### Known Limitation
Publishing of the GitHub pages can fail when using the `GITHUB_TOKEN` secret as the value of the `JEKYLL_PAT` env variable, as opposed to a Personnal Access Token set as a secret. But it might work too :smile: 
