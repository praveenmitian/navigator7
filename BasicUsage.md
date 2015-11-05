This text gives an overview of how to use Navigator7 to structure a Vaadin application.
The best way to learn Navigator7 is to look at the example that comes with the Navigator ("example" package). The application described here is simpler.

To use Navigator7, you typically create:
  * an init-param in your servlet definition (web.xml)
  * a subclass of NavigableApplication
  * a subclass of HeaderFooterFixedAppLevelWindow
  * several page components (typically VerticalLayout or CssLayout descendant)

# web.xml #

You should:
  * Use NavigableApplicationServlet (instead of VaadinServlet)
  * add an init-param to specify the webApplication (class that you create further)

```
	<servlet>
		<servlet-name>vaadin</servlet-name>
		<servlet-class>org.vaadin.navigator7.NavigableApplicationServlet</servlet-class>
		<init-param>
			<param-name>application</param-name>
 			<param-value>myPackage.MyApplication</param-value>
 		</init-param>  
 		<init-param>
 			<description>Navigator7 WebApplication class to start (optionnal)</description>
  			<param-name>webApplication</param-name>
  			<param-value>mypackage.MyWebApplication</param-value>
  		</init-param>
	</servlet>
```

As before, you still specify your "application" class.
Additionnaly, you specify your "webApplication" class created below:

# MyWebApplication #

In, the class below, you specify:
  * a list of page classes (your pages)

```
/** 
 * 
 * @author John Rizzo - BlackBeltFactory.com
 *
 */
public class MyWebApplication extends WebApplication {

    public MyWebApplication() {
        // We need to do that in the constructor (and not later), to ensure that the init method in the ancestor has the pages.
        registerPages(new Class[] {
                DashboardPage.class,
                EditorPage.class,
                TicketPage.class,
                ParamTestPage.class,
                ProductAPage.class,
                ProductBPage.class,
                SeoPage.class,
        });
    }
```


# MyAppLevelWindow #

To work with Navigator7, you must provide a Window extending NavigableAppLevelWindow.
Usually, you don't extend it directly. In the example below, we extend HeaderFooterFixedAppLevelWindow, which provides a template for header/footers and a fixed size layout. It's quite easy to develop other templates, while this template should be the most popular.

Short simplistic version:
```
public class MyAppLevelWindow extends HeaderFooterFixedAppLevelWindow {

    @Override
    protected Component createHeader() {
        retun new Label("Hello, I'm a HEADER.");
    }

    @Override
    protected Component createFooter() {
        return new Label("Hello, I'm a footer!");
    }
}
```

The API enables extending the template very easily. There is a more complex example in the add-on source code.

# MyApplication #

In any Vaadin6 site, you extend com.vaadin.Application. You'll do it as well with Navigator7, but indirectly. Extend NavigableApplication provided by Navigator7 and which extends com.vaadin.Application.
Your class define which (NavigableAppLevel)Window to use: the one that you just created in the previous step.

```
public class MyNavigableApplication extends NavigableApplication {

    public MyNavigableApplication() {
        setTheme("myTheme");
    }

    @Override
    public NavigableAppLevelWindow createNewNavigableAppLevelWindow() {
        return new MyAppLevelWindow();
    }
    
}
```

# Pages #

Finally, you define pages. A page can be any Vaadin Component.

Simplistic example:
```
@Page
public class SimpleContentPage extends VerticalLayout {

    public MyPage1() {
        addComponent( new Label("I'm on a page") );
        addComponent( new Label("I'm on a page too...") );
        addComponent( new Label("There is no much content here") );
    }

}
```

All you have to do is to define the page (SimpleContentPage.class) in you WebApplication descendant as shown above.


# Navigation #

The page above is accessible from this url:

`http://mydomain.com/#SimpleContent`

# Parameters #

You probably create pages to display something...
And you want your pages (with content) to be bookmarkable.

Let's say that you want to display the product, which in your DB has the id 123.

`http://mydomain.com/#Product/123`

or (your choice)

`http://mydomain.com/#Product/id=123`

