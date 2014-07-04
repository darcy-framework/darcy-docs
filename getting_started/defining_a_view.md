# Defining a View

Views are represented in code by the  [`View`](https://github.com/darcy-framework/darcy/blob/master/src/main/java/com/redhat/darcy/ui/View.java) interface. It says views are associated with [`ElementContexts`](https://github.com/darcy-framework/darcy/blob/master/src/main/java/com/redhat/darcy/ui/View.java) (described in [2.1. Fundamentals](darcy_fundamentals.md)) in order for them to look up the elements they are associated with, and requires an `isLoaded` method implementation that the framework and your code can use to determine if a view is fully loaded in whatever context it is assigned to.

## Extend `AbstractView`

Now, you could absolutely write your page objects by simply implementing `View`, but you might wish there was a parent class that could reduce your boilerplate code and provide some syntactic sugar. `AbstractView` is just that class.

```java
public class MyHomePage extends AbstractView {
}
```

Okay, so this will compile, but it's clearly missing its guts. We said views are organizations of elements, so let's add some!

## Elements as fields

When we extend `AbstractView`, we can define elements as fields. This is great because,

* It keeps element definitions in a concise spot.
* We can annotate them and analyze them at runtime (get to this later).
* The element definitions (and therefore, locations) can get reused throughout your view.

```java
import static com.redhat.darcy.ui.Elements.*;

public class MyHomePage extends AbstractView {
  private TextInput login = textInput(By.id("login"));
  private TextInput password = textInput(By.id("password"));
  private Button submit = button(By.id("submit"));

  private Label errorMsg = label(By.class("error"));

  public void login(Credentials credentials) {
    login.clearAndType(credentials.login());
    password.clearAndType(credentials.password());
    submit.click();
    // TODO: Return an instance of the next page object
  }
}
```

As you can see, we refer to elements declaratively by their element type. Each element interface has a simple and convenient API specific to that type. For example, a `TextInput` has a `clearAndType` method, while a `Button` does not. In order to find an element, some locator must be provided, like `By.id("login")`. Folks coming from Selenium WebDriver will welcome the familiar and fluent [`By`](https://github.com/darcy-framework/darcy/blob/master/src/main/java/com/redhat/darcy/ui/By.java) API.

Element fields are assigned instances by the static factory methods on the `Elements` class (I recommend using static imports for syntactic sugar). These are then initialized by `AbstractView`. As in WebDriver's `PageFactory`, elements are _lazily loaded_, that is, no attempt is actually made to find that element until a method is called on the element reference instance. If the element is not found then, an exception will be thrown, _unless_ the method called was `isPresent` or `isDisplayed`. These methods, that test for presence and visibility respectively, will never throw an exception. They will just return false. Once an element is found, whether or not it is continually refound or cached is up to the automation library implementation. Elements found in darcy-webdriver are cached.

> Unlike WebDriver, _all_ UI objects in **darcy** are lazily loaded, not just those defined as class fields. They are merely references who's presence can be tested safely, without nulls or exceptions. So, any element or context you look up may not actually be present. You can test for presence with the `isPresent` method. If you try to interact with one of these objects and they aren't actually present, an exception will be thrown.

## Defining elements at runtime

Element fields are convenient, but not always applicable. Sometimes you don't know the location of an element at compile time; you have to compute it at runtime. In this case, we will look up elements more conventionally, by using an element context directly.

Here's an equivalent version of page object that looks up elements via the context's API instead of declaring elements as fields.

```java
public class MyHomePage extends AbstractView {
  public void login(Credentials credentials) {
    TextInput login = getContext().find().textInput(By.id("login"));
    TextInput password = getContext().find().textInput(By.id("password"));
    Button submit = getContext().find().button(By.id("submit"));

    login.clearAndType(credentials.login());
    password.clearAndType(credentials.password());
    submit.click();
    // TODO: Return an instance of the next page object
  }
}
```

As you can see, it's a little more verbose, but it allows us to put off creating a locator until the method is called, which will be necessary on occasion.

## Load conditions

We mentioned that all views had to implement an `isLoaded` method so that we can test for that view's presence. `AbstractView` is doing this for us, but how does it determine whether or not our view is loaded? The answer is right now, it can't! Fortunately that's an easy fix:

```java
import static com.redhat.darcy.ui.Elements.*;

@RequireAll
public class MyHomePage extends AbstractView {
  private TextInput login = textInput(By.id("login"));
  private TextInput password = textInput(By.id("password"));
  private Button submit = button(By.id("submit"));

  @NotRequired
  Label errorMsg = label(By.class("error"));

  public void login(Credentials credentials) {
    login.clearAndType(credentials.login());
    password.clearAndType(credentials.password());
    submit.click();
    // TODO: Return an instance of the next page object
  }
}
```

Did you see what we changed? We added annotations. One with a class target, `RequireAll`, and one with a field target, `NotRequired`. I bet you can guess what these do! `RequireAll` tells `AbstractView` that in order for this view to be considered loaded, all of the declared element fields must be displayed, except those annotated with `NotRequired`. Now `AbstractView` knows how to determine if our view is loaded or not.

If you ever needed finer grain control in determining if a view is loaded, you can override the protected method `loadCondition` which returns a function that can be used to evaluate whether or not the view is loaded, in addition to annotated elements. If no elements are annotated and `loadCondition` is not overriden, an exception will be thrown on a call to `setContext` or `isLoaded` for that view.

# Next steps

So now we have a page object that is valid at compile and runtime. Unfortunately, our view is still missing something key. I left something out when I said a view is an organization of elements. A view is also an organization of _behaviors_. A view must take ownership of the behavior that is exercised as a result of interacting with its elements. That is, after we click submit, its our view's responsibility to wait for the next page to load before returning from the method. Let's see what facilities **darcy** provides for us to accomplish just that in [tutorial #3](owning_behavior_in_views.md).
