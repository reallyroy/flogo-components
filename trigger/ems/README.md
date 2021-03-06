# EMS
This trigger provides your flogo application the ability to receive JMS Text Messages from a destination

## Pre-requisites

The trigger uses EMS C libraries in order to receive messages from EMS. 
Trigger has been tested against Go v1.12.4 and EMS v8.4 on Mac OS

An installation of EMS 8.4 is required for this trigger to work. 

Once you have installed TIBCO EMS, you will need to make sure that the dynamic libraries are accessible.

Either copy the following dylibs to /usr/local/lib for the trigger to work:

* libtibems64.dylib
* libssl.1.0.0.dylib
* libcrypto.1.0.0.dylib

Alternatively, setting DYLD_LIBRARY_PATH or LD_LIBRARY_PATH to the location of EMS Client Libraries (<EMS_HOME>/ems/8.4/lib) 
should work too (i haven't tested this).


The trigger uses ems client go package (go get github.com/mmussett/ems) which will need to be modified before building. 

Modify the CFLAGS and LDFLAGS paths accordingly in client.go:

```
#cgo darwin CFLAGS: -I/opt/tibco/ems/ems841/ems/8.4/include/tibems
#cgo darwin LDFLAGS: -L/opt/tibco/ems/ems841/ems/8.4/lib -ltibems64
```

## Installation

```bash
flogo install github.com/mmussett/flogo-components/trigger/ems
```

Currently the trigger cannot be installed using Flogo UI due to EMS client dependencies.

## Schema
Outputs and Endpoint:

```json
{
  "settings":[
    {
      "name": "serverUrl",
      "type": "string"
    },
    {
      "name": "destination",
      "type": "string"
    },
    {
      "name": "destinationType",
      "type": "string"
    },
    {
      "name": "user",
      "type": "string"
    },    
    {
      "name": "password",
      "type": "string"
    }             
  ],
  "outputs": [
    {
      "name": "msgText",
      "type": "string"
    }
  ],
  "handler": {
    "settings": [
      {
        "name": "handler_setting",
        "type": "string"
      }
    ]
  }
}
```
## Settings
| Setting   | Description    |
|:----------|:---------------|
| serverUrl  | EMS Server url |
| destination | EMS Destination to receive from |
| destinationType | Either queue or topic |
| user | EMS connection username |
| password | EMS connection password |



## Ouputs
| Output   | Description    |
|:---------|:---------------|
| msgText | Received EMS Message Text |

## Handlers
| Setting   | Description    |
|:----------|:---------------|
| N/A       | awaiting better understanding  |


## Example Configuration

Triggers are configured via the triggers.json of your application. 
The following is and example configuration of the EMS Trigger.

### Log EMS Message
Configure the Trigger to receive EMS Messages
```json
{
  "name": "ems_trigger_app",
  "type": "flogo:app",
  "version": "0.0.1",
  "description": "My flogo application description",
  "appModel": "1.1.0",
  "imports": [
    "github.com/project-flogo/legacybridge",
    "github.com/project-flogo/contrib/activity/log",
    "github.com/project-flogo/contrib/trigger/rest",
    "github.com/project-flogo/flow",
    "github.com/mmussett/flogo-components/trigger/ems"
  ],
  "triggers": [
    {
      "id": "receive_ems_trigger",
      "ref": "github.com/mmussett/flogo-components/trigger/ems",
      "settings": {
        "destination": "queue.sample",
        "destinationType": "queue",
        "password": "",
        "serverURL": "tcp://127.0.0.1:7222",
        "user": "admin"
      },
      "handlers": [
        {
          "settings": null,
          "actions": [
            {
              "ref": "#flow",
              "settings": {
                "flowURI": "res://flow:simple_flow"
              },
              "input": {
                "in": "=$.msgText"
              }
            }
          ]
        }
      ]
    }
  ],
  "resources": [
    {
      "id": "flow:simple_flow",
      "data": {
        "name": "simple_flow",
        "metadata": {
          "input": [
            {
              "name": "in",
              "type": "string",
              "value": "test"
            }
          ],
          "output": [
            {
              "name": "out",
              "type": "string"
            }
          ]
        },
        "tasks": [
          {
            "id": "log",
            "name": "Log Message",
            "activity": {
              "ref": "#log",
              "input": {
                "message": "=$flow.in",
                "addDetails": "false"
              }
            }
          }
        ],
        "links": []
      }
    }
  ]
}
```

## Testing

trigger_test.go is provided to test and can be invoked using:

```
$ go test -v
=== RUN   TestTrigger
2019-06-05 12:41:31.156 INFO   [trigger-ems] - Testing Trigger
2019-06-05 12:41:31.473 INFO   [trigger-ems] - event processing cycle starting
2019-06-05 12:41:35.416 INFO   [trigger-ems] - received message from EMS...
2019-06-05 12:41:35.416 INFO   [trigger-ems] - [hello, world]
2019-06-05 12:41:35.416 INFO   [trigger-ems] - event processing cycle completed
2019-06-05 12:41:35.416 INFO   [trigger-ems] - event processing cycle starting
```

Using the shipped EMS C examples you can send message via tibemsMsgProducer to fire trigger:

```
$ ./tibemsMsgProducer -queue queue.sample "hello, world"
------------------------------------------------------------------------
tibemsMsgProducer SAMPLE
------------------------------------------------------------------------
Server....................... localhost
User......................... (null)
Destination.................. queue.sample
Send Asynchronously.......... false
Message Text.................
	hello, world
------------------------------------------------------------------------
Publishing to destination 'queue.sample'
Published message: hello, world
```



