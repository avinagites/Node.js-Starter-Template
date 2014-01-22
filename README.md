Node.js-Starter Template
===================

A simple but effective Node.js Starter Template using ExpressJS, MongoDB, MongoJS & SocketIO.

<h3>Intro</h3>

This is more of a template than a fully fledged boilerplate. The idea here is to show how to configure each of the above components and to leave behind an easily editable set of files that can be customised into most types of websites or web applications. You can either download the files, install the modules and use it to make whatever you want or follow the walkthrough to learn about how the whole thing works, meant for people just starting out with Node. It's targetted at beginners so, to any experienced Node developers, you can stop reading now! 

<h3>Modules and Pre-requisites</h3>

Express is used as the server-side framework giving us easy ways to provide typical server functionality. Socket.IO allows us to use WebSockets as our data transfer method which gives us real-time capabilities and a nice substitute for AJAX. MongoDB will be our data store and MongoJS our simplified API, making interacting with MongoDB from script easy. To show these modules in action I've made a basic chat application as it's a nice way of showing how the components can be used together and leaves a basic framework that is easily edited to perform all the functionality needed by a typical website.

<h3>Setup</h3>

First thing I'd recommend doing is to get up to date with the latest stable version of Node. This is really easy - make sure you're in your root directory and type the following into the command line:

`````
sudo npm cache clean -f
sudo npm install n –g
sudo n stable 
`````

Check your version of Node by typing:

`````
node -v
`````

At time of writing the latest stable version was 0.10.18. Now create a new directory for the project. Choose a name and type:

`````
mkdir directoryname
cd directoryname
`````

The first thing to put into your new folder is a 'package.json' file to specify your project details. Open up your text editor, create a new json file and copy and paste the below JSON object in. Replace the name, version etc with whatever you wish. The 'start' value is what you'll type into the command line to start your application. I'm going to call my Node script 'script.js' but you can call it whatever you want. Check the version numbers of the dependencies at https://npmjs.org/ and update the JSON accordingly. Add extra dependencies if you want.

`````json
{
    "name": "boilerplate",
    "version": "0.0.1",
    "private": true,
    "author": "Mike Chadwick",
    "description": "A Node.js boilerplate",
    "scripts": {
        "start": "node script.js"
    },
    "dependencies": {
		"express": "3.4.0",
		"socket.io": "0.9.16",
		"mongodb": "1.3.19",
		"mongojs": "0.9.4"
    },
    "engine": "node 0.10.18"
}
`````

For a complete guide to creating a package file take a look here http://package.json.nodejitsu.com/, the above is short but it will work fine. Feel free to add to it if you want. Now we've specified the modules we'll be using, installing them is a breeze. Save the file (with a .json extension) and upload it to your directory. Make sure you're in your new directory and on the command line type:

`````
sudo npm install
`````
<h3>The Server Script</h3>

This will install all the modules / dependencies listed in the package file into a node_modules folder, within your directory. Now the fun begins. First we'll need to create a Node script file to handle all the server side behaviour. Create a new JavaScript file and save it as whatever you specified in your package.json file. As we're using Express as the framework for our application the first thing to do is initialise a new Express app. Type the following into your script:

`````javascript
var express = require( 'express' ),	//	import express module
	app = express();	//	create new express application

`````

Now let's create a HTTP server using the Express 'app' object we've just created. Add it to the list of variables like so:

`````javascript
var express = require( 'express' ),	//	import express module
	app = express(),	//	create new express application
	server = require( 'http' ).createServer( app );	//	create the server to listen for connections
`````
<h3>The 'Public' Folder</h3>

