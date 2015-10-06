All functions are executed as HTTP(S) requests over TCP. All parameters are to
be passed as query parameters unless otherwise indicated. All returned XML/JSON
data is UTF-8 encoded unless otherwise specified. All query parameters should
be UTF-8 and URL encoded, i.e.::

    query-param = URL-ENCODE(UTF8-ENCODE(param=value)).

The functions are divided into two categories. Functions specific to searching
and retrieving of items and the their information such as SEARCH and TV-SEARCH
and functions that are for site/user account management such as CAPS and
REGISTER.

Any conforming implementation should support the CAPS and SEARCH functions.
Other functions are optional and if not supported will return the "203 Function
Not Available" when invoked. 


CAPS
====

The ``CAPS`` function is used to query the server for supported features and
the protocol version and other meta data relevant to the implementation. This
function doesn't require the client to provide any login information but
can be executed out of "login session".


Returned Fields                |
-------------------------------|-------------------------------------------
server/version                 | The version of the protocol implemented by the server. All implementations should be backwards compatible.
limits                         | The limit and defaults to the number of search results returned.
retention                      | Server retention (how many days NZB information is stored before being purged).
category                       | Defines a searchable category which might have any number of subcategories.
category/id                    | Unique category ID, can be either one of the standard category IDs or a site specific ID.
category/name                  | Any descriptive name for the category. Can be site/language specific.                            
category/description           | A description of the contents of the category.
category/subcat                | A subcategory.
category/subcat/id             | Unique category ID, can be either one of the standard category IDs or a site specific ID.
category/subcat/name           | Any descriptive name for the category. Can be site/language specific.
category/subcat/description    | A description of the contents of the category.
groups                         | Defines a list of active, indexed usenet groups.
group/name                     | Name of usenet group.
group/description              | Description of usenet group.
group/lastupdate               | Date and time usenet group was last updated.
genres                         | Defines a list of active genres.
genre/id                       | Id of genre.
genre/name                     | Name of genre.
genre/categoryid               | The category the genre is associate with.


**HTTP Method**

 GET
 
**HTTP Response**

 200 OK
  
Parameters         |
-------------------|--------------------------------------
``t=caps``         | Caps function, must always be "caps".
 

Optional parameters |
--------------------|------------------------------------------------------
``o=xxx``           |Output format, either "JSON" or "XML. Default is "XML".

**Examples**

1. Normal behavior

    **Request**:
        
    ``GET http://servername.com/api?t=caps``

    **Response**:
    
    ``200 OK``

    **Content**:
    
        <?xml version="1.0" encoding="UTF-8"?>
          <caps>
            <!-- server information -->
            <server version="1.0" title="Newznab" strapline="A great usenet indexer" 
                    email="info@newznab.com" url="http://servername.com/" 
                    image="http://servername.com/theme/black/images/banner.jpg"/>

            <!-- limit parameter range -->
            <limits max="100" default="50"/>

            <!-- the server NZB retention -->
            <retention days="400"/>

            <!-- registration available or not -->
            <registration available="yes" open="yes" />

            <!-- 
                 The search functions available at the server 
                 The only currently defined search functions are SEARCH and TV-SEARCH.
                 Any conforming implementation should at least support the basic search.
                 Other search functions are optional.
            -->
            <searching>
                <search available="yes"/>
                <tv-search available="yes"/>
                <movie-search available="no"/>
            </searching>

            <!-- supported categories -->
            <categories>
                <category id="1000" name="Console">
                  <subcat id="1010" name="NDS"/>
                  <subcat id="1020" name="PSP"/>
                </category>
                <category id="2000" name="Movies">
                  <subcat id="2010" name="Foreign"/>
                </category>

                <!-- site specific categories -->
                <category id="1000001" name="Debian"           description="Latest Debian stuff"/>
                <category id="1000002" name="Mandrake 2010"    description="Mandrake 2010">
                  <subcat id="1000003" name="Mandrake 2010 HD" description="Mandrake HD stuff"/>
                  <subcat id="1000004" name="Mandrake 2010 SD" description="Mandrake SD stuff"/>
                </category>
                <!-- etc.. -->
            </categories>               
          </caps>
        </xml>

REGISTER
--------

The ``REGISTER`` function is used for automatically creating and registering
user account.  This is an optional function and may or may not be available at
a site. It is also possible that function is available but currently
registrations at the site are closed. 

The only prerequisite for registering an account is a valid email address
and any server policies.  It is at the server administration discretion to
allow or deny registrations based on for example the validity of the email
address or the the current client host address.

On successful registration a valid username, password and api key are
returned to the caller.  On error an appropriate error code is returned.

**HTTP Method**

``GET``

**HTTP Response**

``200 OK``

Parameters         |
-------------------|----
``t=register``     | Register function, must always be "register"
``email=xxx``      | A valid email address to be used for registration. (URL/UTF-8 encoded).
  
**Examples**

1. Normal behavior

    **Request**:

    ``GET HTTP://servername.com/api?t=register&email=john.joe%40acme.com``

    **Response**:

    ``200 OK``

    **Content**:

        <?xml version="1.0" encoding="UTF-8"?>
        <register username="user123" password="pass123" apikey="abcabcd11234abc" />

2. Denial

    **Request**:

    ``GET HTTP://servername.com/api?t=register&email=john.joe%40acme.com``

    **Response**:

    ``200 OK``

    **Content**:

        <?xml version="1.0" encoding="UTF-8"?>
        <error code="103" description="Registration denied"/>

3. Registration limit imposed

    **Request**:

    ``GET HTTP://servername.com/api?t=register&email=john.joe%40acme.com``

    **Response**:

    ``200 OK``

    **Content**:

        <?xml version="1.0" encoding="UTF-8"?>
        <error code="104" description="No more registrations allowed"/>

