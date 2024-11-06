# Kow.fm V 25

The 25th version of the greatest blog in the world.

This is... kinda based on a jekyll theme. I stripped all the styles and almost all the markup. So it started pretty barebones.

## Goals

Make a blog that won't get deleted if I don't pay a hosting bill. Have something that's very easy to update. Be extremely minimal with markup and design. My hope is that by removing so much that is *"custom"* or *"fancy"*, I can get to the core of what a blog experience requires.

## Run it

```bash
jekyll serve
```

### Customize navigation links

I don't actually now how this part works. I'll need to figure out maybe.

This allows you to set which pages you want to appear in the navigation area and configure order of the links.

For instance, to only link to the `about` and the `portfolio` page, add the following to you `_config.yml`:

```yaml
header_pages:
  - about.md
  - portfolio.md
```

## License

The theme is available as open source under the terms of the [MIT License](http://opensource.org/licenses/MIT).
