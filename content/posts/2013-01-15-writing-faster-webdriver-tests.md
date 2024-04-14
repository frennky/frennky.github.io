---
title:  "Writing faster WebDriver tests"
date:   2013-01-25 22:41:30 +0100
---

Some day ago I met with a friend and had nice chat about testing. Eventually we came to a topic about Selenium. He's been using Selenium mostly for simple test cases, but now he was thinking of using it to test some reporting functionality.

His concern was if Selenium is actually the right tool to help him with that. At that point I said that Selenium is the best free tool for functional testing of web apps that we have now, but it's another thing how good it actually is. It all depends how you use it and for what.

The test case was fairly simple. Login, navigate to reporting, select a report type, input some parameters, generate report and than assert that data in reporting table is correct. The problem was that on average reports have a table with 50 rows and 10 columns, which means that he has 500 cells with data that needs to be verified.

Now before we continue, lets refresh our memory how Selenium actually works. The Selenium server exposes a REST API to client WebDriver libraries. Each command we execute with client library is translated to a request to Selenium server.

When we want to get certain text value from web element, first we execute a command that will select that web element and than we execute a command that will get the text value. That's two request being made to Selenium server. The select request doesn't actually return the web element, it only returns an id that Selenium server generated for that web element, so another request is being made to retrieve text value.

Returning to our problem, that means on average we'll make 500 x 2 = 1000 requests just to select each cell and get it's text value. This of course will make the test very slow.

But how can we make this test run faster? When my friend described the test case, the last step he mentioned was to verify if the report has correct data. He didn't say let's assert each cell, although that's what we must do at the end.

But what if we could just make one assert in a test, one that would require us to make just one request to Selenium server? Well, we can actually do that. Just send a JavaScript that will do all data checking and return whether the report has correct data or not. It's simple as that, and if your web app uses jQuery, it's even easier.

```csharp
var script = "some javascript";
var js = driver as IJavaScriptExecutor;
var isDataCorrect = js.ExecuteScript(script);
Assert.True(isDataCorrect);
```

Have an idea how to make things run faster, post a comment and we can discuss it.
