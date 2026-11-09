# tm1-v12-book
An open textbook on IBM Planning Analytics / TM1 v12: concepts, setup, modelling, TurboIntegrator, rules and the REST API. Unofficial; not affiliated with IBM.

Text licensed under CC BY 4.0 (see [LICENSE](LICENSE)). Code samples licensed under MIT (see [LICENSE-CODE](LICENSE-CODE)).

## Building the book locally

The book is built with [MkDocs](https://www.mkdocs.org/) and the [Material](https://squidfunk.github.io/mkdocs-material/) theme.

```sh
pip install -r requirements.txt
mkdocs serve
```

Then open http://127.0.0.1:8000. Chapters live in [`docs/`](docs/) and the navigation is set in [`mkdocs.yml`](mkdocs.yml).

Every push to `main` publishes the site to GitHub Pages through [`.github/workflows/deploy-docs.yml`](.github/workflows/deploy-docs.yml).
