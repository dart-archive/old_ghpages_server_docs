---
layout: default
title: "The Server Code Explained"
short-title: "Server Code"
description: "Learn about the appengine APIs used by the server"
---

# {{ page.title }}
{: .no_toc}

### Contents
{: .no_toc}

{% include default_toc.html %}

This section highlights certain methods from the server code
and describes the lines that are related to App Engine
and its services.

## Accessing the Cloud Database service {#accessing-datastore}

You can use `context.services` to get access
to the App Engine services, such as
Cloud Database, memcache and so on.
For example, here's how to get a handle
on the Cloud Datastore service:

```dart
context.services.db
```

From this handle, you can call the service's methods, such as `commit()`
and `query()`.

```dart
context.services.db.query(...);
```

## Running runAppEngine()

The `main()` function calls `runAppEngine()` to start App Engine
and provides a method, `requestHandler()` that App Engine calls
for each HTTP request from the client.

```dart
main() {
  runAppEngine(requestHandler);
}
```

## Serving assets

The `requestHandler()` method triages the incoming client requests
based on the URI path and handles the special ones accordingly.
If the URI path does not match any of those tested for,
the program calls the bolded line of code, which
serves client files, such as `index.html`.
In fact, some of the special URI paths are processed
and then redirected to `index.html`.

<pre>
void requestHandler(HttpRequest request) {
  if (request.uri.path == '/items') {
    handleItems(request);
  } else if (request.uri.path == '/clean') {
    handleClean(request).then((_) {
      request.response.redirect(Uri.parse('/index.html'));
    });
  } else if (request.uri.path == '/') {
    request.response.redirect(Uri.parse('/index.html'));
  } else {
    <b>context.assets.serve();</b>
  }
}
</pre>

The `context.assets.serve();` method fetches the assets either from `pub
serve` or from the web directory
depending on whether the option `--dart-pub-serve` is
passed to `gcloud app run`.
This includes compiled JavaScript when running in non-Dart browsers.

When not using `pub serve` no transformations occur and only
simple client assets work.

## Committing items to the Cloud Datastore

Use the `commit()` method to commit an item to the datastore.
The `commit()` method returns a
<a href="http://api.dartlang.org/stable/dart-async/Future-class.html">Future</a>
to perform the operation asynchronously.

<pre>
handleItems(HttpRequest request) {
  if (request.method == 'GET') {
    // Get items from datastore using the readItems method;
    // send them to client in JSON format.
    ...
  } else if (request.method == 'POST') {
    // Validate request, then commit new item to the datastore.
    ...
        return <b>context.services.db.commit</b>(inserts: [item]).then((_)
  ...
  }   
}       
</pre>

The server's `handleClean()` method uses the `commit()` method to delete
all items from the list:

<pre>
Future handleClean(HttpRequest request) {
  return readItems().then((items) {
    var deletes = items.map((item) => item.key).toList();
    return <b>context.services.db.commit(deletes: deletes);</b>
  });
}
</pre>

## Running a query

The code in the server's `readItems()` method
creates a query to retrieve all items in the datastore
associated with the application running from this origin.
It then calls `query.run()`, which returns a Stream to perform
the operation asynchronously.

<pre>
Future&lt;List&lt;Item&gt;&gt; readItems() {
  // Get items from datastore.
  var query = <b>context.services.db.query</b>(
      Item, ancestorKey: rootKey())..order('name');
  return <b>query.run.toList</b>();
}       
</pre>

The `order()` methods sorts the returned list in alphabetical order.

## Using the rootkey

The `rootKey` method is called by `handleItems()` and `readItems()` to
add items to the cloud database.
`ItemsRoot` is defined in the client's `Model.dart` file
and identifies the entity group in which to put the item.

<pre>
rootKey() {
  var rootKey = <b>context.services.db.emptyKey.append(ItemsRoot, id: 1);</b>
}
</pre>

## What next?

It's time to deploy your application:
<a href="../deploy.html">Deploy a Dart Application on App Engine</a>.
