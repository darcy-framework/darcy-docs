# Project Structure
> Note: if you want to just cut to the chase and start working with darcy, feel free to skip this section and move onto [Getting Started](../getting_started/README.md).

**Darcy**, at its core, is just APIs. It provides a declarative and friendly API for consumers to write page objects, as well as a straightforward and flexible API for implementations. **Darcy** does not actually do any automating, that is up to some implementation of this API. If a library can find elements and interact with them, it can be wrapped with darcy's APIs.

## Modules
The most basic, fundamental API to describe user interfaces and their elements is the duty of the [UI module](https://github.com/darcy-framework/darcy-ui).

Those element types are relevant across a number of domains. Where more domain-specific interaction is needed, other modules are used. The [web module](https://github.com/darcy-framework/darcy-web) is one such module. It extends the UI API, providing interfaces for browsers, and locators based on HTML and CSS.

## Implementations
To use darcy, you'll need to pick an implementation of its API to use, just like you need to pick an implementation of `List` in Java. At the moment, the only implementation is [darcy-webdriver](https://github.com/darcy-framework/darcy-webdriver), a wrapper around the [Selenium WebDriver](http://docs.seleniumhq.org) project.