4. Registration disabled

    **Request**:

    ``GET HTTP://servername.com/api?t=register&email=john.joe%40acme.com``

    **Response**:

    ``200 OK``

    **Content**:

        <?xml version="1.0" encoding="UTF-8"?>
        <error code="203" description="Function not available"/>


SEARCH
------

The ``SEARCH`` function searches the index for items matching the search
criteria. On successful search the response contains a list of found items.
Even if search matched nothing an empty response set is created and returned.
This function requires passing the user credentials.

Searches that include categories that are not supported by the server are
still executed but the non-supported categories are simply skipped. This
basically treats such a search simply as a "no match" but allows the same
query to be ran simultaneously against several servers. 

The list of search categories specifies a logical OR condition. I.e. an item
matching the search input in any of the specified categories is considered a
match and is returned. E.g. a search searching for "linux" in "computer" and
"ebook" categories searches for matching items in "computer" and "ebook" but
does not search for example the "movies" category.  Items found in either
group are then combined into a single result set. If the input string for
search is empty all items (within the server/query limits) are returned for
the matching categories. 

When performing the query the categories to be searched are concatenated into
a single query parameter by ``,`` (comma). For example ``cat=200,300,400``,
which is then URL encoded.

The returned XML data stream is RSS 2.0 compliant and also contains
additional information in the extra namespace.

Response-offset field identifies the current subset of all the matches that
are being transmitted in the response. In other words, if a search for
"disco" finds more matches than the server is capable of transmitting in a
single response, the response needs to be split into several responses. Then
it is's the clients responsibility to repeat the same query with same
parameters but specify an increased offset in order to return the next set of
results. 

If offset query parameter is not used response data contains items between 0
offset - limit.  If offset query parameter is out of bounds an empty result
set is returned.

Attrs parameter provides a comma "," separated list of additional (extended)
attributes that the search should return if they are applicable to the
current item.  If attrs is not specified a set of default parameters is
returned.

Important fields of the returned data (RSS)|
-------------------------------------------|------
title                                      | Title of the found item.
guid                                       | A globally unique (GUID) item identifier.
pubdate                                    | The publishing date in RSS date object as specified by RFC822/2822. (not the Usenet date)
category                                   | The category the NZB belongs to. (This is human readable for RSS. More precise category is found in additional data)
enclosure                                  | The NZB url

**HTTP Method**

  ``GET``
  
**HTTP Response**

  ``200 OK``
  
Parameters         |
-------------------|---
``t=search``       | Search function, must always be "search"
``apikey=xxxx``    | User's key as provided by the service provider.
  
Optional parameters|
-------------------|---
``q=xxxx``         | Search input (URL/UTF-8 encoded). Case insensitive.
``group=xxxx``     | List of usenet groups to search delimited by ","
``limit=123``      | Upper limit for the number of items to be returned.
``cat=xxx``        | List of categories to search delimited by ","
``o=xxx``          | Output format, either "JSON" or "XML". Default is "XML".
``attrs=xxx``      | List of requested extended attributes delimeted by ","
``extended=1``     | List all extended attributes (attrs ignored)
``del=1``          | Delete the item from a users cart on download.
``maxage=123``     | Only return results which were posted to usenet in the last x days.
``offset=50``      | The 0 based query offset defining which part of the response we want.

**Examples**

1. Normal behavior

    **Request**:

    ``GET http://servername.com/api?t=search&apikey=xxxxx&q=a%20tv%20show``

    **Response**:

    ``200 OK``

    **Content**:

        <?xml version="1.0" encoding="UTF-8"?>
        <rss version="2.0">

        <channel>
        <title>example.com</tile>
        <description>example.com API results</description>
        <!--  
           More RSS content
         -->          
 
        <!-- offset is the current offset of the response
             total is the total number of items found by the query 
         --> 
        <newznab:response offset="0" total="2344"/>

        <item>        
          <!-- Standard RSS 2.0 Data -->
          <title>A.Public.Domain.Tv.Show.S06E05</title> 
          <guid isPermaLink="true">http://servername.com/rss/viewnzb/e9c515e02346086e3a477a5436d7bc8c</guid> 
          <link>http://servername.com/rss/nzb/e9c515e02346086e3a477a5436d7bc8c&amp;i=1&amp;r=18cf9f0a736041465e3bd521d00a90b9</link> 
          <comments>http://servername.com/rss/viewnzb/e9c515e02346086e3a477a5436d7bc8c#comments</comments>  
          <pubDate>Sun, 06 Jun 2010 17:29:23 +0100</pubDate> 
          <category>TV > XviD</category>  
          <description>Some TV show</description>
          <enclosure url="http://servername.com/rss/nzb/e9c515e02346086e3a477a5436d7bc8c&amp;i=1&amp;r=18cf9f0a736041465e3bd521d00a90b9" length="154653309" type="application/x-nzb" /> 

          <!-- Additional attributes -->
          <newznab:attr name="category" value="2000"/> 
          <newznab:attr name="category" value="2030"/> 
          <newznab:attr name="size"     value="4294967295"/>
        </item>
        </channel>
        </rss>


2. No items matched the search criteria.

    **Request**:

    ``GET http://servername.com/api?t=search&apikey=xxxxx&q=linux%20image``

    **Response**:

    ``200 OK``

    **Content**:

        <?xml version="1.0" encoding="UTF-8"?>
        <rss>
        <channel>
            <newznab:response offset="0" total="0"/>
        </channel>
        </rss>

3. Query could not be completed because user credentials are broken

    **Request**:
    
    ``GET http://servername.com/api?t=search&apikey=xxxxx&q=linux%20image``
    
    **Response**:
    
    ``200  OK``
    
    **Content**:

        <?xml version="1.0" encoding="UTF-8"?>
        <error code="100" description="Incorrect user credentials"/>


