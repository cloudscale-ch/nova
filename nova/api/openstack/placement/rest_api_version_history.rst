REST API Version History
~~~~~~~~~~~~~~~~~~~~~~~~

This documents the changes made to the REST API with every
microversion change. The description for each version should be a
verbose one which has enough information to be suitable for use in
user documentation.

1.0 (Maximum in Newton)
-----------------------

This is the initial version of the placement REST API that was released in
Nova 14.0.0 (Newton). This contains the following routes:

* /resource_providers
* /resource_providers/allocations
* /resource_providers/inventories
* /resource_providers/usages
* /allocations

1.1 Resource provider aggregates
--------------------------------

The 1.1 version adds support for associating aggregates with
resource providers with ``GET`` and ``PUT`` methods on one new
route:

* /resource_providers/{uuid}/aggregates

1.2 Custom resource classes
---------------------------

Placement API version 1.2 adds basic operations allowing an admin to create,
list and delete custom resource classes.

The following new routes are added:

* GET /resource_classes: return all resource classes
* POST /resource_classes: create a new custom resource class
* PUT /resource_classes/{name}: update name of custom resource class
* DELETE /resource_classes/{name}: deletes a custom resource class
* GET /resource_classes/{name}: get a single resource class

Custom resource classes must begin with the prefix "CUSTOM\_" and contain only
the letters A through Z, the numbers 0 through 9 and the underscore "\_"
character.

1.3 member_of query parameter
-----------------------------

Version 1.3 adds support for listing resource providers that are members of
any of the list of aggregates provided using a ``member_of`` query parameter:

* /resource_providers?member_of=in:{agg1_uuid},{agg2_uuid},{agg3_uuid}

1.4 Filter resource providers by requested resource capacity (Maximum in Ocata)
-------------------------------------------------------------------------------

The 1.4 version adds support for querying resource providers that have the
ability to serve a requested set of resources. A new "resources" query string
parameter is now accepted to the `GET /resource_providers` API call. This
parameter indicates the requested amounts of various resources that a provider
must have the capacity to serve. The "resources" query string parameter takes
the form:

``?resources=$RESOURCE_CLASS_NAME:$AMOUNT,$RESOURCE_CLASS_NAME:$AMOUNT``

For instance, if the user wishes to see resource providers that can service a
request for 2 vCPUs, 1024 MB of RAM and 50 GB of disk space, the user can issue
a request to:

`GET /resource_providers?resources=VCPU:2,MEMORY_MB:1024,DISK_GB:50`

If the resource class does not exist, then it will return a HTTP 400.

.. note:: The resources filtering is also based on the `min_unit`, `max_unit`
    and `step_size` of the inventory record. For example, if the `max_unit` is
    512 for the DISK_GB inventory for a particular resource provider and a
    GET request is made for `DISK_GB:1024`, that resource provider will not be
    returned. The `min_unit` is the minimum amount of resource that can be
    requested for a given inventory and resource provider. The `step_size` is
    the increment of resource that can be requested for a given resource on a
    given provider.

1.5 DELETE all inventory for a resource provider
------------------------------------------------

Placement API version 1.5 adds DELETE method for deleting all inventory for a
resource provider. The following new method is supported:

* DELETE /resource_providers/{uuid}/inventories

1.6 Traits API
--------------

The 1.6 version adds basic operations allowing an admin to create, list, and
delete custom traits, also adds basic operations allowing an admin to attach
traits to a resource provider.

The following new routes are added:

* GET /traits: Returns all resource classes.
* PUT /traits/{name}: To insert a single custom trait.
* GET /traits/{name}: To check if a trait name exists.
* DELETE /traits/{name}: To delete the specified trait.
* GET /resource_providers/{uuid}/traits: a list of traits associated
  with a specific resource provider
* PUT /resource_providers/{uuid}/traits: Set all the traits for a
  specific resource provider
* DELETE /resource_providers/{uuid}/traits: Remove any existing trait
  associations for a specific resource provider

Custom traits must begin with the prefix "CUSTOM\_" and contain only
the letters A through Z, the numbers 0 through 9 and the underscore "\_"
character.

1.7 Idempotent PUT /resource_classes/{name}
-------------------------------------------