Ok so now create a 'public' folder within your working directory for all your static files (HTML, CSS, JS, images etc.). Call it whatever you want. Create the directory and an index.html file and pop it in there (stick a h1 tag in there or a page title or something so you can tell if your server's working when we boot it up). Eventually your app's directory structure will look something like this:

`````
├── app
│   ├── script.js
│   ├── package.json
│   ├── public
│       ├── css
│	    	  ├── style.css
│   	  ├── js
│   	  ├── img
│ 	  ├── index.html
│   	  └── chat.html
`````

Now we need to tell the server which port to listen for connections on (I'm using 1337) and point our app to the directory holding all our static files. Add the following lines to your script:

`````javascript
//	listen for connections on port 1337
server.listen( 1337 );

//	specify static file directory to serve to users
//	e.g. html, css, js, images etc	
app.use( express.static( __dirname + '/public' ) );	

`````

Now we can test the Node server is working. Save and upload your script to your project's root directory (not the 'public' directory - this is important). Go to the command line and type to boot the server:

`````
node-dev script.js
`````

Replace 'script.js' with whatever you've named your Node script. Open up your browser and test your application is working, specifying the port in the URL e.g. www.mysite.com:1337. Your index.html file will now be served to the browser. The reason we typed 'node-dev' and not just 'node' on the command line is that the '-dev' bit allows us to make changes to our script file, upload them and have the server automatically restart, saving us from manually having to do it.

<h3>Index (Login) Page</h3>

Below is a snippet of the HTML for the index page. I've kept it nice and simple; we'll use local storage to store the username of the user and start using sockets on the next page when we create our actual chat application.

`````html

          <label for="name">Please enter your name</label>
          <input type="text" class="form-control" id="name" autofocus placeholder="Enter name">
          
          <p class="help-block">Enter your name to login in to chat.</p>
          <button id="submit" class="btn btn-default">Submit</button>
    
   
<script src="//ajax.googleapis.com/ajax/libs/jquery/2.0.3/jquery.min.js"></script>
<script>

	$( '#submit' ).click( function () {
		var name = $( '#name' ).val();
		if( name.length < 3 ) {
			$( '.help-block' ).text( 'Please enter a name with at least 3 characters.' );
		}
		else {
			localStorage.setItem( 'user', name );
			window.location = 'chat.html';		
		}
    });

</script>
</body>
</html>
`````

Now we need to create a chat.html page to handle our chat app. Like everything else that's served to the user we'll need to put it in the public folder.

<h3>MongoDB / MongoJS</h3>

The way the chat app will work is simple; MongoDB will store the user's name and socket id and we'll update the database everytime a user connects or disconnects, allowing us to display who's online at any given time. We'll let people choose whether they want their message to transmit to all the users online or to send a 'private' message to a specific user. Let's begin by configuring MongoDB. On the command line simply type 'mongo' to enter the Mongo shell. First off, here are a few useful Mongo commands. Use these to create your database and collection to store the user details. I've called my database 'chat' and my collection 'users' (you call them whatever you want): 

Show all databases (usually there is a local one created by default).

`````
show dbs
`````
Create a database:

`````
use mydatabasename
`````
Create a 'collection' (the Mongo equivalent of a table)

`````
db.createCollection("collection_name") 
`````
Show all collections in the database

`````
show collections
`````
Insert a record (Mongo equivalent of a row) into the collection

`````
db.collection_name.insert( { attr_name : value, attr2_name : value… etc. } ) 
`````
Remove a record from the collection

`````
db.collection_name.remove( { attr_name : value, attr2_name : value… etc. } )
`````

Now we have our database and collection let's hook it up to our server script. We'll use MongoJS which provides a simple API to let us easily interact with our database. In your Node script, add the following to the list of variables:

`````javascript
ObjectId = require( 'mongodb' ).ObjectID,	//	create ObjectId object to access the id within mongo collections
databaseUrl = "mongodb://localhost:27017/chat", //	specify database name here
collections = [ "users" ],	//	specify collections within database here
db = require( "mongojs" ).connect( databaseUrl, collections );	//	create database object
`````

<h3>Socket.IO</h3>

OK, time to incorporate Socket.IO. Socket.IO gives us WebSocket functionality, bringing with it all the benefits that WebSockets have over HTTP connections. We'll use WebSockets as a replacement for AJAX so we can send data to and from the server to users without the need for a page refresh. Go back to your script file and add the following to your list of variables:

`````javascript
io = require( 'socket.io' ).listen( server );	//	listen for socket events
`````

This will import the Socket.IO module and tell it to listen for socket events via our server. For a full list of all events available through the Socket.IO object created ('io') have a look here: https://github.com/LearnBoost/socket.io/wiki/Exposed-events. The first event we'll use is the 'connection' event, which fires every time a new connection is made to the server from any web page in our application. From this event we can grab the socket details for every user who connects to our app.


When you broadcast a socket event from the server it will emit globally to all sockets connected to the app, unless you specify a specific socket id to send it to. Therefore we'll need to grab the socket id of each user and store it on the client and the server (in our database) to allow us to broadcast to specific users and identify who has disconnected when a user shuts the browser window down. Beneath the public folder declaration in your Node script add the following:

`````javascript
io.sockets.on( 'connection', function ( socket ) {
	// grab socket.id of newly connected user here
	// and do other stuff
});
`````

Now we just need to hook it up on the client-side to establish the connection to the server. We don't need it for the index page as we're using local storage to store the user's name but we'll use it for the chat.html page. The first thing we'll do on here is connect to Socket.IO. I've put the script tags in the document head. You can put them above the closing body tag but establishing a socket connection is necessary to get essential page content, so I'd rather put them in the head. Here's how it looks:

`````html
<script src="/socket.io/socket.io.js"></script>
<script>
	//	connect to socket on server
	var socket = io.connect( 'http://mywebsite.com:1337' ),
		user = localStorage.getItem( 'user' );
			
	if ( user === undefined  ) {
		window.location = 'index.html';			
	}
	else {
		localStorage.removeItem( 'user' );
	}
</script>
`````
