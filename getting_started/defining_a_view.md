# Defining a View

In Darcy, the fundamental UI modelling type is a [```View```][1]. Views can be entire pages, or subsets of a page. You can model a reusable widget (say, a jQuery datatable or calendar plugin) with a View, and reuse it in other Views like you would an element. If you're modelling a web application, think of View as simply DOM. DOM is a tree structure, and a View can start at any point in the tree that is not a leaf. The leaves are elements. The starting point of a View is the View's [```ElementContext```][4], which we'll learn more about later.

Enough details, let's start with the most basic example, the simple page object.

## Extend AbstractView
In darcy, for a type to be a View, it need only fulfill a very simple contract: implement View. You could absolutely write your page objects this way, but you might wish there was a parent class that could reduce your boilerplate code and provide some syntactic sugar. AbstractView is just that class.

```java
public class MyHomePage extends AbstractView {
}
```

Okay, so this will compile, but it's clearly missing its guts. We said Views are organizations of elements, so let's add some.

## Elements as fields
There are two ways to interact with elements. The first we'll introduce is the most common and most straightforward. Defining elements as fields is great because,

* It keeps element definitions in a concise spot.
* We can annotate them and analyze them at runtime (get to this later).
* The element definitions (and therefore, locations) can get reused throughout your class.

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

As you can see, we refer to elements declaratively by their element type. Each element interface has a simple and convenient API specific to that type. For example, a TextInput has a clearAndType method, while a Button does not. Folks coming from Selenium WebDriver will welcome the familiar and fluent [```By```][3] syntax for specifying locators. For web-specific locators, see [darcy-web][5]'s [```By```][6] extension.

Element fields are assigned instances by the factory methods on the ```Elements``` class. These are then initialized by AbstractView. As in WebDriver's PageFactory, elements are _lazily loaded_, that is, no attempt is actually made to find that element until a method is called on the element reference instance. If the element is not found then, an exception will be thrown, _unless_ the method called was ```isDisplayed```. ```isDisplayed``` will never throw an exception. It will just return false. Once an element is found, whether or not it is continually refound or cached is up to the automation library implementation. (Elements found in darcy-webdriver are cached).

## Defining elements at runtime
Element fields are convenient, but not always applicable. Sometimes you don't know the location of an element at compile time; you have to compute it at runtime. In this case, we will look up elements more conventionally, by way of invoking the ElementContext directly.

Here's an equivalent version of page object that looks up elements via the context's API instead of declaring elements as fields.

```java
public class MyHomePage extends AbstractView {
  public void login(Credentials credentials) {
    ElementContext context = getContext();
    
    TextInput login = context.find().textInput(By.id("login"));
    TextInput password = context.find().textInput(By.id("password"));
    Button submit = context.find().button(By.id("submit"));

    login.clearAndType(credentials.login());
    password.clearAndType(credentials.password());
    submit.click();
    // TODO: Return an instance of the next page object
  }
}
```

As you can see, it's a little more verbose, but it allows us to put off creating a `Locator` until the method is called, which will be necessary on occasion.

## Load conditions
We said the contract of a view is that it must implement the `View` interface. The View interface defines three methods: `setContext`, `getContext`, and `isLoaded`. These views have extended `AbstractView`, which has implemented those for us. Now, we can imagine how `setContext` and `getContext` could be implemented, but what about `isLoaded`? How does `AbstractView` know whether to return true or false for that method? The answer is right now, it doesn't! But we can fix that quite easily.

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

Did you see what we changed? We added annotations. One with a class target (```RequireAll```) and one with a field target (```NotRequired```). I bet you can guess what these do! ```RequireAll``` tells ```AbstractView``` that in order for this view to be considered loaded, all of the declared element fields must be displayed, except those annotated with ```NotRequired```. Now ```AbstractView``` knows how to determine if our view is loaded or not.

If you ever needed finer grain control in determining whether a view was loaded or not, you can override the protected method ```loadCondition``` which returns a function that can be used to evaluate whether or not the view is loaded, in addition to annotated elements. If no elements are annotated and ```loadCondition``` is not overriden, an exception will be thrown on a call to ```setContext``` or ```isLoaded``` for that view.

# Next steps

So now we have a page object that is valid at compile and runtime. Unfortunately, our view is still missing something key. I left something out when I said a view is an organization of elements. A view is also an organization of _behaviors_. A view must take ownership of the behavior that is exercised as a result of interacting with its elements. That is, after we click submit, its our view's responsibility to wait for the next page to load before returning from the method. Let's see what facilities darcy provides for us to accomplish just that in [tutorial #3](owning_behavior_in_views.md).

 [1]: https://github.com/darcy-framework/darcy/blob/master/src/main/java/com/redhat/darcy/ui/View.java
 [2]: https://github.com/darcy-framework/darcy/blob/master/src/main/java/com/redhat/darcy/ui/Locator.java
 [3]: https://github.com/darcy-framework/darcy/blob/master/src/main/java/com/redhat/darcy/ui/By.java
 [4]: https://github.com/darcy-framework/darcy/blob/master/src/main/java/com/redhat/darcy/ui/ElementContext.java
 [5]: https://github.com/darcy-framework/darcy-web/
 [6]: https://github.com/darcy-framework/darcy-web/blob/master/src/main/java/com/redhat/darcy/web/By.java
