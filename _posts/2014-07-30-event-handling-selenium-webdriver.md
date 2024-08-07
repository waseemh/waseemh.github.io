---
layout: post
title: "Event Handling in Selenium WebDriver"
comments: true
tags: selenium, webdriver
permalink: event-handling-selenium-webdriver
---
Hooking into events in an automated testing environment can be helpful for debugging, logging and investigation purposes. Moreover, when it comes to executing automated web tests, these aspects play even more important role in maintaining tests and analyzing results.

If you are developing web tests based on Selenium WebDriver framework, I highly recommend using its built-in event mechanism. Adapting such mechanism in your tests infrastructure can be benefcial in the long run, in terms of maintenance and analysis.

Selenium Web Driver 2.0 framework allows to track different events during test execution such as website navigation, elements interactions and thrown exceptions.
As a test developer, I can listen to these events during test execution and provide my own implementation for handling the events once they are triggered by WebDriver.

For example, I can use the On Exception event to take a screenshot of browser instance once exception is thrown by WebDriver. Or maybe use the After Change Value event to track how many times each element have been modified during the tests. Practical imagination has no limits.

In this brief tutorial we are going to introduce the event handling mechanism in WebDriver framework, show how to apply it in your tests and suggest several useful implementations.

### Introduction

The events mechanism in WebDriver is composed of two major objects:

- EventFiringWebDriver: A wrapper of the normal WebDriver API but adds the support of event triggering.
- WebDriverEventListener: An interface with pre-defined events that EventFiringWebDriver instance will trigger.

![Diagram of objects relation](/images/eventdriver.png)

Following the observer pattern, each EventFiringWebDriver instance holds a list of WebDriverEventListener objects. Whenever an event is triggered by WebDriver, all registered listeners will be notified of this event and each listener may react differently to each event.

I'm not going to go over all supported events by WebDriver, since they are self-explanatory in [WebDriver API](https://selenium.googlecode.com/git/docs/api/java/org/openqa/selenium/support/events/WebDriverEventListener.html) documentation.
 
Our first step would be to initialize the EventFiringWebDriver based on the WebDriver instance we already use. In addition, we should create and register an event listener for the EventFiringWebDriver instance.

An example using FirefoxDriver:

```java
//initalize FireFox webdriver instance
WebDriver driver = new FirefoxDriver();

//initalize event-firing driver using Firefox webdriver instance.
EventFiringWebDriver eventDriver = new EventFiringWebDriver(driver); 

//initalize new webdriver event listener
WebDriverEventListener listener = new DefaultWebDriverEventListener();

//register event listener to even-firing webdriver instance
eventDriver.register(listener);
```
 
### Creating Event Listeners
 
After our event listener has been registered in driver, we need to provide implementation for the triggered events. WebDriver API provides two approaches to implement event listeners:

1) Implement WebDriverEventListener interface: This interface includes all the supported events by WebDriver. As a listener, you must provide an implementation of this interface's methods. One major drawback of this approach is that you have to implement all methods in interface. You may find yourself providing empty implementation for irrelevant events in your tests.

{% highlight java %} 
public class DefaultWebDriverEventListener implements WebDriverEventListener {

	public void beforeNavigateTo(String url, WebDriver driver) {
		System.out.println("Driver will navigate to:" + url);
	}

	public void afterNavigateTo(String url, WebDriver driver) {
		System.out.println("Driver navigated to:" + url);
	}

	public void beforeNavigateBack(WebDriver driver) {
		// TODO Auto-generated method stub
		
	}

	public void afterNavigateBack(WebDriver driver) {
		// TODO Auto-generated method stub
		
	}

	public void beforeNavigateForward(WebDriver driver) {
		// TODO Auto-generated method stub
		
	}

	public void afterNavigateForward(WebDriver driver) {
		// TODO Auto-generated method stub
		
	}

	public void beforeFindBy(By by, WebElement element, WebDriver driver) {
		System.out.println("Driver will find element using locator:" + by);
	}

	public void afterFindBy(By by, WebElement element, WebDriver driver) {
		// TODO Auto-generated method stub
		
	}

	public void beforeClickOn(WebElement element, WebDriver driver) {
		// TODO Auto-generated method stub
		
	}

	public void afterClickOn(WebElement element, WebDriver driver) {
		// TODO Auto-generated method stub
		
	}

	public void beforeChangeValueOf(WebElement element, WebDriver driver) {
		// TODO Auto-generated method stub
		
	}

