# url.js
a minimalistic light-weight javascript library for managing url and hash changes events for mono-page (or multi-page) apps that works well with the browser history and back button retrocompatible with fallback code.

Here is a very simple example with a web app with 3 sections: home, products and contact made with url.js:

```javascript
     $each([$html, $fetch], {default: $id("contents")}); // $html/$fetch def outputs to #contents
     $html("<a href='#'>🏠</a> <a href='#products'>🛒</a> <a href='#contact'>📩</a>", $id("menu"));

     $url.onenter(['', 'home'], () => { // #home is also '' (default section)
       $html("Welcome, this is home section.");
     });

     $url.onenter('products', () => {
       $html("Products section. <input type='checkbox' id='unsaved' checked> Unsaved changes");
     });
     $url.onready('products', () => {
       // use onready to perform some task once the section DOM is completely ready, if necessary...
     });
     $url.onexit('products', () => {
       if ($id("unsaved").checked) return false; // if products are not saved avoid page change
     });

     $url.onenter('contact', () => {
       $fetch("./contact.html");
     });
```
(this example uses expressions from https://github.com/iagoFG/alias.js library)

## Usage

Once loaded url.js file you can use following API calls:

### $url.onenter(hash, listener)

Setup a event listener which will be called whenever a new/different hash is pushed into the url. Parameters and listener use the same format than $url.onexit.

   Parameters:
   * _hash_ the hash for which the listener will be called, an array of hashes, or "\*" if this listener must be called whichever the hash value.
   * _listener_ a function which will be called when the hash is detected. You can use named, unnamed or lambda syntax.
 
### $url.onready(hash, listener)

Setup a event listener which will be called whenever a new hash is pushed into the url and all the onenter callbacks has been called. Same parameters and listener than $url.onexit.

### $url.onexit(hash, listener)

Setup a event listener which will be called whenever another hash has been pushed into the url, before any onenter or onready listener is called (this event usually means that screen is going to change and asks you if you want to cancel this change, for example if there are unsaved changes on current page). Also will be called if the page is unloading, for example the beforeunload or unload browser events were detected.

### $url.go(url)

Navigate to the specified url. Whenever you can use this function as long as it reports the url.js core traceability about the navigation made. However of course you can use <a href="url">...</a>, form or window.location = url classic navigation methods.

### $url.back(delta)

Equivalent to window.history.back, however it reports the url.js core traceability about the navigation made.

### $url.replace(url)

Similar to $url.go, however it does not add a navigation history entry... it swaps the current page url only. It is equivalent to window.history.replaceState.

### listener

There are several calls ($url.onexit, $url.onenter and $url.onready) that use a listener. All the listener functions will receive the same following parameters:
 * _hash_ will be the hash for which the listener was invoked, useful when the same listener is used for various or all the possible hash values.
 * _delayedcb_ will be a delayed return callback function. This argument will be null/undefined/false sometimes, when listener cannot delay the return value, for example if the events were triggered from beforeunload browser event.

Listener will return always true|false reporting if the change is allowed or not. The value can be returned in two ways:
 * using directly return true or return false.
 * using delayed return callback (the delayedcb argument). You must check before if delayedcb is indeed defined as long as sometimes delayed return is not allowed. This is an example:

```javascript
     function listener(hash, delayedcb) {
       ...
       if (delayedcb) {
         otherfunction(function(toreturnvalue) {
           delayedcb(toreturnvalue);
         });
         return delayedcb;
       } else {
         return true;
       }
       ...
     }
```
This will return true if cannot be delayed or will call to otherfunction and use it async returned value to feedback the return value asynchronously to url.js calling to delayedcb.

