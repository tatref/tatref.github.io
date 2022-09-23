# Local testing


With drafts
```
zola serve --drafts
```

Without
```
zola serve
```

# Deploy
```
rm -rf docs
zola build --output-dir docs
```

```
git add docs
git commit ...
git push ...
```


