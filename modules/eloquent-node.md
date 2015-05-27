<!--
name: eloquent-node
version : 0.0.1
title : "Eloquent JavaScript - Chapter 20 Node.js"
description: "This is a book about JavaScript, programming, and the wonders of the digital."
homepage : "http://eloquentjavascript.net/"
author : "Marijn Haverbeke"
freshnessDate : 2014-12-14
license : "CC BY-NC"
-->

<!-- @section, "title": "Getting going" -->
So far, you have learned the JavaScript language and used it within a single environment: the browser. This chapter and the next one will briefly introduce you to Node.js, a program that allows you to apply your JavaScript skills outside of the browser. With it, you can build anything from simple command-line tools to dynamic HTTP servers.

These chapters aim to teach you the important ideas that Node.js builds on and to give you enough information to write some useful programs for it. They do not try to be a complete, or even a thorough, treatment of Node.

Whereas you could run the code in previous chapters directly on these pages, since it was either raw JavaScript or written for the browser, the code samples in this chapter are written for Node and won’t run in the browser.

If you want to follow along and run the code in this chapter, start by going to nodejs.org and following the installation instructions for your operating system. Also refer to that website for further documentation about Node and its built-in modules.

<!-- @link, "url" : "https://nodejs.org", "text" : "Install Node.js for your operating system." -->

<!-- @asset, "contentType" : "outlearn/prototype-feature", "text" : "{ \"multiple_choice\": \"The code samples in this chapter cannot be run directly in the browser because\", \"answers\" : [ \"They are raw JavaScript\", \"They require Node.\", \"Because they only work on the newest browsers.\" ], \"correct\" : 1 }"-->

<!-- @section -->

# Background

One of the more difficult problems with writing systems that communicate over the network is managing input and output—that is, the reading and writing of data to and from the network, the hard drive, and other such devices. Moving data around takes time, and scheduling it cleverly can make a big difference in how quickly a system responds to the user or to network requests.

The traditional way to handle input and output is to have a function, such as readFile, start reading a file and return only when the file has been fully read. This is called synchronous I/O (I/O stands for input/output).

Node was initially conceived for the purpose of making asynchronous I/O easy and convenient. We have seen asynchronous interfaces before, such as a browser’s XMLHttpRequest object, discussed in Chapter 17. An asynchronous interface allows the script to continue running while it does its work and calls a callback function when it’s done. This is the way Node does all its I/O.

JavaScript lends itself well to a system like Node. It is one of the few programming languages that does not have a built-in way to do I/O. Thus, JavaScript could be fit onto Node’s rather eccentric approach to I/O without ending up with two inconsistent interfaces. In 2009, when Node was being designed, people were already doing callback-based I/O in the browser, so the community around the language was used to an asynchronous programming style.

<div class="hack-ssh-access">
  <div class="before-click">
    Follow along with these examples in a ready-made NodeJS environment.
  </div>
  <div class="after-click hidden">
    Paste <em>'ssh nodejs@52.4.173.58'</em> into your terminal, and login with password <em>'outlearn'</em>.
  </div>
</div>

<!-- @section -->

# Asynchronicity

I’ll try to illustrate synchronous versus asynchronous I/O with a small example, where a program needs to fetch two resources from the Internet and then do some simple processing with the result.

In a synchronous environment, the obvious way to perform this task is to make the requests one after the other. This method has the drawback that the second request will be started only when the first has finished. The total time taken will be at least the sum of the two response times. This is not an effective use of the machine, which will be mostly idle when it is transmitting and receiving data over the network.

The solution to this problem, in a synchronous system, is to start additional threads of control. (Refer to Chapter 14 for a previous discussion of threads.) A second thread could start the second request, and then both threads wait for their results to come back, after which they resynchronize to combine their results.

In the following diagram, the thick lines represent time the program spends running normally, and the thin lines represent time spent waiting for I/O. In the synchronous model, the time taken by I/O is part of the timeline for a given thread of control. In the asynchronous model, starting an I/O action conceptually causes a split in the timeline. The thread that initiated the I/O continues running, and the I/O itself is done alongside it, finally calling a callback function when it is finished.

