# Working with a Browser



## Getting started with darcy-webdriver

The common entrypoint for all browser automation libraries is the `BrowserFactory`.

```java
import com.redhat.darcy.web.Browser;
import com.redhat.darcy.web.BrowserFactory;
import com.redhat.darcy.webdriver.FirefoxBrowserFactory;

BrowserFactory browserFactory = new FirefoxBrowserFactory();
Browser browser = browserFactory.newBrowser();
```

A call to `newBrowser` launches the FirefoxDriver for us to use. If you needed a remote WebDriver instead, you could use `RemoteBrowserFactory`. The same goes for other browsers.

## URLs

To open a page, we not only need to supply a URL to plug into the browser, but also a view that we expect to be loaded as a result. The browser will set itself as the view's context, block the thread until the view tells us that it is loaded.

```java
String google = "http://www.google.com";
GoogleSearch search = browser.open(google, new GoogleSearch());
```

What if we wanted to associate that URL string with the expected view more closely? We can do just that with the `ViewUrl` type.

```java
@RequireAll
public class GoogleSearch extends AbstractView {
  public static ViewUrl<GoogleSearch> url() {
    return new StaticUrl("http://www.google.com", new GoogleSearch());
  }

  // snip
```

And now to open this page, we can simply do:

```java
GoogleSearch search = browser.open(GoogleSearch.url());
```

I recommend getting creative with your `ViewUrl` instantiation when writing a lot of page objects for your applications. Something like this would be more "DRY":

```java
public class AdviceAnimals extends AbstractView {
  public staic ViewUrl<AdviceAnimals> url() {
    return RedditUrl.subReddit("AdviceAnimals", new AdviceAnimals());
  }

  // snip
```

In this example, should the host for Reddit change, or you want to use a different environment where new beta features are in development, you don't have to change all of the hardcoded URLs inside your views.


