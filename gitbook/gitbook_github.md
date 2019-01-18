# How to upload the gitbook to the github?

## Principle: 
* Save the source code on the branch `master`, 
* Saved the compiled `HTML` file on the branch `gh-pages`

## Steps:
1. Create a new repository
   -   Go to github.com and create a new repository
   -   Remember not to tick "initialize this repository with a README"

2. Go to the command line, enter the folder concerned, then:
   -   `git init`
   -   `git remote add origin https://github.com/WongYatChun/[xxxxxxxxxxx].git`
   -   `git add .`
   -   `git commit -m "sth"`
   -   `git push -u origin master`

3. `Master` branch saves the source code, `gh-pages` saves the compiled `HTML` file
   -    save a copy of `_book` to somewhere else

4. Go back to the command line, enter the folder concered
   -   `git checkout -b "gh-pages"`
   -   Delete all the files (except .git)
   -   copy the contents in the folder `_book` to the folder concerned 
   -   `git add .`
   -   `git commit -m "initialize gh-pages"`
   -   `git push origin gh-pages`

5.  Go to the github.com, switch to the branch `gh-pages`
    -   find `Settings` on the tab
    -   Scroll down the section `GitHub Pages`
    -   "Your site is published at https://wongyatchun.github.io/[xxxxxxx]"

6. Switch back to branch `master`
   -    `git checkout master`