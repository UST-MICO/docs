REST API
========

Structure Overview
------------------

This is how the structure of the API should look like.

-  :http:get:`/services/`

    -  :http:get:`GET /(id)/ </services/(str:id)/>`

    -  :http:get:`GET /(shortName)/ </services/(str:shortName)/>`

        -  :http:get:`GET /(version)/ </services/(str:shortName)/(str:version)/>`

            -  :http:get:`GET /interfaces/ </services/(str:shortName)/(str:version)/interfaces/>`

            -  :http:get:`GET /dependees/ </services/(str:shortName)/(str:version)/dependees/>`

            -  :http:get:`GET /dependers/ </services/(str:shortName)/(str:version)/dependers/>`

            -  :http:get:`GET /status/ </services/(str:shortName)/(str:version)/status/>`

    -  :http:get:`GET /import/ </services/import/>`

        -  :http:post:`POST /(type)/ </services/import/(str:type)/>`

-  :http:get:`/applications/`

    -  :http:get:`GET /(id)/ </applications/(str:id)/>`

    -  :http:get:`GET /(shortName)/ </applications/(str:shortName)/>`

        -  :http:get:`GET /(version)/ </applications/(str:shortName)/(str:version)/>`

            -  :http:get:`GET /services/ </applications/(str:shortName)/(str:version)/services/>`

            -  :http:get:`GET /status/ </applications/(str:shortName)/(str:version)/status/>`

            -  :http:get:`GET /deploy/ </applications/(str:shortName)/(str:version)/deploy/>`

-  :http:get:`/jobs/`

   -  :http:get:`GET /(id)/ </jobs/(str:id)/>`

-  :http:get:`/cluster/*`



Method Details
--------------

.. http:get:: /services/

    **Example Response:**

    .. sourcecode:: json

        [
            {
                "id": "...",
                "shortName": "hello-world",
                "version": "0.0.1",
                "name": "Hello World",
                "description": "...",
            },
            {
                "id": "...",
                "shortName": "foo-bar",
                "version": "0.0.14a",
                "name": "FooBar",
                "description": "...",
            }
        ]

.. http:post:: /services/

    **Example Request:**

    .. sourcecode:: json

        {
            "name": "Hello World",
            "shortName": "hello-world",
            "version": "0.0.1",
            "description": "...",
        }

    **Example Response:**

    .. sourcecode:: json

        {
            "id": "...",
            "name": "Hello World",
            "shortName": "hello-world",
            "version": "0.0.1",
            "description": "...",
            "...": "...",
        }

.. http:get:: /services/(str:id)/

.. http:get:: /services/(str:shortName)/

.. http:get:: /services/(str:shortName)/(str:version)/

    **Example Response:**

    .. sourcecode:: json

        {
            "id": "...",
            "name": "Hello World",
            "shortName": "hello-world",
            "version": "0.0.1",
            "description": "...",
            "...": "...",
            "interfaces": [],
            "dependees": [ "// services this service depends on" ],
            "dependers": [ "// services that depend on this service" ],
        }

.. http:put:: /services/(str:shortName)/(str:version)/

.. http:get:: /services/(str:shortName)/(str:version)/interfaces/

.. http:post:: /services/(str:shortName)/(str:version)/interfaces/

.. http:get:: /services/(str:shortName)/(str:version)/interfaces/(str:shortName)

.. http:put:: /services/(str:shortName)/(str:version)/interfaces/(str:shortName)

.. http:delete:: /services/(str:shortName)/(str:version)/interfaces/(str:shortName)

.. http:get:: /services/(str:shortName)/(str:version)/dependees/

.. http:post:: /services/(str:shortName)/(str:version)/dependees/

.. http:delete:: /services/(str:shortName)/(str:version)/dependees/(str:shortName)

.. http:get:: /services/(str:shortName)/(str:version)/dependers/

.. http:post:: /services/(str:shortName)/(str:version)/dependers/

.. http:delete:: /services/(str:shortName)/(str:version)/dependers/(str:shortName)

.. http:get:: /services/(str:shortName)/(str:version)/status/

.. http:get:: /services/import/

.. http:post:: /services/import/(str:type)/

    **Example Response**

    .. sourcecode:: http

        HTTP/1.1 202 Accepted
        Location: /jobs/12345

.. http:get:: /jobs/(str:id)/

    **Example Response**

    .. sourcecode:: http

        GET /jobs/12345
        HTTP/1.1 200 Ok
        {
            "type": "IMPORT",
            "status": "PENDING",
            ...
            "_links": {
                "self": { "href": "/jobs/12345" },
                "cancel": { "method": "DELETE", "href": "/jobs/12345" },
            },
        }

    **Example Response (Finished Job)**

    .. sourcecode:: http

        HTTP/1.1 303 See Other
        Location: /services/{shortName}/{version} // depends on "type"

.. note:: /applications/ has all service endpoints as well...

.. http:get:: /applications/(str:shortName)/(str:version)/

    .. sourcecode:: json

        {
            "name": "...",
            "description": "...",
            "...": "...",
            "endpoint": {
                // just one "interface description"
            },
            // no dependers, dependees
            "services": [{
                // Service Object
            }]
        }

.. http:post:: /applications/(str:shortName)/(str:version)/deploy/

.. http:get:: /cluster/*


.. todo:: uncomment openapi specification if api changes are merged

.. .. openapi:: openapi.json

