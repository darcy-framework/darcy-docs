# Owning Behavior in Views

When you are using an application, you are reactive. You wait for feedback from the UI before proceeding at nearly every step of the way. You don't go clicking around on buttons that aren't there. You wait until you can see them. You have patience.

Computers don't yet have this luxury. And so, automating interaction with an application necessarily involves constantly syncing up that automation code with the application code. The computer is blindfolded. You can't just tell it where to click, you have to give it a means to _see_ into the application, to solicit feedback, just as you would do implicitly.

Darcy leverages another library for this duty: [synq](https://github.com/darcy-framework/synq). In this tutorial we'll learn the basics of Synq's API and how it cooperates with darcy.

## Using Synq to wait for expected behavior
When you type in a search query and press 'enter' in Google, an asynchronous request is made to retrieve your search results. If we tried to examine or interact with the results too soon, our automation code might find there were no results to interact with, or worse, fail if it assumed there was at least one! It is not the responsibility of everything interacting with that page to know about this caveat, and how to deal with it. Clearly, it is the most maintainable thing to own this in the page object that would represent the Google search page. Let's see what that might look like.

```java
import static com.redhat.synq.Synq.after;

public class GoogleSearch extends AbstractView {
  @Require
  private final TextInput query = textInput(By.id("gbqfq"));

  public List<SearchResult> searchFor(String queryToSearch) {
    query.clearAndType(queryToSearch);

    return after(() -> query.sendKeys('\n'))
      .expect(this::getSearchResults, results -> !results.isEmpty())
      .waitUpTo(30, TimeUnit.SECONDS);
  }

  // The implementation of this method is not important for this example
  public List<SearchResult> getSearchResults() {
    // Get a new list each call to make sure our results are current
    // This uses a custom element implementation... we'll talk about how that works later
    return getContext().element()
        .listOfType(SearchResult.class, By.xpath("//li[@class]"), SearchResultImpl::new);
  }
}
```

So, if you're not familiar with Java 8's lambda expressions, [now's the time to review](http://docs.oracle.com/javase/tutorial/java/javaOO/lambdaexpressions.html). What we're doing here should be pretty readable: after we press enter on the query text box, we should expect some _thing_ to satisfy some _condition_ (in this case, it's a list of search results, and the condition is that the list is not empty), and we want to wait up to 30 seconds for that expectation to be met. Behind the scenes, the enter key is not pressed until we tell synq to wait. When the condition is satisfied, the value under examination is returned (in this case, the list of search results). If the condition is not met within the time limit, a TimeoutException will be thrown.

## A quick overview of synq's domain model
The things that synq can wait for are called [`Event`](https://github.com/darcy-framework/synq/blob/master/src/main/java/com/redhat/synq/Event.java)s. An event is just that: something that may happen in the future and can be awaited. The implementation of a specific event takes care of the waiting. You can get an event to wait for in two different ways: either something else makes event instances for you, or you can construct an event yourself by describing the things that must or must not be true when that event occurs. This is what we have done in our example. When we construct an event this way, we construct a [`PollEvent`](https://github.com/darcy-framework/synq/blob/master/src/main/java/com/redhat/synq/PollEvent.java), that is, the occurrence of this event is determined by _polling_ until that condition is met. It's not as accurate as an event that _listens_ for a trigger, but they're very flexible and easy to construct.

Harnessing the full extent of the synq API will make your automation more reliable and failures more identifiable. Read more about synq's API [here](https://github.com/darcy-framework/synq).

## Transitions

In our previous example we constructed an event from a condition. When we want to await a transition from one view to another within a context, darcy can provide us an event to wait for. This event is aptly named, [`TransitionEvent`](https://github.com/darcy-framework/darcy/blob/master/src/main/java/com/redhat/darcy/ui/TransitionEvent.java). If page objects are to own behavior, then they must also own transitioning from one page to another. We'll use our transition event to accomplish this.

Let's take a look at a login page. When we login successfully, we expect that the account overview page should load thereafter.

```java
@RequireAll
public class Login extends AbstractView {
  private TextInput login = textInput(By.id("login"));
  private TextInput password = textInput(By.id("password"));
  private Button submit = button(By.id("submit"));

  @NotRequired
  private Label errorMsg = label(By.id("error"));

  public AccountOverview loginExpectingSuccess(Credentials credentials) {
    login.clearAndType(credentials.login());
    password.clearAndType(credentials.password());

    return after(submit::click)
        .expect(transition().to(new AccountOverview()))
        .failIf(errorMsg::isDisplayed)
          .throwing(new InvalidLoginException(credentials, errorMsg.readText()))
        .waitUpTo(1, TimeUnit.MINUTES);
  }
}
```

So you should notice some differences from our previous example. First, we are passing a pre-made event to `expect`. This is our transition event. We can create a transition event by calling `transition()` (a protected method on AbstractView) and then passing what view we expect to transition to in `to(View)`.

Secondly, we're using another feature in synq's API: `failIf`. This accepts another event (or can construct a `PollEvent` from a function as we have done here), and the resulting event will wait for //both//. That is, it will react to either event occuring: the transition //or// the error being displayed. If the error is displayed before the transition occurs (which triggers when the transition is complete), then the waiting will halt, an an exception will be thrown. The exception to be thrown is specified in `throwing` as you can see above. If the transition occurs before the error is displayed, the test will happily move along. If neither occurs before the timeout (1 minute in this case), a `TimeoutException` will be thrown. Got all that?

# Next steps

At this point, we've gone over all the material necessary to construct great views, hallmarks of the page object pattern! We can work with elements, and we can block our thread from moving on until certain necessary conditions are met to keep our automation code in sync with the UI it's interacting with. Now how the heck do we actually open one of these pages in a browser? Where do we even get a browser to work with? What's the relationship between Views and ElementContexts? All of these questions will be answered in [the next chapter](browsers_and_contexts.md).
