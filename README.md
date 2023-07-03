# [2mas.github.io](https://2mas.github.io)
Development blog using GitHub Pages and Jekyll based on Minimal Mistakes

---
To run

Run `docker compose up` or

```bash
docker run -it -v="${PWD}:/srv/jekyll" -p 4000:4000 -p 35729:35729 jekyll/jekyll bash
bundle install # or bundle update
bundle exec jekyll serve --livereload --host 0.0.0.0 # Add --force_polling if necessary
```

Visit at http://localhost:4000