4. Query could not be completed because it was malformed

    **Request**:
    
    ``GET http://servername.com/api?t=search&apikey=xxxxx&q=linux%20image``
    
    **Response**:
    
    ``200 OK``
    
    **Content**:

        <?xml version="1.0" encoding="UTF-8"/>
        <error code="200" description="Missing parameter: key"/>

TV-SEARCH
---------

The ``TV-SEARCH`` function searches the index in the TV category for items
matching the search criteria.  The criteria includes query string and in
addition information about season and episode. On successful search the
response contains a list of items that matched the query. Even if the search
matched nothing an empty but valid response is created and returned. This
function requires passing the user credentials.

The returned XML data stream is RSS 2.0 compliant and also contains additional
information in the extra namespace and optionally TV specific information.

**HTTP Method**

 GET

**HTTP Response**

 200 OK

Parameters         |
-------------------|---
``t=tvsearch``     | TV-Search function, must always be "tvsearch".
``apikey=xxx``     | User's key as provided by the service provider.
 
Optional parameters|
-------------------|---
``limit=123``      | Upper limit for the number of items to be returned, e.g. 123.
``rid=xxxx``       | TVRage id of the item being queried.
``cat=xxx``        | List of categories to search delimited by ","
``season=xxxx``    | Season string, e.g S13 or 13 for the item being queried.
``q=xxxx``         | Search input (URL/UTF-8 encoded). Case insensitive.
``ep=xxx``         | Episode string, e.g E13 or 13 for the item being queried.
``o=xml``          | Output format, either "JSON" or "XML". Default is "XML".
``attrs=xxx``      | List of requested extended attributes delimeted by ","
``extended=1``     | List all extended attributes (attrs ignored)
``del=1``          | Delete the item from a users cart on download.
``maxage=123``     | Only return results which were posted to usenet in the last x days.
``offset=50``      | The 0 based query offset defining which part of the response we want.
 
**Examples**

1. Normal behavior

    **Request**:
    ``GET http://servername.com/api?t=tvsearch&apikey=xxx&q=title&season=S03``

    **Response**:
    
    ``200 OK``
    
    **Content**:
    
        <?xml version="1.0" encoding="UTF-8"?>
        <rss version="2.0">
        <channel>
        <title>example.com</title>
        <description>example.com API results</description>
        <!-- 
          More RSS content
        -->

        <!-- offset is the current offset of the response
             total is the total number of items found by the query
        -->
        <newznab:response offset="0" total="1234"/>

        <item>
          <!-- Standard RSS 2.0 data -->
          <title>A.Public.Domain.Tv.Show.S06E05</title> 
          <guid isPermaLink="true">http://servername.com/rss/viewnzb/e9c515e02346086e3a477a5436d7bc8c</guid> 
          <link>http://servername.com/rss/nzb/e9c515e02346086e3a477a5436d7bc8c&amp;i=1&amp;r=18cf9f0a736041465e3bd521d00a90b9</link> 
          <comments>http://servername.com/rss/viewnzb/e9c515e02346086e3a477a5436d7bc8c#comments</comments>  
          <pubDate>Sun, 06 Jun 2010 17:29:23 +0100</pubDate> 
          <category>TV > XviD</category>  
          <description>Some TV show</description>
          <enclosure url="http://servername.com/rss/nzb/e9c515e02346086e3a477a5436d7bc8c&amp;i=1&amp;r=18cf9f0a736041465e3bd521d00a90b9" length="154653309" type="application/x-nzb" />               

          <!-- Additional attributes -->
          <newznab:attr name="category" value="5030"/>
          <newznab:attr name="size"     value="154653309"/>
          <newznab:attr name="season"   value="3"/>
          <newznab:attr name="episode"  value="2"/>
        </item>         

        <item>
          <!-- Standard RSS 2.0 data -->
          <title>A.Public.Domain.Tv.Show.S06E05</title> 
          <guid isPermaLink="true">http://servername.com/rss/viewnzb/e9c515e02346086e3a477a5436d7bc8c</guid> 
          <link>http://servername.com/rss/nzb/e9c515e02346086e3a477a5436d7bc8c&amp;i=1&amp;r=18cf9f0a736041465e3bd521d00a90b9</link> 
          <comments>http://servername.com/rss/viewnzb/e9c515e02346086e3a477a5436d7bc8c#comments</comments>  
          <pubDate>Sun, 06 Jun 2010 17:29:23 +0100</pubDate> 
          <category>TV > XviD</category>  
          <description>Some TV show</description>
          <enclosure url="http://servername.com/rss/nzb/e9c515e02346086e3a477a5436d7bc8c&amp;i=1&amp;r=18cf9f0a736041465e3bd521d00a90b9" length="154653309" type="application/x-nzb" />               

          <!-- Additional attributes -->
          <newznab:attr name="category" value="5000" /> 
          <newznab:attr name="category" value="5030" /> 
          <newznab:attr name="size"     value="4294967295" /> 
          <newznab:attr name="season"   value="3"/>
          <newznab:attr name="episode"  value="1"/>
        </item>         

        <!-- more items to follow -->

        </channel>
        </rss>

MOVIE-SEARCH
------------

The ``MOVIE-SEARCH`` function searches the index for items matching an IMDb ID or
search query.  On successful search the response contains a list of items that
matched the query. Even if the search matched nothing an empty but valid
response is created and returned. This function requires passing the user
credentials.

The returned XML data stream is RSS 2.0 compliant and also contains additional
information in the extra namespace and optionally movie specific information.

**HTTP Method**

 GET

