# nsf_toss_external

# Getting the dataset

- dataset of repositories
    - located in: `./data/oss/original/code_gov/agencies/agency_project_button_url.csv`
    - 2 column dataset that has the url to the code.gove page and the url of the repo button
    - Most of the URLs point to github, but not all
    - Some/most (unverified how many) of the github URLs will return a 404 error
        - Either the URL has a typo in it
        - The agency removed the repository from github
        - The agency made the repository private
        - Probably other reasons too

# Parse the github URL

You might want/need to parse the github URLs to be used in your code.
I'll be using the `"https://github.com/USDA/RIDB"` as an example

1. Make sure you are looking at a github repository
```r
url <- "https://github.com/USDA/RIDB"

# is it a github url?
is_github <- stringr::str_detect(url, '.*github.*')
```
1. Make sure the github URL is valid (does not return 404)
    - You can check [this SO solution](https://stackoverflow.com/questions/23139357/how-to-determine-if-a-url-object-in-r-base-package-returns-404-not-found) to get a `TRUE`/`FALSE` 404 value
2. Parse the User/agency
```r
url_split <- stringr::str_split(url, '/')[[1]]
org <- url_split[length(url_split) - 1]
```
3. Parse the repository name
```r
repo <- url_split[length(url_split)]
```
4. You might also want the org/repo name that is sometimes used
```r
org_repo <- sprintf('%s/%s', org, repo)
```
