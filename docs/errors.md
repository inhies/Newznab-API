Newznab Error Codes
===================

Under normal circumstances i.e. when the HTTP request/response sequence is
successfully completed Newznab implementations always respond with
``HTTP 200 OK``.  However this doesn't mean that the query was semantically
correct. It simply means that the HTTP part of the sequence was successful.
One then must check the actual response body/data to see if the request was
completed without errors. 

In case of a Newznab error the response contains an error code and an a
description of the error.

The error codes have been defined into different ranges. 100-199 Account/user
credentials specific error codes, 200-299 API call specific error codes,
300-399 content specific error codes and finally 900-999 Other error codes.

Error code          | Description
--------------------|----------------------------------------------------------
100                 | Incorrect user credentials
101                 | Account suspended
102                 | Insufficient privileges/not authorized
103                 | Registration denied
104                 | Registrations are closed
105                 | Invalid registration (Email Address Taken)
106                 | Invalid registration (Email Address Bad Format)
107                 | Registration Failed (Data error)
200                 | Missing parameter
201                 | Incorrect parameter
202                 | No such function. (Function not defined in this specification).
203                 | Function not available. (Optional function is not implemented).
300                 | No such item. 
300                 | Item already exists. 
900                 | Unknown error
910                 | API Disabled

Error code example
------------------

1. Query could not be completed because user credentials are broken

    **Request**:
        
    ``GET http://servername.com/api?t=details&apikey=xxxxx&guid=xxxxxxxxx``

    **Response**:
    
    ``200 OK``
    
    **Content**:
    
        <?xml version="1.0" encoding="UTF-8"?>
        <error code="100" description="Incorrect user credentials"/>