**HTTP Response**

 200 OK

Parameters         |
-------------------|---
``t=movie``        | Movie-Search function, must always be "movie".
``apikey=xxx``     | User's key as provided by the service provider.
 
Optional parameters|
-------------------|---
``limit=123``      | Upper limit for the number of items to be returned, e.g. 123.
``imdbid=xxxx``    | IMDB id of the item being queried e.g. 0058935.
``cat=xxx``        | List of categories to search delimited by ","
``genre=xxx``      | A genre string i.e. 'Romance' would match '(Comedy, Drama, Indie, Romance)'
``q=xxxx``         | Search input (URL/UTF-8 encoded). Case insensitive.
``o=xml``          | Output format, either "JSON" or "XML". Default is "XML".
``attrs=xxx``      | List of requested extended attributes delimeted by ","
``extended=1``     | List all extended attributes (attrs ignored)
``del=1``          | Delete the item from a users cart on download.
``maxage=123``     | Only return results which were posted to usenet in the last x days.
``offset=50``      | The 0 based query offset defining which part of the response we want.
 
**Examples**

1. Normal behavior

    **Request**:
        
    ``GET http://servername.com/api?t=movie&apikey=xxx&imdbid=0058935``

    **Response**:
    
    ``200 OK``
    
    **Content**:
    
        <?xml version="1.0" encoding="UTF-8"?>
        <rss version="2.0">
        <channel>
            <title>example.com</title>
            <description>example.com API results</description>
            <!-- 
              More RSS content
            -->

            <!-- offset is the current offset of the response
                 total is the total number of items found by the query
            -->
            <newznab:response offset="0" total="1234"/>

            <item>
              <!-- Standard RSS 2.0 data -->
              <title>A.Public.Domain.Movie.720p.DTS.x264</title> 
              <guid isPermaLink="true">http://servername.com/rss/viewnzb/e9c515e02346086e3a477a5436d7bc8c</guid> 
              <link>http://servername.com/rss/nzb/e9c515e02346086e3a477a5436d7bc8c&amp;i=1&amp;r=18cf9f0a736041465e3bd521d00a90b9</link> 
              <comments>http://servername.com/rss/viewnzb/e9c515e02346086e3a477a5436d7bc8c#comments</comments>  
              <pubDate>Sun, 06 Jun 2010 17:29:23 +0100</pubDate> 
              <category>Movie > XviD</category>  
              <description>Some movie</description>
              <enclosure url="http://servername.com/rss/nzb/e9c515e02346086e3a477a5436d7bc8c&amp;i=1&amp;r=18cf9f0a736041465e3bd521d00a90b9" length="154653309" type="application/x-nzb" />               

              <!-- Additional attributes -->
              <newznab:attr name="category" value="2000" /> 
              <newznab:attr name="category" value="2030" /> 
              <newznab:attr name="size"     value="4294967295" /> 
            </item>         

        </channel>
        </rss>

MUSIC-SEARCH
------------

The ``MUSIC-SEARCH`` function searches the index for items matching music properties.
On successful search the response contains a list of items that matched the
query. Even if the search matched nothing an empty but valid response is
created and returned. This function requires passing the user credentials.

The returned XML data stream is RSS 2.0 compliant and also contains additional
information in the extra namespace and optionally music specific information.

**HTTP Method**

 GET

**HTTP Response**

 200 OK

Parameters         |
-------------------|---
``t=music``        | Music-Search function, must always be "music".
``apikey=xxx``     | User's key as provided by the service provider.
 
Optional Parameters|
-------------------|---
``limit=123``      | Upper limit for the number of items to be returned, e.g. 123.
``album=xxxx``     | Album title (URL/UTF-8 encoded). Case insensitive.
``artist=xxxx``    | Artist name (URL/UTF-8 encoded). Case insensitive.
``label=xxxx``     | Publisher/Label name (URL/UTF-8 encoded). Case insensitive.
``track=xxxx``     | Track name (URL/UTF-8 encoded). Case insensitive.
``year=xxxx``      | Four digit year of release.
``genre=123``      | List of music genre id's to search delimited by ",". See CAPS for available genres.
``cat=xxx``        | List of categories to search delimited by ","
``o=xml``          | Output format, either "JSON" or "XML". Default is "XML".
``attrs=xxx``      | List of requested extended attributes delimited by ","
``extended=1``     | List all extended attributes (attrs ignored)
``del=1``          | Delete the item from a users cart on download.
``maxage=123``     | Only return results which were posted to usenet in the last x days.
``offset=50``      | The 0 based query offset defining which part of the response we want.
 
**Examples**