	public void afterChangeValueOf(WebElement element, WebDriver driver) {
		// TODO Auto-generated method stub
		
	}

	public void beforeScript(String script, WebDriver driver) {
		// TODO Auto-generated method stub
		
	}

	public void afterScript(String script, WebDriver driver) {
		// TODO Auto-generated method stub
		
	}

	public void onException(Throwable throwable, WebDriver driver) {
		// TODO Auto-generated method stub
		
	}

}
{% endhighlight %}

2) Extend AbstractWebDriverEventListener: This abstract class basically implements the WebDriverEventListener interface with empty implementation. Its purpose is to let you override only the events you actually need to listen to.

{% highlight java %} 
import org.openqa.selenium.By;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.support.events.AbstractWebDriverEventListener;

public class DefaultWebDriverEventListener2 extends AbstractWebDriverEventListener{
	
	@Override
	public void beforeNavigateTo(String url, WebDriver driver) {
		System.out.println("Driver will navigate to:" + url);
	}

	@Override
	public void afterNavigateTo(String url, WebDriver driver) {
		System.out.println("Driver navigated to:" + url);
	}
	
	@Override
	public void beforeFindBy(By by, WebElement element, WebDriver driver) {
		System.out.println("Driver will find element using locator:" + by);
	}

}
{% endhighlight %}

Both approaches lead to same result and it's up to you to choose, based on design consideration.
 
### Multiple Event Listeners

As we discussed before, each event-firing WebDriver may include multiple event listeners at the same time. When an event is fired, all registered listeners will be notified about it. For example:

{% highlight java %} 
WebDriver driver = new FirefoxDriver();

EventFiringWebDriver eventDriver = new EventFiringWebDriver(driver);

eventDriver.register(new DefaultWebDriverEventListener());
eventDriver.register(new DefaultWebDriverEventListener2());

eventDriver.get("http://www.google.com");

eventDriver.quit();
{% endhighlight %}

Above example will produce the following output:

    Listener 1 - Driver will navigate to:http://www.google.com
    Listener 2 - Driver will navigate to:http://www.google.com
    Listener 1 - Driver navigated to:http://www.google.com
    Listener 2 - Driver navigated to:http://www.google.com


### Unregistering Event Listeners

If you may need to disable the event listeners on your tests, you can remove the listener from the event-firing WebDriver instance by un-registering it. You may find such option very useful when you want to perform WebDriver operations silently.
WebDriver API provides the following method for un-registering event listeners: public EventFiringWebDriver unregister(WebDriverEventListener eventListener) .
In below example, we initially register two event listeners and un-register first event listener before second navigation operation.

{% highlight java %} 
WebDriver driver = new FirefoxDriver();

EventFiringWebDriver eventDriver = new EventFiringWebDriver(driver);

WebDriverEventListener listener1 = new DefaultWebDriverEventListener();
WebDriverEventListener listener2 = new DefaultWebDriverEventListener2();

eventDriver.register(listener1);
eventDriver.register(listener2);

eventDriver.get("http://www.google.com");

eventDriver.unregister(listener1);

eventDriver.get("http://www.yahoo.com");

eventDriver.quit();
{% endhighlight %}

Above example will produce the following output (notice that Listener 1 wasn't notified of second navigation operation):

    Listener 1 - Driver will navigate to:http://www.google.com
    Listener 2 - Driver will navigate to:http://www.google.com
    Listener 1 - Driver navigated to:http://www.google.com
    Listener 2 - Driver navigated to:http://www.google.com
    Listener 2 - Driver will navigate to:http://www.yahoo.com
    Listener 2 - Driver navigated to:http://www.yahoo.com


### Useful Implementations

After you've successfully integrated the event-firing capability into your web tests, you should think about implementing event listeners according to your tests workflow and requirements. I'm going to list few suggestions which may be useful:

1) Logging library: Integrating event listeners with logging library such as log4j should produce a very informative log of test execution flow.

2) Exception handling: With the help of onException event, you can catch all exception thrown by WebDriver and handle them accordingly. For example: capture screenshots on fail, log missing elements or re-prepare/recover test environment after exception.

3) Popups handling: If you are testing a website with popups appearing randomly, you may find that tests fails due to inability to handle such unexpected popups. Using provided events such as afterClickOn or afterNavigation, you may look for popups and handle them according to your needs.

4) Test Statistics: Gather statistics about number of clicked elements, URL hits or changed values for each element.


I hope that this paper introduced you thorougly to the event handling mechanism in WebDriver.
