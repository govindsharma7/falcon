<img align="right" src="https://raw.github.com/racker/falcon/master/falcon.png" alt="Falcon picture" />

Falcon [![Build Status](https://travis-ci.org/racker/falcon.png)](https://travis-ci.org/racker/falcon)
======

**[Experimental]**

Falcon is a [high-performance Python framework][home] for building cloud APIs. It tries to do as little as possible while remaining [highly effective][benefits].

> Perfection is finally attained not when there is no longer anything to add, but when there is no longer anything to take away.
>
> *- Antoine de Saint-Exupéry*

[home]: http://falconframework.org/index.html
[benefits]: http://falconframework.org/index.html#Benefits

### Design Goals ###

**Fast.** Cloud APIs need to turn around requests quickly, and make efficient use of hardware. Falcon processes requests several times faster than other popular web frameworks.

**Light.** Only the essentials are included, with the "six" Python 3 compatibility module being the only dependency outside the standard library. We work to keep the code lean and mean, making Falcon easier to test, optimize, and deploy. 

**Flexible.** Falcon uses the web-friendly Python language and speaks WSGI. Built-in diagnostics facilitate monitoring and debugging of production systems.

### Install ###

```bash
$ pip install falcon
```

### Test ###

```bash
$ pip install nose && nosetests
```

To test across all supported Python versions:

```bash
$ pip install tox && tox
```

### Usage ###

More/better docs are on the way, but in the meantime, here is a simple example showing how to create a Falcon-based API.

```python
class ThingsResource:
    def on_get(self, req, resp):
        """Handles GET requests"""
        resp.status = falcon.HTTP_200
        resp.body = 'Hello world!'

# falcon.API instances are callable WSGI apps
wsgi_app = api = falcon.API()

# Resources are represented by long-lived class instances
things = ThingsResource()

# things will handle all requests to the '/things' URL path
api.add_route('/things', things)
```

Here is a more involved example, demonstrating getting headers and query parameters, handling errors, and reading/writing message bodies.

```python
class ThingsResource:

    def __init__(self, db):
        self.db = db
        self.logger = logging.getLogger('thingsapi.' + __name__)

    def on_get(self, req, resp, user_id):
        marker = req.get_param('marker', default='')
        limit = req.get_param('limit', default=50)

        # Alternatively, do this in middleware
        token = req.get_header('X-Auth-Token')

        if token is None:
            raise falcon.HTTPUnauthorized('Auth token required',
                                          'Please provide an auth token as '
                                          'part of the request',
                                          'http://docs.example.com/auth')

        # Note: token_is_valid is used as an example
        # and does not actually exist
        if not token_is_valid(token, user_id):
            raise falcon.HTTPUnauthorized('Authentication required',
                                          'The provided auth token is not '
                                          'valid. Please request a new token '
                                          'and try again.',
                                          'http://docs.example.com/auth')

        # Alternatively, do this in middleware
        if not req.client_accepts_json():
            raise falcon.HTTPUnsupportedMediaType(
                'Media Type not Supported',
                'This API only supports the JSON media type.',
                'http://docs.examples.com/api/json')

        try:
            result = self.db.get_things(marker, limit)
        except Exception as ex:
            self.logger.error(ex)

            description = ('Aliens have attacked our base. We will be back'
                           'as soon as we fight them off. We appreciate your'
                           'patience.')

            raise falcon.HTTPServiceUnavailable('Service Outage', description)

        resp.set_header('X-Powered-By', 'Donuts')
        resp.status = falcon.HTTP_200
        resp.body = json.dumps(result)

    def on_post(self, req, resp):
        try:
            raw_json = req.stream.read()
        except Exception:
            raise falcon.HTTPError(falcon.HTTP_748,
                                   'Read Error',
                                   "Could not read the request body. I bet "
                                   "it's them ponies again.")

        try:
            thing = json.loads(raw_json, 'utf-8')
        except ValueError:
            raise falcon.HTTPError(falcon.HTTP_753,
                                   'Malformed JSON',
                                   'Could not decode the request body. The '
                                   'JSON was incorrect.')

        try:
            proper_thing = self.db.add_thing(thing)
        # Note: StorageError is used as an example
        # and does not actually exist
        except StorageError:
            raise falcon.HTTPError(falcon.HTTP_725,
                                   'Database Error',
                                   "Sorry, couldn't write your thing to the "
                                   'database. It worked on my machine.')

        resp.status = falcon.HTTP_201
        resp.set_header('Location', '/{userId}/things/' + proper_thing.id)

wsgi_app = api = falcon.API()

db = StorageEngine()
things = ThingsResource(db)
api.add_route('/{user_id}/things', things)

```

### Contributing ###

Kurt Griffiths (kgriffs) is the creator and current maintainer of the Falcon framework. Pull requests are always welcome. 

Before submitting a pull request, please ensure you have added/updated the appropriate tests (and that all existing tests still pass with your changes), and that your coding style follows PEP 8. 

Commit messages should be formatted using [AngularJS conventions][ajs] (one-liners are OK for now but body and footer may be required as the project matures). 

Comments follow [Google's style guide][goog-style-comments].

[ajs]: http://goo.gl/QpbS7
[goog-style-comments]: http://google-styleguide.googlecode.com/svn/trunk/pyguide.html#Comments

### Legal ###

Copyright 2013 by Rackspace Hosting, Inc.

Falcon image courtesy of [John O'Neill](https://commons.wikimedia.org/wiki/File:Brown-Falcon,-Vic,-3.1.2008.jpg).

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
