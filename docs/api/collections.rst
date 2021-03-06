.. _collections:

Collections
###########

A collection belongs to a bucket and stores records.

A collection is a mapping with the following attribute:

* ``schema``: (*optional*) a JSON schema to validate the collection records
* ``cache_expires``: (*optional*, in seconds) add client cache headers on read-only requests.
  :ref:`More details...<collection-caching>`


.. note::

    By default users are assigned to a bucket that is used for their
    personal data.

    Application can use this default bucket with the ``default``
    shortcut: ie ``/buckets/default/collections/contacts`` will be
    the current user contacts.

    Internally the user default bucket is assigned to an ID that can
    later be used to share data from a user personnal bucket.

    Collections on the "default" bucket are created silently upon first
    access and therefore don't need to be set beforehand.



.. _collection-put:

Creating a collection
=====================


.. http:put:: /buckets/(bucket_id)/collections/(collection_id)

    :synopsis: Creates or replaces a collection object.

    **Requires authentication**

    A collection is the parent object of records. It can be viewed as a container where records permissions are assigned globally.

    **Example Request**

    .. sourcecode:: bash

        $ http put http://localhost:8888/v1/buckets/blog/collections/articles --auth="bob:" --verbose

    .. sourcecode:: http

        PUT /v1/buckets/blog/collections/articles HTTP/1.1
        Accept: application/json
        Accept-Encoding: gzip, deflate
        Authorization: Basic Ym9iOg==
        Connection: keep-alive
        Content-Length: 0
        Host: localhost:8888
        User-Agent: HTTPie/0.9.2

    .. sourcecode:: http

        HTTP/1.1 201 Created
        Access-Control-Expose-Headers: Backoff, Retry-After, Alert
        Content-Length: 159
        Content-Type: application/json; charset=UTF-8
        Date: Thu, 18 Jun 2015 15:36:34 GMT
        Server: waitress

        {
            "data": {
                "id": "articles",
                "last_modified": 1434641794149
            },
            "permissions": {
                "write": [
                    "basicauth:206691a25679e4e1135f16aa77ebcf211c767393c4306cfffe6cc228ac0886b6"
                ]
            }
        }

    .. note::

        In order to create only if it does not exist yet, a ``If-None-Match: *``
        request header can be provided. A ``412 Precondition Failed`` error response
        will be returned if the record already exists.


.. _collection-patch:

Updating a collection
=====================


.. http:patch:: /buckets/(bucket_id)/collections/(collection_id)

    :synopsis: Updates a collection object.

    **Requires authentication**

    A collection is the parent object of records. It can be viewed as
    a container where records permissions are assigned globally.

    **Example Request**

    .. sourcecode:: bash

        $ echo '{"data": {"fingerprint": "9cae1b2d0f2b7d09bcf5c1bf51544274"}}' | http patch http://localhost:8888/v1/buckets/blog/collections/articles --auth="bob:" --verbose

    .. sourcecode:: http

        PATCH /v1/buckets/blog/collections/articles HTTP/1.1
        Accept: application/json
        Accept-Encoding: gzip, deflate
        Authorization: Basic Ym9iOg==
        Connection: keep-alive
        Content-Length: 62
        Content-Type: application/json
        Host: localhost:8888
        User-Agent: HTTPie/0.9.2

        {
            "data": {
                "fingerprint": "9cae1b2d0f2b7d09bcf5c1bf51544274"
            }
        }

    .. sourcecode:: http

        HTTP/1.1 200 OK
        Access-Control-Expose-Headers: Backoff, Retry-After, Alert
        Content-Length: 208
        Content-Type: application/json; charset=UTF-8
        Date: Thu, 18 Jun 2015 15:36:34 GMT
        Server: waitress

        {
            "data": {
                "id": "articles",
                "last_modified": 1434641794149,
                "fingerprint": "9cae1b2d0f2b7d09bcf5c1bf51544274"
            },
            "permissions": {
                "write": [
                    "basicauth:206691a25679e4e1135f16aa77ebcf211c767393c4306cfffe6cc228ac0886b6"
                ]
            }
        }


.. _collection-get:

Retrieving an existing collection
=================================

