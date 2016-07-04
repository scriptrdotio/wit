#Handle vocal commands using wit.ai from scriptr
##Vocal interfaces
The are many cases where an end user needs to send vocal instructions to a device because he cannot use his hands (e.g. when driving, when holding a child, etc.) or because the environment he is working in prevents him to use them (e.g. surgery room to avoid contamination). More generally, the near future should see more importance given to vocal interfaces as a complement to the current tactile ones.
In this article, we will see how to use scriptr to provide you with the ability to trigger the execution of your APIs through vocal instructions.
##wit.ai
wit.ai is a company that provides developers with means to map vocal commands to structured objects composed of intents (command) and entities (command targets and parameters). Using wit.ai is pretty straightforward and does not require much time from you to get started. In the remainder of this document, we will consider that we already have created an intent. Please refer to wit.ai's tutorials and documentation for more on how to create intent based on vocal commands.
#Scriptr handler for wit.ai
##Purpose
The purpose of this handler is to allow you to trigger one of your APIs (scriptr scripts) from a vocal command. The latter is sent by your client app as an audio file (mp3) to the scriptr wit.ai handler, which will invoke wit.ai to identify the corresponding intent, then use the latter to trigger the targeted API. The sole constraints are to provide an "execute()" method in your script that accept a JSON object as a parameter (execute trigger the logic of your API). 
##Components
wit/handleCommand: this is the API to invoke from a client application. The latter needs to pass the audio file containing the vocal command as an attached file ("commands" parameter)

wit/witHandler : this module handles the mapping between the intent that match the vocal command, as returned by wit.ai, and the developer's API

wit/witClient : this module handles the communication with wit.ai's APIs

wit/config: required configuration (wit.ai auth token, intent to script mappings, etc.)
##How to use
- Deploy the aforementioned scripts in your scriptr account, in a folder named "wit"
- Create an intent on https://wit.ai/
- Create a test script in scriptr, which you will use as the API to invoke upon reception of the vocal command, as in the below example
dummy API script
```
function execute(params) {
  return params;
} 
```
- Update the wit/config file with your wit.ai credentials (server token) and create a mapping between your test script and the intent you have created, as in the below example
```
// The URL root to wit.ai's APIs
var witApiUrl = "https://api.wit.ai/";
// Default audio type of voice commands
var audioType = "audio/mpeg3";
// The developer's application ID at wit.ai
var appId = "98k16749-17e6-9dv7-748b-s991b4631258"; // example
// The server to server wit.ai OAuth token
var serverAccessToken = "ASDF7ERTYU9JK0SDFG6FGHDAQ";  // example 
// fill this variable to associate a script name to an intent name and map the entities to the parameter names
// (optional)
// the content of the "mapping" variable below is an example of a mapping
var mapping = {
  "get_nearest_venue": { // this is the intent name
    "script": "./test/getNearestVenue.js", // this is the full path to the script to invoke for the intent
    "params": { // mapping of the entities to parameters
      "distance": "radius", // the distance entity is mapped to the radius parameter
      "local_search_query": "venue" // the local_search_query entity is mapped to the venue parameter
    }
  }
};
```
- From a client application, a REST client or from the scriptr's dashboard, invoke the wit/handleCommand API as follows
```
curl -X POST  -F apsws.time=1432726939153 -F commands=@myVocalCommandFile.mp3 -H 'Authorization: bearer you_scriptr_auth_token' 'https://api.scriptr.io/modules/wit/handleCommand.js'
```
That's all.
##Try it!
Create an intent at wit.ai, called "get_nearest_venue". This intent knows how to interpret vocal instructions such as "get restaurant in a 200 meters perimeter" and returns the following JSON structure accordingly:
```
{
  "msg_id": "e5553a76-b11f-33f5-b345-22d5677s897f989f",
  "_text": "find restaurant in the 200 meters Perimeter",
  "outcomes": [
    {
      "_text": "find venue in the 200 meters Perimeter",
      "intent": "get_nearest_venue",
      "entities": {
        "distance": [
          {
            "value": 200,
            "type": "value",
            "unit": "metre"
          }
        ],
        "local_search_query": [
          {
            "suggested": true,
            "value": "restaurant",
            "type": "value"
          }
        ]
      },
      "confidence": 0.55
    }
  ]
}
```
Also use the dummy API script provided in the source code (wit/test/getNearestVenue) that returns the values of the received parameter. 
Map this script to the intent using the configuration file given as an example in the "How to use" paragraph.
Create an audio file (mp3) that contains the following vocal instruction: "get nearest gas station in a five hundred meters perimeter" and pass it to the following CURL instruction that points to your scriptr account.
curl -X POST  -F apsws.time=1432726939153 -F commands=@command.mp3 -H 'Authorization: bearer YOUR_TOKEN' 'https://api.scriptr.io/modules/wit/handleCommand.js'

Below is what you should get as output:
```
>> curl -X POST  -F apsws.time=1432726939153 -F commands=@command.mp3 -H 'Authorization: bearer YOUR_TOKEN' 'https://api.scriptr.io/modules/wit/handleCommand.js'
2015-05-27 11:42:26,446 LOG [modules/wit/handleCommand] instanciating wit client
2015-05-27 11:42:34,983 LOG [modules/wit/handleCommand] received following response form wit.ai {"status":"200","headers":{"Connection":"keep-alive","Transfer-Encoding":"chunked","Content-Type":"application/json","Date":"Wed, 27 May 2015 11:42:34 GMT","Server":"nginx/1.8.0"},"body":"{\n  \"msg_id\" : \"e5553a76-b11f-33f5-b345-22d5677s897f989f\",\n  \"_text\" : \"find nearest gas station in 500 meter perimeter\",\n  \"outcomes\" : [ {\n    \"_text\" : \"find nearest gas station in 500 meter perimeter\",\n    \"intent\" : \"get_nearest_venue\",\n    \"entities\" : {\n      \"distance\" : [ {\n        \"value\" : 500,\n        \"type\" : \"value\",\n        \"unit\" : \"metre\"\n      } ],\n      \"local_search_query\" : [ {\n        \"suggested\" : true,\n        \"value\" : \"nearest gas station\",\n        \"type\" : \"value\"\n      } ]\n    },\n    \"confidence\" : 0.55\n  } ]\n}","timeout":false}
2015-05-27 11:42:34,983 LOG [modules/wit/handleCommand] next API invocation : {"script":"./test/getNearestVenue.js","params":{"radius":500,"venue":"nearest gas station"}}
 
{
    "metadata": {
        "requestId": "0de441b8-0585-4ced-ad40-57fd2f103aad",
        "status": "success",
        "statusCode": "200"
    },
    "result": {
        "radius": 500,
        "venue": "nearest gas station"
    }
}
```
