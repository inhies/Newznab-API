This document describes the NEWZNAB Usenet Searching Web API. The API is
designed to be implemented by Usenet indexing sites, i.e. sites that index
Usenet newsgroups through some means, typically by downloading and inspecting
the NTTP headers. The API is aimed for NZB aware client applications  to allow
them to perform Usenet searches against Newznab servers and receive NZB
information in order to facilitate direct downloading from Usenet without
having to download any NTTP headers.

This document does not describe the actual implementation of either the client
or the server but just describes the HTTP(S) interface and request/response
sequences.

Intended readers are server and client implementers.

Notation
--------
    
This document uses the following notations:

Parameters:

* ``t=c`` denotes a required HTTP query parameter.
* ``[o=json | o=xml]`` denotes optional parameters with possible values.
