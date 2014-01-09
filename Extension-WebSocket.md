CoOPS WebSocket extension specification 1.0draft3
================================

OVERVIEW
--------
WebSocket is an extension for the CoOps protocol and every implementation does not necessary need to implement it. WebSockets provide faster and lighter way for server and client to communicate.

See [https://github.com/foyt/coops-spec/](https://github.com/foyt/coops-spec/) for the basic protocol specification   

STATUS
--------
This document is currently a draft of the specification.

Join Request
----------------------
When extension is enabled in server unsecureWebSocketUrl and/or secureWebSocketUrl should appear in extensionProperties/websockets key-value map.

Example of join request with WebSocket extension: 

    { 
      "response": { 
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
	}
	
*Format of the unsecureWebSocketUrl and secureWebSocketUrl is following:*

    protocol ":" "//" host ":" port path token

**protocol** is either ws or wss depending on whether socket is secure or not.<br/>
**host** is a host where server is located<br/>
**path** path to a file<br/>
**token** optional token which can be used to authenticate incoming socket connection <br/>

*Example url*

    "ws://www.example.com:80/path/to/file/token"    
    
Messages
--------

Instead of standard protocol API calls communication between server and client is done via WebSocket messages. 

**Message format**

Messages always include "type" property that indicates a type of a message. Other properties depend on the message type.

    {
      "type": "MESSAGE TYPE",
      "PROPERTY NAME": "PROPERTY VALUE"
    }

**Messages to server**

***patch***

Client sends a patch to server. Request has to contain revisionNumber field so server can check for version conflicts. Before sending next patch client should wait for patchAccepted or patchRejected message.

    {
      "type": "patch",
      "patch": "patch content in used diff format",
      "revisionNumber": number of revision patch is meant for,
      "properties": {
        "key": "value"
      },
      "extensions": {
	    "cursors": {
          "selectStart": 199,
          "selectEnd": 200
	    }
      }
    }
    
**Messages to client**

***patch***

Server sends a patch to a client.

    {
      "type": "patch",
      "sessionId": "1a79a4d60de6718e8e5b326e338ae533",
      "patch": "patch content in used diff format",
      "revisionNumber": number of revision patch is meant for,
      "checksum": content checksum for integrity checks (optional),
      "properties": {
        "key": "value"
      },
      "extensions": {
	    "cursors": {
          "selectStart": 199,
          "selectEnd": 200
	    }
      }
    }
