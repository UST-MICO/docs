REST API
========

Structure Overview
------------------

This is how the structure of the API looks like.

-  :http:get:`/services`

   -  /<id>

      -  /serviceDescriptions

         -  /<servicedescriptionId>

      -  /dependsOn

         -  /<dependsonId>

      -  /versions

-  :http:get:`/serviceDescriptions`

   -  /<id>

-  :http:get:`/applications`

   -  /<id>

      -  /serviceDescriptions

         -  /<servicedescriptionId>

      -  /dependsOn

         -  /<dependsonId>

-  :http:get:`/dependsOns`

   -  /<id>

      -  /serviceDependencies


.. todo:: add models to documentation

Method Details
--------------

.. openapi:: openapi.json