1. Normal behavior

    **Request**:
      
    ``GET http://servername.com/api?t=music&apikey=xxx&album=Groovy&extended=1``

    **Response**:
    
    ``200 OK``
    
    **Content**:
        
        <?xml version="1.0" encoding="UTF-8"?>
        <rss version="2.0">
        <channel>
            <title>example.com</title>
            <description>example.com API results</description>
            <!-- 
              More RSS content
            -->

            <!-- offset is the current offset of the response
                 total is the total number of items found by the query
            -->
            <newznab:response offset="0" total="1234"/>

            <item>
              <!-- Standard RSS 2.0 data -->
              <title>A.Public.Domain.Album.Name</title> 
              <guid isPermaLink="true">http://servername.com/rss/viewnzb/e9c515e02346086e3a477a5436d7bc8c</guid> 
              <link>http://servername.com/rss/nzb/e9c515e02346086e3a477a5436d7bc8c&amp;i=1&amp;r=18cf9f0a736041465e3bd521d00a90b9</link> 
              <comments>http://servername.com/rss/viewnzb/e9c515e02346086e3a477a5436d7bc8c#comments</comments>  
              <pubDate>Sun, 06 Jun 2010 17:29:23 +0100</pubDate> 
              <category>Music > MP3</category>  
              <description>Some music</description>
              <enclosure url="http://servername.com/rss/nzb/e9c515e02346086e3a477a5436d7bc8c&amp;i=1&amp;r=18cf9f0a736041465e3bd521d00a90b9" length="154653309" type="application/x-nzb" />               

              <!-- Additional attributes -->
              <newznab:attr name="category" value="3000" /> 
              <newznab:attr name="category" value="3010" /> 
              <newznab:attr name="size"     value="144967295" /> 
              <newznab:attr name="artist"   value="Bob Smith" /> 
              <newznab:attr name="album"    value="Groovy Tunes" /> 
              <newznab:attr name="publisher" value="Epic Music" /> 
              <newznab:attr name="year"     value="2011" /> 
              <newznab:attr name="tracks"   value="track one|track two|track three" /> 
              <newznab:attr name="coverurl" value="http://servername.com/covers/music/12345.jpg" /> 
              <newznab:attr name="review"   value="This album is great" /> 
            </item>         

        </channel>
        </rss>


BOOK-SEARCH
-----------

The ``BOOK-SEARCH`` function searches the index for items matching book properties.
On successful search the response contains a list of items that matched the
query. Even if the search matched nothing an empty but valid response is
created and returned. This function requires passing the user credentials.

The returned XML data stream is RSS 2.0 compliant and also contains additional
information in the extra namespace and optionally music specific information.

**HTTP Method**

 GET

**HTTP Response**

 200 OK

Parameters         |
-------------------|---
``t=book``         | Book-Search function, must always be "book".
``apikey=xxx``     | User's key as provided by the service provider.

Optional Parameters|
-------------------|---
``limit=123``      | Upper limit for the number of items to be returned, e.g. 123.
``title=xxxx``     | Book title (URL/UTF-8 encoded). Case insensitive.
``author=xxxx``    | Author name (URL/UTF-8 encoded). Case insensitive.
``o=xml``          | Output format, either "JSON" or "XML". Default is "XML".
``attrs=xxx``      | List of requested extended attributes delimited by ","
``extended=1``     | List all extended attributes (attrs ignored)
``del=1``          | Delete the item from a users cart on download.
``maxage=123``     | Only return results which were posted to usenet in the last x days.
``offset=50``      | The 0 based query offset defining which part of the response we want.
 
**Examples**

1. Normal behavior

    **Request**:
        
    ``GET http://servername.com/api?t=book&apikey=xxx&author=Charles%20Dack&extended=1``

    **Response**:
 
    ``200 OK``
    
    **Content**:
        
        <?xml version="1.0" encoding="UTF-8"?>
        <rss version="2.0">
        <channel>
            <title>example.com</title>
            <description>example.com API results</description>
            <!-- 
              More RSS content
            -->

            <!-- offset is the current offset of the response
                 total is the total number of items found by the query
            -->
            <newznab:response offset="0" total="1234"/>

            <item>
              <!-- Standard RSS 2.0 data -->
              <title>A.Public.Domain.Book.Name</title> 
              <guid isPermaLink="true">http://servername.com/rss/viewnzb/e9c515e02346086e3a477a5436d7bc8c</guid> 
              <link>http://servername.com/rss/nzb/e9c515e02346086e3a477a5436d7bc8c&amp;i=1&amp;r=18cf9f0a736041465e3bd521d00a90b9</link> 
              <comments>http://servername.com/rss/viewnzb/e9c515e02346086e3a477a5436d7bc8c#comments</comments>  
              <pubDate>Sun, 06 Jun 2010 17:29:23 +0100</pubDate> 
              <category>Misc > Ebook</category>  
              <description>Some book</description>
              <enclosure url="http://servername.com/rss/nzb/e9c515e02346086e3a477a5436d7bc8c&amp;i=1&amp;r=18cf9f0a736041465e3bd521d00a90b9" length="154653309" type="application/x-nzb" />               

              <!-- Additional attributes -->
              <newznab:attr name="category" value="7020" /> 
              <newznab:attr name="size"     value="144967295" /> 
              <newznab:attr name="author"   value="Charles Dack" /> 
              <newznab:attr name="title"    value="Weather and Folk Lore of Peterborough and District" /> 
              <newznab:attr name="review"   value="This book is a classic" /> 
            </item>         

        </channel>
        </rss>		 
	 
DETAILS
-------

The ``DETAILS`` function returns all information for a particular Usenet (NZB) item.
The response contains the generic RSS part + full extra information + full
type/category specific information.
  
**HTTP Method**

  GET

**HTTP Response**

  200 OK
  
Parameters         |
-------------------|---
``t=details``      | Details function, must always be "details".
``id=xxxx``        | The GUID of the item being queried. 
``apikey=xxxx``    | User's key as provided by the service provider.
  
Optional Parameters|
-------------------|--
``o=xxx``          | Output format, either "JSON" or "XML". Default is "XML".

**Examples**

