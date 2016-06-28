---
layout: default
title: "Dart on the Server"
description: "Information on server-side programming for Dart."
has-permalinks: false
---

# {{ page.title }}
{: .no_toc}

### Contents
{: .no_toc}

{% include default_toc.html %}

Dart is a great fit for server applications.

* Runs nearly everywhere
* Does nearly everything
* Easy to develop and debug
* Scales out

The Dart VM runs anywhere you can get a Linux, Windows, or Mac prompt.
It has been ported to i32, x64, ARM, ARM64, and MIPS.

You can access files, UDP and TCP sockets, HTTP, WebSockets, and more with the
[dart:io libraries][dart_io_libs]. Write a server, connect to a REST service,
parse JSON, and more.

Debugging and profiling your server app is easy with
[Observatory](http://dart-lang.github.io/observatory/). Connect
to your VM with your browser, and get real-time insight into your running
app.

Use [Isolates][isolate-docs] for secure, concurrent apps.
Spawn an isolate to run Dart
functions and libraries in an _isolated heap_, and take advantage of
multiple CPU cores.

Asynchronous code is easy to write, with the new _async/await/yield_
language features.

RPC services are easy to create and consume, with the
[rpc package](https://github.com/dart-lang/rpc).
You can write your RPC server in Dart, and generate client libraries for
Java, C#, and more.

### Examples

Build an HTTP server:

``` dart
import 'dart:io';

main() async {
  var server = await HttpServer.bind(InternetAddress.LOOPBACK_IP_V4, 8080);
  print("Serving at ${server.address}:${server.port}");

  await for (var request in server) {
    request.response
      ..headers.contentType = new ContentType("text", "plain", charset: "utf-8")
      ..write('Hello, world')
      ..close();
  }
}
```

Server WebSockets:

``` dart
import 'dart:io';
import 'dart:async';

handleMsg(msg) {
  print('Message received: $msg');
}

main() {
  runZoned(() async {
    var server = await HttpServer.bind('127.0.0.1', 4040);
    await for (var req in server) {
      if (req.uri.path == '/ws') {
        // Upgrade a HttpRequest to a WebSocket connection.
        var socket = await WebSocketTransformer.upgrade(req);
        socket.listen(handleMsg);
      };
    }
  },
  onError: (e) => print("An error occurred."));
}
```

Get HTTP resources:

``` dart
import 'package:http/http.dart' as http;

main() async {
  var contents = await http.read('http://example.com/');
  print(contents);
}
```

### Get started

Start with our [tutorials](https://www.dartlang.org/docs/tutorials/).
Once you're coding, consult the
[API documentation](https://api.dartlang.org/apidocs/channels/stable/dartdoc-viewer/home).

* [Tutorial: Write Command-Line Apps](https://www.dartlang.org/docs/tutorials/cmdline/)
* [Tutorial: Write HTTP Clients & Servers](https://www.dartlang.org/docs/tutorials/httpserver/)
{% comment %}
- Add this back if the example/text is updated.
{% endcomment %}

You may want to learn about Dart's asynchrony support.
Start with [Asynchrony support](https://www.dartlang.org/docs/dart-up-and-running/ch02.html#asynchrony),
a section in the [language tour](https://www.dartlang.org/docs/dart-up-and-running/ch02.html).
If you want more, you can deep dive into
[Dart Language Asynchrony Support: Phase 1](https://www.dartlang.org/articles/await-async/) and
[Dart Language Asynchrony Support: Phase 2](https://www.dartlang.org/articles/beyond-async/).

### Get support

Join our
[cloud@dartlang.org](https://groups.google.com/a/dartlang.org/forum/#!forum/cloud)
mailing list and chat with other engineers building
server-side Dart apps.

Post your "how-to" questions to
[Stack Overflow](http://stackoverflow.com/tags/dart).

And of course, we welcome [bug reports](http://dartbug.com/new).

### Libraries

There are lots of options for building server-side and command-line
apps with Dart. Here are just some of our favorites.

Out of the box, [Dart supports TLS/SSL](tls-ssl.html).

[dart:io][dart_io_libs] provides core functionality, such as files, directories,
sockets, HTTP, and lots more. [Shelf][shelf] is a middleware system for
web servers, with a robust plugin system. [Redstone][redstone] is a
server-side microframework for Dart. [MySQL driver][mysql] helps you
connect to MySQL from Dart. [PostgreSQL driver][postgres] helps you
connect to PostgreSQL from Dart. [Args][args] is a configurable parser of
command-line args. [Http][http] helps you easily connect to HTTP resources.

Check [pub.dartlang.org][pub] for more options and libraries, including
ORM, Redis, Mongo, authentication, Google APIs, and more.

### Learn more

Watch this video, recorded at the
[Dart Developer Summit](https://www.dartlang.org/events/2015/summit/),
about moving from Node.js to Dart.

<iframe width="640" height="360" src="https://www.youtube.com/embed/NHsmiY0rFS8?list=PLOU2XLYxmsIIQorIS8gagUiMau9S84vZV&amp;showinfo=0" frameborder="0" allowfullscreen></iframe>

Then, dive deep with our [articles](https://www.dartlang.org/articles/), examples,
and other documentation.

* [Beware the Nest o' Pirates: Write a Server App](codelab/)
  shows you how to use the RPC package to write a RESTful server
  that you can message from the command line or from a client app.
* [Articles on a variety of Dart subjects](https://www.dartlang.org/articles/),
   many written by Dart engineers
* [Dart by Example][examples] has tons of small code snippets for servers
  and consoles

### Hosting Dart servers

You have many options. Here are a few:

* Google Cloud Platform; for details on Dart support, see
  [Dart and Google Cloud Platform](google-cloud-platform)
* [Heroku](https://github.com/igrigorik/heroku-buildpack-dart)  
* [Sourcevoid](https://www.sourcevoid.com/)

In addition, Dart runs on most cloud providers, such as
Amazon's EC2, Digital Ocean, and more.

[dart_io_libs]: https://api.dartlang.org/apidocs/channels/stable/dartdoc-viewer/dart:io
[isolate-docs]: https://api.dartlang.org/apidocs/channels/stable/dartdoc-viewer/dart:isolate
[shelf]: https://pub.dartlang.org/packages/shelf
[redstone]: http://redstonedart.org/
[mysql]: https://pub.dartlang.org/packages/sqljocky
[postgres]: https://pub.dartlang.org/packages/postgresql
[pub]: https://pub.dartlang.org
[args]: https://pub.dartlang.org/packages/args
[http]: https://pub.dartlang.org/packages/http
[examples]: https://www.dartlang.org/dart-by-example/#dart-io-and-command-line-apps