The 1.7 version changes handling of `PUT /resource_classes/{name}` to be a
create or verification of the resource class with `{name}`. If the resource
class is a custom resource class and does not already exist it will be created
and a ``201`` response code returned. If the class already exists the response
code will be ``204``. This makes it possible to check or create a resource
class in one request.

1.8 Require placement 'project_id', 'user_id' in PUT /allocations
-----------------------------------------------------------------

The 1.8 version adds ``project_id`` and ``user_id`` required request parameters
to ``PUT /allocations``.

1.9 Add GET /usages
--------------------

The 1.9 version adds usages that can be queried by a project or project/user.

The following new routes are added:

``GET /usages?project_id=<project_id>``

   Returns all usages for a given project.

``GET /usages?project_id=<project_id>&user_id=<user_id>``

   Returns all usages for a given project and user.

1.10 Allocation candidates (Maximum in Pike)
--------------------------------------------

The 1.10 version brings a new REST resource endpoint for getting a list of
allocation candidates. Allocation candidates are collections of possible
allocations against resource providers that can satisfy a particular request
for resources.

1.11 Add 'allocations' link to the ``GET /resource_providers`` response
-----------------------------------------------------------------------

The ``/resource_providers/{rp_uuid}/allocations`` endpoint has been available
since version 1.0, but was not listed in the ``links`` section of the
``GET /resource_providers`` response. The link is included as of version 1.11.

1.12 PUT dict format to /allocations/{consumer_uuid}
----------------------------------------------------

In version 1.12 the request body of a ``PUT /allocations/{consumer_uuid}``
is expected to have an `object` for the ``allocations`` property, not as
`array` as with earlier microversions. This puts the request body more in
alignment with the structure of the ``GET /allocations/{consumer_uuid}``
response body. Because the `PUT` request requires `user_id` and
`project_id` in the request body, these fields are added to the `GET`
response. In addition, the response body for ``GET /allocation_candidates``
is updated so the allocations in the ``alocation_requests`` object work
with the new `PUT` format.

1.13 POST multiple allocations to /allocations
----------------------------------------------

Version 1.13 gives the ability to set or clear allocations for more than
one consumer uuid with a request to ``POST /allocations``.

1.14 Add nested resource providers
----------------------------------

The 1.14 version introduces the concept of nested resource providers. The
resource provider resource now contains two new attributes:

* ``parent_provider_uuid`` indicates the provider's direct parent, or null if
  there is no parent. This attribute can be set in the call to ``POST
  /resource_providers`` and ``PUT /resource_providers/{uuid}`` if the attribute
  has not already been set to a non-NULL value (i.e. we do not support
  "reparenting" a provider)
* ``root_provider_uuid`` indicates the UUID of the root resource provider in
  the provider's tree. This is a read-only attribute

A new ``in_tree=<UUID>`` parameter is now available in the ``GET
/resource-providers`` API call. Supplying a UUID value for the ``in_tree``
parameter will cause all resource providers within the "provider tree" of the
provider matching ``<UUID>`` to be returned.

1.15 Add 'last-modified' and 'cache-control' headers
----------------------------------------------------

Throughout the API, 'last-modified' headers have been added to GET responses
and those PUT and POST responses that have bodies. The value is either the
actual last modified time of the most recently modified associated database
entity or the current time if there is no direct mapping to the database. In
addition, 'cache-control: no-cache' headers are added where the 'last-modified'
header has been added to prevent inadvertent caching of resources.

1.16 Limit allocation candidates
--------------------------------

Add support for a ``limit`` query parameter when making a
``GET /allocation_candidates`` request. The parameter accepts an integer
value, `N`, which limits the maximum number of candidates returned.

1.17 Add 'required' parameter to the allocation candidates (Maximum in Queens)
------------------------------------------------------------------------------

Add the `required` parameter to the `GET /allocation_candidates` API. It
accepts a list of traits separated by `,`. The provider summary in the response
will include the attached traits also.

1.18 Support ?required=<traits> queryparam on GET /resource_providers
---------------------------------------------------------------------

Add support for the `required` query parameter to the `GET /resource_providers`
API. It accepts a comma-separated list of string trait names. When specified,
the API results will be filtered to include only resource providers marked with
all the specified traits. This is in addition to (logical AND) any filtering
based on other query parameters.

Trait names which are empty, do not exist, or are otherwise invalid will result
in a 400 error.

