---
layout: post
title:  "Curl"
date:   2011-03-18 20:09:30 +0100
---

# curl

Curl has quite a list of options. Following is the list of oftem used options, not a long list, so easy to memorize:

```
-d, --data <data>   HTTP POST data
    --data-raw <data> HTTP POST data, '@' allowed
-H, --header <header/@file> Pass custom header(s) to server
-i, --include       Include protocol response headers in the output
-k, --insecure      Allow insecure server connections when using SSL
-L, --location      Follow redirects
-o, --output <file> Write to file instead of stdout
-x, --proxy [protocol://]host[:port] Use this proxy
-O, --remote-name   Write output to a file named as the remote file
-X, --request <command> Specify request command to use
-s, --silent        Silent mode
-T, --upload-file <file> Transfer local FILE to destination
-u, --user <user:password> Server user and password
-v, --verbose       Make the operation more talkative
```

## Downloading file

```bash
curl http://localhost > output.txt

curl -LO http://localhost

curl -Lo index.htm http://localhost

curl -L http://localhost/file.tar.gz | tar -xz
```

## Posting json

```bash
curl -d @filename -H 'Content-Type: application/json' -X POST http://localhost

curl -d '{ "key": "value" }' -H 'Content-Type: application/json' -X POST http://localhost
```

## Uploading file

```bash
curl -T file.tar.gz http://localhost
```