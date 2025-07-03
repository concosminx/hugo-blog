### Commands 

1. create the site `hugo new site hugo-blog --fomat yaml`
2. add a new doc `hugo new docs/test.md`
3. init the git repo `git init`
4. install a theme: 
   - see the [docs](https://github.com/adityatelange/hugo-PaperMod/wiki/Installation)
   - install the theme `git clone https://github.com/adityatelange/hugo-PaperMod themes/PaperMod --depth=1`
   - update the theme `cd themes/PaperMod` and `git pull`
   - add the theme as a submodule `git submodule add --depth=1 https://github.com/adityatelange/hugo-PaperMod.git themes/PaperMod`
   - use this when reclone `git submodule update --init --recursive`
   - modify `hugo.yaml` and set the theme `theme: PaperMod`
5. check the site with `hugo server` 
6. upload to git `add`, `commit`, `git branch -M main`, `git remote add origin ...`, `git push -u origin main`
7. create `gh-pages` branch in github
8. set the `rw` permissions for the workflow `Settings -> Actions`
9. create the deploy workflow
10. edit the base URL in `hugo.yaml`

### Add new stuff
- create a post `hugo new posts/first.md`
- check the theme [options](https://adityatelange.github.io/hugo-PaperMod/)
