---
title:  "FeedResult - Kreiranje custom ActionResult-a"
date:   2011-07-13 23:49:30 +0100
---

U ovom postu biće prikazano kako može da se napravi custom ActionResult koji vraća feed kao odgovor na zahteva browser-a.

## ActionResult

Svaka akcija kontrolera vraća objekat koji proizilazi iz apstraktne [ActionResult](http://msdn.microsoft.com/en-us/library/system.web.mvc.actionresult.aspx) klase. ASP.NET MVC sadrži nekoliko klasa koje nasleđuju ActionResult:

- ContentResult
- EmptyResult
- FileResult (apstraktna)
  - FileContentResult
  - FilePathResult
  - FileStreamResult
- HttpUnauthorizedResult
- JavaScriptResult
- JsonResult
- RedirectResult
- RedirectToRouteResult
- ViewResultBase (apstraktna)
  - PartialViewResult
  - ViewResult

ActionResult klasa sadrži samo jednu metodu, tako da ako pravimo custom ActionResult potrebno je da preklopimo samo ExecuteResult metodu. Iako, u našem primeru možemo direktno da nasledimo ActionResult klasu, mi ćemo naslediti [FileResult](http://msdn.microsoft.com/en-us/library/system.web.mvc.fileresult.aspx).

S obzirom da je MVC open source projekat, evo i [source](http://aspnet.codeplex.com/SourceControl/BrowseLatest)-a ove dve klase:

```csharp
public abstract class ActionResult {
    public abstract void ExecuteResult(ControllerContext context);
}
 
 public abstract class FileResult : ActionResult {
 
    protected FileResult(string contentType) {
        if (String.IsNullOrEmpty(contentType)) {
            throw new ArgumentException(MvcResources.Common_NullOrEmpty, "contentType");
        }
 
        ContentType = contentType;
    }
 
    private string _fileDownloadName;
 
    public string ContentType {
        get;
        private set;
    }
 
    public string FileDownloadName {
        get {
            return _fileDownloadName ?? String.Empty;
        }
        set {
            _fileDownloadName = value;
        }
    }
 
    public override void ExecuteResult(ControllerContext context) {
        if (context == null) {
            throw new ArgumentNullException("context");
        }
 
        HttpResponseBase response = context.HttpContext.Response;
        response.ContentType = ContentType;
 
        if (!String.IsNullOrEmpty(FileDownloadName)) {
            // From RFC 2183, Sec. 2.3:
            // The sender may want to suggest a filename to be used if the entity is
            // detached and stored in a separate file. If the receiving MUA writes
            // the entity to a file, the suggested filename should be used as a
            // basis for the actual filename, where possible.
            ContentDisposition disposition = new ContentDisposition() { FileName = FileDownloadName };
            string headerValue = disposition.ToString();
            context.HttpContext.Response.AddHeader("Content-Disposition", headerValue);
        }
 
        WriteFile(response);
    }
 
    protected abstract void WriteFile(HttpResponseBase response);
 
}
```

Kako bi browser shvatio da se radi o feed-u, potrebno je da u response-u dodamo header Content-Disposition sa vrednošću `application/rss+xml`. Ovo je ujedno i razlog zašto ćemo naslediti FileResult umesto ActionResult.

## Atom i RSS

[Atom](http://en.wikipedia.org/wiki/Atom_(standard)) i [Rss](http://en.wikipedia.org/wiki/RSS) predstavljaju dijalekte XML-a, ali na svu sreću nije potrebno da se bavimo parsiranjem XML-a. U .NET-u postoji skup klasa koje su namenjen za rad sa strukturama podataka kakve su Atom i Rss feed-ovi. U prostoru imena [System.ServiceModel.Syndication](http://msdn.microsoft.com/en-us/library/system.servicemodel.syndication.aspx) nalazi se nekoliko klasa:

- SyndicationFeed
- SyndicationItem
- SyndicationContent
- itd.

## Implementacija

Uzećemo za primer blog aplikaciju, odnosno Post klasu kao model za koji ćemo generisati feed:

```csharp
public class Post
{
    public int PostId { get; set }
    public string Title { get; set; }
    public string Body { get; set; }
    public string Permalink { get; set; }
    public DateTime Posted { get; set; }
}
```

FeedController je krajnje jednostavan i mislim da nema potrebe detaljnije objašnjavati šta radi:

```csharp
public class FeedController : Controller
{
    private BlogContext db = new BlogContext();
     
    public ActionResult Atom()
    {
        var posts = db.Posts.All();
        return FeedResult(posts, FeedFormat.Atom10);
    }
     
    public ActionResult Rss()
    {
        var posts = db.Posts.All();
        return FeedResult(posts, FeedFormat.Rss20);
    }   
}
```

FeedResult izgledaće ovako:

```csharp
public class FeedResult : FileResult
{
    private IEnumerable items;
    private FeedFormat feedFormat;
    private Uri currentUrl;
    public SyndicationFeed Feed { get; set; }
 
    public FeedResult(IEnumerable items, FeedFormat format)
        : base("application/rss+xml")
    {
        this.items = items;
        this.feedFormat = format;
    }
 
    public override void ExecuteResult(ControllerContext context)
    {
        this.currentUrl = context.RequestContext.HttpContext.Request.Url;
        base.ExecuteResult(context);
    }
 
    protected override void WriteFile(HttpResponseBase response)
    {
        Feed = new SyndicationFeed("Feed title", "Feed description", this.currentUrl, GetSyndicationItems());            
        Feed.Authors.Add(new SyndicationPerson("my@email.com", "My Name", "MyBlogUrl"));            
        Save(XmlWriter.Create(response.Output));
    }
 
    private void Save(XmlWriter writer)
    {
        switch (this.feedFormat)
        {
            case FeedFormat.Atom10:
                Feed.SaveAsAtom10(writer);
                break;
            case FeedFormat.Rss20:
                Feed.SaveAsRss20(writer);
                break;
            default:
                Feed.SaveAsRss20(writer);
                break;
        }
        writer.Close();
    }
 
    private List GetSyndicationItems()
    {
        var items = new List();
        foreach (var post in this.items)
        {
            var item = new SyndicationItem
            {
                Title = SyndicationContent.CreatePlaintextContent(post.Title),
                Content = SyndicationContent.CreateHtmlContent(post.Body),
                Id = post.PostId.ToString(),
                PublishDate = post.Posted,
                LastUpdatedTime = post.Posted
            };
            item.AddPermalink(new Uri(post.Permalink));
            items.Add(item);
        }
        return items;
    }
}
 
public enum FeedFormat
{
    Atom10,
    Rss20
}
```

Mislim da kod FeedResult-a dovoljno sam sebe objašnjava, treba samo obratiti pažnju na dve metode koje su preklopljne: WriteFile i ExecuteResult. ExecuteResult je metoda nasleđena od ActionResult-a i nju okida [ControllerActionInvoker](http://msdn.microsoft.com/en-us/library/system.web.mvc.controlleractioninvoker.aspx). WriteFile je nasleđena on FileResult klase i ona je odgovorna za pisanje sadržaja fajla u response. U slučaju nekog fajla to bi bio niz bajtova, a u našem slučaju to je sadržaj XML fajla (Atom ili RSS feed).
