# Event Tracking Guide

## Basics of web tracking

Look at the user tracking in context of an open session. Anonymous visitor visits your webiste and browses your content. You may want to capture user activity by tracking events, updating user attributes and identifying user by unique ID in the right moment (e.g. sign up form submit).

### Identify users
By default Exponea tracks users only by cookie id. The identify function assings current user given id or ids. Use the same id in import wizard to match users.

    multiple_identifiers = {
      'registered': 12345, 
      'email': 'user@domain.com'
    };
    exponea.identify(multiple_identifiers)
  
### Update user attributes
Create or update user attributes. Use attributes for storing static data (age, gender, location). Timestamp of update isn't persisted.

    user_attributes = {
      email: 'john@rambo.com',
      property2: 123
    };
    exponea.update(user_attributes)
  
### Track custom events
Adds new event to profile of current user. Events contain timestamp and are better for analyses than user attributes.

    event_attributes = {
      property1: 'value1',
      property2: 123
    };
    exponea.update(user_attributes)
  
## How to track events to Exponea

There are three ways how to track events:

1. Insert tracking code directly into your website - the best way, requires a direct change in website's code.
2. Use a tag manager for inserting tracking code - good if GTM is already on the site and unreliable because ad blockers block GTM.
3. Write content modifying rules in Exponea app - convenient way when you know what you are doing.

## Advanced tracking tips

### Tracking clicks
Listens to click on given CSS selector.

        $(‘document.body’).on(‘click’, ‘#selector’, function() { 
            exponea.track(“clicked event"); 
        });

### Tracking forms
Listens to submit on given CSS selector ([jQuery Forms Submit docs](https://api.jquery.com/submit/)).

        $('document.body').on("submit", "form", function(e){
            exponea.track(“form submitted"); 
        }

### Tracking links
Turns on tracking of a outbound link.

         exponea.trackLink('#link_element', 'link event name', {
               'custom property1': 'click on #linkelement'
           });

Alternative: use custom HTML attribute 'data-exponea-track' to mark links to be tracked

        <a href="http://www.outboudsite.com" data-exponea-track> Outbound link </a>

## Tracking examples
Here's collection of code snippets created by Exponea consultants for various use-cases. It's a good source of inpiration, but you shouldn't reproduce everything literally.
The most of Examples assume that jQuery is loaded. You can check it's existence and load it if necessary. Or learn how to track without jQuery.

### Add a product to the basket with full product details
Listens to clicks on "Add to basket" button, extracts product data and tracks "add to cart" event containing product data.

     $('#frm-productInfo-buyBoxForm-addToBasket').click(function() {
         var price = $('div.buy-info > ul.save-pay > li').text().match(/[0-9]+,?[0-9]*/);
            exponea.track('add to cart', {
                'title': $('.itemreviewed').text(),
                'price': $('p.product-price > strong').text(),
                'price-before': price[0],
                'product-availability': $('p.product-availability > span').text(),
                'product-availability-class': $('p.product-availability > span').attr('class'),
                'product-image': $('#product-images > div > ul > li > a > img:first').attr('src'),
                'product-location': document.location.pathname
            });
        });

### Track view category
Check if is on category page and tracks the category name.

        if ($('.group-product-list .group-header h1').text()) {
            exponea.track('view category', {name: $('.group-product-list .group-header h1').text()});
        }

### Identify user by email without submiting a form
Insight: The blur event is sent to an element when it loses focus ([jQuery Forms Blur docs](https://api.jquery.com/submit/)).

        $('input[type=email]').on("blur", function() {
            var exp_email = $('input[type=email]').val();
            if (exp_email.match(exp_emailRegex)) {
        		exponea.identify(exp_email);
                exponea.update({
                    'email': exp_email
                });
            }
        });

Variant without jQuery        

        document.querySelector("input[name='email']").addEventListener("blur", function(evt) {
        	if(this.value != null && this.value.trim()) {
        		exponea.track("blur", { email: this.value.trim(),
        		location: window.location.href,
        		path: window.location.pathname });
        	}
        });

### Track purchases with total price and order id

        var exp_floatRegex = /([0-9]+\.?[0-9]*)/;
        
        var exp_price = $('tr.price > td').text().replace(',', '.').match(exp_floatRegex)[0];
        var exp_orderid = $('.mt-reset strong').first().text()
        
        exponea.track('purchase', {price: exp_price, order_id: exp_orderid});

### Track view article in media publisher using Open Graph meta tag
Many CMS store document details in meta attributes. Here's how to extract them.

    exponea.track('view article', {
        'title':jQuery("meta[property='og:title']").attr('content'),
        'url':jQuery("meta[property='og:url']").attr('content'),
        'author':jQuery("meta[name='Author']").attr('content'),
        'description':jQuery("meta[name='description']").attr('content'),
        'published time':jQuery("meta[property='article:published_time']").attr('content'),
        'paywall shown':pw,
        'paywall variant':pw_variant,
      	'logged in': logged_in,
        'id': infi_article_id,
        'article tags':tags
    });;
    
### How to track UTM parameters to custom event
Session_start event contains UTM parameters. Sometimes it's handy to extract them manually.

        var utm_source;
        var utm_term;
        var utm_medium;
        var utm_campaign;
        var utm_content;
        
        window.location.search.replace(/[?&]+utm_source=([^&]*)/gi, function (str, value) {
           utm_source = decodeURIComponent(value); });
        
        window.location.search.replace(/[?&]+utm_term=([^&]*)/gi, function (str, value) {
           utm_term = decodeURIComponent(value); });
        
        window.location.search.replace(/[?&]+utm_medium=([^&]*)/gi, function (str, value) {
           utm_medium = decodeURIComponent(value); });
        
        window.location.search.replace(/[?&]+utm_campaign=([^&]*)/gi, function (str, value) {
           utm_campaign = decodeURIComponent(value); });
        
        window.location.search.replace(/[?&]+utm_content=([^&]*)/gi, function (str, value) {
           utm_content = decodeURIComponent(value); });
        
        exponea.track('view offer', {
            'utm_source':utm_source,
            'utm_term':utm_term,
            'utm_medium':utm_medium,
            'utm_campaign':utm_campaign,
            'utm_content':utm_content,
        });
    
### Dynamically load jQuery with callback
When jQuery isn't already on site, you can load it dynamically by creating custom tang.

        (function() {
            // Load the script
            var script = document.createElement("SCRIPT");
            script.src = 'https://ajax.googleapis.com/ajax/libs/jquery/1.7.1/jquery.min.js';
            script.type = 'text/javascript';
            document.getElementsByTagName("head")[0].appendChild(script);
        
            // Poll for jQuery to come into existance
            var checkReady = function(callback) {
                if (window.jQuery) {
                    callback(jQuery);
                }
                else {
                    window.setTimeout(function() { checkReady(callback); }, 100);
                }
            };
        
            // Start polling...
            checkReady(function($) {
                // Use $ here... and send data to Exponea from here
            });
        })();

## Data extraction examples

### Extracting product price
This example extracts price from a shopping cart summary, changes decimal numbers delimiter from CEE "," to US "." and gets only numbers (ignores currency sign and any other strings).

        var exponea_totalPrice = $('#basket-list > tfoot > tr > td.total > strong')
            .html().replace(",", ".").match(/[1-9]*.?[1-9]*/)[0];


