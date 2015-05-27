# TurboGraft
Turbograft extends [Turbolinks](https://github.com/rails/turbolinks), allowing you to perform partial page refreshes.

`Graft` - (noun) a shoot or twig inserted into a slit on the trunk or stem of a living plant, from which it receives sap.

In botony, one can take parts of a tree and splice it onto another tree.  The DOM is a tree.  In this library, we're cutting off sub-trees of the DOM and splicing new ones on.

 Turbolinks works by intercepting navigation requests and loading them via Ajax when possible, swapping the body tag of the document with the newly loaded copy. Turbograft builds on this to allow you to perform a partial page refresh on specified DOM nodes by adding a refresh key. This allows you reduce page load time, while the feeling of a native, single-page application.

## One render path
Turbograft gives you the ability to maintain a single, canonical render path for views. Your ERB views are the single definition of what will be rendered, without the worry of conditionally fetching snippets of HTML from elsewhere. This approach leads to clear, simplified code.
## Client-side performance
Partial page refreshes mean that CSS and JavaScript are only reloaded when you need them to be. Turbograft improves on the native, single-page application feel for the user while keeping these benefits inherited from Turbolinks.
## Simplicity
Turbograft was built with simplicity in mind. It intends to offer the smallest amount of overhead required on top of a traditional Rails stack to solve the problem of making a Rails app feel native to the browser.

## Status
[![Gem Version](https://badge.fury.io/rb/turbograft.svg)](http://badge.fury.io/rb/turbograft)
[![Build Status](https://travis-ci.org/Shopify/turbograft.svg?branch=master)](http://travis-ci.org/Shopify/turbograft)

## Installation

* Remove `gem "turbolinks"` from your Gemfile
* Add `gem "turbograft"` to your Gemfile
* Run `bundle install`
* Remove `//= require turbolinks` from _app/assets/javascripts/application.js_
* Add `//= require turbograft` to _app/assets/javascripts/application.js_

## Usage
### Partial page refresh

```html
<div id="content" refresh="page">
  ...
</div>
```


```html
<a href="#" id="partial-refresh-page" refresh="page" onclick="event.preventDefault(); Page.refresh({url: '<%= page_path(@next_id) %>',onlyKeys: ['page']});">Refresh the page</a>
```

This performs a `GET` request, but our client state is maintained. Using the refresh attribute, we tell TurboGraft to grab the new page, but only refresh elements where refresh="page".  This is the lowest-level way to use TurboGraft.

`refresh` attributes on your DOM nodes can be considered somewhat analoguous to how `class` will apply styles to any nodes with that class.  That is to say, many nodes can be decorated `refresh="foo"` and all matching nodes will be replaced with `onlyKeys: ['foo']`.  Each node with `refresh` must have its own unique ID (this is how nodes are matched during the replacement stage).  At the moment, `refresh` does not support multiple keys (e.g., `refresh="foo bar"`) like the `class` attribute does.

### onlyKeys
You can specify multiple refresh keys on a page, and you can tell TurboGraft to refresh on one or more refresh keys for a given action.

```html
<button id='refresh-a-and-b' href="<%= page_path(@id) %>" onclick="event.preventDefault(); Page.refresh({url: '<%= page_path(@id) %>', onlyKeys: ['section-a', 'section-b']});">Refresh Section A and B</button>
```

### exceptKeys
You can also tell TurboGraft to refresh the page, but exclude certain elements from being refreshed.

```html
<button id='refresh-a-and-b' href="<%= page_path(@id) %>" onclick="event.preventDefault(); Page.refresh({url: '<%= page_path(@id) %>', exceptKeys: ['section-a', 'section-b']});">Refresh everything but Section A and B</button>
```

### refresh-never
The `refresh-never` attribute will cause a node only appear once in the `body` of the document. This can be used to include and initialize a tracking pixel or script just once inside the body.

```html
<div refresh-never>
  <%= link_to "Never refresh", page_path(@next_id), id: "next-page-refresh-never", refresh: "page" %>
</div>
```

## tg-remote

The `tg-remote` option allows you to query methods on or submit forms to different endpoints, and gives partial page replacement on specified refresh keys depending on the response status.

It requires your `<form>`, `<a>`, or `<button>` to be marked up with:

* `tg-remote`: (optionally valueless for `<form>`, but requires an HTTP method for links) the HTTP method you wish to call on your endpoint
* `href`: (if node is `<a>` or `<button>`) the URL of the endpoint you wish to hit
* `refresh-on-success`: (optional) The refresh keys to be refreshed, using the body of the response. This is space-delimited
* `full-refresh-on-success-except`: (optional) Replaces body except for specififed refresh keys, using the body of the XHR which has succeeded
* `refresh-on-error`: (optional) The refresh keys to be refreshed, but using body of XHR which has failed. Only works with error 422. If the XHR returns and error and you do not supply a refresh-on-error, nothing is changed
* `full-refresh-on-error-except`: (optional) Replaces body except for specified refresh keys, using the body of the XHR which has failed.  Only works with error 422
* `remote-once`: (optional) The action will only be performed once. Removes `remote-method` and `remote-once` from element after consumption
* `full-refresh`: Rather than using the content of the XHR response for partial page replacement, a full page refresh is performed. If `refresh-on-success` is defined, the page will be reloaded on these keys. If `refresh-on-success` is not defined, a full page refresh is performed. Defaults to true if neither refresh-on-success nor refresh-on-error are provided
* `tg-remote-norefresh`: Prevents `Page.refresh()` from being called, allowing methods to be executed without updating client state

Note that as `refresh-on-*` pertains to partial refreshes and `full-refresh-on-*-except` pertains to full refreshes, they are incompatible with each other and should not be combined.

### Examples

Call a remote method:

```html
<a href="#" tg-remote="post" refresh-on-success="page section-a section-b">Remote-method</a>
```

The Rails way:

```erb
<%= link_to "Remote method", method_path, 'refresh-on-success' => 'page section-a section-b', 'full-refresh' => 'true', 'tg-remote' => 'post' %>
```

Post to a remote form:

```html
<div id="results" refresh="results">
  Use the field below to submit some content, and get a result.
</div>
<form tg-remote="" action="/pages/submit" method="post" refresh-on-success="results" refresh-on-error="results">
  <input name="form-input" type="text">
  <button type="submit">Submit</button>
</form>
```

### tg-remote events

* `turbograft:remote:init`: Before XHR is sent
* `turbograft:remote:start`: When XHR is sent
* `turbograft:remote:always`: Always fires when XHR is complete
* `turbograft:remote:success`: Always fires when XHR was successful
* `turbograft:remote:fail`: Always fires when XHR failed
* `turbograft:remote:fail:unhandled`: Fires after `turbograft:remote:fail`, but when no partial replacement with refresh-on-error was performed (because no `refresh-on-error` was supplied)

Each event also is sent with a copy of the XHR, as well as a reference to the element that initated the `remote-method`.

### tg-static

With the `tg-static` attribute decorating a node, we can make sure that this node is not replaced during a fullpage refresh.  Contrast this to partial page refreshes, where we normally specify the set of elements that need to change.  With `tg-static`, we can define a set of elements (by annotating them with this attribute) that must never change.

The internal state of any nodes marked with `tg-static` will remain, even though the entire page has been swapped out.  A partial page refresh with `onlyKeys` targeting a node inside of the `tg-static` node is also possible, persisting your static element but swapping the innards.

Though, if you were to refresh the page at a higher level -- e.g., refreshing an ancestor of the `tg-static`, the static aspect is no longer obeyed and it is replaced!

Examples of where this may be useful include:

- running `<video>` or `<audio>` element
- a client-controlled static nav

### refresh-always

For the lazy developer in all of us, we can use the attribute `refresh-always` when we want to be sure we've absolutely replaced a certain element, if it exists.  An example of such a node you may want to apply this might be an unread notification count -- always being sure to update it if it exists in the response.

### tg-remote-noserialize

When serializing forms for tg-remote calls, turbograft will check to ensure inputs meet the following criteria:

- the input has a `name` attribute
- the input does not have the `disabled` attribute

and

- the input does not have the `tg-remote-noserialize` attribute
- no ancestor of the input has the `tg-remote-noserialize` attribute

The `tg-remote-noserialize` is useful in scenarios where a whole section of the page should be editable, i.e. not `disabled`, but should only conditionally be submitted to the server.

## Example App

There is an example app that you can boot to play with TurboGraft.  Open the console and network inspector and see it in action!  This same app is also used in the TurboGraft browser testing suite.

```
./server
```

## jQuery and memory leaks

When turbograft replaces or removes a node it uses native DOM API to do so. If any objects use jQuery to listen to events on a node then these objects will leak when the node is replaced because jQuery will still have references to it. To clean these up you'll need to tell jQuery that they're removed. This can be done with something like:

```coffeescript
document.addEventListener 'page:after-node-removed', (event) ->
  $(event.data).remove()
```

## Contributing

1. Fork it ( http://github.com/Shopify/turbograft/fork )
2. Create your feature branch (`git checkout -b my-new-feature`)
3. Commit your changes (`git commit -am 'Add some feature'`)
4. Write some tests, and edit the example app
5. Push to the branch (`git push origin my-new-feature`)
6. Create new Pull Request

## Testing

- `./server` and visit http://localhost:3000/teaspoon to run the JS test suite
- `bundle exec rake test` to run the browser test suite