Control flow for synchronous and asynchronous I/O
Another way to express this difference is that waiting for I/O to finish is implicit in the synchronous model, while it is explicit, directly under our control, in the asynchronous one. But asynchronicity cuts both ways. It makes expressing programs that do not fit the straight-line model of control easier, but it also makes expressing programs that do follow a straight line more awkward.

In Chapter 17, I already touched on the fact that all those callbacks add quite a lot of noise and indirection to a program. Whether this style of asynchronicity is a good idea in general can be debated. In any case, it takes some getting used to.

But for a JavaScript-based system, I would argue that callback-style asynchronicity is a sensible choice. One of the strengths of JavaScript is its simplicity, and trying to add multiple threads of control to it would add a lot of complexity. Though callbacks don’t tend to lead to simple code, as a concept, they’re pleasantly simple yet powerful enough to write high-performance web servers.

<!-- @section -->

# The node command

When Node.js is installed on a system, it provides a program called node, which is used to run JavaScript files. Say you have a file hello.js, containing this code:

```
var message = "Hello world";
console.log(message);
```

You can then run node from the command line like this to execute the program:

```
$ node hello.js
Hello world
```

The console.log method in Node does something similar to what it does in the browser. It prints out a piece of text. But in Node, the text will go to the process’ standard output stream, rather than to a browser’s JavaScript console.

If you run node without giving it a file, it provides you with a prompt at which you can type JavaScript code and immediately see the result.

```javascript
$ node
> 1 + 1
2
> [-1, -2, -3].map(Math.abs)
[1, 2, 3]
> process.exit(0)
$
```

<!-- @task, "text" : "Run node and try executing some commands of your choice."-->

The process variable, just like the console variable, is available globally in Node. It provides various ways to inspect and manipulate the current program. The exit method ends the process and can be given an exit status code, which tells the program that started node (in this case, the command-line shell) whether the program completed successfully (code zero) or encountered an error (any other code).

To find the command-line arguments given to your script, you can read `process.argv`, which is an array of strings. Note that it also includes the name of the node commands and your script name, so the actual arguments start at index 2. If `showargv.js` simply contains the statement `console.log(process.argv)`, you could run it like this:

```javascript
$ node showargv.js one --and two
["node", "/home/marijn/showargv.js", "one", "--and", "two"]
```

<!-- @task, "text" : "Create and run showargv.js as described in the text."-->

All the standard JavaScript global variables, such as Array, Math, and JSON, are also present in Node’s environment. Browser-related functionality, such as document and alert, is absent.

The global scope object, which is called window in the browser, has the more sensible name global in Node.

<!-- @section -->

# Modules

Beyond the few variables I mentioned, such as console and process, Node puts little functionality in the global scope. If you want to access other built-in functionality, you have to ask the module system for it.

The CommonJS module system, based on the require function, was described in Chapter 10. This system is built into Node and is used to load anything from built-in modules to downloaded libraries to files that are part of your own program.

When require is called, Node has to resolve the given string to an actual file to load. Pathnames that start with "/", "./", or "../" are resolved relative to the current module’s path, where "./" stands for the current directory, "../" for one directory up, and "/" for the root of the file system. So if you ask for "./world/world" from the file /home/marijn/elife/run.js, Node will try to load the file /home/marijn/elife/world/world.js. The .js extension may be omitted.

When a string that does not look like a relative or absolute path is given to require, it is assumed to refer to either a built-in module or a module installed in a node_modules directory. For example, require("fs") will give you Node’s built-in file system module, and require("elife") will try to load the library found in node_modules/elife/. A common way to install such libraries is by using NPM, which I will discuss in a moment.

To illustrate the use of require, let’s set up a simple project consisting of two files. The first one is called main.js, which defines a script that can be called from the command line to garble a string.

```javascript
var garble = require("./garble");

// Index 2 holds the first actual command-line argument
var argument = process.argv[2];

console.log(garble(argument));
```

The file garble.js defines a library for garbling strings, which can be used both by the command-line tool defined earlier and by other scripts that need direct access to a garbling function.

```javascript
module.exports = function(string) {
  return string.split("").map(function(ch) {
    return String.fromCharCode(ch.charCodeAt(0) + 5);
  }).join("");
};
```

Remember that replacing module.exports, rather than adding properties to it, allows us to export a specific value from a module. In this case, we make the result of requiring our garble file the garbling function itself.

