# Automating the Web

Though we've been using web pages as an easy example, everything we've talked about so far applies to all kinds of user interfaces, not just HTML. Okay, we used locators that tied to id's of web elements, but locator strategies like those can also apply to [mobile](http://selendroid.io/quickStart.html#nativeAppTest) and native applications as well, depending on the automation library.

To actually get started, however, is more context dependent. There is no common API for "opening a view"&mdash;this depends on the type of context we're working with. For the web, there are browsers. Browsers open URLs, and URLs correspond to expected views. And before we can do that, we first have to open a browser of course! [Let's start there.](working_with_a_browser.md)

