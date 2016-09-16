---
layout: post
status: publish
published: true
title: How to assert Logback calls with Mockito
author:
  display_name: Alfredo Díaz
  url: http://www.therore.net/
author_url: http://www.therore.net/
date: '2016-09-14 21:24:42 +0200'
date_gmt: '2016-09-14 21:24:42 +0200'
categories:
- Uncategorized
tags:
- logback
- mockito
- junit
- unit-testing
comments: []
---

<p>
<a href="/images/logback_with_mockito.jpg"><img src="/images/logback_with_mockito.jpg" alt="logback with mockito" width="300" height="201" class="alignnone size-medium wp-image-330" style="float: left; padding: 30px"/></a>
When a modern robust application has to deal with failures, it tries to continue operating gracefully keeping a normal external behaviour. Usually each of these failures are reported to a logging system in order that they can be monitorized. Not only failures are reported, any relevant event is logged with the same purpose.
</p>

Focusing on unit testing, it is quite common that when we test a piece of code, we get an output with no apparent errors and on the other hand the logging system has received notification of errors. **It becomes clear that to test properly operating of the [SUT](https://en.wikipedia.org/wiki/System_under_test), logged events also have to be verified**.

I have created a Junit rule to facilitate assertions on logging events with logback.

The unit-test class LogbackRuleTest shows some examples and requires the following dependencies.
{% highlight xml %}
  <dependency>
    <groupId>net.therore.spring.mockito</groupId>
    <artifactId>therore-spring-mockito</artifactId>
    <version>LATEST</version>
  </dependency> 
  <dependency>
    <groupId>net.therore.spring.mockito</groupId>
    <artifactId>therore-spring-mockito</artifactId>
    <version>LATEST</version>
  </dependency> 
  <dependency>
      <groupId>org.projectlombok</groupId>
      <artifactId>lombok</artifactId>
      <version>LATEST</version>
      <scope>provided</scope>
  </dependency>
</pre>
{% endhighlight %}

{% highlight java  %}
import lombok.extern.slf4j.Slf4j;
import net.therore.spring.mockito.logback.LogbackRule;
import org.junit.Rule;
import org.junit.Test;

import static net.therore.spring.mockito.logback.EventMatchers.errorWithException;
import static net.therore.spring.mockito.logback.EventMatchers.text;
import static org.mockito.Matchers.argThat;
import static org.mockito.Mockito.*;

@Slf4j
public class LogbackRuleTest {

    @Rule
    public LogbackRule rule = new LogbackRule();

    @Test
    public void happyPath() {
        log.info("info message");

        // verify that no error with exception has been logged
        verify(rule.getLog(), never()).contains(argThat(errorWithException()));
    }

    @Test(expected = AssertionError.class)
    public void exceptionPath() {
        log.error("error message", new RuntimeException());

        // verify that no error with exception has been logged
        verify(rule.getLog(), never()).contains(argThat(errorWithException()));
    }

    @Test(expected = AssertionError.class)
    public void combinedExceptionPath() {
        log.info("info message");
        log.error("error message", new RuntimeException());

        // verify that no error with exception has been logged
        verify(rule.getLog(), never()).contains(argThat(errorWithException()));
    }

    @Test
    public void testContainingSpecificMessage() {
        log.info("specific message");

        // verify that the message "specific message" has been logged
        verify(rule.getLog(), atLeastOnce()).contains(argThat(text("specific message")));
    }

    @Test
    public void testContainingCombinedSpecificMessage() {
        log.info("specific message");
        log.info("other message");

        // verify that the message "specific message" has been logged
        verify(rule.getLog(), atLeastOnce()).contains(argThat(text("specific message")));
    }

    @Test(expected = AssertionError.class)
    public void testNotContainingSpecificMessage() {
        log.info("specific message");

        // verify that the message "specific message" has not been logged
        verify(rule.getLog(), never()).contains(argThat(text("specific message")));
    }

    @Test(expected = AssertionError.class)
    public void testNotContainingCombinedSpecificMessage() {
        log.info("other message");
        log.info("specific message");

        // verify that the message "specific message" has not been logged
        verify(rule.getLog(), never()).contains(argThat(text("specific message")));
    }

}
{% endhighlight %}
