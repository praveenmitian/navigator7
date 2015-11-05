#gather functionalities implemented or not, discussed in the forums that Vaadin users would like to see in a navigation system.


# Namings #

The preferred and most natural word for a class representing a page is "Page".

"Application" should be used in its "Web Application" meaning (one instance per ServletContext). It should not mean "HttpSession" as it do in Vaadin v6.

"Window" is perfect for floating windows and modal dialog boxes. It could also be used for meaning "meta container of a page in a browser tab". It should be named SomethingWindow, not just Window then. It should have a special class or interface more specific than just "Window". For example "AppLevelWindow".


# Configuration Shift to Servlet Context #

As summary, I'd rename com.vaadin.Application to com.vaadin.UserContext, and make it less central in the design, with things moved to the ServletContext scoped WebApplication. People would extend WebApplication and tell the new class name in the web.xml (init param of Vaadin servlet).

http://vaadin.com/forum/-/message_boards/message/155212#_19_message_159969


# Vaadin Initialization #

Should Vaadin still be initialized through a servlet?
http://vaadin.com/forum/-/message_boards/message/155212?_19_delta=10&_19_keywords=&_19_advancedSearch=false&_19_andOperator=true&cur=3#_19_message_216481

It should be possible also for a batch job to initialize it (through one method call) because it needs Vaadin to produce URLs for mails. Safe guards would not enable Vaadin to be initialized twice in the same ServletContext (if any).

http://vaadin.com/forum/-/message_boards/message/155212?_19_delta=10&_19_keywords=&_19_advancedSearch=false&_19_andOperator=true&cur=3#_19_message_221383



# Annotation and code configuration. #

Annotations should be used to identify the components that are "pages". They should not extend a special component, any Vaadin Component could be a page.
Annotation scanning could automatically detect pages.
Alternatively, annotation scanning could be turned off, and some Java code in the (web) application initialization code could identify the page classes.


# Hidden Pages #

It should be possible to configure some pages, so they don't change the URI.
I would typically error pages. It would be also the pages of a wizard (the URI would not change (optionnal) between the pages of a wizard).

The typical way to access these pages would be a forward. Maybe it's the forward method that should take a "keepPreviousUri" boolean parameter. The page definitions would have nothing special.

# Interceptors #

An interceptors (kind filters) mechanism should enable to plug-in satellite ability without bloating the heart of the navigation system.
Standard interceptors should be provided for:
- URI parameters processing
- Display exception information to the developer.
- Ask confirmation before leaving a page

Additional interceptors could be written for:
- read Spring security annotations on a page and decide if it should be displayed or not,
- ...

Custom interceptors would seldom be written by application developpers.


# Thread Locals #

Vaadin should enable the application developper to get importants objects "out of the blue" by calling a static method. The servlet scoped WebApplication is an example.

FileUpload request should not "escape" Vaadin request initialization mechanism anymore.
http://vaadin.com/forum/-/message_boards/message/155212?_19_delta=10&_19_keywords=&_19_advancedSearch=false&_19_andOperator=true&cur=3#_19_message_217065


# Fewer "Session Expired" messages #

When clicking a link to another page, the session should silently be renewed, as we don't need the current main windows (everything should be rebuilt).
Most applications would also probably like auto-relogin for users (with cookies) feature.

# Enable batch jobs to build URLs #

Batach jobs (sending mails) should be enabled to use Vaadin to build URLs, even if there is no (threadLocal) http request.


http://vaadin.com/forum/-/message_boards/message/155212?_19_delta=10&_19_keywords=&_19_advancedSearch=false&_19_andOperator=true&cur=2#_19_message_204826


# Reusable page instances #

A page could be configured as "reusable" and the instance would be kept in the HttpSession. That would not be the default behaviour.
http://vaadin.com/forum/-/message_boards/message/155212?_19_delta=10&_19_keywords=&_19_advancedSearch=false&_19_andOperator=true&cur=3#_19_message_224760
http://vaadin.com/forum/-/message_boards/message/155212?_19_delta=10&_19_keywords=&_19_advancedSearch=false&_19_andOperator=true&cur=5#_19_message_255325


# Parameters Injection #

We need the attributes of a page to be automatically filled with parameters. Both positional and named parameters should be possible. A parameter could be optional or required.

Maybe parameters should be positional and required by default. Named parameters would be used for complex situations.

The system must prevent (Exception) building invalid URIs (when placing links to other pages). An URI build is invalid if the given parameter's type does not match the corresponding attribute type; or if there are more parameters than attribute; for example.