1. Normal behavior

    **Request**:
        
    ``GET http://servername.com/api?t=details&apikey=xxxxx&guid=xxxxxxxxx``

    **Response**:
    
    ``200 OK``
    
    **Content**:
    
        <?xml version="1.0" encoding="UTF-8"?>
        <rss version="2.0">
        <channel>
          <item>
            <!-- Standard RSS 2.0 Data -->
            <title>A.Public.Domain.Tv.Show.S06E05</title> 
            <guid isPermaLink="true">http://servername.com/rss/viewnzb/e9c515e02346086e3a477a5436d7bc8c</guid> 
            <link>http://servername.com/rss/nzb/e9c515e02346086e3a477a5436d7bc8c&amp;i=1&amp;r=18cf9f0a736041465e3bd521d00a90b9</link> 
            <comments>http://servername.com/rss/viewnzb/e9c515e02346086e3a477a5436d7bc8c#comments</comments>  
            <pubDate>Sun, 06 Jun 2010 17:29:23 +0100</pubDate> 
            <category>TV > XviD</category>  
            <description>Some TV show</description>
            <enclosure url="http://servername.com/rss/nzb/e9c515e02346086e3a477a5436d7bc8c&amp;i=1&amp;r=18cf9f0a736041465e3bd521d00a90b9" length="154653309" type="application/x-nzb" /> 

            <!-- 
                 Additional attributes 
                 Details function returns all possible attributes that are 1) known and 2) applicable
                 for the item requested.
            -->
            <newznab:attr name="category"   value="2000" /> 
            <newznab:attr name="category"   value="2030" /> 
            <newznab:attr name="size"       value="4294967295" /> 
            <newznab:attr name="files"      value="107" /> 
            <newznab:attr name="poster"     value="example@4u.net (example)" /> 
            <newznab:attr name="grabs"      value="1" /> 
            <newznab:attr name="comments"   value="0" /> 
            <newznab:attr name="usenetdate" value="Tue, 22 Jun 2010 06:54:22 +0100" />  
            <newznab:attr name="group"      value="alt.binaries.teevee" /> 
    
          </item>  
        </channel>
        </rss>

2. Query could not be completed because it was malformed

    **Request**:
        
    ``GET http://servername.com/api?t=details&apikey=xxxxx&guid=xxxxxxxxx``

    **Response**:
    
    ``200 OK``
    
    **Content**:
    
        <?xml version="1.0" encoding="UTF-8"/>
        <error code="200" description="Missing parameter: key"/>
      
3. Query could not be completed because no such item was available

    **Request**:
        
    ``GET http://servername.com/api?t=details&apikey=xxxxx&guid=xxxxxxxxx``

    **Response**:
    
    ``200 OK``
    
    **Content**:
    
        <?xml version="1.0" encoding="UTF-8"/>
        <error code="300" description="No such GUID"/>
        
4. Query could not be completed because user credentials are broken

    **Request**:
        
    ``GET http://servername.com/api?t=details&apikey=xxxxx&guid=xxxxxxxxx``

    **Response**:
    
    ``200 OK``
    
    **Content**:
        
        <?xml version="1.0" encoding="UTF-8"/>
        <error code="100" description="Incorrect user credentials"/>     

GETNFO
------

The ``GETNFO`` function returns an nfo file for a particular Usenet (NZB) item.
  
**HTTP Method**

  GET

**HTTP Response**

  200 OK
  
Parameters         |
-------------------|---
  t=getnfo         | Details function, must always be "getnfo".
  id=xxxx          | The GUID of the item being queried. 
  apikey=xxxx      | User's key as provided by the service provider.
  
Optional parameters|
-------------------|---
  raw=1            | If provided returns just the nfo file without the rss container
  o=xxx            | Output format, either "JSON" or "XML". Default is "XML".

**Examples**

1. Normal behavior

    **Request**:
        
    ``GET http://servername.com/api?t=getnfo&apikey=xxxxx&guid=xxxxxxxxx``

    **Response**:
    
    ``200 OK``
    
    **Content**:
    
        <?xml version="1.0" encoding="UTF-8"?>
        <rss version="2.0">
        <channel>
          <item>
            <!-- Standard RSS 2.0 Data -->
            <title>A.Public.Domain.Tv.Show.S06E05</title> 
            <guid isPermaLink="true">http://servername.com/details/e9c515e02346086e3a477a5436d7bc8c</guid> 
            <link>http://servername.com/nfo/e9c515e02346086e3a477a5436d7bc8c</link> 
            <pubDate>Sun, 06 Jun 2010 17:29:23 +0100</pubDate> 
            <description>This is the nfo file</description>
            <enclosure url="http://servername.com/nfo/e9c515e02346086e3a477a5436d7bc8c&ampenclosure=1&amp;i=1&amp;r=18cf9f0a736041465e3bd521d00a90b9" length="154653309" type="application/x-nzb" /> 
          </item>  
        </channel>
        </rss>

2. Query could not be completed because it was malformed

    **Request**:
        
    ``GET http://servername.com/api?t=getnfo&apikey=xxxxx&id=xxxxxxxxx``

    **Response**:
    
    ``200 OK``
    
    **Content**:
    
        <?xml version="1.0" encoding="UTF-8"/>
        <error code="200" description="Missing parameter: id"/>
      
3. Query could not be completed because no such item was available

    **Request**:
        
    ``GET http://servername.com/api?t=getnfo&apikey=xxxxx&id=xxxxxxxxx``

    **Response**:
    
    ``200 OK``
    
    **Content**:
    
        <?xml version="1.0" encoding="UTF-8"/>
        <error code="300" description="No such GUID"/>
        
4. Query could not be completed because user credentials are broken

    **Request**:
        
    ``GET http://servername.com/api?t=getnfo&apikey=xxxxx&id=xxxxxxxxx``

    **Response**:
    
    ``200 OK``
    
    **Content**:
        
        <?xml version="1.0" encoding="UTF-8"/>
        <error code="100" description="Incorrect user credentials"/>     		
		
GET
---

The ``GET`` function returns an nzb for a guid.
  
**HTTP Method**

  GET

**HTTP Response**

  200 OK
  
Parameters         |
-------------------|---
  t=get            | Details function, must always be "get".
  id=xxxx          | The GUID of the item being queried. 
  apikey=xxxx      | User's key as provided by the service provider.
  
Optional parameters|
-------------------|---
  del=1            | If provided removes the nzb from the users cart (if present)

