# Gunrock Web Server
This web server is a simple server used in ECS 150 for teaching about multi-threaded programming and operating systems. This version of the server can only handle one client at a time and simply serves static files. Also, it will close each connection after reading the request and responding, but generally is still HTTP 1.1 compliant.

This server was written by Sam King from UC Davis and is actively maintained by Sam as well. The `http_parse.c` file was written by [Ryan Dahl](https://github.com/ry) and is licensed under the BSD license by Ryan.

## Quickstart
To compile and run the server, open a terminal and execute the following commands:
```bash
$ git clone https://github.com/kingst/gunrock_web.git
$ cd gunrock_web
$ make
$ ./gunrock_web
```

To test it out, you can either open up a web browser on the same machine and give it this url `http://localhost:8080/hello_world.html` or if you want to use curl on the command line you can test it out like this:
```bash
$ # get a basic HTML file
$ curl http://localhost:8080/hello_world.html
$ # get a basic HTML file with more detailed information
$ curl -v http://localhost:8080/hello_world.hml
$ # head a basic HTML file
$ curl --head http://localhost:8080/hello_world.html
$ # test out a file that does not exist (404 status code)
$ curl -v http://localhost:8080/hello_world2.html
$ # test out a POST, which isn't supported currently (405 status code)
$ curl -v -X POST http://localhost:8080/hello_world.html
```

We also included a full website that you can use for testing, try pointing your browser to: `http://localhost:8080/bootstrap.html`

## Command line arguments
Your C++ program must be invoked exactly as follows:

```bash
$ ./gunrock_web [-d basedir] [-p port] [-t threads] [-b buffers]
```

The command line arguments to your web server are to be interpreted as
follows.

- **basedir**: this is the root directory from which the web server should
  operate. The server should try to ensure that file accesses do not access
  files above this directory in the file-system hierarchy. Default: current
  working directory (e.g., `.`).
- **port**: the port number that the web server should listen on; the basic web
  server already handles this argument. Default: 10000.
- **threads**: the number of worker threads that should be created within the web
  server. Must be a positive integer. Default: 1.
- **buffers**: the number of request connections that can be accepted at one
  time. Must be a positive integer. Note that it is not an error for more or
  less threads to be created than buffers. Default: 1.

For example, you could run your program as:
```
$ ./gunrock_web -d static -p 8003 -t 8 -b 16
```

In this case, your web server will listen to port 8003, create 8 worker threads for
handling HTTP requests, allocate 16 buffers for connections that are currently
in progress (or waiting), and use SFF scheduling for arriving requests.

## Key concepts
The main idea behind this server is to make adding handlers as easy as writing a function. The `FileService.cpp` is a simple service that will read a file from the `static` directory and serve it back to the client as HTML. If you want to write new handlers, you'd do it by adding the new service and inheriting from `HttpService`, adding your source file to the `Makefile` and registering your service with the main `gunrock.cpp` file as a new service.

To match services to requests, the main `gunrock.cpp` logic tries to find the first path prefix match that it can, and when it finds a match it forwards the request on to the service for handling.

From within the service, you set the body of the request, or if there is an error you set the appropriate status code in the response object.

## Key files
To make this server multithreaded, you're going to need to modify the main `gunrock.cpp` file. You'll need to modify this file so that client requests are handled by a pool of threads with some priority logic to handle high priority files first. See the project README for more details.

## Other files
- **FileService** - Main file service, where the application logic for reading files goes
- **HTTP** - Higher level HTTP object, interfaces with the `http_parser`
- **http_parser** - HTTP protocol parsing state machine and callback functionality
- **HTTPRequest** - The HTTP request object, this is filled in by the framework
- **HTTPResponse** - The HTTP response object, the data for the response is filled in by the service
- **HttpUtils** - Simple utility functions for working with HTTP data
- **MyServerSocket** - High level abstraction on top of server sockets, accepts connections from new clients
- **MySocket** - High level abstraction on top of sockets, used by the framework to read requests and write responses
- **gunrock** - The main function + basic request handling
