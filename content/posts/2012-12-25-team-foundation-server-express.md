---
title:  "Team Foundation Server Express"
date:   2012-12-25 22:10:30 +0100
---

Of all the new developer tools and products that Microsoft launched this fall, Team Foundation Server Express got most of my attention. The fact that it's free and that it comes with reasonable limitations compared to full version, shows that Microsoft is really dedicated to making it's platform stronger.

This blog post won't cover all the features that are included or not, I'll just focus on those that are most interesting.

## What's included in the box :)

- Source control - same as in full version with history, notifications, security and other features.
- Work item tracking - same as in full version with few project templates and ability to add new templates, customize work items etc.
- Build automation - no limitations here, you can have multiple build agents that are on multiple machines if that's what you need.

## What's not included in the box :(

- Only supports SQL Express - sad thing about this is that even if you have a full SQL Server you can't use it for TFS Express, it will install SQL Express and use that instance.
- SharePoint and Reporting integration - not sure what to say about this since I wasn't a big fan of it and never had much fun with it (too complicated setup).
- Single Server configuration - no proxies.

If you're that big that you need this feature than you should have enough funds to purchase a full licence.
Great thing about Visual Studio 2012 Express editions is that they now support version control, which means that you can use those tools with TFS Express. Also, if you have earlier versions of Visual Studio, TFS Express will work with them as well and with Team Explorer Everywhere you can use TFS Express with Eclipse.

To make things clear Team Foundation Server Express is free, but it supports up to 5 users. Now this may seem as a deal breaker, but if you think about it and mind what's in the box, I think this is quite reasonable.

This version is designed for small teams, but if you need a few more users you can always get them by purchasing a CAL.

Finally, if you ever decide to upgrade to full TFS, the process is simple and you won't lose any data including source control history.