**Examples**

1. Normal behavior

    **Request**:
        
    ``GET http://servername.com/api?t=get&apikey=xxxxx&guid=xxxxxxxxx``

    **Response**:
    
    ``200 OK``
    
    **Content**:
    
		<?xml version="1.0" encoding="UTF-8"?>
		<!DOCTYPE nzb PUBLIC "-//newzBin//DTD NZB 1.1//EN" "http://www.newzbin.com/DTD/nzb/nzb-1.1.dtd">
		<nzb xmlns="http://www.newzbin.com/DTD/2003/nzb">
		...

2. Query could not be completed because it was malformed

    **Request**:
        
    ``GET http://servername.com/api?t=get&apikey=xxxxx&id=xxxxxxxxx``

    **Response**:
    
    ``200 OK``
    
    **Content**:
    
        <?xml version="1.0" encoding="UTF-8"/>
        <error code="200" description="Missing parameter: id"/>
      
3. Query could not be completed because no such item was available

    **Request**:
        
    ``GET http://servername.com/api?t=get&apikey=xxxxx&id=xxxxxxxxx``

    **Response**:
    
    ``200 OK``
    
    **Content**:
    
        <?xml version="1.0" encoding="UTF-8"/>
        <error code="300" description="No such GUID"/>
        
4. Query could not be completed because user credentials are broken

    **Request**:
        
    ``GET http://servername.com/api?t=get&apikey=xxxxx&id=xxxxxxxxx``

    **Response**:
    
    ``200 OK``
    
    **Content**:
        
        <?xml version="1.0" encoding="UTF-8"/>
        <error code="100" description="Incorrect user credentials"/>     		
		
		
CART-ADD
--------

The ``CART-ADD`` function adds an item to the users cart.
  
**HTTP Method**

  GET

**HTTP Response**

  200 OK
  
Parameters         |
-------------------|---
``t=cartadd``      | Cart add function, must always be "cartadd".
``id=xxxx``        | The GUID of the item being added to the cart. 
``apikey=xxxx``    | User's key as provided by the service provider.
  
Optional Parameters|
-------------------|---
``o=xxx``          | Output format, either "JSON" or "XML". Default is "XML".

**Examples**

1. Default behavior

    **Request**:
        
    ``GET http://servername.com/api?t=cartadd&id=12344234234234&apikey=xxxxx``

    **Response**:
    
    ``200 OK``
    
    **Content**:
    	
        <?xml version="1.0" encoding="UTF-8"?>
        <cartadd id="123" />	

2. Query could not be completed because it was malformed

    **Request**:
        
    ``GET http://servername.com/api?t=cartadd&apikey=xxxxx&guid=xxxxxxxxx``

    **Response**:
    
    ``200 OK``
    
    **Content**:
        
        <?xml version="1.0" encoding="UTF-8"/>
        <error code="200" description="Missing parameter: id"/>
      
3. Query could not be completed because no such item was available

    **Request**:
        
    ``GET http://servername.com/api?t=cartadd&apikey=xxxxx&guid=xxxxxxxxx``

    **Response**:
    
    ``200 OK``
    
    **Content**:
    
        <?xml version="1.0" encoding="UTF-8"/>
        <error code="300" description="No such GUID"/>
        
4. Query could not be completed because user credentials are broken

    **Request**:
        
    ``GET http://servername.com/api?t=cartadd&apikey=xxxxx&guid=xxxxxxxxx``

    **Response**:
    
    ``200 OK``
    
    **Content**:
        
        <?xml version="1.0" encoding="UTF-8"/>
        <error code="100" description="Incorrect user credentials"/>
		
        
5. Query could not be completed because item already exists

    **Request**:
        
    ``GET http://servername.com/api?t=cartadd&apikey=xxxxx&guid=xxxxxxxxx``

    **Response**:
    
    ``200 OK``
    
    **Content**:
        
        <?xml version="1.0" encoding="UTF-8"/>
        <error code="310" description="Item already exists"/>		

CART-DEL
--------

The ``CART-DEL`` function deletes an item from the users cart.
  
**HTTP Method**

  GET

**HTTP Response**

  200 OK
  
Parameters         |
-------------------|---
``t=cartdel``      | Cart del function, must always be "cartdel".
``id=xxxx``        | The GUID of the item being delete from the cart. 
``apikey=xxxx``    | User's key as provided by the service provider.
  
Optional Parameters|
-------------------|---
``o=xxx``          | Output format, either "JSON" or "XML". Default is "XML".

**Examples**

1. Default behavior

    **Request**:
        
    ``GET http://servername.com/api?t=cartdel&id=12344234234234&apikey=xxxxx``

    **Response**:
    
    ``200 OK``
    
    **Content**:
    	
        <?xml version="1.0" encoding="UTF-8"?>
        <cartdel id="123" />	

2. Query could not be completed because it was malformed

    **Request**:
        
    ``GET http://servername.com/api?t=cartdel&apikey=xxxxx&guid=xxxxxxxxx``

    **Response**:
    
    ``200 OK``
    
    **Content**:
        
        <?xml version="1.0" encoding="UTF-8"/>
        <error code="200" description="Missing parameter: id"/>
      
3. Query could not be completed because no such item was available

    **Request**:
        
    ``GET http://servername.com/api?t=cartdel&apikey=xxxxx&guid=xxxxxxxxx``

    **Response**:
    
    ``200 OK``
    
    **Content**:
    
        <?xml version="1.0" encoding="UTF-8"/>
        <error code="300" description="No such GUID"/>
        
