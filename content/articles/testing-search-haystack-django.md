Title: Testing Search (Haystack) in Django
Date: 2013-07-21 13:44
Category: Software
Tags: software, search, testing, code
Slug: search-django-testing
Author: Ben Reilly
Summary: Testing search functionality with Django
Status: published


[Django’s build-in testing framework](https://docs.djangoproject.com/en/dev/topics/testing/overview/) is extremely handy. As long as you use the ORM with a supported data store, a test database is used for the duration of the tests and is cleaned up in between unit tests. There is no need for elaborate mocking – something I had grown accustom to in .NET.

Here is a quick sample, edited for brevity:

```
$ ./manage.py test appname -v 2

Creating test database for alias 'default' ('test_projectname')…
Syncing...
Creating tables ...
test_first (projectname.test.SampleTestClass) ... ok
test_second (projectname.test.SampleTestClass) ... ok
test_third (projectname.test.SampleTestClass) ... ok
Ran 3 tests in 1.260s
OK
Destroying test database for alias 'default' ('test_projectname')…
```

But if you are using some external source of data, it is necessary to create a mock or some fake environment (as Django does).

[Haystack](http://haystacksearch.org/) is a handy library that abstracts out the details of various search engines. You get some powerful features build into something like [Elasticsearch](http://www.elasticsearch.org/) – high availability, full text search, spelling correct, more like this, etc – in some functions and data structures familiar to Django using developers.

But if you are [integration testing](https://en.wikipedia.org/wiki/Integration_testing), the tests are calling your views directly and your views are updating or retrieving data from an external search engine, you are going to potentially have a bad time. Stuff will be persisted between unit tests and your results will be likely be inconsistent.

The solution is pretty simple actually. Fire up a new index, override the settings such that the new index is the target for the Haystack calls for the duration of tests, and clear the index between tests.

```
TEST_INDEX = {
    'default': {
        'ENGINE': 'haystack.backends.elasticsearch_backend.ElasticsearchSearchEngine',
        'URL': 'http://127.0.0.1:9200/',
        'TIMEOUT': 60 * 10,
        'INDEX_NAME': 'test_index',
    },
}


@override_settings(HAYSTACK_CONNECTIONS=TEST_INDEX)
class BaseTestCase(TestCase):

    def setUp(self):
        haystack.connections.reload('default')
        super(BaseTestCase, self).setUp()

    def tearDown(self):
        call_command('clear_index', interactive=False, verbosity=0)
```

[GIST here](https://gist.github.com/bwreilly/6050981)