.. http:get:: /buckets/(bucket_id)/collections/(collection_id)

    :synopsis: Returns the collection object.

    **Requires authentication**

    **Example Request**

    .. sourcecode:: bash

        $ http get http://localhost:8888/v1/buckets/blog/collections/articles --auth="bob:" --verbose

    .. sourcecode:: http

        GET /v1/buckets/blog/collections/articles HTTP/1.1
        Accept: */*
        Accept-Encoding: gzip, deflate
        Authorization: Basic Ym9iOg==
        Connection: keep-alive
        Host: localhost:8888
        User-Agent: HTTPie/0.9.2


    **Example Response**

    .. sourcecode:: http

        HTTP/1.1 200 OK
        Access-Control-Expose-Headers: Backoff, Retry-After, Alert, Last-Modified, ETag
        Content-Length: 159
        Content-Type: application/json; charset=UTF-8
        Date: Thu, 18 Jun 2015 15:52:31 GMT
        Etag: "1434642751314"
        Last-Modified: Thu, 18 Jun 2015 15:52:31 GMT
        Server: waitress

        {
            "data": {
                "id": "articles",
                "last_modified": 1434641794149
            },
            "permissions": {
                "write": [
                    "basicauth:206691a25679e4e1135f16aa77ebcf211c767393c4306cfffe6cc228ac0886b6"
                ]
            }
        }


.. _collection-delete:

Deleting a collection
=====================

.. http:delete:: /buckets/(bucket_id)/collections/(collection_id)

    :synopsis: Deletes a specific collection and **everything under it**.

    **Requires authentication**

    **Example Request**

    .. sourcecode:: bash

        $ http delete http://localhost:8888/v1/buckets/blog/collections/articles --auth="bob:" --verbose

    .. sourcecode:: http

        DELETE /v1/buckets/blog/collections/articles HTTP/1.1
        Accept: */*
        Accept-Encoding: gzip, deflate
        Authorization: Basic Ym9iOg==
        Connection: keep-alive
        Content-Length: 0
        Host: localhost:8888
        User-Agent: HTTPie/0.9.2

    **Example Response**

    .. sourcecode:: http

        HTTP/1.1 200 OK
        Access-Control-Expose-Headers: Backoff, Retry-After, Alert
        Content-Length: 71
        Content-Type: application/json; charset=UTF-8
        Date: Thu, 18 Jun 2015 15:54:02 GMT
        Server: waitress

        {
            "data": {
                "deleted": true,
                "id": "articles",
                "last_modified": 1434642842010
            }
        }


.. _collection-json-schema:

Collection JSON schema
======================

**Requires setting** ``kinto.experimental_collection_schema_validation`` to ``True``.

A `JSON schema <http://json-schema.org/>`_ can optionally be associated to a
collection.

Once a schema is set, records will be validated during creation or update.

If the validation fails, a ``400 Bad Request`` error response will be
returned.

.. note::

    JSON schema is quite verbose and not an ideal solution for every use-case.
    However it is universal and supported by many programming languages
    and environments.


Set or replace a schema
-----------------------

Just modify the ``schema`` attribute of the collection object:

**Example request**

.. code-block:: bash

    $ echo '{
      "data": {
        "schema": {
          "title": "Blog post schema",
          "type": "object",
          "properties": {
              "title": {"type": "string"},
              "body": {"type": "string"}
          },
          "required": ["title"]
        }
      }
    }' | http PATCH "http://localhost:8888/v1/buckets/default/collections/articles" --auth admin: --verbose

.. code-block:: http

    PATCH /v1/buckets/default/collections/articles HTTP/1.1
    Accept: application/json
    Accept-Encoding: gzip, deflate
    Authorization: Basic YWRtaW46
    Connection: keep-alive
    Content-Length: 236
    Content-Type: application/json; charset=utf-8
    Host: localhost:8888
    User-Agent: HTTPie/0.8.0

    {
        "data": {
            "schema": {
                "properties": {
                    "body": {
                        "type": "string"
                    },
                    "title": {
                        "type": "string"
                    }
                },
                "required": [
                    "title"
                ],
                "title": "Blog post schema",
                "type": "object"
            }
        }
    }

**Example response**

.. code-block:: http

    HTTP/1.1 200 OK
    Access-Control-Expose-Headers: Backoff, Retry-After, Alert, Content-Length
    Content-Length: 300
    Content-Type: application/json; charset=UTF-8
    Date: Fri, 21 Aug 2015 12:31:40 GMT
    Etag: "1440160300818"
    Last-Modified: Fri, 21 Aug 2015 12:31:40 GMT
    Server: waitress

    {
        "data": {
            "id": "articles",
            "last_modified": 1440160300818,
            "schema": {
                "properties": {
                    "body": {
                        "type": "string"
                    },
                    "title": {
                        "type": "string"
                    }
                },
                "required": [
                    "title"
                ],
                "title": "Blog post schema",
                "type": "object"
            }
        },
        "permissions": {
            "write": [
                "basicauth:780f1ecd9f57b01bef79608b45916d3bddd17f83461ac6240402e0ffff3596c5"
            ]
        }
    }



Records validation
------------------

Once a schema has been defined, the posted records must match it:

.. code-block:: bash

    $ echo '{"data": {
        "body": "Fails if no title"
    }}' | http POST http://localhost:8888/v1/buckets/blog/collections/articles/records --auth "admin:"

