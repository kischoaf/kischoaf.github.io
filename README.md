# kieranschoaf.com

Source for [kieranschoaf.com](https://kieranschoaf.com), the portfolio of Kieran Schoaf, a network engineer focused on Cisco infrastructure and Python automation.

Built with [Jekyll](https://jekyllrb.com) and deployed to GitHub Pages with GitHub Actions.

## Local development

Requires the Ruby version in `.ruby-version` and Bundler.

```bash
bundle install
bundle exec jekyll serve
```

The site runs at http://localhost:4000.

## Where things live

| Path | Contents |
| --- | --- |
| `_config.yml` | Site metadata, bio, navigation |
| `_data/` | Experience, education, projects, skills, homelab |
| `projects/` | Project case study pages |
| `_posts/` | Field Notes posts |
| `_includes/`, `_layouts/` | Page sections and templates |
| `_sass/` | Styles |

## Deployment

Every push to `main` builds the site, checks it for broken internal links and images, and publishes it to GitHub Pages. Pull requests run the same build and checks without publishing.
