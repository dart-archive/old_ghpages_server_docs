---
layout: default
title: "Dart and Google Cloud Platform"
short-title: "Google Cloud Platform"
description: "An introduction to Dart support for services in Google Cloud Platform."
has-permalinks: false
---

# {{ page.title }} <img src="images/GoogleCloudPlatform-logo.png" alt="logo for Google Cloud Platform">

**Note**: There are many ways to host a Dart server. You might also be
interested in [Firebase][] and [Sourcevoid][].

[Google Cloud Platform](https://cloud.google.com/)
provides a set of cloud-based services that allow you to build,
test, and deploy applications on Google's highly scalable
and reliable infrastructure for your web, mobile, and backend solutions.

Cloud libraries and prepackaged Docker images developed for the
Dart platform provide support for the following services:

App Engine Custom Runtimes for the Flex Environment:
: See [Dart on App Engine Custom Runtimes for the Flex
  Environment](app-engine-flex) to learn how to run your Dart
  application on Google App Engine using the flexible environment.

Google Compute Engine
: See [Running on Google Compute
  Engine](https://github.com/dart-lang/dart_docker/tree/master/hello#running-on-google-compute-engine) for information
  on how to use Google Compute Engine for your application.

Google Container Engine
: See
  [Running on Google Container
  Engine](https://github.com/dart-lang/dart_docker/tree/master/hello#running-on-google-container-engine)
  for information on how to take advantage of clusters of containers.

**Note**: Google App Engine Managed VMs is deprecated and replaced with
[App Engine Custom Runtimes for the Flexible
Environment](https://cloud.google.com/appengine/docs/flexible/custom-runtimes/).
For more information, see [Dart on App Engine Custom Runtimes for the Flex
Environment](app-engine-flex).

[Firebase]: https://firebase.google.com/
[Heroku]: https://github.com/igrigorik/heroku-buildpack-dart
[Sourcevoid]: https://www.sourcevoid.com/
