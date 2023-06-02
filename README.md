# [2mas.github.io](https://2mas.github.io)
Development blog using GitHub Pages and Jekyll based on Minimal Mistakes

---
To run (powershell ${PWD})
```
docker run -it -v="${PWD}:/srv/jekyll" -p 4000:4000 jekyll/jekyll bash
# bundle update
# bundle exec jekyll serve --livereload --host 0.0.0.0 --force_polling
```
Visit at http://localhost:4000