1.19 Include generation and conflict detection in provider aggregates APIs
--------------------------------------------------------------------------

Enhance the payloads for the `GET /resource_providers/{uuid}/aggregates`
response and the `PUT /resource_providers/{uuid}/aggregates` request and
response to be identical, and to include the ``resource_provider_generation``.
As with other generation-aware APIs, if the ``resource_provider_generation``
specified in the `PUT` request does not match the generation known by the
server, a 409 Conflict error is returned.

1.20 Return 200 with provider payload from POST /resource_providers
-------------------------------------------------------------------

The `POST /resource_providers` API, on success, returns 200 with a payload
representing the newly-created resource provider, in the same format as the
corresponding `GET /resource_providers/{uuid}` call. This is to allow the
caller to glean automatically-set fields, such as UUID and generation, without
a subsequent GET.

1.21 Support ?member_of=<aggregates> queryparam on GET /allocation_candidates
-----------------------------------------------------------------------------

Add support for the `member_of` query parameter to the `GET
/allocation_candidates` API. It accepts a comma-separated list of UUIDs for
aggregates. Note that if more than one aggregate UUID is passed, the
comma-separated list must be prefixed with the "in:" operator. If this
parameter is provided, the only resource providers returned will be those in
one of the specified aggregates that meet the other parts of the request.

1.22 Support forbidden traits on resource providers and allocations candidates
------------------------------------------------------------------------------

Add support for expressing traits which are forbidden when filtering
``GET /resource_providers`` or ``GET /allocation_candidates``. A forbidden
trait is a properly formatted trait in the existing ``required`` parameter,
prefixed by a ``!``. For example ``required=!STORAGE_DISK_SSD`` asks that the
results not include any resource providers that provide solid state disk.

1.23 Include code attribute in JSON error responses
---------------------------------------------------

JSON formatted error responses gain a new attribute, ``code``, with a value
that identifies the type of this error. This can be used to distinguish errors
that are different but use the same HTTP status code. Any error response which
does not specifically define a code will have the code
``placement.undefined_code``.

1.24 Support multiple ?member_of queryparams
--------------------------------------------

Add support for specifying multiple ``member_of`` query parameters to the ``GET
/resource_providers`` API. When multiple ``member_of`` query parameters are
found, they are AND'd together in the final query. For example, issuing a
request for ``GET /resource_providers?member_of=agg1&member_of=agg2`` means get
the resource providers that are associated with BOTH agg1 and agg2. Issuing a
request for ``GET /resource_providers?member_of=in:agg1,agg2&member_of=agg3``
means get the resource providers that are associated with agg3 and are also
associated with *any of* (agg1, agg2).

1.25 Granular resource requests to ``GET /allocation_candidates``
-----------------------------------------------------------------

``GET /allocation_candidates`` is enhanced to accept numbered groupings of
resource, required/forbidden trait, and aggregate association requests. A
``resources`` query parameter key with a positive integer suffix (e.g.
``resources42``) will be logically associated with ``required`` and/or
``member_of`` query parameter keys with the same suffix (e.g. ``required42``,
``member_of42``). The resources, required/forbidden traits, and aggregate
associations in that group will be satisfied by the same resource provider in
the response. When more than one numbered grouping is supplied, the
``group_policy`` query parameter is required to indicate how the groups should
interact. With ``group_policy=none``, separate groupings - numbered or
unnumbered - may or may not be satisfied by the same provider. With
``group_policy=isolate``, numbered groups are guaranteed to be satisfied by
*different* providers - though there may still be overlap with the unnumbered
group. In all cases, each ``allocation_request`` will be satisfied by providers
in a single non-sharing provider tree and/or sharing providers associated via
aggregate with any of the providers in that tree.

The ``required`` and ``member_of`` query parameters for a given group are
optional.  That is, you may specify ``resources42=XXX`` without a corresponding
``required42=YYY`` or ``member_of42=ZZZ``. However, the reverse (specifying
``required42=YYY`` or ``member_of42=ZZZ`` without ``resources42=XXX``) will
result in an error.

The semantic of the (unnumbered) ``resources``, ``required``, and ``member_of``
query parameters is unchanged: the resources, traits, and aggregate
associations specified thereby may be satisfied by any provider in the same
non-sharing tree or associated via the specified aggregate(s).
