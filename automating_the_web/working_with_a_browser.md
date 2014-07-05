# Working with a Browser

The common entrypoint for all browser automation libraries is the `BrowserFactory`, however each library will have its own setup specifics.

## Getting started with darcy-webdriver

To use Selenium WebDriver as the automation library implementation, import darcy-webdriver into your project using Maven:

```xml
<groupId>com.redhat.darcy</groupId>
<artifactId>darcy-webdriver</artifactId>
<version>0.1-SNAPSHOT</version>
```

There are several `BrowserFactory` implementations to choose from in the darcy-webdriver library, each corresponding to an available WebDriver implementation, such as FirefoxDriver or RemoteWebDriver. Within each specific browser factory, you can fluently describe any DesiredConditions or other options, like a Firefox profile.

```java
import com.redhat.darcy.web.Browser;
import com.redhat.darcy.web.BrowserFactory;
import com.redhat.darcy.webdriver.FirefoxBrowserFactory;

FirefoxProfile fp = new FirefoxProfile();

// Set some options on the profile

BrowserFactory browserFactory = new FirefoxBrowserFactory()
    .usingProfile(fp);
Browser browser = browserFactory.newBrowser();
```

Each call to `newBrowser` launches a FirefoxDriver for us to use, and gives us a `Browser` object that controls it. Finally! Let's get started with it.

## URLs

To open a page, we not only need to supply a URL to plug into the browser, but also a view that we expect to be loaded as a result. The browser will set itself as the view's context and block the thread until the view tells us that it is loaded.

```java
String google = "http://www.google.com";
GoogleSearch search = browser.open(google, new GoogleSearch());
```

Notice that `open` returns the view that we passed in as an argument. This allows us to instantiate the view, open its URL, wait for it to be loaded, and assign it to a variable, all in one line.

What if we wanted to associate that URL string with the expected view more closely? We can do just that with the `ViewUrl` type.

```java
@RequireAll
public class GoogleSearch extends AbstractView {
  public static ViewUrl<GoogleSearch> url() {
    return new StaticViewUrl("http://www.google.com", GoogleSearch::new);
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
    return RedditUrl.subReddit("AdviceAnimals", AdviceAnimals::new);
  }

  // snip
```

In this example, should the URL scheme for subreddits change, or you want to use a different environment where new beta features are in development, you don't have to change anything in your view code.

# Next steps

At this point, you know how to describe views, work with elements and transitions, open a browser, and use it to open a URL pointing to a view. In other words, you're ready to write some serious automation. I recommend trying to automate some basic tasks, and exploring the APIs available to you in `Browser` and the various elements. Questions? Comments? Concerns? File an issue or open a pull request on [**darcy**'s GitHub](http://github.com/darcy-framework).
