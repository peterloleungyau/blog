This repository hosts the source files of my blog at https://peterloleungyau.github.io/

The site is powered by Hugo, with ox-hugo to export the org files to md files.

## Setup after cloning
After cloning, also need to fix the them submodules by
```
git submodule init
git submodule update
```

## Preview posts
To start server with drafts enabled:
```
hugo server -D
```

Then visit http://localhost:1313/ to view the posts.

## To publish
After checking the drafts, can build the final pages with:
```
hugo -D
```

Then publish with:
```
cd public
git add .
git commit -m "updated"
git push
```
