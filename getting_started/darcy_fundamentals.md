# Fundamentals

## Views and Elements

Darcy's domain model consists of two main constructs: ```View```s and ```Element```s. Views are your typical page objects, minus the stipulation that they have to represent _whole pages_; they can also represent other collections of elements (such as a widget on a page, or a reusable menu). In fact that's really just what a View is: a organization of elements. The APIs for your Views are derived from behavior your user's get from those elements. Darcy provides an opinionated structure and API for defining Views and tying them together. We'll go over that in these tutorials.

Elements are the concise UI atoms that make up a View. Elements are 1:1 with your typical fundamental UI element types: text inputs, buttons, select boxes, etc. Darcy only provides an API for these elements, and leaves it up to a wrapper around some automation library to implement them. [darcy-webdriver](https://github.com/darcy-framework/darcy-webdriver) is just such a wrapper: it implements darcy's APIs with [Selenium WebDriver](http://docs.seleniumhq.org/).

## darcy-web

By default, darcy's API's are not even web specific. UI is a common language that exists far beyond the browser. However, browsers are ripe for automating and have their own unique domain. For this, there is the tiny [darcy-web](https://github.com/darcy-framework/darcy-web) project, which provides some web-specific structure to darcy.

## "Custom" elements

What is a table? Is it an element? Or is it a collection of more atomic elements? What is a calendar widget? Is the Calendar an element? In darcy, they can, and should be, both a View and an Element. They can behave like a singular element by simply implementing [```Element```](https://github.com/darcy-framework/darcy/tree/master/src/main/java/com/redhat/darcy/ui/elements) (or some extension thereof), and integrate a collection of smaller elements into a higher level API by being a View. We'll learn about how to write and integrate custom element implementations later.

# Next steps

You've added a darcy implementation as a dependency to your project and you understand the basics of darcy's domain model. [Let's start writing some code](defining_a_view.md)!
