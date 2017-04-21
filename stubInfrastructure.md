Stub Infrastructure
=======
This document describes stub infracture used for "Mobile Ordering Orchestration" service.

We will be implementing an infrastructure to use stub in place of actual EAS ( EAS FULL FORM ) service. This infrastructure will be used for development enviornment so that MOO service can be developed with depending on EAS service.

The implemention for this model will include a JSON file which will store a replica of JSON response which can be received from EAS service. For the implementation we are using [bluebird.js](http://bluebirdjs.com/docs/getting-started.html), The bluebird library is already in use in MOO service to provide promises object and transform callback into promises.

## The Stub Implementation
Stubs are implemented in a directory structure in MOO service under stubs directory, we will be having seperate JSON file for each API request parameter. The idea is to provide with unique and realtime like respone to each EAS service call.

Here is an example of stub implementaion and directory structure
```
        {project_home}/stubs/dreams for Dreams stubs
        {project_home}/stubs/sf for SF stubs
        {project_home}/stubs/eas for EAS stubs
```
The idea here is, when we make a backend call to,
For example:
```
        http://sf.disney.com/assembly/api/v1/itineraries/54321
```
Then this will load response from file:
```
        {project_home}/stubs/sf/assembly/api/v1/itineraries/54321.json
```

### JSON file layout
The stubs directory will contain JSON file, This JSON file will have JSON object with object key as body and status code. We will expose this object so that it can be accesable from the callee function.

Here is an example of JSON layout and how it will manage response body and status code.
```
        module.exports = {
          "body": {
            "responseKeyOne": "response value",
            "responseKeyTwo": "response value"
          },
          "statusCode": 200
        };
```

## Node Implementation to call stub infrastructure
Node implementation will be managed by a enviornment variable which will decide whether to call EAS service or stub infrastucture.

Here is as example showing node implementation call:
```
        // check for enviornment variable to call EAS service
        if (EAS_ENVIORNMENT){
          request(reqOptions)
          .pipe(res); //request is a npm library used to make HTTP/HTTPS calls
        } else {

          //call node implementation for stub infrastructure
          stubInfrastructure(destinationId)
          .then(function (response){
            //response Object with JSON data having response body and status code
          })
          .catch(function (error){
            //error response from stub infrastructure
          })
        }
```

stubInfrastructure function in the above example will read JSON file stored in stub directory and make use of [bluebird.js](http://bluebirdjs.com/docs/getting-started.html) to return JSON object as a promise.