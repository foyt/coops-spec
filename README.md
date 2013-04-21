CoOPS protocol specification 1.0draft 1
====================================

OVERVIEW
--------

CoOPS is a protocol which allows users to simultaneously edit a file in a server. The protocol focuses on the core of the simultaneous editing, leaving out other functions - such as user management, file creation, file deletion, etc. These functions are extensions to the basic protocol. The protocol defines the client-server communication using HTTP protocol.

The protocol assumes that each file is located in a address that can be used for file identification (URL). All protocol operations are relative to this address. This allows the protocol to be used in variety of different contexts, such as editing of html document, a paragraph of a wiki page or a svg image. Because the editing environment (blog, wiki, etc. server program) usually contains user registration and authentication functions these are left out of the scope of protocol. Communication between server and client is done via HTTP protocol using JSON formatted data.

Protocol also defines way to transfer meta-information besides normal file content. 

It makes sense to use different diff / patch algorithms for different file types. For this reason, client and the server negotiate used diff / patch algorithmic before starting.	

REST API
--------
TODO Overview

### Status Codes
TODO Status Codes

### End points

**(GET) /**

<table>
  <tr><td>Path</td><td>/</td></tr>
  <tr><td>Method</td><td>GET</td></tr>
  <tr><td>Errors</td><td></td></tr>
  <tr><td>&nbsp;&nbsp;&nbsp;Not Found</td><td>When file does not exist</td></tr>
  <tr><td>Parameters</td><td>/</td></tr>
  <tr><td>&nbsp;&nbsp;&nbsp;revisionNumber</td><td>(Query) Optional parameter that specifies file revision to be returned.</td></tr>
  <tr><td>Description</td><td>Returns file and file meta-information. If revision number parameter is specified method returns file of that specified revision otherwise last revision is returned</td></tr>
</table>

**Response**

<table>
  <tr><td>fileId</td><td>contains file id. Format of the id depends on implementation</td></tr>
  <tr><td>revisionNumber</td><td>Revision number of returned file.</td></tr>
  <tr><td>name</td><td>Name of the file</td></tr>
  <tr><td>content</td><td>Contents of the file as text. Binary files are serialized into text form</td></tr>
  <tr><td>contentType</td><td>Content type of the file. If file needs to be edited by some specific editor "editor" -parameter can be used to specify which one</td></tr>
  <tr><td>properties</td><td>File metadata as key-value pairs.</td></tr>
</table>

**Example response**

    { 
      "response": {
        "fileId": "112233445566778899aabbcc",
        "revisionNumber": 123,
        "name": "name",
        "content": "CONTENT",
        "contentType": "text/html;editor=CKEditor",
        "properties": {
            "backgroundColor": "green"
        }
	  }
	}
	
**(PUT) /**

**(PATCH) /**

**(GET) /join**
	
EXTENSIONS
----------
TODO Overview

### WebSocket
TODO Describe WebSocket extension