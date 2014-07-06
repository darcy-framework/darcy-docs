# Project Structure
> Note: if you want to just cut to the chase and start working with darcy, feel free to skip this section and move onto [Getting Started](../getting_started/README.md).

At its core, **darcy** is just APIs. It provides a declarative and friendly API for consumers to write page objects, as well as a straightforward and flexible API for implementations. It does not actually do any automating, that is up to some implementation of this API. If a library can find elements and interact with them, it can be an implementation.

## Modules
The most basic, fundamental API to describe user interfaces and their elements is the duty of the [UI module](https://github.com/darcy-framework/darcy-ui).

Those element types are relevant across a number of domains. Where more domain-specific interaction is needed, other modules are used. The [web module](https://github.com/darcy-framework/darcy-web) is one such module. It extends the UI API, providing interfaces for browsers, and locators based on HTML and CSS.

To aid in synchronizing the automation thread with conditions and events occurring in the UI, the [**synq**](https://github.com/darcy-framework/synq) library is used.

## Implementations
At the moment, the only implementation is [darcy-webdriver](https://github.com/darcy-framework/darcy-webdriver), a wrapper around the [Selenium WebDriver](http://docs.seleniumhq.org) project.
