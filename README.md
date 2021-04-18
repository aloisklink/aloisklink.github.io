# aloisklink.github.io

Uses the [minimal-mistakes](https://mmistakes.github.io/minimal-mistakes/)
template.

## Local testing

Ruby bundler is used to install dependencies, and on Ubuntu, it can be installed
with:

```bash
sudo apt install ruby-dev ruby-bundler
```

Then, first set the install path to a local folder, so we don't need to run with `sudo`:

```bash
bundle config set path 'vendor/bundle'
```

Finally, you can install the Ruby dependencies with:

```bash
bundle install
```

And then serve the bundle at http://localhost:4000 by running:

```bash
bundle exec jekyll serve
```
