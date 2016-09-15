---
layout: page
title: What is WatirWorks
date:   2015-07-06 23:50:00
categories: Waitr, Ruby, Cucumber
---

WatirWorks IS essentially [Cucumber](https://cucumber.io/){:target="\_blank"}.

[Cucumber](https://cucumber.io/){:target="\_blank"} is a tool traditionally used for [BDD](https://en.wikipedia.org/wiki/Behavior-driven_development){:target="\_blank"}.
It allows tests to be written out in a human readable language (gherkin).
WatirWorks, in the hands of an tester, can be made into
a powerfully simple front end regression framework.
Combined with watir-webdriver the ruby code is simplified even more.

An intended benefit of WatirWorks is that it’s super easy to create
automation *so simple that even a manual tester could do it*,
I liken it to a ‘kiddie pool’ of web automation.
Its not ‘record & playback’ so when the code changes underneath you,
you don’t have to re-record the entire test. Most of the time
you simply need to rename one of the selectors.

A majority of the predefined rules are ‘selector’ based.
Actions are usually taken upon selectors such as links, buttons,
checkboxes, forms, text_fields, dropdowns. A test case
in its simplest form would comprise of one ‘Given’ rule,
one ‘When’ rule and one ‘Then’ rule…

> `Given...` I am in a defined state… like logged out, or logged in on the page under test  
> `When...` I do something like type text, click a link or submit a form  
> `Then...` I validate the desired outcome, like the url or text exists on the page

See the simple example below

### Feature File:

in Gherkin syntax

{% highlight gherkin linenos %}
Feature: Get to Login page
      As a public user, I want click on the login link so that I can get to the login page.

  Scenario: 1 Get to the login page
    Given I am on the "http://the-internet.herokuapp.com" page
      When I click on the "css=a[href$=login]" element
        Then the url should include "login"    
{% endhighlight %}

### Ruby Code:
each line of the above gherkin has its own [matching](http://www.regular-expressions.info){:target="\_blank"} ruby script


`Given I am on the "http://the-internet.herokuapp.com" page`

{% highlight ruby linenos %}
Given /^I am on the "(.*)" page$/ do |url|
  start_time = Time.now
  if ( url =~ /^\/(.*)/ )
    @browser.goto(BASE_URL + "#{url}")
    puts @browser.url
  else
    @browser.goto url
  end
  end_time = Time.now
  elapsed_seconds = (end_time - start_time).round(2)
  puts "Response Time: #{elapsed_seconds} seconds."
end
{% endhighlight %}


`When I click on the "css=a[href$=login]" element`

{% highlight ruby linenos %}
When /^I click on the "(\w{2,9})=(.*)" element$/ do |attribute, value|
  selector = @browser.element(:"#{attribute}" => value)
  selector.wait_until_present
  eval = selector.exists?
  if eval == true
    selector.click
    else
      fail("FAIL!!! I couldn't click the '#{attribute}=#{value}' element")
  end
end 
{% endhighlight %}


`Then the url should include "login"`

{% highlight ruby linenos %}
Then /^the url should include "(.*)"$/ do |included_url|
  my_url = @browser.url
  eval = @browser.url.include? "#{included_url}"
  if eval == true
    puts("TRUE!!! '#{included_url}' is included in '#{my_url}'")
  else
    fail("FAIL!!!! '#{included_url}' is NOT included in '#{my_url}'")
  end
end
{% endhighlight %}


### iTerm:

Example output when I run the step test scenario from above:

{% highlight console %}
  Feature: Get to Login page
Scenario: 1 Get to the login page
  Given I am on the "http://the-internet.herokuapp.com" page
      Response Time: 0.91 seconds.
  When I click on the "css=a[href$=login]" element
  Then the url should include "login"
      TRUE!!! 'login' is included in 'http://the-internet.herokuapp.com/login'

1 scenario (1 passed)
3 steps (3 passed)
0m1.419s            
{% endhighlight %}
