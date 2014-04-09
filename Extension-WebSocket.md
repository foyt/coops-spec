CoOps WebSocket extension specification 1.0.0-draft4
================================

OVERVIEW
--------
WebSocket is an extension for the CoOps protocol and every server does not necessary need to implement it. The purpose of the extension is to provide a faster and lighter way for server and client to communicate.

See [https://github.com/foyt/coops-spec/](https://github.com/foyt/coops-spec/) for the basic protocol specification.

Identifier
----------
Extension identifier is **webSocket-1.0.0**

STATUS
--------
This document is currently a draft of the specification.

Join Request
------------
When the extension is enabled join request extensions part should contain webSocket-1.0.0 object that describes how to open the socket.

extension object may contain **ws** and **wss** fields that hold urls into unencrypted (ws) and encrypted WebSocket urls.

Absent field indicates that server does not support either unencrypted (ws) or encrypted (wss) WebSockets.

The extension (as well as the main protocol) assumes that each file is located in an address that can be used for file identification but depending on the server implementation it can also contain other information such as security tokens or other information needed by the server to complete opening of the socket.

Example of join request with WebSocket extension: 

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
	    "websockets-1.0.0": {
          "ws": "ws://www.example.com/ws/742523daef59db4b718409f46de05d0c",
          "wss": "wss://www.example.com/ws/742523daef59db4b718409f46de05d0c",
	    }
      }
	}
    
Messages
--------

Instead of standard protocol API calls communication between server and client is done via WebSocket messages. 

**Message format**

Messages are always stringified JSON objects that contain "type and "data" properties. As a general thumb rule WebSocket messages try to replicate methods from basic protocol where the type specifies what kind of message is in question and the data field contains the actual data as JSON object.

**Example:**
  
    {
      "type": "message-type",
      "data": {
        "message": "data"
      }
    }
    
### Messages from client

#### patch

Patches a file. Message is a WebSocket equivalent for [Basic protocol patch request](https://github.com/foyt/coops-spec/#patch--patch-request)

**Data**

* **revisionNumber** - number of revision patch is meant for
* **sessionId** - unique collaboration session id
* **patch** - an optional field that contains changes to the file content in used diff format
* **properties** - an optional field that contains changed metadata as key-value pairs
* **extensions** - an optional field that contains extension changes as key-value pairs

On successful patch server sends a *update* message in return. If the patching fails on conflict *patchRejected* message is sent and on other errors server sends a *patchError* message.   

**Example**

        {
          "type": "patch",
          "data": {
            revisionNumber : 123, 
            sessionId: "742523daef59db4b718409f46de05d0c"m
            patch: "patch content in used diff format", 
            properties: {
              "key": "value"
            },
            extensions: {
              "extension": {
                "key": "value"
	          }
            }
          }
        }

### Messages from server

#### update

Sends an update to the client. Message is a WebSocket equivalent for [Basic protocol update request](https://github.com/foyt/coops-spec/#get-update-update-request)

**Data**

* **sessionId** - unique collaboration session id from whom the patch came from
* **revisionNumber** - number of patch revision
* **patch** - an optional field that contains changes to the file content in used diff format
* **properties** - an optional field that contains changed metadata as key-value pairs
* **extensions** - an optional field that contains extension changes as key-value pairs

Update messages are used to notify clients of file updates. Updates can be originated from some other session or from the client it self in which case the session id matches the clients sessionId. 

When the patch is originated from the client itself it should be treated as patch acceptance notification (server has accepted the patch and applied it into the file in the server) otherwise the patch should be applied into the client file,

**Example**

        {
          "type": "update",
          "data": {
            revisionNumber : 123, 
            sessionId: "742523daef59db4b718409f46de05d0c"m
            patch: "patch content in used diff format", 
            properties: {
              "key": "value"
            },
            extensions: {
              "extension": {
                "key": "value"
	          }
            }
          }
        }
        
#### patchRejected     

Sends a notification to the client that patch it sent was rejected. 

**Data**

        {
          type: "patchRejected",
          data: {
            code: code,
            message: "Reject reason"
          }
        }
                
When client receives a patchRejected message it means that the last update it sent was rejected and the client should wait for an update message and try again.

**Example**

        {
          "type": "patchRejected",
          "data": {
            code : 409, 
            message: "Server version does not match client version"
          }
        }

        
#### patchError     

Sends a notification to the client that patch it sent caused an error. 

**Data**

        {
          type: "patchError",
          data: {
            code: code,
            message: "Error message"
          }
        }
                
When client receives a patchError message it means that the last patch it sent caused an error on the server. 

**Example**

        {
          "type": "patchError",
          "data": {
            code : 400, 
            message: "Server does not support extension 'bogus'"
          }
        }
