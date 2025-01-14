Commonspeak2
---

Commonspeak2 leverages publicly available datasets from Google BigQuery to generate content discovery and subdomain wordlists.

As these datasets are updated on a regular basis, the wordlists generated via Commonspeak2 reflect the current technologies used on the web.

By using the Golang client for BigQuery, we can stream the data and process it very quickly. The future of this project will revolve around improving the quality of wordlists generated by creating automated filters and substitution functions.

Let's turn creating wordlists from a manual task, into a reproducible and reliable science with BigQuery.


I just want the wordlists...
----
We will update [wordlists.assetnote.io](https://wordlists.assetnote.io) website with any wordlists generated the Commonspeak2 tool.

Wordlists are automatically generated at the end of each month and uploaded to this site. Further details here: https://github.com/assetnote/wordlists


Instructions & Usage
----

If you're compiling or running Commonspeak2 from source:

* [Golang 1.10 or above](https://storage.googleapis.com/golang/getgo/installer_linux)
* [Glide](https://github.com/Masterminds/glide)
* [Google Cloud Service Account with access to BigQuery](https://cloud.google.com/bigquery/docs/reference/libraries#client-libraries-install-go)

If you're using the pre-built binaries:

* Download the newest release [here](https://github.com/assetnote/commonspeak2/releases)

Upon completing the above steps, Commonspeak2 can be used in the following ways:

### Subdomains

Currently subdomains are extracted from HackerNews and HTTPArchive's latest scans. Unlike the previous revision of Commonspeak, the datasets and queries have been optimised to contain valid data that occurs often in the wild. 

`⟩ ./commonspeak2 --project crunchbox-160315 --credentials credentials.json subdomains -o subdomains.txt`

```
INFO[0000] Generated SQL template for HackerNews.        Mode=Subdomains
INFO[0000] Generated SQL template for HTTPArchive.       Mode=Subdomains
INFO[0000] Executing BigQuery SQL... this could take some time.  Mode=Subdomains Source=hackernews
INFO[0019] Total rows extracted 71415.                   Mode=Subdomains Silent=false Source=hackernews Verbose=false
INFO[0019] Executing BigQuery SQL... this could take some time.  Mode=Subdomains Source=httparchive
INFO[0075] Total rows extracted 484701.                  Mode=Subdomains Silent=false Source=httparchive Verbose=false
```

### Words with extensions

Using a single query on GitHub's dataset, we can extract every path filtered by file extension. This can be done with:

`⟩ ./commonspeak2 --project crunchbox-160315 --credentials credentials.json ext-wordlist -e jsp -l 100000 -o jsp.txt`


```
INFO[0000] Executing BigQuery SQL... this could take some time.  Extensions=jsp Limit=100000 Mode=WordsWithExt Source=Github
INFO[0013] Total rows extracted 100000.                  Mode=WordsWithExt Source=Github
```

Any set of extensions can be passed via the `-e` flag, i.e. `-e aspx,php,html,js`.

### Deleted files

*Contributed by [mhmdiaa](https://twitter.com/mhmdiaa)*

Using GitHub's commits dataset, we can extract what may be files that developers decided to delete from their public repositories. These files may contain sensitive data. This can be done with:

`⟩ ./commonspeak2 --project crunchbox-160315 --credentials credentials.json deleted-files -l 50000 -o deleted.txt`


```
INFO[0000] Executing BigQuery SQL... this could take some time.  Limit=50000 Mode=DeletedFiles Source=Github
INFO[0013] Total rows extracted 50000.                  Mode=DeletedFiles Source=Github
```


### Features in Active Development

Feel free to send pull requests to complete the features below, add datasets or improve the architecture of this project. Thank you!

**Routes Based Extraction**

We can create SQL statements that cover routing patterns in almost any web framework. For now we support the following web frameworks to extract path's from:

- Rails  [working implementation ✅]
- NodeJS [to be implemented ❎]
- Tomcat [to be implemented ❎]

This data can be extracted using the following command:

`⟩ ./commonspeak2 --project crunchbox-160315 --credentials credentials.json routes --frameworks rails -l 100000 -o rails-routes.txt`

WARNING: running the above query will cost you **lots** of money (over $20 per framework). Commonspeak2 will prompt to confirm that this is OK. To skip this prompt use the `--silent` flag.

When this is ran on for Rails routes, Commonspeak2 does the following:

1) Pulls Rails routes from `config/routes.rb` using Regex and the latest Github dataset.
2) Processes the data, converts it into paths and does contexual replacements to make the path valid (i.e. converting `/:id` to `/1234`)
3) Normalizes the path, finally saving to disk after all the processing is complete.

**Scheduled Wordlist Generation**

Planned feature to use a cron-like system to allow for wordlist generation from BigQuery to happen continuously.

When this command is introduced, we will insert the `--schedule` parameter to any of our pre-existing commands covered in this README like so:

`⟩ ./commonspeak2 --project crunchbox-160315 --credentials credentials.json --schedule weekly routes --frameworks nodejs,tomcat -l 100000 -o nodejs-tomcat-routes.txt`

The above query will run a weekly BigQuery and save the output to `./nodejs-tomcat-routes.txt`.

**Substitutions and Alterations**

Generate smart substitutions and alterations for the datasets that it makes sense for. For example, converting string values from `/admin/users/:id` to `/admin/users/1234` (contextually aware of the number).

## Building commonspeak2 package

The module uses glide for package manager. Note that more recent versions of golang use go mod so you will have to take this into account.

- Clone this repository in the following local folder: ```$GOPATH/src/github.com/assetnote/commonspeak2```
- When making any changes to the data/ folder, regenerate the files using https://github.com/go-bindata/go-bindata by running ```go-bindata -o assets/assets.go -pkg assets data/... ```
- Update dependencies using ```glide up```. Make sure you run this after running go-bindata
- run ```go build``` and commonspeak2 executable should be created

Credits
----

Shubham Shah [@infosec_au](https://twitter.com/infosec_au)

Michael Gianarakis [@mgianarakis](https://twitter.com/mgianarakis)

License
----

```
   Copyright 2018 Assetnote

   Licensed under the Apache License, Version 2.0 (the "License");
   you may not use this file except in compliance with the License.
   You may obtain a copy of the License at

       http://www.apache.org/licenses/LICENSE-2.0

   Unless required by applicable law or agreed to in writing, software
   distributed under the License is distributed on an "AS IS" BASIS,
   WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
   See the License for the specific language governing permissions and
   limitations under the License.
```

[Assetnote Pty. Ltd.](https://assetnote.io/) - Twitter [@assetnote](https://twitter.com/assetnote)
