# Fundamentals

## Views, Elements, and Contexts

**Darcy**'s domain model consists of three main constructs: views, elements, and contexts. All of which you're looking at right now!

### Elements

> <label for="TextInput">Label <input type="text" name="TextInput" value="TextInput"></label> <input type="button" value="Button"> <select><option>SelectOption</option></select> <a href="">Link</a> <input type="file">

Elements are the most fundamental of the three, and you are probably already very familiar with them. Elements are your typical, atomic user interface objects like text inputs and buttons. That is, they are not typically described as being composed of several parts; the element is _the_ part. In darcy, elements are the glue that binds the API to some automation library behind the scenes.

UI objects that are composed of several elements can behave like one singular element (such as a Calendar widget), however those are not truly fundamental elements. Anything that contains or operates on the fundamental element types is a view.

### Views

Views are your page objects. This chapter you're reading could be described as a view. It has paragraph text, links, and some example elements above. Views describe some collection of elements and their collective behavior, without the linguistic stipulation that they ought to represent _whole pages_; they can and should also represent _parts_ of pages, such as a shared navigation menu, or a calendar widget that is composed of several smaller elements. The table of contents on the left is a great candidate for a view that is not a whole page and could reused for every chapter. You're encouraged to house views within views in a "compositional" style where appropriate. Darcy provides an opinionated structure and API for defining views and tying them together.

### Contexts

A context is anything that can find something else. They have knowledge of other stuff, such as elements or other contexts. A context that can find elements is an element context. A context that can find other contexts is a parent context. For example, the web browser you may be currently viewing this page with is both: it can find elements and other browser windows that it has opened, say if you open a link in a new window. Frames are also contexts. Each window or frame can only find the elements within themselves&mdash;that is, other contexts' elements are not contained within any other context. A context can only find _its_ elements or _its_ other contexts.

> If you're coming from Selenium WebDriver, this is a little bit different than how WebDriver works out of the box. A single driver refers to only one window or frame at a time, but it may switch to different ones. In darcy, a browser or frame reference will always point to the same browser or frame, and this will never change. When you need to interact with a new window, you will find it and get a reference to it. Now you will have two references to two different windows.

Contexts are generally what define the type of application you're working with. If you're automating a web application, your context is a browser (or frame). If you're automating an Android app, your context is an application in the Android operating system. What all contexts have in common is that they can lookup references to elements or other contexts, but beyond that they will have general specific API's depending on what a user can typicall do with them.

## Tying them together

A view cannot work with the elements it describes without a context. A view is merely a description of what some UI looks like and how it works. You can imagine a view without a context, but you wouldn't be able to actually _see_ it. A context provides the eyes and hands with which a view becomes realized. Without a context, those elements a view describes cannot be found in any way. It knows not where to look! By setting the context to a view, such as a web browser or a mobile application, that view then uses that context to look up those elements. In this way, a view is not necessarily tied to the type of context it may be presented in.

# Next steps

You've added a **darcy** implementation as a dependency to your project and you understand the basics of **darcy**'s domain model. [Let's start writing some code!](defining_a_view.md)
