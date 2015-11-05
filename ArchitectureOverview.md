Navigator7 is inspired by building BlackBeltFactory.com with Vaadin 6.
We have implemented many replies from forum posts over the months in our application.

One day, we have extracted and reengineered that infrastructure code out of our application to create Navigator7.

The architecture of Navigator7 is also inspired by the Navigator add-on from Joonas.

Before you read about the architecture, please read about the [BasicUsage](BasicUsage.md).

The framework changed and
THIS DESCRIPTION IS OUTDATED (but it may help anyway). It mainly lacks the WebApplication class (and your descendant)

![http://navigator7.googlecode.com/files/Navigator7-Architecture.jpg](http://navigator7.googlecode.com/files/Navigator7-Architecture.jpg)

## AppLevelWindow ##

As Joonas confirmed that the API lacks the notion of application level window, we created such a class extending Window. It contains nothing, except the implicit fact it plays the role of an Application.mainWindow. Concretely, one instance of AppLevelWindow = 1 tab in your browser.

## NavigableAppLevelWindow ##

It's descendant, NavigableAppLevelWindow, is responsible of displaying a page, and everything eventually needed around (as header/footer). It has a reference to:
  * the current page
  * the Navigator.

## Navigator ##

For each NavigableAppLevelWindow instance, there is a Navigator instance. Navigator is responsible of handling URI changes (fired by a UriFragmentUtility). NavigatorConfig will map the page name (as "Home") in the URI ("#Home/id=123") to the page class (HomePage.class). Then Navigator will instantiate the page and give it to the NavigableAppLevelWindow that knows where to place it (inside the template).

The Navigator will notify the current page (implementing ParamChangeListener) of any uri change.

## UriAnalyzer ##

UriAnalyzer hierarchy helps your application to be bookmarkable. It manages the translation between the URI string (as "#Home/id=123") and your application objects (as HomePage.class and long = 123).
  * UriAnalyzer: page name exctraction
  * ParamUriAnalyzer: parameters manipulated as strings (as "id=123")
  * EntityUriAnalyzer: translation from a parameter String value (as "123") into an object (as new Product()).
  * MyUriAnalyzer (your code): tells how to find entity (if JPA, as simple as em.find(entityClass, value)).


## NavigableApplication ##

Refers the NavigatorConfig and MyUriAnalyzer instances.
Implements the ThreadLocal pattern to refer the application and the main window (NavigableAppLevelWindow) from anywhere. Useful, in your pages for example, to get the UriAnalyzer instance and handle the parameter changes.