.. code-block:: http

    HTTP/1.1 400 Bad Request
    Access-Control-Expose-Headers: Backoff, Retry-After, Alert
    Content-Length: 192
    Content-Type: application/json; charset=UTF-8
    Date: Wed, 10 Jun 2015 10:17:01 GMT
    Server: waitress

    {
        "code": 400,
        "details": [
            {
                "description": "u'title' is a required property",
                "location": "body",
                "name": "title"
            }
        ],
        "errno": 107,
        "error": "Invalid parameters",
        "message": "u'title' is a required property"
    }



Schema migrations
-----------------

*Kinto* does not take care of schema migrations. But it gives the basics for clients
to manage it.

If the validation succeeds, the record will receive a ``schema`` field with the
schema version (i.e. the collection current ``last_modified`` timestamp).

It becomes possible to use this ``schema`` field as a filter on the collection
records endpoint in order to obtain the records that were not validated against a particular
version of the schema.

For example, ``GET /buckets/default/collections/articles/records?min_schema=123456``.


Remove a schema
---------------

In order to remove the schema of a collection, just modify the ``schema`` field
to an empty mapping.


**Example request**

.. code-block:: bash

    echo '{"data": {"schema": {}} }' | http PATCH "http://localhost:8888/v1/buckets/default/collections/articles" --auth admin: --verbose

.. code-block:: http

    PATCH /v1/buckets/default/collections/articles HTTP/1.1
    Accept: application/json
    Accept-Encoding: gzip, deflate
    Authorization: Basic YWRtaW46
    Connection: keep-alive
    Content-Length: 26
    Content-Type: application/json; charset=utf-8
    Host: localhost:8888
    User-Agent: HTTPie/0.8.0

    {
        "data": {
            "schema": {}
        }
    }

**Example response**

.. code-block:: http

    HTTP/1.1 200 OK
    Access-Control-Expose-Headers: Backoff, Retry-After, Alert, Content-Length
    Content-Length: 171
    Content-Type: application/json; charset=UTF-8
    Date: Fri, 21 Aug 2015 12:27:04 GMT
    Etag: "1440159981842"
    Last-Modified: Fri, 21 Aug 2015 12:26:21 GMT
    Server: waitress

    {
        "data": {
            "id": "articles",
            "last_modified": 1440159981842,
            "schema": {}
        },
        "permissions": {
            "write": [
                "basicauth:780f1ecd9f57b01bef79608b45916d3bddd17f83461ac6240402e0ffff3596c5"
            ]
        }
    }


.. _collection-caching:

Collection caching
==================

With the ``cache_expires`` attribute on a collection, it is possible to add client
cache control response headers for read-only requests.
The client (or cache server or proxy) will use them to cache the collection
records for a certain amount of time, in seconds.

For example, set it to ``3600`` (1 hour):

.. code-block:: bash

    echo '{"data": {"cache_expires": 3600} }' | http PATCH "http://localhost:8888/v1/buckets/default/collections/articles" --auth admin:

From now on, the cache control headers are set for the `GET` requests:

.. code-block:: bash

    http  "http://localhost:8888/v1/buckets/default/collections/articles/records" --auth admin:

.. code-block:: http
    :emphasize-lines: 3,8

    HTTP/1.1 200 OK
    Access-Control-Expose-Headers: Backoff, Retry-After, Alert, Content-Length, Next-Page, Total-Records, Last-Modified, ETag, Cache-Control, Expires, Pragma
    Cache-Control: max-age=3600
    Content-Length: 11
    Content-Type: application/json; charset=UTF-8
    Date: Mon, 14 Sep 2015 13:51:47 GMT
    Etag: "1442238450779"
    Expires: Mon, 14 Sep 2015 14:51:47 GMT
    Last-Modified: Mon, 14 Sep 2015 13:47:30 GMT
    Server: waitress
    Total-Records: 0

    {
        "data": [...]
    }


If set to ``0``, the collection records become explicitly uncacheable (``no-cache``).

.. code-block:: bash

    echo '{"data": {"cache_expires": 0} }' | http PATCH "http://localhost:8888/v1/buckets/default/collections/articles" --auth admin:

.. code-block:: http
    :emphasize-lines: 3,8,10

    HTTP/1.1 200 OK
    Access-Control-Expose-Headers: Backoff, Retry-After, Alert, Content-Length, Next-Page, Total-Records, Last-Modified, ETag, Cache-Control, Expires, Pragma
    Cache-Control: max-age=0, must-revalidate, no-cache, no-store
    Content-Length: 11
    Content-Type: application/json; charset=UTF-8
    Date: Mon, 14 Sep 2015 13:54:51 GMT
    Etag: "1442238450779"
    Expires: Mon, 14 Sep 2015 13:54:51 GMT
    Last-Modified: Mon, 14 Sep 2015 13:47:30 GMT
    Pragma: no-cache
    Server: waitress
    Total-Records: 0

    {
        "data": []
    }

.. note::

    This can also be forced from settings, see :ref:`configuration section <configuration-client-caching>`.
