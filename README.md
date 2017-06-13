# A simple Node client to get Enertiv data

## What are we trying to do ?

* Get data about Power in ITP from the enertiv server
* Try and make this data available on a website (realtime)

## Why are we using Node ?

## How to install Node

## Steps

### 1) Create HTTP Request to get access server

* Our first step here is to get credentials to access the server.
* We have some [sample details](https://api.enertiv.com/o/applications/) that you can use
* Replace the login details that you get here in the code below.
* Type the below code in `client.js`


```javascript

var loginData = querystring.stringify({
	'client_id':'abcd',
	'client_secret': '1234abcd',
	'grant_type': 'password',
	'username': 'itpower',
	'password': 'itpower123',
})

console.log(loginData);

```

* Now run the script with `node client.js`, you should see the following output

```
client_id=abcd&client_secret=1234abcd&grant_type=password&username=itpower&password=itpower123

```

* We also need to setup HTTP request options so that the server knows the details of the request it is receiving.
(Add the below snippet in `client.js`)

```javascript

// set up the HTTPS request options. You'll modify and
// reuse this for subsequent calls:
var loginRequestOptions = {
  rejectUnauthorized: false,
  method: 'POST',
  host: 'api.enertiv.com',
  port: 443,
  path: '/o/token/',
  headers: {
    'Content-Type': 'application/x-www-form-urlencoded',
    'Content-Length': loginData.length
  }
};

console.log(loginRequestOptions);

```
* Now run the script with `node client.js`, you should see the following output

```

{ rejectUnauthorized: false,
  method: 'POST',
  host: 'api.enertiv.com',
  port: 443,
  path: '/o/token/',
  headers:
   { 'Content-Type': 'application/x-www-form-urlencoded',
     'Content-Length': 94 } }

```
* We have created **loginData** which contains then login credentials and **loginRequestOptions** that has the details of the request.
* Now to make the actual HTTP request
* This function **https.request** takes the *httpsRequestOptions** as a parameter along with a callback function. For now, all our callback does is say wether the request went through or not. (Paste the below snippet in `client.js`)
* *Note: if the request was successful it will give us the response, 200*

```javascript

var request = https.request(
  loginRequestOptions, function(response){
    console.log("Response is : " + response.statusCode)
  });	// start it
request.write(loginData); // add  body of  POST request
request.end();

```

* Run `node client.js` You should see the below output

```
Response is : 200
```

* When we create a successful request to the server now, it will give us an access token. This is a token that says that we have the right access required to get data from the server.
* Therefore, once we successfully get access, we will save this token locally using the function **saveToken**( add the function below to `client.js`)


```javascript

var accessToken;
function saveToken(response) {
  var result = '';		// string to hold the response
  // as each chunk comes in, add it to the result string:
  response.on('data', function (data) {
    result += data;
  });

  // when the final chunk comes in, print it out:
  response.on('end', function () {
    result = JSON.parse(result);
    accessToken = result.access_token;
    console.log(result);
  });
}
```

* Also, change the callback in https.request function to saveToken. It should look like this

```javascript

var request = https.request(loginRequestOptions, saveToken);	s

```

* Run `node client.js` and you should see the below output

```
{ scope: 'read write',
  expires_in: 36000,
  access_token: 'hkW1czbvAnXgTxvYAK6uv4a0KoacwU',
  refresh_token: 'ZnMNwqz7Bg5mHJ5jrLYtxUV2AOSOeQ',
  token_type: 'Bearer' }
```


* ### **Checkpoint 1** ( Your code should now be looking something like this)

```javascript
var querystring = require('querystring');
var https = require('https');


var loginData = querystring.stringify({
	'client_id':'c99b7f5dec0d6a0f6178',
	'client_secret': '575af139440e5ae453d6171d14efd8ce3a4f3005',
	'grant_type': 'password',
	'username': 'mmg542@nyu.edu',
	'password': 'energyatitp',
})

console.log(loginData);

// set up the HTTPS request options. You'll modify and
// reuse this for subsequent calls:
var loginRequestOptions = {
  rejectUnauthorized: false,
  method: 'POST',
  host: 'api.enertiv.com',
  port: 443,
  path: '/o/token/',
  headers: {
    'Content-Type': 'application/x-www-form-urlencoded',
    'Content-Length': loginData.length
  }
};

console.log(loginRequestOptions);

var request = https.request(loginRequestOptions, saveToken);	// start it
request.write(loginData); // add  body of  POST request
request.end();

var accessToken;
function saveToken(response) {
  var result = '';		// string to hold the response
  // as each chunk comes in, add it to the result string:
  response.on('data', function (data) {
    result += data;
  });

  // when the final chunk comes in, print it out:
  response.on('end', function () {
    result = JSON.parse(result);
    accessToken = result.access_token;
    console.log(result);
  });
}


```

###2) Get Data from the server

* Now that we are able to access the server,let's get some data from the server.
* We need to create a new set of options for the second request.
* Add this code snippet in `client.js` after **loginRequestOptions** is declared.

```javascript
// this is already in client.js
var loginRequestOptions = {
  rejectUnauthorized: false,
  method: 'POST',
  host: 'api.enertiv.com',
  port: 443,
  path: '/o/token/',
  headers: {
    'Content-Type': 'application/x-www-form-urlencoded',
    'Content-Length': loginData.length
  }
};

// Add the new options here
var dataRequestOptions = {
  rejectUnauthorized: false,
  method: 'GET',
  host: 'api.enertiv.com',
  port: 443,
  path: '/o/token/',
  headers: {
    'Content-Type': 'application/x-www-form-urlencoded',
    'Content-Length': loginData.length
  }
};

```
* Let's create a function to request for the client information (that is, information of who YOU are).
* We will create a function called **getClientInfo(path,token)** This function takes parameters **path** (the endpoint that you are trying to get data from) and **token** (this is the access token that the server gave us when we logged in).
* Add this to the bottom of `client.js`

```javascript
var clientData;

function getClientInfo(path, token) {
  dataRequestOptions.path = path;
  dataRequestOptions.headers = {
    'Authorization': 'Bearer ' + token
  }
  request = https.get(dataRequestOptions, function (response) { // make the API call
    var result = '';
    // as each chunk comes in, add it to the result string:
    response.on('data', function (data) {
      result += data;
    });

    // when the final chunk comes in, print it out:
    response.on('end', function () {
      result = JSON.parse(result);
      clientData = result;
      console.log('****************');
      console.log(clientData);
      console.log('****************');
      console.log(accessToken);
      console.log('****************');
    });
  });
}
```

* Now let's change the code so that instead of showing us our accessToken, the **saveToken** function will use the accessToken to get the client information.
* Add the below line in `client.js` in the **saveToken** function after defining the **accessToken**

```javascript

		getClientInfo('/api/client/', accessToken);

```

* the endpoint `/api/client/` can be found in the list of [defined enertiv endpoints](https://api.enertiv.com/docs/)

* That is, the saveToken function should now look like this

```javascript
function saveToken(response) {
  var result = '';		// string to hold the response
  // as each chunk comes in, add it to the result string:
  response.on('data', function (data) {
    result += data;
  });

  // when the final chunk comes in, print it out:
  response.on('end', function () {
    result = JSON.parse(result);
    accessToken = result.access_token;
    getClientInfo('/api/client/', accessToken);
    console.log(result);
  });
}
```


* Test again by running `node client.js`, you should get something like this

```

{ scope: 'read write',
  expires_in: 36000,
  token_type: 'Bearer',
  access_token: '69KpF8GtK1Jl2ugy3EJz3g3TjtMNqh',
  refresh_token: '2vDTSZnWC6Edti1utF0JpDZteQ2uel' }
****************
[ { id: 'ad4394d7-ad81-4d1c-adea-51b3213c291c',
    parent_client: null,
    subscription: 'Basic',
    name: 'NYU ITP',
    address_1: null,
    address_2: null,
    city: null,
    state: null,
    suite: null,
    floor: null,
    zip: null,
    zip4: null,
    logo: 'logos/ITP_2_sIrP1nN.png',
    locations: [ '5ae33444-387d-46f6-9be0-3c4b54f53561' ],
    move_in_date: null,
    move_out_date: null,
    property_values: {} } ]
****************
69KpF8GtK1Jl2ugy3EJz3g3TjtMNqh
****************

```

* Let us now change the endpoint to try and get some different data.
* There is an endpoint that will give us information about the location where we are running the enertiv system (in this case, ITP). [endpoint](https://api.enertiv.com/docs/#!/location/location_read)
This endpoint requires a `location_id` which we got in our previous response.
* So, let us change the **getClientInfo** function parameters to look like this -

```javascript
// getClientInfo('/api/client/', accessToken);
getClientInfo('/api/location/5ae33444-387d-46f6-9be0-3c4b54f53561/', accessToken);
```

* run `node client.js` and you will get a complete list of all the sublocations in ITP.

* ### **Checkpoint 2** ( Your code should now be looking something like this)

``` javascript

var querystring = require('querystring');
var https = require('https');


var loginData = querystring.stringify({
	'client_id':'c99b7f5dec0d6a0f6178',
	'client_secret': '575af139440e5ae453d6171d14efd8ce3a4f3005',
	'grant_type': 'password',
	'username': 'mmg542@nyu.edu',
	'password': 'energyatitp',
})

console.log(loginData);

// set up the HTTPS request options. You'll modify and
// reuse this for subsequent calls:
var loginRequestOptions = {
  rejectUnauthorized: false,
  method: 'POST',
  host: 'api.enertiv.com',
  port: 443,
  path: '/o/token/',
  headers: {
    'Content-Type': 'application/x-www-form-urlencoded',
    'Content-Length': loginData.length
  }
};

var dataRequestOptions = {
  rejectUnauthorized: false,
  method: 'GET',
  host: 'api.enertiv.com',
  port: 443,
  path: '/o/token/',
  headers: {
    'Content-Type': 'application/x-www-form-urlencoded',
    'Content-Length': loginData.length
  }
};

console.log(loginRequestOptions);

var request = https.request(loginRequestOptions, saveToken);	// start it
request.write(loginData); // add  body of  POST request
request.end();

var accessToken;
function saveToken(response) {
  var result = '';		// string to hold the response
  // as each chunk comes in, add it to the result string:
  response.on('data', function (data) {
    result += data;
  });

  // when the final chunk comes in, print it out:
  response.on('end', function () {
    result = JSON.parse(result);
    accessToken = result.access_token;
    getClientInfo('/api/location/5ae33444-387d-46f6-9be0-3c4b54f53561/', accessToken);
    console.log(result);
  });
}

var clientData;

function getClientInfo(path, token) {
  dataRequestOptions.path = path;
  dataRequestOptions.headers = {
    'Authorization': 'Bearer ' + token
  }
  request = https.get(dataRequestOptions, function (response) { // make the API call
    var result = '';
    // as each chunk comes in, add it to the result string:
    response.on('data', function (data) {
      result += data;
    });

    // when the final chunk comes in, print it out:
    response.on('end', function () {
      result = JSON.parse(result);
      clientData = result;
      console.log('****************');
      console.log(clientData);
      console.log('****************');
      console.log(accessToken);
      console.log('****************');
    });
  });
}
```

###3) Getting realtime data

* We can also choose one equipment on the floor, and get its power consumption data in real time

* To do this, we will use the following [endpoint](https://api.enertiv.com/docs/#!/equipment/equipment_data_list)
* This endpoint requires us to pass the equipment id, along with start and end time. It also requires the time interval of data required.
* Let us try and get the data for the first minute in 2017. Replace the getClientInfo function call with the below code in `client.js`

```javascript
    // construct url

    var fromTime = new Date('01/01/2017T00:00:00Z');
    var toTime = new Date('01/01/2017T00:02:00Z');

    var url = '/api/equipment/a40be1ed-5a9d-4b35-b500-0aff698e8c79/data/?fromTime=' +
    fromTime.toISOString() +
    '&toTime=' +
    toTime.toISOString() +
    '&interval=min';

    getClientInfo(url, accessToken);

```
* Run `node client.js` and you should see the below

```
{ units: 'kW',
  names: [ 'HVAC Motor, Signaling Device 2' ],
  data:
   [ { x: '2017-01-01T00:00:00Z',
       'HVAC Motor, Signaling Device 2': 0 },
     { x: '2017-01-01T00:01:00Z',
       'HVAC Motor, Signaling Device 2': 0 } ] }

```

* Now let us change the definition of **fromTime** and **toTime** so that it is the most recent minute.

```javascript

var toTime = new Date();
toTime.setHours(toTime.getHours());
var fromTime = new Date(toTime);

var durationInMinutes = 3;
console.log(toTime);
console.log(fromTime);
fromTime.setMinutes(toTime.getMinutes() - durationInMinutes);

```

* Run 'node client.js' again and you should see something similar the below -

```

{ units: 'kW',
  names: [ 'HVAC Motor, Signaling Device 2' ],
  data:
   [ { x: '2017-06-12T00:18:00Z',
       'HVAC Motor, Signaling Device 2': 3.2997 } ] }

```

* The last thing that we want to do is call the http request every 10s/30s so that we get a constant stream of realtime data
* To do this, wrap the http request call in a setInterval function.
* That is, replace the below code snippet...

```javascript

var request = https.request(loginRequestOptions, saveToken);	// start it
request.write(loginData); // add  body of  POST request
request.end();

```

with the below code snippet.

```javascript
setInterval(function() {
  var request = https.request(httpsRequestOptions, saveToken);	// start it
  request.write(loginData);                       // add  body of  POST request
  request.end();
  console.log('Hello');
}, 10000);

```

* when you run `node client.js` now, you will see the data being sent every 10s/30s
## Express
* How to install Express?
`$ npm install express`

* Add express to the `client.js`
```javascript
var express = require('express'); // include the express library
var server = express();           // create a server using express

// start the server:
server.listen(8080);

server.use('/', express.static('public'));   // set a static file directory

// send message to the client
function handleRequest(request, response) {
  response.send(clientData);         // send message to the client
  response.end();                 // close the connection
}

// define what to do when the client requests `/data`:
server.get('/data', handleRequest);         // GET request
```
* At this point, the `client.js` is complete.

## Visualization using p5
* Add a `public` folder with an empty p5 sketch template
* In the sketch.js, use `loadJSON` method to get data from `http://localhost:8080/data` every 10 seconds.
* parse data into `device`, `time`, `usage`, and make a chart.

* Another p5 example is in the `enertive_p5_static_json` folder
[demo](http://alpha.editor.p5js.org/mathura/sketches/Sy86Na7-x)

## References
* [enertiv bitbucket](https://bitbucket.org/enertiv/enertiv-client/)
* [enertiv endpoints](https://api.enertiv.com/docs/)

## Glossary
* **Access Token** - An equivalent of a password given to us by the server. It gives us access to login to the server.
* **HTTP Request**
* **GET and POST Requests** -
* **endpoints** - a unique URL that represents an object/data or collection of objects/data
