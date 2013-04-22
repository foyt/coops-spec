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
  <tr><td>&nbsp;&nbsp;&nbsp;404&nbsp;-&nbsp;Not Found</td><td>When file does not exist</td></tr>
  <tr><td>Parameters</td><td>/</td></tr>
  <tr><td>&nbsp;&nbsp;&nbsp;revisionNumber</td><td>(Query) Optional parameter that specifies file revision to be returned.</td></tr>
  <tr><td>Description</td><td>Returns file and file meta-information. If revision number parameter is specified method returns file of that specified revision otherwise last revision is returned</td></tr>
</table>

**Response**

<table>
  <tr><td>revisionNumber</td><td>Revision number of returned file.</td></tr>
  <tr><td>name</td><td>Name of the file</td></tr>
  <tr><td>content</td><td>Contents of the file as text. Binary files are serialized into text form</td></tr>
  <tr><td>contentType</td><td>Content type of the file. If file needs to be edited by some specific editor "editor" -parameter can be used to specify which one</td></tr>
  <tr><td>properties</td><td>File metadata as key-value pairs (JSON Object).</td></tr>
</table>

**Example response**

    { 
      "response": {
        "revisionNumber": 123,
        "name": "name",
        "content": "CONTENT",
        "contentType": "text/html;editor=CKEditor",
        "properties": {
            "backgroundColor": "green"
        }
	  }
	}
	
**(PATCH) /**

<table>
  <tr><td>Path</td><td>/</td></tr>
  <tr><td>Method</td><td>PUT</td></tr>
  <tr><td>Errors</td><td></td></tr>
  <tr><td>&nbsp;&nbsp;&nbsp;404&nbsp;-&nbsp;Not Found</td><td>When file does not exist</td></tr>
  <tr><td>&nbsp;&nbsp;&nbsp;409&nbsp;-&nbsp;Conflict</td><td>When server version does not match client version</td></tr>
  <tr><td>Parameters</td><td>/</td></tr>
  <tr><td>&nbsp;&nbsp;&nbsp;revisionNumber</td><td>(JSON) number of revision patch is meant for</td></tr>
  <tr><td>&nbsp;&nbsp;&nbsp;name</td><td>(JSON) optional field that instructs name change if specified</td></tr>
  <tr><td>&nbsp;&nbsp;&nbsp;patch</td><td>(JSON) optional field that contains changes to content in used diff format</td></tr>
  <tr><td>&nbsp;&nbsp;&nbsp;properties</td><td>(JSON) optional field that contains changed metadata as key-value pairs (JSON Object)</td></tr>
  <tr><td>Description</td><td>Patches a file and returns updated file.</td></tr>
</table>

**Response**

<table>
  <tr><td>revisionNumber</td><td>Revision number of returned file.</td></tr>
  <tr><td>name</td><td>Name of the file</td></tr>
  <tr><td>content</td><td>Contents of the file as text. Binary files are serialized into text form</td></tr>
  <tr><td>contentType</td><td>Content type of the file. If file needs to be edited by some specific editor "editor" -parameter can be used to specify which one</td></tr>
  <tr><td>properties</td><td>File metadata as key-value pairs (JSON Object)</td></tr>
</table>

**Example response**

    { 
      "response": {
        "revisionNumber": 123,
        "name": "name",
        "content": "CONTENT",
        "contentType": "text/html;editor=CKEditor",
        "properties": {
            "backgroundColor": "green"
        }
	  }
	}

**(GET) /join**

<table>
  <tr><td>Path</td><td>/join</td></tr>
  <tr><td>Method</td><td>GET</td></tr>
  <tr><td>Errors</td><td></td></tr>
  <tr><td>&nbsp;&nbsp;&nbsp;404&nbsp;-&nbsp;Not Found</td><td>When file does not exist</td></tr>
  <tr><td>&nbsp;&nbsp;&nbsp;501&nbsp;-&nbsp;Not Implemented</td><td>When server does not support any algorithms provided by client and/or when unsupported CoOPS version is specified</td></tr>
  <tr><td>Parameters</td><td>/</td></tr>
  <tr><td>&nbsp;&nbsp;&nbsp;algorithm</td><td>(Query) algorithm(s) supported by client. Parameter can be repeated to indicate support for multiple algorithms. In this case algorithms should be ordered descendingly from most favourable to least favourable algorithm</td></tr>
  <tr><td>&nbsp;&nbsp;&nbsp;protocolVersion</td><td>(Query) used CoOPS protocol version.</td></tr>
  <tr><td>Description</td><td>Client calls method when joining file collaboration and resolves used algorithm and extensions.</td></tr>
</table>

**Response**

<table>
  <tr><td>algorithm</td><td>Revision number of returned file.</td></tr>
  <tr><td>extensions</td><td>Revision number of returned file.</td></tr>
</table>

**Example response**

    { 
      algorithm: "diff-match-patch",
        extensions: [
		  "websockets", 
		  "cursors"
		]
	  }
	}
		
EXTENSIONS
----------
TODO Overview

### WebSocket
TODO Describe WebSocket extension