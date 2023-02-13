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



Image sequence to gif

```
ffmpeg.exe -framerate 1 -i "%d.png" -filter_complex "[0:v]fps=1,split[a][b];[b]palettegen[p];[a][p]paletteuse,setpts=1*PTS[v]" -map '[v]' out.gif
```