CoOPS protocol specification 1.0.0
==================================

OVERVIEW
--------

CoOPS is a protocol which allows users to simultaneously edit a file in a server. The protocol focuses on the core of the simultaneous editing, leaving out other functions - such as user management, file creation, file deletion, etc. These functions are extensions to the basic protocol. The protocol defines the client-server communication using HTTP protocol.

The protocol assumes that each file is located in an address that can be used for file identification (URL). All protocol operations are relative to this address. This allows the protocol to be used in variety of different contexts, such as editing of html document, a paragraph of a wiki page or a svg image. Because the editing environment (blog, wiki, etc. server program) usually contains user registration and authentication functions these are left out of the scope of protocol. Communication between server and client is done via HTTP protocol using JSON formatted data.

Protocol also defines way to transfer meta-information besides normal file content. 

It makes sense to use different diff / patch algorithms for different file types. For this reason, client and the server negotiate used diff / patch algorithm before starting.

STATUS
--------
Protocol version is currently 1.0.0

REST API
--------

### Status Codes

<table width="100%">
  <tr><td>200 - OK</td><td>Used when request has succeeded.</td></tr>
  <tr><td>404 - Not Found</td><td>When requested file could not be found</td></tr>
  <tr><td>409 - Conflict</td><td>When request has failed because of a conflict</td></tr>
  <tr><td>500 - Internal Server Error</td><td>When request has failed unexpectedly</td></tr>
  <tr><td>501 - Not Implemented</td><td>When client has requested for a unsupported feature</td></tr>
</table>

Implementations may use other status codes besides ones used in protocol.

### (GET) / (load request)

Returns a file and file meta-information. If revision number parameter is specified method returns file of that specified revision otherwise last revision is returned

<table width="100%">
  <tr><td>Path</td><td>/</td></tr>
  <tr><td>Method</td><td>GET</td></tr>
  <tr><td>Errors</td><td></td></tr>
  <tr><td width="120px">&nbsp;&nbsp;&nbsp;404 - Not Found</td><td>When file does not exist</td></tr>
  <tr><td>Parameters</td><td></td></tr>
  <tr><td>&nbsp;&nbsp;&nbsp;revisionNumber</td><td>(Query) Optional parameter that specifies file revision to be returned.</td></tr>
</table>

**Response**

Status code should be 200 when request is successful.  

<table width="100%">
  <tr><td>revisionNumber</td><td>Revision number of a returned file.</td></tr>
  <tr><td>content</td><td>Contents of the file as text. Binary files are serialized into text form</td></tr>
  <tr><td>contentType</td><td>Content type of the file. If file needs to be edited by some specific editor "editor" -parameter can be used to specify which one</td></tr>
  <tr><td>properties</td><td>File metadata as key-value pairs (JSON Object).</td></tr>
</table>

**Example response**

    { 
      "revisionNumber": 123,
      "content": "CONTENT",
      "contentType": "text/html;editor=CKEditor",
      "properties": {
        "name": "Name of the file",
        "backgroundColor": "green"
      }
	}

### (GET) /update (update request)

Client calls the method in order to check new updates into the file.

<table width="100%">
  <tr><td>Path</td><td>/update</td></tr>
  <tr><td>Method</td><td>GET</td></tr>
  <tr><td>Errors</td><td></td></tr>
  <tr><td width="165px">&nbsp;&nbsp;&nbsp;400 - Bad request </td><td>When revisionNumber parameter is missing</td></tr>
  <tr><td>Parameters</td><td></td></tr>
  <tr><td>&nbsp;&nbsp;&nbsp;sessionId</td><td>(Query) Unique collaboration session id.</td></tr>
  <tr><td>&nbsp;&nbsp;&nbsp;revisionNumber</td><td>(Query) Revision number of client file.</td></tr>
</table>

**Response**

Status code should 200 (OK) when server has updates for client and 204 (No Content) when updates are not available. When server sends 204 no content should be sent.

<table width="100%">
  <tr><td>sessionId</td><td>Unique collaboration session id</td></tr>
  <tr><td>revisionNumber</td><td>Revision number of the patch</td></tr>
  <tr><td>checksum</td><td>Content checksum for integrity checks (optional)</td></tr>
  <tr><td>patch</td><td>Changes into the file content in used diff format</td></tr>
  <tr><td>properties</td><td>Changed properties as key-value pairs (JSON Object)</td></tr>
  <tr><td>extensions</td><td>Changed extension values as key-value map (JSON Object). Extension names act as keys and changes are represented in key-value pairs (JSON Object) </td></tr>
