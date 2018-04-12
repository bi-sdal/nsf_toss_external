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

## The root endpoint

This is a REST API, so all querries will be in the form of a web URL.
The querries all begin with `https://api.github.com`, this is the "root endpoint".
All other querries are added as a suffix to this root endpoint

```bash
curl https://api.github.com
{
  "current_user_url": "https://api.github.com/user",
  "current_user_authorizations_html_url": "https://github.com/settings/connections/applications{/client_id}",
  "authorizations_url": "https://api.github.com/authorizations",
  "code_search_url": "https://api.github.com/search/code?q={query}{&page,per_page,sort,order}",
  "commit_search_url": "https://api.github.com/search/commits?q={query}{&page,per_page,sort,order}",
  "emails_url": "https://api.github.com/user/emails",
  "emojis_url": "https://api.github.com/emojis",
  "events_url": "https://api.github.com/events",
  "feeds_url": "https://api.github.com/feeds",
  "followers_url": "https://api.github.com/user/followers",
  "following_url": "https://api.github.com/user/following{/target}",
  "gists_url": "https://api.github.com/gists{/gist_id}",
  "hub_url": "https://api.github.com/hub",
  "issue_search_url": "https://api.github.com/search/issues?q={query}{&page,per_page,sort,order}",
  "issues_url": "https://api.github.com/issues",
  "keys_url": "https://api.github.com/user/keys",
  "notifications_url": "https://api.github.com/notifications",
  "organization_repositories_url": "https://api.github.com/orgs/{org}/repos{?type,page,per_page,sort}",
  "organization_url": "https://api.github.com/orgs/{org}",
  "public_gists_url": "https://api.github.com/gists/public",
  "rate_limit_url": "https://api.github.com/rate_limit",
  "repository_url": "https://api.github.com/repos/{owner}/{repo}",
  "repository_search_url": "https://api.github.com/search/repositories?q={query}{&page,per_page,sort,order}",
  "current_user_repositories_url": "https://api.github.com/user/repos{?type,page,per_page,sort}",
  "starred_url": "https://api.github.com/user/starred{/owner}{/repo}",
  "starred_gists_url": "https://api.github.com/gists/starred",
  "team_url": "https://api.github.com/teams",
  "user_url": "https://api.github.com/users/{user}",
  "user_organizations_url": "https://api.github.com/user/orgs",
  "user_repositories_url": "https://api.github.com/users/{user}/repos{?type,page,per_page,sort}",
  "user_search_url": "https://api.github.com/search/users?q={query}{&page,per_page,sort,order}"
}
```

## Availiable Endpoints

You can get all availiable endpoints under the [GitHub Apps > Avaliable Endpoints section of the documentation](https://developer.github.com/v3/apps/available-endpoints/).

### Statistics

One of the more interesting bits would be on the [Statistics](https://developer.github.com/v3/repos/statistics/) section.

#### Get contributors list with additions, deletions, and commit counts

https://developer.github.com/v3/repos/statistics/#get-contributors-list-with-additions-deletions-and-commit-counts

The documentation says you can run 

```
GET /repos/:owner/:repo/stats/contributors
```

To replicate the call for our example github repository we can run

```bash
curl https://api.github.com/repos/USDA/RIDB/stats/contributors
```

Here's a partial output of the results

```
[
  {
    "total": 1,
    "weeks": [
      {
        "w": 1422144000,
        "a": 0,
        "d": 0,
        "c": 0
      },
      {
        "w": 1422748800,
        "a": 0,
        "d": 0,
        "c": 0
      },
      {
        "w": 1423353600,
        "a": 0,
        "d": 0,
        "c": 0
      },
```

# Getting the Github API working in R

As you can see the benefit of haveing a REST api is that all you need is to construct the correct URL string,
and the API will return a JSON reponse back to you.

## Using `httr`

If you construct the full REST API call as a string, you can pass it into the `httr::GET` function,
and the results will be returned back to you in R.

```r
> results <- httr::GET('https://api.github.com/repos/USDA/RIDB/stats/contributors')
> results
Response [https://api.github.com/repos/USDA/RIDB/stats/contributors]
  Date: 2018-04-12 17:20
  Status: 200
  Content-Type: application/json; charset=utf-8
  Size: 64.2 kB
[
  {
    "total": 1,
    "weeks": [
      {
        "w": 1422144000,
        "a": 0,
        "d": 0,
        "c": 0
      },
...
```

Now when you `View(results)`, rstudio has a nice way for you to interact with this kind of hierarchical data.

### How many querries you have left

Now that you have the results in R, you can traverse it like a list.
Here's how you can look at how many querroes you have left

```r
> results$header$`x-ratelimit-remaining`
[1] "54"
```

### Result content

Unless you speak unicode, the raw content results will make no sense
```r
head(results$content)
1] 5b 0a 20 20 7b 0a
```

Since we know the github API returns things in JSON, we can convert the unicode to text to JSON

Take a look at this JSON and jsonlite vignette: https://cran.r-project.org/web/packages/jsonlite/vignettes/json-aaquickstart.html

```r
json_results <- jsonlite::fromJSON(base::rawToChar(results$content))
```

In this example, the JSON returns a list for each week of the repository.
Each week is stored as a dataframe with the POXIX-time, num additions, deletions, and commits

```r
> head(json_results$weeks[[1]])
           w a d c
1 1422144000 0 0 0
2 1422748800 0 0 0
3 1423353600 0 0 0
4 1423958400 0 0 0
5 1424563200 0 0 0
6 1425168000 0 0 0
```

## Using `gh`
