# Q notes

My gitbook for documenting q related experience and knowledge.

## How to add chapters

Add your markdown posts to the `_posts` folder.


## Deploy Locally with Jekyll Serve

```cmd
jekyll serve
```

## How to generate TOC

The jekyll-gitbook theme leverages [jekyll-toc][4] to generate the *Contents* for the page.
The TOC feature is not enabled by default. To use the TOC feature, modify the TOC
configuration in `_config.yml`:
```yaml
toc:
    enabled: true
```

OR

add the following script in MarkDown file which need TOC:

```yaml
toc:
    enabled: true
```
