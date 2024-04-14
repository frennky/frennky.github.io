---
title:  "Disabling Google Ads"
date:   2012-12-15 22:41:30 +0100
---

Some time ago I was doing some Selenium Webdriver tests for a website with Google Ads. For QA environment the ads were disabled through some website configuration, but for Staging environment everything was set up as on Live, and so the ads were enabled.

However, this caused some problems when running Webdriver tests. What happened is that on some pages the Webdriver would timeout because the requests made by Google Ads scripts were taking too long.

I needed a solution to disable the Google Ads without changing the configuration of website or environment. It turned out simple, just added these two lines to hosts file on a machine where Webdriver was running and the problem was gone.

```
127.0.0.0 pagead2.googlesyndication.com
127.0.0.0 googleads.g.doubleclick.net
```

If you're running tests locally, add this to your machine, and if you're using a Selenium Grid (Selenium Server), than add this on each node that will be used.

You can guess what this will do. The Google ads scripts will make a request to a local host instead to Google servers, and that will cause the scripts to finish whatever they are doing, instead of making Webdriver timeout.

Note that this isn't a change in website configuration or environment, these changes were made on test agent machines (client machines).

Have you had this problem? How did you work around it?
