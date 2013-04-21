CoOPS protocol specification draft 1
====================================

OVERVIEW
--------

CoOPS is a protocol which allows users to simultaneously edit a file in a server. The protocol focuses on the core of the simultaneous editing, leaving out other functions - such as authentication, user management, etc. These functions are extensions to the basic protocol. The protocol defines the client-server communication using HTTP protocol.

The protocol assumes that each file is located in a address that can be used for file identification (URL). All protocol operations are relative to this address. This allows the protocol to be used in variety of different contexts, such as editing of html document, a paragraph of a wiki page or a svg image. Because the editing environment (blog, wiki, etc. server program) usually contains user registration and authentication functions these are left out of the scope of protocol. Communication between server and client is done via HTTP protocol using JSON formatted data.

Protocol also defines way to transfer meta-information besides normal file content. 

It makes sense to use different diff / patch algorithms for different file types. For this reason, client and the server negotiate used diff / patch algorithmic before starting.	

REST API
--------
TODO Overview

### Status Codes
TODO Status Codes

### End points

|Path|/|
|Method|GET|
|Description|Returns file and file meta-information. If revision number parameter is specified method returns file of that specified revision otherwise last revision is returned|
|Errors|			
| Not Found|When file does not exist|
| Unauthorized|When user is not recognized|
| Forbidden|When user does not have permission to file|
|Parameters|		
| revisionNumber|(Query) Optional parameter that specifies file revision to be returned.|
	
    { 
      "response": {
        "revisionNumber": 123,
        "content": "CONTENT",
        "properties": {
            "backgroundColor": "green"
        }
	  }
	}

EXTENSIONS
----------
TODO Overview

### WebSocket
TODO Describe WebSocket extension