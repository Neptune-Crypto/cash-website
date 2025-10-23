# The neptune.cash website

This site is made with [Zola][zola], a static site generator.

**Note:** as of October 2025, [neptune.cash](https://neptune.cash/) lives as a typscript repository served by [bun](https://bun.com/).

[zola]: https://www.getzola.org/

## Overview

The file [`config.toml`](./config.toml) contains basic settings.

To create or modify dynamic content (blog posts, tutorials, articles), go to
the relevant sub-directories in the [`content/`](./content) directory.  If you
want to include static content, like images, PDFs, or other downloads, they go
into the [`static/`](./static) directory. You can then link to those files by
assuming they're in the web root.

If you want to change the front page, or a similar layout-heavy page, or if you
want to change the way that dynamic content is rendered, or add another category
of dynamic content: that lives in the [`templates/`](./templates) directory.

Styling lives in the [`sass/`](./sass) directory.
 
## Running locally

Install the [Zola command-line tool][zola-install] (`snap install --edge zola`
on Ubuntu.)

To statically generate the site, type `zola build`:

```
$ zola build
Building site...
Checking all internal links with anchors.
> Successfully checked 0 internal link(s) with anchors.
-> Creating 2 pages (0 orphan) and 1 sections
Done in 197ms.
```

This places a statically generated website in `public/`.

To automatically rebuild the site when files change, type `zola serve`:

```
$ zola serve
Building site...
Checking all internal links with anchors.
> Successfully checked 0 internal link(s) with anchors.
-> Creating 2 pages (0 orphan) and 1 sections
Done in 161ms.

Listening for changes in /home/sshine/Projects/neptune/neptune.cash-website{config.toml, content, sass, static, templates}
Press Ctrl+C to stop

Web server is available at http://127.0.0.1:1111
```

[zola-install]: https://www.getzola.org/documentation/getting-started/installation/

## Deploying to production

Pushing to `master` will trigger deployment on https://neptune.cash/

Pushing to `preview` will trigger deployment on https://preview.neptune.cash/
