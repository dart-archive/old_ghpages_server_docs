---
layout: default
title: "Beware the Nest o' Pirates: Write a Server App"
description: "Learn how you can easily write a RESTful server using Dart."
has-permalinks: true
---

# {{ page.title }}

Learn how to create a RESTful web server using the
Dart language and libraries, featuring the
[RPC package](https://github.com/dart-lang/rpc).
In this code lab, you create a server with API methods exposed as
GET, POST, and DELETE requests.  Once the server's API is complete,
you use it to generate the corresponding API for a Dart client app.
Finally, you extend the server using the HTTP server library
to serve the Dart client directly.

This code lab assumes that you are comfortable reading Dart code.
If you'd like a more thorough introduction to the Dart language, see
the [client code lab](https://www.dartlang.org/codelabs/darrrt/) or the
[language tour](https://www.dartlang.org/docs/dart-up-and-running/ch02.html).

**This app lets you store a list of pirates to a Dart server. Try it!**

<iframe class="running-app-frame"
        style="width:900px;height:410px;"
        src="https://pirates-nest.appspot.com/piratebadge.html">
</iframe>

<hr>

<div class="piratemap" markdown="1" style="min-height:325px">

## Map

* [Step 0: Set up](#set-up)
* [Step 1: View the server code](#step-one)
* [Step 2: Annotate the server API](#step-two)
* [Step 3: Launch the server](#step-three)
* [Step 4: Extend the server API](#step-four)
* [Step 5: Generate the client API](#step-five)
* [Step 6: Connect the client to the server](#step-six)
* [Step 7: Serve the client from the server](#step-seven)
* [What next?](#whatnext)
* [Summary](#summary)

</div>

<hr>
<p>&nbsp;</p>

## Step 0: Set up {#set-up}

In this step, you download Dart and get the sample code.

### &#9875; Get Dart.

<div class="trydart-step-details" markdown="1">
If you haven't already done so,
[download the Dart SDK](https://www.dartlang.org/downloads/).
Unzip the archive, which creates a directory called `dart-sdk`.

The Dart SDK download includes several Dart
tools that you'll need, incuding `dart`, `pub`, and `dartanalyzer`.
If `dart` is not recognized at the command line,
add `<path-to-the-SDK>/dart-sdk/bin` to your path.

<aside class="alert alert-info" markdown="1">
**Note:**
This code lab works with any IDE or editor, but the instructions
assume that you're using the command line.
</aside>

</div>

### &#9875; Get Dartium.

<div class="trydart-step-details" markdown="1">
You need a way to test the client app.
You have two options. You can:

* Compile the app to JavaScript and open the page in any browser.
* Open the app in Dartium.

If you prefer the latter option, and you don't have Dartium already,
[download Dartium](https://www.dartlang.org/downloads/archive/).
Note that the executable is called Chromium on your file system.
</div>

### &#9875; Get the sample code.

<div class="trydart-step-details" markdown="1">
Download the sample code from the
[one-hour-codelab GitHub repo](https://github.com/dart-lang/one-hour-codelab)
using one of the following options:

<ul markdown="1">
<li markdown="1">
  Download the ZIP file,
  [one-hour-codelab-master.zip](https://github.com/dart-lang/one-hour-codelab/archive/master.zip).
  Unzip the ZIP file, which creates a directory called
  `one-hour-codelab-master`.
</li>

<li markdown="1">
  Clone the repo. For example, from the command line:

```
git clone https://github.com/dart-lang/one-hour-codelab.git
```

This creates a directory named `one-hour-codelab`.
</li>
</ul>

</div>

### &#9875; Look at the one-hour-codelab sample.

<div class="trydart-step-details" markdown="1">
The `server` directory contains the following files and directories:

`1-starter`
: Basic server (does not run)

`2-simple`
: Supports a few simple methods using GET requests

`3-*`
: Intentionally omitted as Step 3 does not require additional code

`4-extended`
: Adds more advanced methods including support for POST, and DELETE requests

`5-generated`
: Includes the generated client API

`6-client`
: Includes the PirateBadge client app,
  modified to use the server for storing and retrieving pirate names

`7-serve`
: Includes the final version of the server,
  modified to serve the client app

`working-dir`
: A work area that initially contains directories and files from `1-starter`

</div>

### &#9875; Get the curl utility.

<div class="trydart-step-details" markdown="1">
You need a way to test the server.
You can do some of the testing in a browser,
but Step 4 requires a tool, such as curl, that lets you construct
POST and DELETE requests.

_curl_ is a tool used for transferring data to or from a server using
a supported protocol, such as HTTP, FTP, or IMAP.
(Other tools, such as wget, perform the same function, but
this code lab uses curl.)
curl is already installed on most Macs and many Linux distributions.

If you don't have curl, go to the
[curl download](http://curl.haxx.se/download.html) page,
scroll down until you find the right version for your OS,
and then download and install it.
On Linux, you might be able to get curl by running:

```
apt-get install curl
```
</div>

## Step 1: View the server code {#step-one}

In this step, you open the source files for the server and familiarize
yourself with the code. Note that this version of the server doesn't run.
You'll fix the code in [Step 2](#step-two).

<div class="trydart-note" markdown="1">
<strong>Note:</strong>
Throughout this code lab, edit the files in `working-dir`.
You can use the files in the numbered directories to compare to your code
or to recover if you get off track.
</div>

<div class="trydart-step-details" markdown="1">
Initially, the `working-dir` files and directories are identical to
those in `1-starter`:

![The 1-starter directory structure](images/server-directory-structure.png)

`messages.dart`
: Specifies the message format used between client and server.

`piratesapi.dart`
: Specifies the methods that the server makes remotely available.

`piratesnest.dart`
: Contains a `main()` function, which implements the main server entrypoint.

`utils.dart`
: Contains utility methods for generating and validating pirate names.

</div>

### &#9875; Walk through the code.

<div class="trydart-step-details" markdown="1">
Get familiar with the server code.

#### lib/server/piratesapi.dart

<pre>
library pirate.server;

import 'package:rpc/api.dart';

import '../common/messages.dart';
import '../common/utils.dart';

// This class defines the interface that the server provides.
class PiratesApi {
  final Map&lt;String, Pirate&gt; <b>_pirateCrew</b> = {};
  final PirateShanghaier <b>_shanghaier</b> =
      new PirateShanghaier();

  PiratesApi() {
    var captain = new Pirate.fromString('Lars the Captain');
    _pirateCrew[captain.toString()] = captain;
  }

  // Returns a list of the pirate crew.
  List&lt;Pirate&gt; <b>listPirates()</b> {
    return _pirateCrew.values.toList();
  }

  // Generates (shanghais) and returns a new pirate.
  // Does not add the new pirate to the crew.
  Pirate <b>shanghaiAPirate()</b> {
    var pirate = _shanghaier.shanghaiAPirate();
    if (pirate == null) {
      throw new InternalServerError('Ran out of pirates!');
    }
    return pirate;
  }
}
</pre>

<div class="trydart-filename">lib/server/piratesapi.dart</div>

<p>&nbsp;</p>
&#128273; <strong> Key information </strong>

* The server maintains a list (stored as a Map) of pirates, `_pirateCrew`,
  which constitutes the "nest of pirates." This list initially contains
  only "Lars the Captain".

* The server creates `_shanghaier`, an instance of PirateShanghaier,
  which handles the generation of pirate names.
   Generating a pirate name is referred to as "shanghaiing a pirate".

* The PiratesApi class defines two methods:
  `listPirates()` returns the `_pirateCrew` as a list;
  `shanghaiAPirate()` returns a new Pirate object with a valid pirate name
   (example: Anne the Brave).

#### bin/piratesnest.dart

<pre>
library piratesnest;

import 'dart:io';

import 'package:logging/logging.dart';
import 'package:rpc/rpc.dart';

import 'package:server_code_lab/server/piratesapi.dart';

<b>final ApiServer _apiServer = new ApiServer(prettyPrint: true);</b>

<b>main()</b> async {
  // Add a bit of logging...
  <b>Logger.root..level = Level.INFO</b>
             <b>..onRecord.listen(print);</b>

  // Set up a server serving the pirate API.
  <b>_apiServer.addApi(new PiratesApi());</b>
  <b>HttpServer server =</b>
      <b>await HttpServer.bind(InternetAddress.ANY_IP_V4, 8088);</b>
  <b>server.listen(_apiServer.httpRequestHandler);</b>
  <b>print('Server listening on http://${server.address.host}:'</b>
        <b>'${server.port}');</b>
}
</pre>

<div class="trydart-filename">bin/piratesnest.dart</div>

<p>&nbsp;</p>
&#128273; <strong> Key information </strong>

* The RPC package requires you to create an `ApiServer` instance for
  routing the HTTP requests to your methods.

* This file contains the entrypoint for the server&mdash;the `main()`
  function. Running this file starts up the server.

* After creating an instance of HttpServer and binding it to port 8088,
  the server listens to that port using the API specified in the PiratesApi
  class.

* The logging code generates print statements for each
  message that the server handles.

#### lib/common/utils.dart

<pre>
library pirate.utils;

import 'dart:math' show Random;

import 'messages.dart';

// Proper pirate names.
const List&lt;String&gt; <b>pirateNames</b> = const [
  "Anne", "Bette", "Cate", "Dawn", "Elise", "Faye", "Ginger",
  "Harriot", "Izzy", "Jane", "Kaye", "Liz", "Maria", "Nell",
  "Olive", "Pat", "Queenie", "Rae", "Sal", "Tam", "Uma",
  "Violet", "Wilma", "Xana", "Yvonne", "Zelda", "Abe",
  "Billy", "Caleb", "Davie", "Eb", "Frank", "Gabe", "House",
  "Icarus", "Jack", "Kurt", "Larry", "Mike", "Nolan",
  "Oliver", "Pat", "Quib", "Roy", "Sal", "Tom", "Ube",
  "Val", "Walt", "Xavier", "Yvan", "Zeb"
];

// Proper pirate appellations.
const List&lt;String&gt; <b>pirateAppellations</b> = const [
  "Awesome", "Captain", "Even", "Fighter", "Great",
  "Hearty", "Jackal", "King", "Lord", "Mighty",
  "Noble", "Old", "Powerful", "Quick", "Red",
  "Stalwart", "Tank", "Ultimate", "Vicious",
  "Wily", "aXe", "Young", "Brave", "Eager",
  "Kind", "Sandy", "Xeric", "Yellow", "Zesty"
];

// Clearly invalid pirate appellations.
const List&lt;String&gt; <b>_forbiddenAppellations</b> = const [
  '', 'sweet', 'handsome', 'beautiful', 'weak', 'wuss',
  'chicken', 'fearful'
];

// Helper method for validating whether the given pirate is truly a pirate!
bool <b>truePirate(Pirate pirate)</b> => pirate.name != null &&
    pirate.name.trim().isNotEmpty &&
    pirate.appellation != null &&
    !_forbiddenAppellations
        .contains(pirate.appellation.toLowerCase());

// Shared class for shanghaiing (generating) pirates.
class PirateShanghaier {
  static final Random indexGen = new Random();

  Pirate <b>shanghaiAPirate({String name, String appellation})</b> {
    var pirate = new Pirate();
    pirate.name = name != null
        ? name
        : pirateNames[indexGen.nextInt(pirateNames.length)];
    pirate.appellation = appellation != null
        ? appellation
        : pirateAppellations[
        indexGen.nextInt(pirateAppellations.length)];
    return truePirate(pirate) ? pirate : null;
  }
}
</pre>
<div class="trydart-filename">lib/common/utils.dart</div>

<p>&nbsp;</p>
&nbsp;&#128273; <strong> Key information </strong>

* Servers and other command-line apps generally rely on the dart:io library.
  Client apps generally rely on the dart:html library.
  `utils.dart` doesn't rely on dart:io or dart:html and can be used by both
  the server and the client, so it lives in `lib/common/`.

* The `pirateNames` list contains first names (like "Anne" and "Mike");
  `pirateAppellations` contains labels (like "Old" and "Mighty").
  These are used to generate valid pirate names ("Anne the Mighty").

* Thanks to the `truePirate()` function, the server rejects any pirate name
  that uses one of the `_forbiddenAppellations`;
  for example "Markus the Wuss" is not allowed.

#### lib/common/messages.dart

<pre>
library pirate.messages;

// This class is used to send data back and forth between the
// client and server. It is automatically serialized and
// deserialized by the RPC package.
<b>class Pirate</b> {
  String name;
  String appellation;

  // A message class must have a default constructor taking no
  // arguments.
  <b>Pirate()</b>;

  // It is fine to have other named constructors.
  <b>Pirate.fromString(String pirateName)</b> {
    var parts = pirateName.split(' the ');
    name = parts[0];
    appellation = parts[1];
  }

  String toString() => name.isEmpty ? '' : '$name the $appellation';
}
</pre>
<div class="trydart-filename">lib/common/messages.dart</div>

<p>&nbsp;</p>
&nbsp;&#128273; <strong> Key information </strong>

* `messages.dart` specifies the message format used between client and server.
  Both apps share the code, which is possible because it does not rely
  on dart:io or dart:html.

* Besides defining properties, the Pirate class can convert Pirate objects
  to and from their string components.

</div>

## Step 2: Annotate the server API {#step-two}

In this step, you add annotations to the server's API so the RPC library
understands which methods are to be exposed as remotely available.

<div class="trydart-step-details" markdown="1">

### &#9875; Run pub get.

The pub package manager makes sure that you have all the packages that
the app needs.

```
pub get
```

### &#9875; Edit lib/server/piratesapi.dart.

Add an `@ApiClass` annotation to the `PiratesApi` class.
Add `@ApiMethod` annotations to the `listPirates()` and
`shanghaiAPirate()` methods.

<pre>
library pirate.server;

import 'package:rpc/api.dart';

import '../common/messages.dart';
import '../common/utils.dart';

// This class defines the interface that the server provides.
<b>@ApiClass(version: 'v1')</b>
class PiratesApi {
  final Map&lt;String, Pirate&gt; _pirateCrew = {};
  final PirateShanghaier _shanghaier = new PirateShanghaier();

  PiratesApi() {
    var captain = new Pirate.fromString('Lars the Captain');
    _pirateCrew[captain.toString()] = captain;
  }

  // Returns a list of the pirate crew.
  <b>@ApiMethod(path: 'pirates')</b>
  List&lt;Pirate&gt; listPirates() {
    return _pirateCrew.values.toList();
  }

  // Generates (shanghais) and returns a new pirate.
  // Does not add the new pirate to the crew.
  <b>@ApiMethod(path: 'shanghai')</b>
  Pirate shanghaiAPirate() {
    var pirate = _shanghaier.shanghaiAPirate();
    if (pirate == null) {
      throw new InternalServerError('Ran out of pirates!');
    }
    return pirate;
  }
}
</pre>
<div class="trydart-filename">lib/server/piratesapi.dart</div>

<p>&nbsp;</p>
&nbsp;&#128273; <strong> Key information </strong>

* The [RPC](https://github.com/dart-lang/rpc) package allows
  you to create RESTful server-side Dart APIs.

* Identify the top-level API class with the `ApiClass` annotation
  to expose this class as a public API. A version number is required.

* Identify remotely accessible methods with the `ApiMethod` annotation.
  Unless otherwise specified with the optional `method` parameter,
  all methods are assumed to send GET requests.

* The required `path` parameter in the `ApiMethod` annotation specifies
  how to invoke the method via an HTTP request, as you will see in
  [Step 3](#step-three).

</div>

## Step 3: Launch the server {#step-three}

In this step, you start up the server and make an HTTP request to
verify that it's running.

<div class="trydart-step-details" markdown="1">

### &#9875; Check the code.

Use `dartanalyzer` to check the code for errors
(unless you're using a tool, such as WebStorm,
that automatically analyzes code).
To run [dartanalyzer](https://github.com/dart-lang/analyzer_cli#dartanalyzer)
from the command line,
do the following from the `working-dir` directory:

```
dartanalyzer lib/server/piratesapi.dart
```

You should see `No issues found.` If the code has problems, you
can fix it or use the code from `2-simple`.

### &#9875; Start up the server.

Use the `dart` command to launch the server:

```
dart bin/piratesnest.dart
```

You should see the following output:

```
[INFO] rpc: Adding /piratesApi/v1 to set of valid APIs.
Server listening on http://0.0.0.0:8088
```

&nbsp;&#128273; <strong> Key information </strong>

<ul markdown="1">
<li markdown="1">The `INFO` message is displayed by the logging code.
</li>

<li markdown="1"> When you start up the server,
  you might see a dialog asking, 'Do you want the application "dart"
  to accept incoming network connections?' This dialog,
  from the computer's firewall, is asking if you want to be able to
  connect to the Dart server from a different computer. Since you
  are using localhost, you can "Deny" the request and it will still work.
</li>

<li markdown="1"> If you are running the server from an IDE,
  you might also see:

```
Observatory listening on http://127.0.0.1:53264
```

  Observatory is Dart's profiling tool&mdash;you can ignore this message.
</li>
</ul>

### &#9875; Test the server.

You can test the server in a browser, or by using the curl utility.

<aside class="alert alert-info" markdown="1">
**Note:**
Open a new shell to run any curl commands. Logging messages
appear in the shell where you launched the server.
</aside>

To make an HTTP request to the server, first construct a URL. For example:

<img src="images/url.png" width="75%" alt="a sample HTTP request" />

The server listens on `localhost:8088`, as specified in the
`main()` method. The class name, version,
and method name are specified in the ApiClass.

You can test the server either in a browser, or by using curl.

For example, to request the list of pirates from the server,
do one of the following:

<ul markdown="1">
<li markdown="1">Navigate to
  [http://localhost:8088/piratesApi/v1/pirates](http://localhost:8088/piratesApi/v1/pirates)
  in a browser.
</li>

<li markdown="1">Type the following at the command line:

```
curl http://localhost:8088/piratesApi/v1/pirates
```
</li>
</ul>

Initially, the crew contains only the captain:

```
[
 {
  "name": "Lars",
  "appellation": "Captain"
 }
]
```

The window running the server displays logging messages for
this GET command.

&nbsp;&#128273; <strong> Key information </strong>

<ul markdown="1">
<li markdown="1"> Don't be alarmed if you see the following warning
when messaging the server from the browser:

```
[WARNING] rpc:
Response
  Status Code: 400
  ...
  RPC Error with status: 400 and message: Invalid request, missing API name
  and/or version: http://localhost:8088/favicon.ico.
```
</li>
</ul>

</div>

## Step 4: Extend the server API {#step-four}

In this step, you extend the server API to support POST
and DELETE requests so that you can store and delete pirate names
in the list of pirates.

<div class="trydart-step-details" markdown="1">

### &#9875; Edit lib/server/piratesapi.dart.

To support adding a pirate,
add a `hirePirate()` method to the PiratesApi class.

<pre>
// This class defines the interface that the server provides.
@ApiClass(version: 'v1')
class PiratesApi {
  ...
  <b>@ApiMethod(method: 'POST', path: 'pirate')</b>
  <b>Pirate hirePirate(Pirate newPirate) {</b>
    <b>// Make sure this is a real pirate...</b>
    <b>if (!truePirate(newPirate)) {</b>
      <b>throw new BadRequestError(</b>
          <b>'$newPirate cannot be a pirate. \'Tis not a pirate name!');</b>
    <b>}</b>
    <b>var pirateName = newPirate.toString();</b>
    <b>if (_pirateCrew.containsKey(pirateName)) {</b>
      <b>throw new BadRequestError(</b>
          <b>'$newPirate is already part of your crew!');</b>
    <b>}</b>

    <b>// Add the pirate to the crew.</b>
    <b>_pirateCrew[pirateName] = newPirate;</b>
    <b>return newPirate;</b>
  <b>}</b>
  ...
}
</pre>
<div class="trydart-filename">lib/server/piratesapi.dart</div>

<p>&nbsp;</p>
&#128273; <strong> Key Information </strong>

* Annotating the method with `ApiMethod` exposes it as available remotely.

* The optional `method` parameter identifies the request type:
  `hirePirate` is a POST request.

* The required `path` parameter specifies how to access this method
  from an HTTP request.

* For POST requests, the RPC package requires that the method take a message
  class as a parameter. A message class must have a public constructor and
  public fields. Our message class, Pirate, is defined in `messages.dart`.

* If the pirate is successfully added to the crew, the server returns the
  Pirate object as a response message.

* The POST request's body (which is in JSON) is automatically deserialized
  into a Pirate object, and the returned Pirate object is
  automatically serialized into the response body as JSON.

<p>&nbsp;</p>
To support deleting a pirate,
add a `firePirate()` method to the PiratesApi class.

<pre>
// This class defines the interface that the server provides.
@ApiClass(version: 'v1')
class PiratesApi {
  ...
  <b>@ApiMethod(</b>
      <b>method: 'DELETE', path: 'pirate/{name}/the/{appellation}')</b>
  <b>Pirate firePirate(String name, String appellation) {</b>
    <b>var pirate = new Pirate()</b>
      <b>..name = Uri.decodeComponent(name)</b>
      <b>..appellation = Uri.decodeComponent(appellation);</b>
    <b>var pirateName = pirate.toString();</b>
    <b>if (!_pirateCrew.containsKey(pirateName)) {</b>
      <b>throw new NotFoundError('Could not find pirate \'$pirate\'!' +</b>
          <b>'Maybe they\'ve abandoned ship!');</b>
    <b>}</b>
    <b>return _pirateCrew.remove(pirateName);</b>
  <b>}</b>
  ...
}
</pre>
<div class="trydart-filename">lib/server/piratesapi.dart</div>

<p>&nbsp;</p>
&#128273; <strong> Key Information </strong>

* Annotating the method with `ApiMethod` exposes it as available remotely.

* The optional `method` parameter identifies `firePirate` as a DELETE request.

* The required path parameter, set here to '`pirate/{name}/the/{appellation}`',
  specifies how to access this method from an HTTP request.
  `name` and `appellation` are path parameters that are parsed by the
  RPC package and passed to the method as String values.

### &#9875; Check the code.

Run `dartanalyzer` on `piratesapi.dart`. From the top of `working-dir`:

```
dartanalyzer lib/server/piratesapi.dart
```

If the analyzer finds issues, you can debug them, or you can
use the code in `4-extended`.

### &#9875; Start up the server.

Restart the server.

```
dart bin/piratesnest.dart
```

### &#9875; Test the new API.

Verify the new API for adding and deleting pirates.
This step requires curl.

Constructing a curl command for a POST or DELETE request is
a little different than for a GET request.
If you recall, we annotated the `hirePirate` method like this:

<pre>
@ApiMethod(method: 'POST', path: '<b>pirate</b>')
</pre>

To access the `hirePirate` method, use `pirate` as the last bit of the URL.
You can pass data in curl using the `-d` parameter, so the following command
adds "Shams the Destroyer" to the list of pirates:

On Mac and Linux:

```
curl -d '{"name":"Shams", "appellation":"Destroyer"}' http://localhost:8088/piratesApi/v1/pirate
```

On Windows:

```
curl -d "{\"name\":\"Shams\",\"appellation\":\"Destroyer\"}" http://localhost:8088/piratesApi/v1/pirate
```

The `firePirate` method was annotated like this:

<pre>
@ApiMethod(method: 'DELETE', path: '<b>pirate/{name}/the/{appellation}</b>')
</pre>

The following command deletes "Shams the Destroyer":

```
curl -X DELETE http://localhost:8088/piratesApi/v1/pirate/Shams/the/Destroyer
```

Play with the server by adding and deleting pirates.
You might try the following:

* Add the same pirate twice.
* Delete the same pirate twice.
* Add a pirate with a _forbidden_ appellation (as defined in `utils.dart`),
  such as "Brad the Beautiful" or "Horatio the Wuss".

&nbsp;&#128273; <strong> Key information </strong>

* The DELETE request (`firePirate`) demonstrates how to specify parameters
  directly in the HTTP request. The POST request (`hirePirate`) demonstrates
  how to use a message class for passing values.
  Either approach is valid depending on your needs:

    * Only path and query parameters can be specified in the URL directly.

    * The message class enables using more advanced data structures.

</div>

## Step 5: Generate the client API {#step-five}

In the previous step, you added POST and DELETE methods to the
PiratesApi class.
In this step, you use the generator from the RPC package to
create a PiratesApi class that can be used by a client app.
After you have completed this step, a new `lib/client/`
directory contains the generated API.

![The 5-generated directory structure](images/generated-clientapi-directory-structure.png)

<div class="trydart-step-details" markdown="1">

### &#9875; Create the client API using the RPC generator.

From within the `working-dir` directory run the following commands:

```
pub global activate rpc
pub global run rpc:generate client -i lib/server/piratesapi.dart -o lib/client
```

You should see the following output:

```
[SUCCESS] piratesapi v1 @ lib/client
```

&nbsp;&#128273; <strong> Key information </strong>

* The RPC generator creates a <code>lib/client/<em>name</em>.dart</code> file,
  where _name_ is based on the name of the server API class. For example,
  if PiratesApi were defined in a file named `lib/server/ahoy.dart`,
  the client filename would still be `lib/client/piratesapi.dart`.

* This file allows you to connect your client app to the server, as you will
  see in [Step 6](#step-six).

### &#9875; View the generated RESTful API.

You can view the full listing of the generated file in the
`lib/client/` directory, but let's look an overview of the code:

<pre>
library server_code_lab.piratesApi.client;
...

<b>class PiratesApi</b> {

  final commons.ApiRequester _requester;

  <b>PiratesApi(http.Client client, {core.String rootUrl: "http://localhost:8080/", core.String servicePath: "piratesApi/v1/"}) :</b>
    ...

  async.Future&lt;Pirate&gt; <b>firePirate(core.String name, core.String appellation)</b> {
    ...
  }

  async.Future&lt;Pirate&gt; <b>hirePirate(Pirate request)</b> {
    ...
  }

  async.Future&lt;core.List&lt;Pirate&gt;&gt; <b>listPirates()</b> {
    ...
  }

  async.Future&lt;Pirate&gt; <b>shanghaiAPirate()</b> {
    ...
  }
}

class PirateFactory {
  static Pirate fromJson(core.Map _json) {
    ...
  }

  static core.Map toJson(Pirate message) {
    ...
  }
}
</pre>
<div class="trydart-filename">lib/client/piratesapi.dart</div>

<p>&nbsp;</p>
&nbsp;&#128273; <strong> Key information </strong>

* For each method marked with `ApiMethod` in `server/piratesapi.dart`,
  the RPC generator creates a corresponding method in `client/piratesapi.dart`.

* The RPC generator assumes that the server is messaging on
  `http://localhost:8080/`.  Our server uses port 8088,
  so the client app must override the default URL when instantiating
  this class, as you will see in the next step.

* These generated methods return Futures, indicating that they
  return before their work is complete. In the next step you'll see
  how to write code that uses these asynchronous methods.

</div>

## Step 6: Connect the client to the server {#step-six}

This step shows how to call the server from
a Dart client using the generated client API.

To save time, we've modified the Pirate example from the
[client code lab](https://www.dartlang.org/codelabs/darrrt/)
with additional UI for creating and maintaining a nest of pirates.
We've also updated the UI to use
[material design](http://www.google.com/design/spec/material-design/introduction.html#introduction-goals),
as shown in the following screen shot:

<img src="images/piratebadgeUI.png" alt="The PirateBadge UI" style="border:1px solid #021a40" >

<div class="trydart-step-details" markdown="1">

### &#9875; Get the client app.

**Copy the `web` directory from `6-client` to your package.**

After you have copied the `web` directory to your working directory,
the package should look like this:

![The 6-starter directory structure](images/client-server-directory-structure.png)

The bolded files implement the client app, PirateBadge.

`piratebadge.css`
: The styling for the browser page.

`piratebadge.dart`
: The Dart code specifying the app's behavior.

`piratebadge.html`
: The browser page for displaying the PirateBadge app.

### &#9875; View the code.

You can view the full listing of the client code in the
`web/` directory, but let's look at an overview of piratebadge.dart:

<pre>
...

import 'package:http/browser_client.dart';
import 'package:server_code_lab/client/piratesapi.dart';
import 'package:server_code_lab/common/messages.dart';
import 'package:server_code_lab/common/utils.dart';

...

// By default the generated client code uses
// 'http://localhost:8080/'. Since our server is running on
// port 8088 we override the default url when instantiating
// the generated PiratesApi class.
<b>final String _serverUrl = 'localhost:8088/';</b>
final BrowserClient _client = new BrowserClient();
<b>PiratesApi _api;</b>
<b>PirateShanghaier _shanghaier;</b>

Future main() async {
  // We need to determine if the client is using http or https
  // and use the same protocol for the client stub requests
  // (the protocol includes the ':').
  var protocol = window.location.protocol;
  <b>_api = new PiratesApi(_client, rootUrl: '$protocol//$_serverUrl');</b>
  _shanghaier = new PirateShanghaier();

  ...
}

Future refreshList() async {
  <b>List&lt;Pirate&gt; pirates = await _api.listPirates();</b>
  ...
}

void updateBadge(Event e) {
  String inputName = (e.target as InputElement).value.trim();
  <b>var pirate = _shanghaier.shanghaiAPirate(name: inputName);</b>
  ...
}

Future storeBadge(Event e) async {
  var pirateName = badgeNameElement.text;
  if (pirateName == null || pirateName.isEmpty) return null;
  var pirate = new Pirate.fromString(pirateName);
  try {
    <b>await _api.hirePirate(pirate);</b>
  } catch (error) {
    ...
  }
  ...
}

...

Future removeBadge(Event e) async {
  ...
  var option = pirateList.options.elementAt(idx);
  var pirate = new Pirate.fromString(option.label);
  try {
    <b>await _api.firePirate(pirate.name, pirate.appellation);</b>
  } catch (error) {
    ...
  }
  ...
}

Future removeAllBadges(Event e) async {
  for (var option in pirateList.options) {
    var pirate = new Pirate.fromString(option.label);
    try {
      <b>await _api.firePirate(pirate.name, pirate.appellation);</b>
    } catch (error) {
      // ignoring errors.
    }
  }
  ...
}

void generateBadge(Event e) {
  var pirate = _shanghaier.shanghaiAPirate();
  setBadgeName(pirate);
}

...

void <b>addRippleEffect(MouseEvent e)</b> {
  ...
}
</pre>
<div class="trydart-filename">web/piratebadge.dart</div>

<p>&nbsp;</p>
&nbsp;&#128273; <strong> Key information </strong>

* The generated server API assumes that the server uses `localhost:8080`,
  but our server is actually serving on port 8088, so the client has to set
  it explicitly.

* The client creates `_api`, an instance of the client-side PiratesApi class.

* The client calls the server through the `_api` instance, using
  the API generated in [Step 5](#step-five).
  For example, the client fetches the current pirate crew asynchronously by
  calling `_api.listPirates()`.

* The client's UI uses
  [material design](http://www.google.com/design/spec/material-design/introduction.html#introduction-goals).
  The material design visuals are mostly implemented in `piratebadge.css`.
  When clicked,
  a material design button animates&mdash;a wave ripples outward from the
  point of contact. The `addRippleEffect` method,
  implemented in Dart code, provides this effect.

### &#9875; Run the code.

Before launching the client, run `pub get` and launch the server.

```
pub get
dart bin/piratesnest.dart
```

You have a few options for launching the client.

<ul markdown="1">
<li markdown="1">Launch the client using Dartium.

Start up Dartium (remember that it's called Chromium on your file system).
Use **File > Open File...** and select `piratebadge.html` from the `web`
directory of your package.
</li>

<li markdown="1">Launch the client using your IDE.
Right-click `web/piratebadge.html` and choose the `Run in Dartium` option
(or equivalent) from the menu.
</li>

<li markdown="1"> Run `pub build` and launch the generated version in any browser.
The following opens the app in your default browser on a Mac.

```
pub build
open build/web/piratebadge.html
```
</li>
</ul>

&nbsp;&#128273; <strong> Key information </strong>

* Running `pub build` prints the following:

  "WARNING: dart:mirrors support in dart2js is
  experimental, and not recommended.
  This implementation of mirrors is
  incomplete, and often greatly increases
  the size of the generated JavaScript code."

  You can ignore this warning. The HTTP library uses mirrors to instantiate
  platform-specific classes on the VM and, in this case,
  does not cause code bloat.

* When running the client app, you might see these logging statements:

  Refused to set unsafe header "user-agent"<br>
  Refused to set unsafe header "content-length"

  These messages are caused by the generated code trying to set these
  header fields. You can ignore these messages.

* If Dartium's status indicator spins indefinitely, don't worry.
  The PirateBadge app might well be loaded and running.

</div>

## Step 7: Serve the client from the server {#step-seven}

In this step, you modify the server so that it serves the client
in the browser&mdash;you no longer have to explicitly launch the
client app.

<div class="trydart-step-details" markdown="1">

### &#9875; Edit pubspec.yaml, and run pub get.

Add the `http_server` dependency to the `pubspec.yaml` file, which
is directly under `working-dir`.

``` yaml
name: server_code_lab
version: 0.1.0
author: Dart Team <misc@dartlang.org>
description: Code-lab server sample.
environment:
  sdk: '>=1.9.0 <2.0.0'
dependencies:
  _discoveryapis_commons: ^0.1.0
  browser: ^0.10.0+2
  crypto: ^0.9.0
  http: ^0.11.1
  <b>http_server: ^0.9.5+1</b>
  logging_handlers: ^0.8.0
  rpc: ^0.3.0
```
<div class="trydart-filename">pubspec.yaml</div>

<p>&nbsp;</p>
Update the dependencies:

```
pub get
```

If you aren't familiar with caret syntax for pubspec files, see
[Caret syntax](https://www.dartlang.org/tools/pub/dependencies.html#caret-syntax),
a section in
[Pub Dependencies](https://www.dartlang.org/tools/pub/dependencies.html).

### &#9875; Edit bin/piratesnest.dart.

Import the `dart:async` and `http_server` packages.

<pre>
<b>import 'dart:async';</b>
import 'dart:io';

<b>import 'package:http_server/http_server.dart';</b>
import 'package:logging/logging.dart';
import 'package:rpc/rpc.dart';
</pre>
<div class="trydart-filename">bin/piratesnest.dart</div>

<p>&nbsp;</p>
&nbsp;&#128273; <strong> Key information </strong>

* The `http_server` package is used to serve the client app.

<p></p>
Add the following code to create a virtual directory for serving the
client from the `build/web` directory.

<pre>
final ApiServer _apiServer = new ApiServer(prettyPrint: true);

<b>final String _buildPath =</b>
    <b>Platform.script.resolve('../build/web/').toFilePath();</b>
<b>final VirtualDirectory _clientDir = </b>
    <b>new VirtualDirectory(_buildPath);</b>
</pre>
<div class="trydart-filename">bin/piratesnest.dart</div>

<p></p>
Add a request handler that examines the request and vectors it to
the right place.

<pre>
<b>Future requestHandler(HttpRequest request) async {</b>
  <b>if (request.uri.path.startsWith('/piratesApi')) {</b>
    <b>// Handle the API request.</b>
    <b>var apiResponse;</b>
    <b>try {</b>
      <b>var apiRequest = new HttpApiRequest.fromHttpRequest(request);</b>
      <b>apiResponse =</b>
          <b>await _apiServer.handleHttpApiRequest(apiRequest);</b>
    <b>} catch (error, stack) {</b>
      <b>var exception =</b>
          <b>error is Error ? new Exception(error.toString()) : error;</b>
      <b>apiResponse = new HttpApiResponse.error(</b>
          <b>HttpStatus.INTERNAL_SERVER_ERROR, exception.toString(),</b>
          <b>exception, stack);</b>
    <b>}</b>
    <b>return sendApiResponse(apiResponse, request.response);</b>
  <b>} else if (request.uri.path == '/') {</b>
    <b>// Redirect to the piratebadge.html file. This will initiate</b>
    <b>// loading the client application.</b>
    <b>request.response.redirect(Uri.parse('/piratebadge.html'));</b>
  <b>} else {</b>
    <b>// Serve the requested file (path) from the virtual directory,</b>
    <b>// minus the preceeding '/'. This will fail with a 404 Not Found</b>
    <b>// if the request is not for a valid file.</b>
    <b>var fileUri = new Uri.file(_buildPath)</b>
        <b>.resolve(request.uri.path.substring(1));</b>
    <b>_clientDir.serveFile(new File(fileUri.toFilePath()), request);</b>
  <b>}</b>
<b>}</b>
</pre>
<div class="trydart-filename">bin/piratesnest.dart</div>

<p>&nbsp;</p>
&#128273; <strong> Key information </strong>

* If the incoming request begins with the `/piratesApi` string
  (for example, if it's a curl command) vector that request to ApiServer.

* If the incoming request doesn't specify any file,
  serve `/piratebadge.html` (the default); this loads the client app.

* Otherwise, serve the requested file. Specifying an invalid file
  results in a 404 error.

<p></p>
Instruct the server to listen on the `requestHandler` method.

<pre>
main() async {
  // Add a bit of logging...
  Logger.root..level = Level.INFO
             ..onRecord.listen(print);

  // Set up a server serving the pirate API.
  _apiServer.addApi(new PiratesApi());
  HttpServer server =
      await HttpServer.bind(InternetAddress.ANY_IP_V4, 8088);
  server.listen(<b>requestHandler</b>);
  print('Server listening on http://${server.address.host}:'
        '${server.port}');

}
</pre>
<div class="trydart-filename">bin/piratesnest.dart</div>

<p>&nbsp;</p>

### &#9875; Run the code.

Update the dependencies.

```
pub get
```

Run the analyzer on the code, if your IDE or editor didn't already do so.

```
dartanalyzer bin/piratesnest.dart
```

Build the client app.

```
pub build
```

Kill any previous instances of the server and relaunch.

```
dart bin/piratesnest.dart
```

In your browser, navigate to
<a href="http://localhost:8088/piratebadge.html" target="_blank">http://localhost:8088/piratebadge.html</a>.

Play with the server by creating a pirate crew.

Can you add the same pirate twice?
: The client's UI protects you from some of the errors that
  you could trigger when constructing an HTTP request.

Can you create a pirate name with a custom appellation, such as Magnus the Magnificent?
: The client app doesn't use the full functionality
  that the server provides&mdash;and that's ok!

</div>

## What next? {#whatnext}

Now that you've played with a Dart server,
what is next? Here are some suggestions.

<div class="trydart-step-details" markdown="1">

### &#9875; Deploy your application to AppEngine.

[Step 8](https://github.com/dart-lang/one-hour-codelab/tree/appengine-auth/server/8-appengine)
of the appengine-auth branch of the one-hour-codelab repo
modifies the server so that you can deploy it to AppEngine.

### &#9875; Add authentication.

[Step 9](https://github.com/dart-lang/one-hour-codelab/tree/appengine-auth/server/9-auth)
of the appengine-auth branch of the one-hour-codelab repo
modifies the server to require a Google Account login.

### &#9875; Make your server API discoverable.

The RPC package allows you to expose your API in the Discovery document format.
This means that clients can download the API description in JSON and generate
client stubs to call the server API using a Discover document generator,
such as the
[Discovery API generator for Dart](https://github.com/dart-lang/discoveryapis_generator).

You can read more at
[Google APIs Discovery Service](https://developers.google.com/discovery/).

### &#9875; Read the tutorials.

Learn more about Dart from
[The Dart Tutorials](https://www.dartlang.org/docs/tutorials/).
In particular, the
[Write HTTP Clients & Servers](https://www.dartlang.org/docs/tutorials/httpserver/)
tutorial has more information on server-side programming.

<hr>

</div>

## Summary {#summary}

This code lab introduced you to the RPC package,
a powerful tool for creating a RESTful server API
without writing a lot of boilerplate code.

<div class="trydart-step-details" markdown="1">

You learned:

* How to specify a server's API, using the RPC package.

* How to create the corresponding client API using the RPC generator.

* How to create libraries that a client and server can share, by writing
  code that depends on neither dart:io nor dart:html.

* How to use the API generated by the RPC package.

* How to modify the server to serve the client directly.

### &#9875; Feedback

Please provide feedback to the appropriate repo:

* [server docs repo](https://github.com/dart-lang/server/issues)
* [one-hour-codelab repo](https://github.com/dart-lang/one-hour-codelab/issues)
* [rpc repo](https://github.com/dart-lang/rpc/issues)

</div>