</table>

**Example response**

    [{ 
      "sessionId": "1a79a4d60de6718e8e5b326e338ae533",
      "revisionNumber": 123,
      "checksum": "1a79a4d60de6718e8e5b326e338ae533",
      "patch": "patch text",
      "properties": {
        "name": "Name of the file",
        "backgroundColor": "green"
      },
      "extensions": {
	    "cursors": {
          "selectStart": 199,
          "selectEnd": 200
	    }
      }
	}, ... ]
		
### (PATCH) / (patch request)

Patches a file.

<table width="100%">
  <tr><td>Path</td><td>/</td></tr>
  <tr><td>Method</td><td>PUT</td></tr>
  <tr><td>Errors</td><td></td></tr>
  <tr><td>&nbsp;&nbsp;&nbsp;404 - Not Found</td><td>When file does not exist</td></tr>
  <tr><td>&nbsp;&nbsp;&nbsp;409 - Conflict</td><td>When server file revision does not match client file revision</td></tr>
  <tr><td>Parameters</td><td></td></tr>
  <tr><td>&nbsp;&nbsp;&nbsp;revisionNumber</td><td>(JSON) number of revision patch is meant for</td></tr>
  <tr><td>&nbsp;&nbsp;&nbsp;patch</td><td>(JSON) optional field that contains changes to the file content in used diff format</td></tr>  
  <tr><td>&nbsp;&nbsp;&nbsp;properties</td><td>(JSON) optional field that contains changed metadata as key-value pairs (JSON Object)</td></tr>
  <tr><td>&nbsp;&nbsp;&nbsp;extensions</td><td>(JSON) optional field that contains extension changes as key-value pairs (JSON Object)</td></tr>
</table>

**Response**

Successful request returns status code 204 (No Content) and does not return any content. 

### (GET) /join (join request)

Client calls the method in order to join the collaboration session.

<table width="100%">
  <tr><td>Path</td><td>/join</td></tr>
  <tr><td>Method</td><td>GET</td></tr>
  <tr><td>Errors</td><td></td></tr>
  <tr><td>&nbsp;&nbsp;&nbsp;404 - Not Found</td><td>When file does not exist</td></tr>
  <tr><td width="155px">&nbsp;&nbsp;&nbsp;501 - Not Implemented</td><td>When server does not support any algorithms provided by client and/or when unsupported CoOPS version is specified</td></tr>
  <tr><td>Parameters</td><td></td></tr>
  <tr><td>&nbsp;&nbsp;&nbsp;algorithm</td><td>(Query) algorithm(s) supported by client. Parameter can be repeated to indicate support for multiple algorithms. In this case algorithms should be ordered descendingly from most favourable to least favourable algorithm</td></tr>
  <tr><td>&nbsp;&nbsp;&nbsp;protocolVersion</td><td>(Query) used CoOPS protocol version.</td></tr>
</table>

**Response**

Status code should be 200 if the request is successful

<table width="100%">
  <tr><td>sessionId</td><td>Unique collaboration session id</td></tr>
  <tr><td>algorithm</td><td>Revision number of returned file.</td></tr>
  <tr><td>revisionNumber</td><td>Current revision number of the file</td></tr>
  <tr><td>content</td><td>Current content of the file</td></tr>
  <tr><td>contentType</td><td>Current content type of the file</td></tr>
  <tr><td>properties</td><td>Current properties of the file as key-value pairs (JSON Object)</td></tr>
  <tr><td>extensions</td><td>Supported extensions as key-value map (JSON Object). Extension names are listed as keys and values are settings of extension as key-value pair (JSON Object) </td></tr>
</table>

**Example response**

    { 
      "sessionId": "1a79a4d60de6718e8e5b326e338ae533",
      "algorithm": "diff-match-patch",
      "revisionNumber": 123,
      "content": "CONTENT",
      "contentType": "text/html;editor=CKEditor",
      "properties": {
        "name": "Name of the file",
        "backgroundColor": "green"
      },
      "extensions": {
	    "websockets": {
          "unsecureWebSocketUrl": "ws://www.example.com:80/path/to/file/token",
          "secureWebSocketUrl": "wss://www.example.com:8443/path/to/file/token"
	    }
      }
	}