The function splits the string it is given into single characters by splitting on the empty string and then replaces each character with the character whose code is five points higher. Finally, it joins the result back into a string.

We can now call our tool like this:

```
$ node main.js JavaScript
Of{fXhwnuy
```

<!-- @task, "text" : "Write main.js and garble.js and try running them."-->

<!-- @section -->

# Installing with NPM

NPM, which was briefly discussed in Chapter 10, is an online repository of JavaScript modules, many of which are specifically written for Node. When you install Node on your computer, you also get a program called npm, which provides a convenient interface to this repository.

For example, one module you will find on NPM is figlet, which can convert text into ASCII art—drawings made out of text characters. The following transcript shows how to install and use it:

```javascript
$ npm install figlet
npm GET https://registry.npmjs.org/figlet
npm 200 https://registry.npmjs.org/figlet
npm GET https://registry.npmjs.org/figlet/-/figlet-1.0.9.tgz
npm 200 https://registry.npmjs.org/figlet/-/figlet-1.0.9.tgz
figlet@1.0.9 node_modules/figlet
$ node
> var figlet = require("figlet");
> figlet.text("Hello world!", function(error, data) {
    if (error)
      console.error(error);
    else
      console.log(data);
  });
  _   _      _ _                            _     _ _
 | | | | ___| | | ___   __      _____  _ __| | __| | |
 | |_| |/ _ \ | |/ _ \  \ \ /\ / / _ \| '__| |/ _` | |
 |  _  |  __/ | | (_) |  \ V  V / (_) | |  | | (_| |_|
 |_| |_|\___|_|_|\___/    \_/\_/ \___/|_|  |_|\__,_(_)
```

After running npm install, NPM will have created a directory called node_modules. Inside that directory will be a figlet directory, which contains the library. When we run node and call require("figlet"), this library is loaded, and we can call its text method to draw some big letters.

Somewhat unexpectedly perhaps, instead of simply returning the string that makes up the big letters, figlet.text takes a callback function that it passes its result to. It also passes the callback another argument, error, which will hold an error object when something goes wrong or null when everything is all right.

This is a common pattern in Node code. Rendering something with figlet requires the library to read a file that contains the letter shapes. Reading that file from disk is an asynchronous operation in Node, so figlet.text can’t immediately return its result. Asynchronicity is infectious, in a way—every function that calls an asynchronous function must itself become asynchronous.

There is much more to NPM than npm install. It reads package.json files, which contain JSON-encoded information about a program or library, such as which other libraries it depends on. Doing npm install in a directory that contains such a file will automatically install all dependencies, as well as their dependencies. The npm tool is also used to publish libraries to NPM’s online repository of packages so that other people can find, download, and use them.

This book won’t delve further into the details of NPM usage. Refer to [npmjs.org](http://npmjs.org) for further documentation and for an easy way to search for libraries.


<!-- @task, "text" : "Install figlet."-->

<!-- @task, "text" : "Install another Node module you find at npmjs.org."-->

<!-- @section -->

# The file system module

One of the most commonly used built-in modules that comes with Node is the "fs" module, which stands for file system. This module provides functions for working with files and directories.

For example, there is a function called readFile, which reads a file and then calls a callback with the file’s contents.

```javascript
var fs = require("fs");
fs.readFile("file.txt", "utf8", function(error, text) {
  if (error)
    throw error;
  console.log("The file contained:", text);
});
```

The second argument to readFile indicates the character encoding used to decode the file into a string. There are several ways in which text can be encoded to binary data, but most modern systems use UTF-8 to encode text, so unless you have reasons to believe another encoding is used, passing "utf8" when reading a text file is a safe bet. If you do not pass an encoding, Node will assume you are interested in the binary data and will give you a Buffer object instead of a string. This is an array-like object that contains numbers representing the bytes in the files.

```javascript
var fs = require("fs");
fs.readFile("file.txt", function(error, buffer) {
  if (error)
    throw error;
  console.log("The file contained", buffer.length, "bytes.",
              "The first byte is:", buffer[0]);
});
```

<!-- @asset, "contentType" : "outlearn/prototype-feature", "text" : "{ \"multiple_choice\": \"If you do not provide an encoding to the readFile() method, the system will\", \"answers\" : [ \"Use `utf8` as the default.\", \"Throw an error.\", \"Not return a string.\" ], \"correct\" : 2 }"-->

<!-- @task, "text" : "Read in a file encoded in UTF-8 using fs."-->

<!-- @task, "text" : "Read in a binary file."-->

A similar function, writeFile, is used to write a file to disk.

```javascript
var fs = require("fs");
fs.writeFile("graffiti.txt", "Node was here", function(err) {
  if (err)
    console.log("Failed to write file:", err);
  else
    console.log("File written.");
});
```

Here, it was not necessary to specify the encoding since writeFile will assume that if it is given a string to write, rather than a Buffer object, it should write it out as text using its default character encoding, which is UTF-8.


<!-- @task, "hasDeliverable" : true, "text" : "Paste in your code that writes a file using fs.writeFile."-->

The "fs" module contains many other useful functions: readdir will return the files in a directory as an array of strings, stat will retrieve information about a file, rename will rename a file, unlink will remove one, and so on. See the documentation at nodejs.org for specifics.


<!-- @task, "text" : "Print all the files in a directory using fs.readdir."-->

Many of the functions in "fs" come in both synchronous and asynchronous variants. For example, there is a synchronous version of readFile called readFileSync.

```javascript
var fs = require("fs");
console.log(fs.readFileSync("file.txt", "utf8"));
```

Synchronous functions require less ceremony to use and can be useful in simple scripts, where the extra speed provided by asynchronous I/O is irrelevant. But note that while such a synchronous operation is being performed, your program will be stopped entirely. If it should be responding to the user or to other machines on the network, being stuck on synchronous I/O might produce annoying delays.

<!-- @section -->

# The HTTP module

Another central module is called "http". It provides functionality for running HTTP servers and making HTTP requests.

This is all it takes to start a simple HTTP server:

```javascript
var http = require("http");
var server = http.createServer(function(request, response) {
  response.writeHead(200, {"Content-Type": "text/html"});
  response.write("<h1>Hello!</h1><p>You asked for <code>" +
                 request.url + "</code></p>");
  response.end();
});
server.listen(8000);
```

If you run this script on your own machine, you can point your web browser at http://localhost:8000/hello to make a request to your server. It will respond with a small HTML page.

<!-- @task, "text" : "Start an HTTP server using the sample above."-->

The function passed as an argument to createServer is called every time a client tries to connect to the server. The request and response variables are objects representing the incoming and outgoing data. The first contains information about the request, such as its url property, which tells us to what URL the request was made.

To send something back, you call methods on the response object. The first, writeHead, will write out the response headers (see Chapter 17). You give it the status code (200 for “OK” in this case) and an object that contains header values. Here we tell the client that we will be sending back an HTML document.

Next, the actual response body (the document itself) is sent with response.write. You are allowed to call this method multiple times if you want to send the response piece by piece, possibly streaming data to the client as it becomes available. Finally, response.end signals the end of the response.

The call to server.listen causes the server to start waiting for connections on port 8000. This is the reason you have to connect to localhost:8000, rather than just localhost (which would use the default port, 80), to speak to this server.

To stop running a Node script like this, which doesn’t finish automatically because it is waiting for further events (in this case, network connections), press Ctrl-C.

A real web server usually does more than the one in the previous example—it looks at the request’s method (the method property) to see what action the client is trying to perform and at the request’s URL to find out which resource this action is being performed on. You’ll see a more advanced server later in this chapter.

To act as an HTTP client, we can use the request function in the "http" module.

```javascript
var http = require("http");
var request = http.request({
  hostname: "eloquentjavascript.net",
  path: "/20_node.html",
  method: "GET",
  headers: {Accept: "text/html"}
}, function(response) {
  console.log("Server responded with status code",
              response.statusCode);
});
request.end();
```

The first argument to request configures the request, telling Node what server to talk to, what path to request from that server, which method to use, and so on. The second argument is the function that should be called when a response comes in. It is given an object that allows us to inspect the response, for example to find out its status code.

Just like the response object we saw in the server, the object returned by request allows us to stream data into the request with the write method and finish the request with the end method. The example does not use write because GET requests should not contain data in their request body.

To make requests to secure HTTP (HTTPS) URLs, Node provides a package called https, which contains its own request function, similar to http.request.

<!-- @section -->

# Streams

We have seen two examples of writable streams in the HTTP examples—namely, the response object that the server could write to and the request object that was returned from http.request.

Writable streams are a widely used concept in Node interfaces. All writable streams have a write method, which can be passed a string or a Buffer object. Their end method closes the stream and, if given an argument, will also write out a piece of data before it does so. Both of these methods can also be given a callback as an additional argument, which they will call when the writing to or closing of the stream has finished.

It is possible to create a writable stream that points at a file with the fs.createWriteStream function. Then you can use the write method on the resulting object to write the file one piece at a time, rather than in one shot as with fs.writeFile.

Readable streams are a little more involved. Both the request variable that was passed to the HTTP server’s callback function and the response variable passed to the HTTP client are readable streams. (A server reads requests and then writes responses, whereas a client first writes a request and then reads a response.) Reading from a stream is done using event handlers, rather than methods.

Objects that emit events in Node have a method called on that is similar to the addEventListener method in the browser. You give it an event name and then a function, and it will register that function to be called whenever the given event occurs.

Readable streams have "data" and "end" events. The first is fired every time some data comes in, and the second is called whenever the stream is at its end. This model is most suited for “streaming” data, which can be immediately processed, even when the whole document isn’t available yet. A file can be read as a readable stream by using the fs.createReadStream function.

The following code creates a server that reads request bodies and streams them back to the client as all-uppercase text:

```javascript
var http = require("http");
http.createServer(function(request, response) {
  response.writeHead(200, {"Content-Type": "text/plain"});
  request.on("data", function(chunk) {
    response.write(chunk.toString().toUpperCase());
  });
  request.on("end", function() {
    response.end();
  });
}).listen(8000);
```

The chunk variable passed to the data handler will be a binary Buffer, which we can convert to a string by calling toString on it, which will decode it using the default encoding (UTF-8).

The following piece of code, if run while the uppercasing server is running, will send a request to that server and write out the response it gets:

```javascript
var http = require("http");
var request = http.request({
  hostname: "localhost",
  port: 8000,
  method: "POST"
}, function(response) {
  response.on("data", function(chunk) {
    process.stdout.write(chunk.toString());
  });
});
request.end("Hello server");
```

The example writes to process.stdout (the process’ standard output, as a writable stream) instead of using console.log. We can’t use console.log because it adds an extra newline character after each piece of text that it writes, which isn’t appropriate here.

<!-- @task, "text" : "Run the example script on your machine."-->

<!-- @section -->

# A simple file server

Let’s combine our newfound knowledge about HTTP servers and talking to the file system and create a bridge between them: an HTTP server that allows remote access to a file system. Such a server has many uses. It allows web applications to store and share data or give a group of people shared access to a bunch of files.

When we treat files as HTTP resources, the HTTP methods GET, PUT, and DELETE can be used to read, write, and delete the files, respectively. We will interpret the path in the request as the path of the file that the request refers to.

We probably don’t want to share our whole file system, so we’ll interpret these paths as starting in the server’s working directory, which is the directory in which it was started. If I ran the server from /home/marijn/public/ (or C:\Users\marijn\public\ on Windows), then a request for /file.txt should refer to /home/marijn/public/file.txt (or C:\Users\marijn\public\file.txt).

We’ll build the program piece by piece, using an object called methods to store the functions that handle the various HTTP methods.

```javascript
var http = require("http"), fs = require("fs");

var methods = Object.create(null);

http.createServer(function(request, response) {
  function respond(code, body, type) {
    if (!type) type = "text/plain";
    response.writeHead(code, {"Content-Type": type});
    if (body && body.pipe)
      body.pipe(response);
    else
      response.end(body);
  }
  if (request.method in methods)
    methods[request.method](urlToPath(request.url),
                            respond, request);
  else
    respond(405, "Method " + request.method +
            " not allowed.");
}).listen(8000);
```

This starts a server that just returns 405 error responses, which is the code used to indicate that a given method isn’t handled by the server.

The respond function is passed to the functions that handle the various methods and acts as a callback to finish the request. It takes an HTTP status code, a body, and optionally a content type as arguments. If the value passed as the body is a readable stream, it will have a pipe method, which is used to forward a readable stream to a writable stream. If not, it is assumed to be either null (no body) or a string and is passed directly to the response’s end method.

To get a path from the URL in the request, the urlToPath function uses Node’s built-in "url" module to parse the URL. It takes its pathname, which will be something like /file.txt, decodes that to get rid of the %20-style escape codes, and prefixes a single dot to produce a path relative to the current directory.

```javascript
function urlToPath(url) {
  var path = require("url").parse(url).pathname;
  return "." + decodeURIComponent(path);
}
```

If you are worried about the security of the urlToPath function, you are right. We will return to that in the exercises.

We will set up the GET method to return a list of files when reading a directory and to return the file’s content when reading a regular file.

One tricky question is what kind of Content-Type header we should add when returning a file’s content. Since these files could be anything, our server can’t simply return the same type for all of them. But NPM can help with that. The mime package (content type indicators like text/plain are also called MIME types) knows the correct type for a huge number of file extensions.

If you run the following npm command in the directory where the server script lives, you’ll be able to use require("mime") to get access to the library:

```javascript
$ npm install mime
npm http GET https://registry.npmjs.org/mime
npm http 304 https://registry.npmjs.org/mime
mime@1.2.11 node_modules/mime
When a requested file does not exist, the correct HTTP error code to return is 404. We will use fs.stat, which looks up information on a file, to find out both whether the file exists and whether it is a directory.

methods.GET = function(path, respond) {
  fs.stat(path, function(error, stats) {
    if (error && error.code == "ENOENT")
      respond(404, "File not found");
    else if (error)
      respond(500, error.toString());
    else if (stats.isDirectory())
      fs.readdir(path, function(error, files) {
        if (error)
          respond(500, error.toString());
        else
          respond(200, files.join("\n"));
      });
    else
      respond(200, fs.createReadStream(path),
              require("mime").lookup(path));
  });
};
```

Because it has to touch the disk and thus might take a while, fs.stat is asynchronous. When the file does not exist, fs.stat will pass an error object with a code property of "ENOENT" to its callback. It would be nice if Node defined different subtypes of Error for different types of error, but it doesn’t. Instead, it just puts obscure, Unix-inspired codes in there.

We are going to report any errors we didn’t expect with status code 500, which indicates that the problem exists in the server, as opposed to codes starting with 4 (such as 404), which refer to bad requests. There are some situations in which this is not entirely accurate, but for a small example program like this, it will have to be good enough.

The stats object returned by fs.stat tells us a number of things about a file, such as its size (size property) and its modification date (mtime property). Here we are interested in the question of whether it is a directory or a regular file, which the isDirectory method tells us.

We use fs.readdir to read the list of files in a directory and, in yet another callback, return it to the user. For normal files, we create a readable stream with fs.createReadStream and pass it to respond, along with the content type that the "mime" module gives us for the file’s name.

The code to handle DELETE requests is slightly simpler.

```javascript
methods.DELETE = function(path, respond) {
  fs.stat(path, function(error, stats) {
    if (error && error.code == "ENOENT")
      respond(204);
    else if (error)
      respond(500, error.toString());
    else if (stats.isDirectory())
      fs.rmdir(path, respondErrorOrNothing(respond));
    else
      fs.unlink(path, respondErrorOrNothing(respond));
  });
};
```

You may be wondering why trying to delete a nonexistent file returns a 204 status, rather than an error. When the file that is being deleted is not there, you could say that the request’s objective is already fulfilled. The HTTP standard encourages people to make requests idempotent, which means that applying them multiple times does not produce a different result.

```javascript
function respondErrorOrNothing(respond) {
  return function(error) {
    if (error)
      respond(500, error.toString());
    else
      respond(204);
  };
}
```

When an HTTP response does not contain any data, the status code 204 (“no content”) can be used to indicate this. Since we need to provide callbacks that either report an error or return a 204 response in a few different situations, I wrote a respondErrorOrNothing function that creates such a callback.

This is the handler for PUT requests:

```javascript
methods.PUT = function(path, respond, request) {
  var outStream = fs.createWriteStream(path);
  outStream.on("error", function(error) {
    respond(500, error.toString());
  });
  outStream.on("finish", function() {
    respond(204);
  });
  request.pipe(outStream);
};
```

Here, we don’t need to check whether the file exists—if it does, we’ll just overwrite it. We again use pipe to move data from a readable stream to a writable one, in this case from the request to the file. If creating the stream fails, an "error" event is raised for it, which we report in our response. When the data is transferred successfully, pipe will close both streams, which will cause a "finish" event to fire on the writable stream. When that happens, we can report success to the client with a 204 response.

The full script for the server is available at eloquentjavascript.net/code/file_server.js. You can download that and run it with Node to start your own file server. And of course, you can modify and extend it to solve this chapter’s exercises or to experiment.

The command-line tool curl, widely available on Unix-like systems, can be used to make HTTP requests. The following session briefly tests our server. Note that -X is used to set the request’s method and -d is used to include a request body.

```shell
$ curl http://localhost:8000/file.txt
File not found
$ curl -X PUT -d hello http://localhost:8000/file.txt
$ curl http://localhost:8000/file.txt
hello
$ curl -X DELETE http://localhost:8000/file.txt
$ curl http://localhost:8000/file.txt
File not found
```

The first request for file.txt fails since the file does not exist yet. The PUT request creates the file, and behold, the next request successfully retrieves it. After deleting it with a DELETE request, the file is again missing.

<!-- @task, "text" : "Test the server with the sample curl commands."-->

<!-- @section -->

# Error handling

In the code for the file server, there are six places where we are explicitly routing exceptions that we don’t know how to handle into error responses. Because exceptions aren’t automatically propagated to callbacks but rather passed to them as arguments, they have to be handled explicitly every time. This completely defeats the advantage of exception handling, namely, the ability to centralize the handling of failure conditions.

What happens when something actually throws an exception in this system? Since we are not using any try blocks, the exception will propagate to the top of the call stack. In Node, that aborts the program and writes information about the exception (including a stack trace) to the program’s standard error stream.

This means that our server will crash whenever a problem is encountered in the server’s code itself, as opposed to asynchronous problems, which will be passed as arguments to the callbacks. If we wanted to handle all exceptions raised during the handling of a request, to make sure we send a response, we would have to add try/catch blocks to every callback.

This is not workable. Many Node programs are written to make as little use of exceptions as possible, with the assumption that if an exception is raised, it is not something the program can handle, and crashing is the right response.

Another approach is to use promises, which were introduced in Chapter 17. Those catch exceptions raised by callback functions and propagate them as failures. It is possible to load a promise library in Node and use that to manage your asynchronous control. Few Node libraries integrate promises, but it is often trivial to wrap them. The excellent "promise" module from NPM contains a function called denodeify, which takes an asynchronous function like fs.readFile and converts it to a promise-returning function.

```javascript
var Promise = require("promise");
var fs = require("fs");

var readFile = Promise.denodeify(fs.readFile);
readFile("file.txt", "utf8").then(function(content) {
  console.log("The file contained: " + content);
}, function(error) {
  console.log("Failed to read file: " + error);
});
```

For comparison, I’ve written another version of the file server based on promises, which you can find at eloquentjavascript.net/code/file_server_promises.js. It is slightly cleaner because functions can now return their results, rather than having to call callbacks, and the routing of exceptions is implicit, rather than explicit.

I’ll list a few lines from the promise-based file server to illustrate the difference in the style of programming.

The fsp object that is used by this code contains promise-style variants of a number of fs functions, wrapped by Promise.denodeify. The object returned from the method handler, with code and body properties, will become the final result of the chain of promises, and it will be used to determine what kind of response to send to the client.

```javascript
methods.GET = function(path) {
  return inspectPath(path).then(function(stats) {
    if (!stats) // Does not exist
      return {code: 404, body: "File not found"};
    else if (stats.isDirectory())
      return fsp.readdir(path).then(function(files) {
        return {code: 200, body: files.join("\n")};
      });
    else
      return {code: 200,
              type: require("mime").lookup(path),
              body: fs.createReadStream(path)};
  });
};

function inspectPath(path) {
  return fsp.stat(path).then(null, function(error) {
    if (error.code == "ENOENT") return null;
    else throw error;
  });
}
```

The inspectPath function is a simple wrapper around fs.stat, which handles the case where the file is not found. In that case, we replace the failure with a success that yields null. All other errors are allowed to propagate. When the promise that is returned from these handlers fails, the HTTP server responds with a 500 status code.

<!-- @section -->

# Summary

Node is a nice, straightforward system that lets us run JavaScript in a nonbrowser context. It was originally designed for network tasks to play the role of a node in a network. But it lends itself to all kinds of scripting tasks, and if writing JavaScript is something you enjoy, automating everyday tasks with Node works wonderfully.

NPM provides libraries for everything you can think of (and quite a few things you’d probably never think of), and it allows you to fetch and install those libraries by running a simple command. Node also comes with a number of built-in modules, including the "fs" module, for working with the file system, and the "http" module, for running HTTP servers and making HTTP requests.

All input and output in Node is done asynchronously, unless you explicitly use a synchronous variant of a function, such as fs.readFileSync. You provide callback functions, and Node will call them at the appropriate time, when the I/O you asked for has finished.

<!-- @section -->

# Exercises

## Content negotiation, again

In Chapter 17, the first exercise was to make several requests to eloquentjavascript.net/author, asking for different types of content by passing different Accept headers.

Do this again, using Node’s http.request function. Ask for at least the media types text/plain, text/html, and application/json. Remember that headers to a request can be given as an object, in the headers property of http.request’s first argument.

Write out the content of the responses to each request.

<!-- @task, "hasDeliverable" : true, "text" : "Complete the Content negotiation, again exercise and paste in your code."-->

## Fixing a leak

For easy remote access to some files, I might get into the habit of having the file server defined in this chapter running on my machine, in the /home/marijn/public directory. Then, one day, I find that someone has gained access to all the passwords I stored in my browser.

What happened?

If it isn’t clear to you yet, think back to the urlToPath function, defined like this:

```javascript
function urlToPath(url) {
  var path = require("url").parse(url).pathname;
  return "." + decodeURIComponent(path);
}
```

Now consider the fact that paths passed to the "fs" functions can be relative—they may contain "../" to go up a directory. What happens when a client sends requests to URLs like the ones shown here?

```
http://myhostname:8000/../.config/config/google-chrome/Default/Web%20Data
http://myhostname:8000/../.ssh/id_dsa
http://myhostname:8000/../../../etc/passwd
```

Change urlToPath to fix this problem. Take into account the fact that Node on Windows allows both forward slashes and backslashes to separate directories.

Also, meditate on the fact that as soon as you expose some half-baked system on the Internet, the bugs in that system might be used to do bad things to your machine.

<!-- @task, "hasDeliverable" : true, "text" : "Complete the Fixing a leak exercise and paste in your code."-->


## Creating directories

Though the DELETE method is wired up to delete directories (using fs.rmdir), the file server currently does not provide any way to create a directory.

Add support for a method MKCOL, which should create a directory by calling fs.mkdir. MKCOL is not one of the basic HTTP methods, but it does exist, for this same purpose, in the WebDAV standard, which specifies a set of extensions to HTTP, making it suitable for writing resources, not just reading them.

<!-- @task, "hasDeliverable" : true, "text" : "Complete the Creating directories exercise and paste in your code."-->



## A public space on the web

Since the file server serves up any kind of file and even includes the right Content-Type header, you can use it to serve a website. Since it allows everybody to delete and replace files, it would be an interesting kind of website: one that can be modified, vandalized, and destroyed by everybody who takes the time to create the right HTTP request. Still, it would be a website.

Write a basic HTML page that includes a simple JavaScript file. Put the files in a directory served by the file server and open them in your browser.

Next, as an advanced exercise or even a weekend project, combine all the knowledge you gained from this book to build a more user-friendly interface for modifying the website from inside the website.

Use an HTML form (Chapter 18) to edit the content of the files that make up the website, allowing the user to update them on the server by using HTTP requests as described in Chapter 17.

Start by making only a single file editable. Then make it so that the user can select which file to edit. Use the fact that our file server returns lists of files when reading a directory.

Don’t work directly in the code on the file server, since if you make a mistake you are likely to damage the files there. Instead, keep your work outside of the publicly accessible directory and copy it there when testing.

If your computer is directly connected to the Internet, without a firewall, router, or other interfering device in between, you might be able to invite a friend to use your website. To check, go to whatismyip.com, copy the IP address it gives you into the address bar of your browser, and add :8000 after it to select the right port. If that brings you to your site, it is online for everybody to see.


<!-- @task, "hasDeliverable" : true, "text" : "Complete the A public space on the web exercise and paste in your code."-->
