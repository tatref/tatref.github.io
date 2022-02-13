# Deployment / update


zola serve --drafts

zola serve

rm -rf docs
zola build --output-dir docs

git add docs


