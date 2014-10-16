# Working with a Browser

The common entrypoint for all browser automation libraries is the `BrowserFactory`, however each library will have its own setup specifics.

## Getting started with darcy-webdriver

To use Selenium WebDriver as the automation library implementation, import darcy-webdriver into your project using Maven:

```xml
<groupId>com.redhat.darcy</groupId>
<artifactId>darcy-webdriver</artifactId>
<version>0.1-SNAPSHOT</version>
```

To use snapshot versions, you'll need Sonatype's snapshot repo in your pom or settings.xml.

```xml
<repositories>
    <repository>
        <id>central-snapshots</id>
        <url>https://oss.sonatype.org/content/repositories/snapshots</url>
        <releases><enabled>false</enabled></releases>
        <snapshots><enabled>true</enabled></snapshots>
    </repository>
</repositories>
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

## Open event

Interaction with the browser is asynchronous. The opened page might have some redirects, or some Javascript or AJAX to load. In the `Browser` API, there is a method `open(String, View)` that accepts a URL string, and a view that you expect to be loaded, with the browser as its context, after this operation should complete. We've seen these kinds of expectations before when we wrote our views: transitions. The same concept applies here. We are constructing a transition event that should happen after we open the provided URL.

In light of this, `open` returns an `Event`!

```java
String google = "http://www.google.com";
Event<GoogleSearch> openSearch = browser.open(google, new GoogleSearch());
```

And like all events, it can be awaited:

```java
openSearch.waitUpTo(1, ChronoUnit.MINUTES);
```

Or, customized further:

```java
openSearch = browser.open(google, new GoogleSearch())
    .failIf(browser.transition().to(new Http404Page());
```

Of course, you will likely want to chain the event interaction, so you can simply get back your view:

```java
String google = "http://www.google.com";
GoogleSearch search = browser.open(google, new GoogleSearch())
    .waitUpTo(1, ChronoUnit.MINUTES);
```

You'll notice many of the Browser APIs return events for precisely the same reason.

## View URLs

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
GoogleSearch search = browser.open(GoogleSearch.url())
    .waitUpTo(1, ChronoUnit.MINUTES);
```

I recommend getting creative with your `ViewUrl` instantiation when writing a lot of page objects for your applications. Something like this would be more "DRY":

```java
public class AdviceAnimals extends AbstractView {
  public static ViewUrl<AdviceAnimals> url() {
    return RedditUrl.subReddit("AdviceAnimals", AdviceAnimals::new);
  }

  // snip
```

In this example, should the URL scheme for subreddits change, or you want to use a different environment where new beta features are in development, you don't have to change anything in your view code.

# Next steps

At this point, you know how to describe views, work with elements and transitions, open a browser, and use it to open a URL pointing to a view. In other words, you're ready to write some serious automation! I recommend trying to automate some basic tasks, and exploring the APIs available to you in `Browser` and the various elements. Questions? Comments? Concerns? File an issue or open a pull request on [**darcy**'s GitHub](http://github.com/darcy-framework).