Your ProductPage class just needs to:
  * implement ParamChangeListener
  * either
  * use @Param to annotate attributes for automatic injection.
  * use the UriAnalyzer in the paramChanged() method.

## @Param (int) ##

```
public class ProductPage extends VerticalLayout implements ParamChangeListener {

    @Param(pos=0, required=true) int id; 

    Label productDescriptionLabel = new Label();
    
    public ProductPage() {
        // Main info about the current product
        addComponent(productDescriptionLabel);

        // Link to product of the month.
        pr = new ParamPageResource(ProductPage.class, 44455));
        addComponent(new Link("Don't miss the product of the month:" , pr));
    }

    @Override
    public void paramChanged(NavigationEvent event) {
        // Nothing to do to retrieve id, because it has been injected by Navigator7
        
        Product product = .... call your DAO to find the product from id.
        setProduct(product);        
    }

    protected void setProduct(Product product) {
        productDescriptionLabel.setValue(product.getDescription());
    }
}
```

## @Param (Product) ##

```
public class ProductPage extends VerticalLayout implements ParamChangeListener {

    @Param(pos=0, required=true) Product product; 

    Label productDescriptionLabel = new Label();
    
    public ProductPage() {
        // Main info about the current product
        addComponent(productDescriptionLabel);

        // Link to product of the month.
        Product productOfTheMonth = .... (get from DAO probably)
        pr = new ParamPageResource(ProductPage.class, productOfTheMonth));
        addComponent(new Link("Don't miss the product of the month:" , pr));
    }

    @Override
    public void paramChanged(NavigationEvent event) {
        // Nothing to do to retrieve product, because it has been
        // queried by MyUriAnalyzer and injected by Navigator7
        
        setProduct(product);        
    }

    protected void setProduct(Product product) {
        productDescriptionLabel.setValue(product.getDescription());
    }
}
```

```
/** Example of EntityUriAnalyzer.
 * Your Navigator7 can live with the deafault provided ParamUriAnalyzer (only working with Strings and simple types).
 * If you have a DB you probably want an EntityUriAnalyzer.
 * 
 * @author John Rizzo - BlackBeltFactory.com
 */
@Configurable   // for Spring injection
public class MyUriAnalyzer extends EntityUriAnalyzer<BaseEntity> {

    @PersistenceContext
    EntityManager entityManager;   // Injected by Spring, for example.

    @Override
    public BaseEntity findEntity(Class<? extends BaseEntity> entityClass, String pk) {
        return entityManager.find(entityClass, pk);
    }

    @Override
    public String getEntityFragmentValue(BaseEntity entity) {
        // It's especially easy here because the primary key is in a common ancestor.
        return entity.getId().toString();
    }

}

```

## UriAnalyzer (manual) ##

```
public class ProductPage extends VerticalLayout implements ParamChangeListener {

    Label productDescriptionLabel = new Label();
    
    public ProductPage() {
        // Main info about the current product
        addComponent(productDescriptionLabel);

        // Link to product of the month.
        pr = new PageResource(ProductPage.class, "44455"));
        addComponent(new Link("Don't miss the product of the month:" , pr));
    }

    @Override
    public void paramChanged(NavigationEvent event) {
        // Thread local pattern to get the Application (which has the UriAnalyser)
        ParamUriAnalyzer analyzer = NavigableApplication.getCurrent().getUriAnalyzer();
        
        // 1st parameter as a String.
        Long id = analyzer.getMandatoryLong(event.getParams(), 0);  // Position 0.
        if (id == null) { return; }  // analyzer already displayed a message to the end-user.

        // other version: 1st parameter as an Entity (JPA or anything else).
        EntityUriAnalyzer eAnalyzer = (EntityUriAnalyzer)analyzer;
        Product product = eAnalyzer.getMandatoryEntity(event.getParams(), 0);  // Position 0.
        if (product == null) { return; }  // analyzer already displayed a message to the end-user.
        
        setProduct(product);        
    }

    protected void setProduct(Product product) {
        productDescriptionLabel.setValue(product.getDescription());
    }
}
```


That's it for the very high level basics.