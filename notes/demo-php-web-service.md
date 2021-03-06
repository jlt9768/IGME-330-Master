# Demo: PHP-driven Web Service

## Overview

- We can use the PHP scripting language to return data in the JSON and JSON-P formats - thus creating our own web service:
  - https://en.wikipedia.org/wiki/JSON
  - https://en.wikipedia.org/wiki/JSONP

## I. Simple Web Service (JSON only)

- This web service returns a random joke in the JSON format
- Here is a working version: http://igm.rit.edu/~acjvks/courses/2018-fall/330/php/get-a-joke.php

### I-A. The *server* code (in PHP)

**get-a-joke.php**

```php
<?php
/*
	Name: get-a-joke.php
	Description: Returns a single random joke in JSON format
	Author: Keyser Soze
	Last Modified: 11/5/2018
*/

// $jokes contains our data
// this is an indexed array of associative arrays
// the associative arrays are jokes -  they have an 'q' key and an 'a' key
$jokes = array(
	array("q"=>"What do you call a very small valentine?","a"=>"A valen-tiny!"),
	array("q"=>"What did the dog say when he rubbed his tail on the sandpaper?","a"=>"Ruff, Ruff!"),
	array("q"=>"Why don't sharks like to eat clowns?","a"=>"Because they taste funny!"),
	array("q"=>"What did the boy cat say to the girl cat?","a"=>"You're Purr-fect!"),
	array("q"=>"What is a frog's favorite outdoor sport?","a"=>"Fly Fishing!")
);

// get a random element of the $jokes array
// there are a bunch more PHP array functions at: http://php.net/manual/en/ref.array.php
$numJokes = count($jokes);
$randomJoke = $jokes[mt_rand(0, $numJokes - 1)];

// json_encode() turns an associative array into a string of well-formed JSON
$string = json_encode($randomJoke);

// Sends the correct HTTP header
// header("Access-Control-Allow-Origin: *");
header('content-type:application/json');
echo $string;

?>
```

### I-B. The *client* code (Node.js JavaScript)

- You can create a node project like we did here - [node-and-web-services-2.md](./node-and-web-services-2.md) - and run this code. 
- **SUCCESS!** - It will call the web service and print out a random joke:

```
***The joke is:***
What do you call a very small valentine?

A valen-tiny!
```

- Here's the Node.js code:

**random-joke/index.js**

```js
// #1 - import the request module, which is used to download data over http
const request = require('request');

// #2 - set our URL
let url = "http://igm.rit.edu/~acjvks/courses/2018-fall/330/php/get-a-joke.php";

// #3 - make the request
// the second parameter below is a callback function (an ES6 arrow function in this case)
// which is called when the data is downloaded
request(url, (err, response, body) => {
    // if there's no error, and if the server's status code is 200 (i.e. "Ok")
    if(!err && response.statusCode == 200){
    	// A - convert the downloaded text to a JavaScript Object
        let obj = JSON.parse(body); 
        let text = `${obj.q}\n\n${obj.a}`;
        // B - log it out
        console.log(`***The joke is:***\n ${text}`);
    }else{
    	console.log(`err=${err}`);
    }
});
```

### I-C. The *client* code (Web Browser JavaScript - `jQuery.ajax()`)

- Here we are going to try to download this JSON data utilizing `jQuery.ajax()`

**get-a-joke-jquery-ajax-json-start.html**

```html
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="utf-8" />
 	<title>Get a joke JSON!</title>

 <!-- Import jQuery -->
  <script src="https://code.jquery.com/jquery-1.11.1.min.js"></script>
  
  <script>
  	"use strict";
	const URL = "http://igm.rit.edu/~acjvks/courses/2018-fall/330/php/get-a-joke.php";
	window.onload = init;
	
	function init(){
		document.querySelector("#search").onclick = getData;
	}
	
	// MY FUNCTIONS
	function getData(){
		let url = URL;
		console.log("loading " + url);
		
		// use jQuery
	$.ajax({
		  dataType: "json",
		  url: url,
		  data: null,
		  success: jsonLoaded
		});

	
	}
	

	function jsonLoaded(obj){
		console.log("obj stringified = " + JSON.stringify(obj));

		/*
			Write code to display the .q and .a properties of the joke
		*/

		//document.querySelector("#content").innerHTML = bigString;
	}

 </script>
  
  
</head>
<body>
 <h1>Jokes!</h1>


<button type="button" id="search">Get Joke!<br />:-O</button>

<h2>Results</h2>
 <div id="content">
 <p>No data yet!</p>
 </div>
 

</body>
</html>
```

