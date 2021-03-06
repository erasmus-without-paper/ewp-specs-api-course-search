Simple Course Replication API
=============================

* [What is the status of this document?][statuses]
* [See the index of all other EWP Specifications][develhub]


Summary
-------

This document describes the **Simple Course Replication API**. This API can be
implemented by any HEI, even it is does not take part in EWP mobility process.
Once implemented, it allows the clients to replicate the catalogue of courses
conducted on this HEI. This in turn allows the clients to design rich course
searching user experience.

In the draft stages of the design process, this API was referred to as "Course
Search API". However, as the result of
[this issue](https://github.com/erasmus-without-paper/ewp-specs-api-course-replication/issues/3),
we have dropped this name along with some of its initially planned *filtering*
functionality.

As it has been explained in the [Courses API][courses-api], `Course` is just
one of the types of **learning opportunity**. This API MAY expose all types of
learning opportunities, not only courses. See Courses API for details.


Performance note
----------------

Once you implement this API, you effectively allow your requesters to download
the entire listing of your programmes and courses, and keep it synchronized
later on. This is probably a good idea from a business viewpoint (because your
HEIs' course catalogues will be easier discovered in external systems), but
it's necessarily so good an idea *for your servers*.

Depending on EWP's popularity and the number of LOS objects in your system,
implementing this particular API MAY introduce a significant load on your
servers - primarily via the Courses API, not this one (especially if you decide
to make these two APIs available anonymously).

Therefore:

 * To avoid potential problems with performance, proper caching is recommended
   (not only in *this* API, but - primarily - in the Courses API).

 * Keep in mind, that this functionality is *not* essential for EWP's mobility
   workflow, and you are free to "skip" this API if you're afraid of
   performance issues. It is also perfectly okay to first make it available,
   and then **change your mind later** (if it proves troublesome).


Request method
--------------

 * Requests MUST be made with either HTTP GET or HTTP POST method. Servers MUST
   support both these methods. Servers SHOULD reject all other request methods.

 * Clients are advised to use POST when passing large number of parameters
   (servers MAY set a limit on their GET query string length).


Request parameters
------------------

Parameters MUST be provided in the regular `application/x-www-form-urlencoded`
format.


### `hei_id` (required)

ID of the institution in which the courses are conducted. This parameter MUST
be validated by the server even if the server covers only a single HEI.


### `modified_since` (optional)

A datetime string in the [`xs:dateTime` format][xs-datetime], e.g.
`2004-02-12T15:19:21+01:00`.

If not given, then servers MUST return a full list of all their LOS IDs.
This includes courses not currently conducted (if the server keeps record of
such courses).

If `modified_since` is given, then the server SHOULD filter the returned LOS
IDs to the ones which have been *modified* after the given point in time.

 * Servers MAY include courses which were *not* modified. For example, if the
   server only *suspects* that the course was modified, then it is okay to
   include the course's ID in the response.

 * Servers MAY ignore the `modified_since` parameter completely, and *always*
   respond with the full list of course IDs. If, for some reason, the server
   cannot reliably identify when courses are updated, then it's even *better*
   to do so.

 * Servers MAY cache the response (so that it will be generated faster for the
   next client), but if they do, then they MUST do it in a way which will
   allow the clients to receive all the changes in a standard, predictable way
   (though with a delay).

   For example, if you want to cache the response for 24 hours, then such
   cached response MUST also include all courses modified since 24 hours
   *before* the moment such response was generated. Otherwise, the clients
   would be missing some changes.


Security
--------

This version of this API uses [standard EWP Authentication and Security,
Version 2][sec-v2]. Server implementers choose which security methods they
support by declaring them in their manifest's API-entry.

This API (as well as the Courses API), provides data which is also usually
accessible to the anonymous public by other channels. It is RECOMMENDED for
server implementers to not be overly strict on security methods they require
(i.e. it is RECOMMENDED to *not* require extra layers of encryption in requests
and responses - TLS seems more than enough). Server implementers MAY also
consider allowing this API to be accessed by anonymous clients.


Handling of invalid parameters
------------------------------

 * General [error handling rules][error-handling] apply.
 * Invalid (or unknown) `hei_id` values MUST result in a HTTP 400 error
   response.


Response
--------

If the request was valid, then servers MUST respond with a valid XML document
described by the [response.xsd](response.xsd) schema. See the schema
annotations for further information.


Data model entities involved in the response
--------------------------------------------

 * Learning Opportunity Specification


[develhub]: http://developers.erasmuswithoutpaper.eu/
[statuses]: https://github.com/erasmus-without-paper/ewp-specs-management#statuses
[registry-spec]: https://github.com/erasmus-without-paper/ewp-specs-api-registry
[discovery-api]: https://github.com/erasmus-without-paper/ewp-specs-api-discovery
[echo]: https://github.com/erasmus-without-paper/ewp-specs-api-echo
[error-handling]: https://github.com/erasmus-without-paper/ewp-specs-architecture#error-handling
[institutions-api]: https://github.com/erasmus-without-paper/ewp-specs-api-institutions
[courses-api]: https://github.com/erasmus-without-paper/ewp-specs-api-courses
[sec-v2]: https://github.com/erasmus-without-paper/ewp-specs-sec-intro/tree/stable-v2
[xs-datetime]: https://www.w3.org/TR/xmlschema11-2/#dateTime