4. Query could not be completed because user credentials are broken

    **Request**:
        
    ``GET http://servername.com/api?t=cartdel&apikey=xxxxx&guid=xxxxxxxxx``

    **Response**:
    
    ``200 OK``
    
    **Content**:
        
        <?xml version="1.0" encoding="UTF-8"/>
        <error code="100" description="Incorrect user credentials"/>		
		
COMMENTS
--------

The ``COMMENTS`` function returns all comments known about a release.
  
**HTTP Method**

  GET

**HTTP Response**

  200 OK
  
Parameters         |
-------------------|---
``t=comments``     | Comments function, must always be "comments".
``guid=xxxx``      | The GUID of the item being queried. 
``apikey=xxxx``    | User's key as provided by the service provider.
  
Optional Parameters|
-------------------|---
``o=xxx``          | Output format, either "JSON" or "XML". Default is "XML".

**Examples**

1. Default behavior

    **Request**:
        
    ``GET http://servername.com/api?t=comments&apikey=xxxxx&guid=xxxxxxxxx``

    **Response**:
    
    ``200 OK``
    
    **Content**:
    
        <?xml version="1.0" encoding="UTF-8"?>
        <rss version="2.0">
        <channel>
          <item>
            <!-- Standard RSS 2.0 Data -->
            <title>username_of_poster</title> 
            <guid isPermaLink="true">http://servername.com/rss/viewnzb/e9c515e02346086e3a477a5436d7bc8c</guid> 
            <link>http://servername.com/rss/viewnzb/e9c515e02346086e3a477a5436d7bc8c</link> 
            <pubDate>Sun, 06 Jun 2010 17:29:23 +0100</pubDate> 
            <description>Comment about the item</description>
          </item>  
        </channel>
        </rss>

2. Query could not be completed because it was malformed

    **Request**:
        
    ``GET http://servername.com/api?t=comments&apikey=xxxxx&guid=xxxxxxxxx``

    **Response**:
    
    ``200 OK``
    
    **Content**:
        
        <?xml version="1.0" encoding="UTF-8"/>
        <error code="200" description="Missing parameter: key"/>
      
3. Query could not be completed because no such item was available

    **Request**:
        
    ``GET http://servername.com/api?t=comments&apikey=xxxxx&guid=xxxxxxxxx``

    **Response**:
    
    ``200 OK``
    
    **Content**:
    
        <?xml version="1.0" encoding="UTF-8"/>
        <error code="300" description="No such GUID"/>
        
4. Query could not be completed because user credentials are broken

    **Request**:
        
    ``GET http://servername.com/api?t=comments&apikey=xxxxx&guid=xxxxxxxxx``

    **Response**:
    
    ``200 OK``
    
    **Content**:
        
        <?xml version="1.0" encoding="UTF-8"/>
        <error code="100" description="Incorrect user credentials"/>

COMMENTS-ADD
------------

The ``COMMENTS-ADD`` function adds a comment to a release.
  
**HTTP Method**

  GET

**HTTP Response**

  200 OK
  
Parameters         |
-------------------|---
``t=commentadd``   | Comments-add function, must always be "commentadd".
``guid=xxxx``      | The GUID of the item being queried. 
``apikey=xxxx``    | User's key as provided by the service provider.
``text=xxxx``      | The comment being added (URL/UTF-8 encoded). 
  
Optional Parameters|
-------------------|---
``o=xxx``          | Output format, either "JSON" or "XML". Default is "XML".

**Examples**

1. Default behavior

    **Request**:
     
    ``GET HTTP://servername.com/api?t=commentadd&apikey=xxxxx&guid=xxxxxxxxx&text=comment``

    **Response**:
    
    ``200 OK``
    
    **Content**:
    
        <?xml version="1.0" encoding="UTF-8"?>
        <commentadd id="123" />

2. Query could not be completed because no such item was available

    **Request**:
        
    ``GET HTTP://servername.com/api?t=commentadd&apikey=xxxxx&guid=xxxxxxxxx&text=comment``

    **Response**:
    
    ``200 OK``
    
    **Content**:
    
        <?xml version="1.0" encoding="UTF-8"?>
        <error code="300" description="No such item"/>

3. Adding comments via API is not enabled

    **Request**:
        
    ``GET HTTP://servername.com/api?t=commentadd&apikey=xxxxx&guid=xxxxxxxxx&text=comment``

    **Response**:
    
    ``200 OK``
    
    **Content**:
    
        <?xml version="1.0" encoding="UTF-8"?>
        <error code="203" description="Function not available"/>

USER
----

The ``USER`` function is used for retrieving information about a user account 

**HTTP Method**

  GET

**HTTP Response**

  200 OK

Parameters         |
-------------------|---
``t=user``         | User function, must always be "user"
``username=xxx``   | A valid username (URL/UTF-8 encoded).
  
**Examples**

1. Default behavior 

    **Request**:
        
    ``GET HTTP://servername.com/api?t=user&username=user123``

    **Response**:
    
    ``200 OK``
    
    **Content**:
        
        <?xml version="1.0" encoding="UTF-8"?>
        <user username="user123" grabs="123" role="User" apirequests="100" downloadrequests="100" 
        movieview="1" musicview="1" consoleview="1" createddate="2011-08-23 12:31:47" />

2. Query could not be completed because no such item was available

    **Request**:
        
    ``GET HTTP://servername.com/api?t=user&username=user123``

    **Response**:
    
    ``200 OK``
    
    **Content**:
    
        <?xml version="1.0" encoding="UTF-8"?>
        <error code="300" description="No such item"/>

3. Function disabled

    **Request**:
        
    ``GET HTTP://servername.com/api?t=user&username=user123``

    **Response**:
    
    ``200 OK``
    
    **Content**:
    
        <?xml version="1.0" encoding="UTF-8"?>
        <error code="203" description="Function not available"/>
	 
