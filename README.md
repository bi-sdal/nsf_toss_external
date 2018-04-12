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

# Exploring the Github v3 API

The main REST API v3 documentation is here: https://developer.github.com/v3/

We can run the `curl` command to querry the API

```bash
curl -i https://api.github.com/users/octocat/orgs
```

The results may look like this:
```bash
HTTP/1.1 200 OK
Server: GitHub.com
Date: Thu, 12 Apr 2018 15:57:43 GMT
Content-Type: application/json; charset=utf-8
Content-Length: 5
Status: 200 OK
X-RateLimit-Limit: 60
X-RateLimit-Remaining: 59
X-RateLimit-Reset: 1523552263
Cache-Control: public, max-age=60, s-maxage=60
Vary: Accept
ETag: "98f0c1b396a4e5d54f4d5fe561d54b44"
X-GitHub-Media-Type: github.v3; format=json
Access-Control-Expose-Headers: ETag, Link, Retry-After, X-GitHub-OTP, X-RateLimit-Limit, X-RateLimit-Remaining, X-RateLimit-Reset, X-OAuth-Scopes, X-Accepted-OAuth-Scopes, X-Poll-Interval
Access-Control-Allow-Origin: *
Strict-Transport-Security: max-age=31536000; includeSubdomains; preload
X-Frame-Options: deny
X-Content-Type-Options: nosniff
X-XSS-Protection: 1; mode=block
Referrer-Policy: origin-when-cross-origin, strict-origin-when-cross-origin
Content-Security-Policy: default-src 'none'
X-Runtime-rack: 0.012705
X-GitHub-Request-Id: 8000:5D3D:AE335A:15477B4:5ACF81F7

[

]
```
The things to watch out for are the `X-RateLimit-Limit` and `X-RateLimit-Remaining` values.
If you do not have an API key there is a [rate limit](https://developer.github.com/v3/#rate-limiting) of 60 querries per hour.
This is a good way to not use up your actual API key rates if you want to play around with what comes out of the REST API call.