- A. **FAIL!** - Here's the error message: 

```
Access to XMLHttpRequest at 'http://igm.rit.edu/~acjvks/courses/2018-fall/330/php/get-a-joke.php' from origin 'null'
has been blocked by CORS policy: No 'Access-Control-Allow-Origin' header is present on the requested resource.
```

- B. CORS stands for "Cross-Origin Resource Sharing" 

- "Cross-Origin Resource Sharing (CORS) is a mechanism that uses additional HTTP headers to tell a browser to let a web application running at one origin (domain) have permission to access selected resources from a server at a different origin." - https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS

- C. In this case, we can enable CORS for this web service because we are the ones that wrote it. Add this header to your PHP script (*before* the `echo()` statement):

```php
header("Access-Control-Allow-Origin: *");
```

- D. Now check the Network tab in the Web Inspector to see this new header:

![Screenshot](_images/php-web-service-1.jpg)

- E. Try your browser client again - you should be able to download the JSON now!

- F. ***Summary: Web browsers can NOT directly download JSON data from another domain unless CORS is enabled (or the browser's security restrictions are turned off).***

<hr><hr>

## II. Web Service with JSON and JSON-P
- So what if the web service you want to use does not enable CORS, and you don't have any control over the service? What do you do?
- Answer: JSON-P to the rescue!
- The web service below accepts a parameter for a named callback function, and if one is specified, it will wrap the JSON data inside of that callback function before returning it - check it out here: http://igm.rit.edu/~acjvks/courses/2018-fall/330/php/get-a-joke-2.php?callback=jsonloaded
- JSON wrapped inside of a callback function is called JSON-P
- JSON-P CAN be downloaded cross-domain, even if CORS is NOT turned on.
- NOTE: naming the callback function `jsonLoaded` is completely arbitrary - call it whatever you want - just be sure that your JS is changed to match the new function name

### II-A. The *server* code (in PHP)

**get-a-joke-2.php**

```php
<?php
/*
	Name: get-a-joke-2.php
	Description: Returns a single random joke in JSON or JSON-P format
	Author: Keyser Soze
	Last Modified: 11/5/2018
*/
if(array_key_exists('callback', $_GET)){
	$callback = $_GET['callback'];
}else{
	$callback = NULL;
}

// $jokes contains our data
// this is an indexed array of associative arrays
// the associative arrays are jokes -  they have an 'q' key and an 'a' key
$jokes = array(
	array("q"=>"What do you call a very small valentine?","a"=>"A valen-tiny!"),
	array("q"=>"What did the dog say when he rubbed his tail on the sandpaper?","a"=>"Ruff, Ruff!"),
	array("q"=>"Why don't sharks like to eat clowns?","a"=>"Because they taste funny!"),
	array("q"=>"What did the boy cat say to the girl cat?","a"=>"You're Purr-fect!"),
	array("q"=>"What is a frog's favorite outdoor sport?","a"=>"Fly Fishing!")
);

// get a random element of the $jokes array
// there are a bunch more PHP array functions at: http://php.net/manual/en/ref.array.php
$numJokes = count($jokes);
$randomJoke = $jokes[mt_rand(0, $numJokes - 1)];

// json_encode() turns an associative array into a string of well-formed JSON
$string = json_encode($randomJoke);

if ($callback){
	// Sends the correct HTTP header
	header('content-type:application/javascript');
	$string = $callback . "(" . $string . ")";
}else{
	// Sends the correct HTTP header
	header('content-type:application/json');
}

echo $string;

?>
```

### II-B. The *client* code (JavaScript utilizing `jQuery.ajax()`)

**get-a-joke-jquery-ajax-jsonp-start.html**

```html
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="utf-8" />
 	<title>Get a joke JSONP!</title>

 <!-- Import jQuery -->
  <script src="https://code.jquery.com/jquery-1.11.1.min.js"></script>
  
  <script>
  	"use strict";
	const URL = "http://igm.rit.edu/~acjvks/courses/2018-fall/330/php/get-a-joke-2.php";
	window.onload = init;
	
	function init(){
		document.querySelector("#search").onclick = getData;
	}
	
	// MY FUNCTIONS
	function getData(){
		let url = URL;
		console.log("loading " + url);
		
		// use jQuery
	$.ajax({
		  dataType: "jsonp",
		  url: url,
		  data: null,
		  success: jsonLoaded
		});

	
	}
	
	function jsonLoaded(obj){
		console.log("obj stringified = " + JSON.stringify(obj));
		/*
			Write code to display the .q and .a properties of the joke
		*/
	
		//document.querySelector("#content").innerHTML = bigString;
	}

 </script>
  
</head>
<body>
 <h1>Jokes!</h1>

<button type="button" id="search">Get Joke!<br />:-O</button>

<h2>Results</h2>
 <div id="content">
 <p>No data yet!</p>
 </div>
 
</body>
</html>
```


### II-C. The *client* code (JavaScript utilizing XSS)

- Now we will see how jQuery is successfully downloading JSON-P from another domain
- Basically, it dynamically creates a &lt;script> tag and set the `.src` property of that tag to the address of the JSON-P web service. 
- Once this JSON-P file (which is a JavaScript code file) is downloaded, the callback function is called, and passes the JSON data that was "Wrapped" in the call
- thsi technique is alterantively called "DOM Injection", "Script Tag hack", or XSS (Cross-Site Scripting)
- Below, we will download JSON-P without the help of a library

**get-a-joke-DOM-injection-jsonp-start.html**

```html
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="utf-8" />
 	<title>Get Joke -  DOM Injection Version</title>
 	<link href='https://fonts.googleapis.com/css?family=Audiowide' rel='stylesheet' type='text/css'>
 	<style>
 	*{
 		font-family:sans-serif;
 	}
 	
 	header{
 		background-color: crimson;
 		margin:0;
 		padding-left:5px;
 	}
 	
 	h1{
 		font-size:4.5em;
 		font-family: "Audiowide";
 		margin-top:0;
 		margin-bottom:0;
 		color:black;
 		letter-spacing:.025em;
 	}
 	
 	h2{
 		font-size:1.5em;
 		font-family: "Audiowide";
 		margin-top:0;
 		color:#eee;
 		letter-spacing:.04em;
 		white-space: nowrap;
 	}
 	
 	section>div{
 		background-color:#ccc;
 		margin:1em;
 		padding:.5em;
 	}
 	</style>

  <script>
  	"use strict";
	const URL = "http://igm.rit.edu/~acjvks/courses/2018-fall/330/php/get-a-joke-2.php";
	
	window.onload = init;
	
	function init(){
		document.querySelector("#search").onclick = search;
	}
	
	// MY FUNCTIONS
	function search(){
		// build url
		let url = URL;
		url += "?callback=jsonLoaded";
		
		
		// create <script> element and add to page
	
		console.log("loading: " + url);
	}
	

	function jsonLoaded(obj){
		console.log("obj stringified = " + JSON.stringify(obj));
		
		// get rid of <script> tag we made for this request
		
		
		let q = obj.q;
		let a = obj.a;

		/*
			Write code to display the .q and .a properties of the joke
		*/
		
		document.querySelector("#content").innerHTML = bigString;
		
	}

 </script>
  
  
</head>
<body>
<header>
 <h1>Joker Finder!</h1>
 <h2>The Joke tool you can trust&reg;</h2>
</header>
	<p id='status'><i>Status: Ready to laugh!</i></p>
	<p>
		<button id="search">Search!</button>
	</p>
	<hr>
	<h3>Results:</h3>
	<p id="content">???</p>
 
</body>
</html>
```
