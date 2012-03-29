# Analytics

how do you easily manage web analytics in code?

# Example code from a few months ago

    @@@ js
    $('#layouts-common-left-nav... a').live('click', 
      function() {
        var categoryName = $(this).text();
        analyticsResponder({
          action: 'latest_trends', 
          category: 'navigation', 
          label: categoryName
        });
    });

analytics.js -> google_analytics_deprecated.js

# Why don't we like this again?
&nbsp;

# Why don't we like this again?

    @@@ js
    describe('Clicking product title', function () {
      var selector = '.br-sf-widget-merchant...';
      itShouldFireGAEvent('click', 
        selector, 
        ['product', 
         'bloomreach_pdp:more_click', 
         '-The-Fuchsia-of-Fashion-Dress', 
         1
    ]);});

Multiply this by 8 billion

# How much time does this take to write/maintain?

* How many unit tests do you actually need to know your code works?

# How much time does this take to write/maintain?

* How many unit tests do you actually need to know your code works?
* How often do we add new instrumentation code?

# How much time does this take to write/maintain?

* How many unit tests do you actually need to know your code works?
* How often do we add new instrumentation code?
    * **ALL THE TIME**

# How much time does this take to write/maintain?

* How many unit tests do you actually need to know your code works?
* How often do we add new instrumentation code?
    * **ALL THE TIME**
* DRY?

# How much time does this take to write/maintain?

* How many unit tests do you actually need to know your code works?
* How often do we add new instrumentation code?
    * **ALL THE TIME**
* DRY?
* Multiple analytics toolkits?

# How much time does this take to write/maintain?

* How many unit tests do you actually need to know your code works?
* How often do we add new instrumentation code?
    * **ALL THE TIME**
* DRY?
* Multiple analytics toolkits?
    * **Omniture custom links**

# We live in the future

I was told it would be more awesome than this

# I want to see this

    @@@ ruby
    %div.row{ :analytics => {
      :ga => ['user_actions', 'attribute_selection', "length:#{label}"],
      :event_to_bind => 'mouseup'
    }}

# In fact, I want to see this

    @@@ ruby
    %div.row{ :analytics => {
      :ga => ['my_category', 'my_action', "my_value"],
      :omniture => ['my>link>name', 'event10', '1;1'],
      :event_to_bind => 'mouseup'
    }}

# Oh, and…

    @@@ ruby
    link_to "awesomeness", "/here",
      :analytics => {
        :ga => ['user_actions', 'whoa', 'awesome'],
        :event_to_bind => 'mouseup'
      }

# So what is this actually doing?

this:

    @@@ ruby
    :ga => ['user_actions', 'whoa', 'awesome']

becomes:

    @@@ js
    data-analytics-ga="%5B%22user_actions%22%2C%22whoa%22%2C%22awesome%22%5D"

* note that the array is JSON encoded and escaped
* this makes it very easy to decode later in javascript

# To be totally clear

    @@@ html
    <a href="/link" 
      data-analytics-ga="%5B%22user_actions%22%2C%22whoa%22%2C%22awesome%22%5D"
      data-analytics-omniture="%5B%22my%3Elink%3Ename%22%2C%22event10%22%2C%221%3B1%22%5D"
      data-analytics-event="mouseup">
        awesomeness
    </a>

* :event_to_bind is optional, and writes out "mouseup" if not explicitly set

# What gets sent to Google?

    [0] 'category' => 'user_actions'
    [1] 'action' => 'whoa'
    [2] 'label' => 'awesome'
    [3] 'value' => defaults to 1

# What gets sent to Omniture?

    [0] 'linkName' => 'my>link>name'
    [1] 'events' => 'event10,event15' ... gets event80 merged in
    [2] 'products' => '1;1' ... [inventory_classification.id, product.id]

* don't set event80, it gets stuck on there automagically
* if you need products but don't need custom events, set events to nil

# Unit tests are there to cover the bases

    @@@ js
    describe('when data attributes on an element', function () {
      $.each(['mouseup', 'click', ...], function (i, event) {
        describe('explicit ' + event, function () {
          it("should fire GA on event : " + event, function () {
            setFixture(event, 
              "%5B%22thing%22%2C%22other%20thing%22%5D", "");
            triggerEvent(event);
            expect(testStub.callback)
              .toHaveBeenCalledWith(["thing", "other thing"]);
          });
        });
      });
    });

Multiply this by 1

# We can even test that the correct thing appears in the browser

    Then I should have the following data present for google analytics:
      | selector  | category | action           | label           |
      | .facebook | social   | btb_pdp:facebook | <Might have:id> |
      | .twitter  | social   | btb_pdp:twitter  | <Might have:id> |
      | .email    | social   | btb_pdp:email    | <Might have:id> |

    Then clicking "The Story" within ".product-detail-page" \ 
      would trigger GA event: category: "user_actions", \
      action: "pdp_tab", and label: "The Story"

* It's really hard to see that something actually got sent to Google, but we can at least see that the correct thing was written in the page

# Where is the code?

The ruby side lives in three places:

* *lib/modcloth/core_ext/action_view*
* *lib/modcloth/core_ext/haml*
* *lib/analytics_methods.rb*

This uses alias_method_chain to inject code into Rails helpers and Haml. Currently we don't mix directly into ERB.

# Where is the javascript?

* *public/javascripts/modcloth/analytics/core.js*
    * Sets live bindings on presence of data-analytics-event
* *public/javascripts/modcloth/analytics/ga.js*
    * Initializes Google Analytics
    * Provides callback methods to decode data and push events onto GA stack
    * Tells core.js to look for data-analytics-ga
* *public/javascripts/modcloth/analytics/omniture/customLinks.js*
    * Provides callback methods for sending custom links
    * Merges custom link data with page-level data
    * Tells core.js to look for data-analytics-omniture
    * Tells core.js to look for data-analytics-omnitureAjax

# Wait, what's this about omnitureAjax?

Our chief weapon is surprise, fear and surprise

# Two chief weapons

* Use *omniture* if you are in a normal page.
* Use *omnitureAjax* if you are instrumenting something in Quicklook. The data merging needs to be done differently, as Quicklook masks itself as a new page view to Omniture.

# Ruthless efficiency

* *click* almost never works for instrumentation
* This gets most of the cases, but our site has a lot of javascript patterns that complicate things

# Next steps

* Pull the ruby code out of e-comm and release as an open source gem?
* Extend omniture javascript to more easily support extra params (and be readable)
* Push forward with integration testing
    * When we push things onto GA/Omniture event stack, we can also push info onto our own call stack
    * We can analyze our own call stack in integration tests
        * Already some headway here, but had inconsistent results

?? Other next steps?

# Side notes

* Redirections: We've broken campaign tracking several times because we redirect without including campaign tokens
* Include data analysts

