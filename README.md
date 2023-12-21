Built in Pelican.


Local development preview:

```bash
$ poetry run pelican -r -l
```

Deployment:

```bash
$ poetry run pelican content -s publishconf.py
$ poetry run ghp-import output -b gh-pages
$ git push origin gh-pages
```
