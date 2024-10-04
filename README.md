# 🌐 Webserv 🌐
<br>
<br>
This project is all about creating your own HTTP server, a fundamental piece of the internet's backbone. <br>
By building it yourself, you’ll see how the Hypertext Transfer Protocol (HTTP) drives the web, <br>
allowing browsers to communicate with servers to fetch and display web pages. <br> <br>

This project is made in collabaration with 🫖 [Adri Rommers](https://github.com/arommers) and [Sven van Hasselt](https://github.com/svenvanhasselt) 🫖
<br> <br>

[Video](https://github.com/user-attachments/assets/0704e46d-c065-41a0-9bfe-815065c3c738)


## 🐉Table of Contents

- [About](#About)
- [Server](#Server)
- [Request & Response](#Request-&-Response)
- [Common Gateway Interface (CGI)](#CGI)
- [Configuration File](#Configuration-File)
- [Configuration Parser](#Configuration-Parser)
- [Installation](#Installation)
- [Resources](#Resources)

## 🦕About
The goal of the project is to build a C++ compatible HTTP web server from scratch. <br>
The web server can handle HTTP `GET`, `POST`, and `DELETE` Requests, <br>
and can serve static files from a specified root directory or dynamic content using CGI. <br> <br>

### 🗿 HTTP (Hypertext Transfer Protocol)
HTTP *(Hypertext Transfer Protocol)* is a protocol for sending and receiving information over the internet. <br>
It is the foundation of the World Wide Web *(www)* and is used by web browsers and web servers to communicate with each other. <br>
When you click a link or submit a form, your browser sends an HTTP request, <br>
and the server responds with the content you need—or an error message if something goes wrong. <br> <br>

### 🗿 HTTP consists of requests and responses:
When a client *(such as a web browser)* wants to retrieve a webpage from a server, <br>
it sends an HTTP request to the server. <br>
The server then processes the request and sends back an HTTP response. <br>

```script

                                           HTTP Message can be either 
                                             a request or response.
                                                /             \
                                               /               \
                                              /                 \
----------------------------------------------                   -----------------------------------------------

	HTTP request:								HTTP response:
	consists of a 								consists of a
	- request line 🟥,					        	- status line 🔴,
	- headers 🟨,								- headers 🟡,
	- optional message body 🟩.						- optional message body 🟢.
Example:								Example:
GET /index.html HTTP/1.1						HTTP/1.1 200 OK
Host: localhost:8080							Content-Type: text/html
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64)			Content-Length: 1234
									<Message Body>
----------------------------------------------------------------------------------------------------------------

```
### 🗿 HTTP Request States:

Either the client or the server may exchange messages using any HTTP request method. <br>
The choice of method affects the state between the client and server. <br>
`GET` **(read-only request)** requests do not alter the server’s state, while `POST` and `DELETE` can. <br>

- `GET` &nbsp;&nbsp; : &nbsp;&nbsp; The path to resource *(e.g. /index.html)*. <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; In case of success *(or non-error)*, GET returns a representation of the resource <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; in response body and HTTP response status code of **200 (OK)**. <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; In an error case, it most often returns a **404 (NOT FOUND)** or **400 (BAD REQUEST)**. <br>

- `POST` &nbsp;&nbsp; : &nbsp;&nbsp; POST method is most often utilized to create new resources. <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; On successful creation, HTTP response code **201 (Created)** is returned. <br>

- `DELETE` &nbsp;&nbsp; : &nbsp;&nbsp; It deletes a resource specified in URI. <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; On successful deletion, it returns HTTP response status code **204 (No Content)**. <br> <br>


## 🦖Server

### 🗿 Server Core: 
Responsible for the low-level networking, such as creating and managing sockets, <br>
handling input and output streams, and managing the flow of data between the server and clients. <br> <br>

### 🗿 Server Setup with Socket Programming:
A server's main function is to listen for incoming client connections, <br>
process their requests, and send back appropriate responses. <br>
This is done through a combination of socket creation and communication mechanisms. <br> <br>


<details>
  <summary><strong>Step 1: Create a Socket</strong></summary>

### 🧭 Step 1: Create a Socket
The first step in creating a server is to establish a socket. <br>
A socket is like a door through which data flows between a server and a client. <br> <br>

In C++ *(and C)*, you can create a socket using the `socket()` function:

```cpp
int server_fd = socket(AF_INET, SOCK_STREAM, 0);
```
- `AF_INET` means the server will use IPv4 addresses.
- `SOCK_STREAM` indicates that we're using TCP *(which is connection-oriented and reliable)*.
- `0` specifies the default protocol *(TCP in this case)*. <br>
If `socket()` returns a value less than 0, an error occurred.

<br>
<br>

</details>

<details>
  <summary><strong>Step 2: Bind the Socket</strong></summary>

### 🧭 Step 2: Bind the Socket
After creating the socket, you need to assign an address and port to it. <br>
This is called binding. The server will listen for connections on this address and port. <br>

```cpp
struct sockaddr_in address;
address.sin_family = AF_INET;
address.sin_addr.s_addr = INADDR_ANY;  // Binds to all available interfaces
address.sin_port = htons(8080);        // Convert port number to network byte order

bind(server_fd, (struct sockaddr*)&address, sizeof(address));
```
- `INADDR_ANY` allows the server to accept connections on any of the machine's network interfaces.
- `htons()` converts the port number to network byte order.

<br>
<br>

</details>

<details>
  <summary><strong>Step 3: Listen for Connections</strong></summary>

### 🧭 Step 3: Listen for Connections
Once the socket is bound to an address, the server needs to listen for incoming client connections. <br>

```cpp
listen(server_fd, 3);
```
server_fd is the file descriptor of the socket. <br>
The second argument specifies the number of pending connections <br>
that can be queued up before the server starts rejecting them. <br>

<br>
<br>

</details>

---

<details>
  <summary><strong>Step 4: Accept Incoming Connection</strong></summary>

### 🧭 Step 4: Accept Incoming Connections
When a client attempts to connect, the server needs to accept the connection using `accept()`. <br>
This function returns a new socket descriptor that the server uses to communicate with the client. <br> <br>

```cpp
int new_socket = accept(server_fd, (struct sockaddr*)&address, (socklen_t*)&addrlen);
```
`accept()` extracts the first connection request from the queue and creates a new socket for communication with that client. <br>

<br>
<br>

</details>

<details>
  <summary><strong>Step 5: Communicate (Send/Receive Data)</strong></summary>

### 🧭 Step 5: Communicate (Send/Receive Data)
Once the server accepts a connection, it can send and receive messages with the client. <br>
This is done using `read()` and `write()` *(or recv() and send())*. <br> <br>

**Receiving data from the client:** <br>
```cpp
char buffer[1024] = {0};
read(new_socket, buffer, 1024);  // Reads data sent by the client
```
<br>

**Sending data back to the client:** <br>
```cpp
const char* hello = "HTTP/1.1 200 OK\nContent-Type: text/plain\nContent-Length: 12\n\nHello world";
send(new_socket, hello, strlen(hello), 0);  // Sends a simple HTTP response
```

<br>
<br>

</details>

<details>
  <summary><strong>Step 6: Close the Socket</strong></summary>

### 🧭 Step 6: Close the Socket
After communicating with the client, it's important to close the socket to free up system resources. <br>

```cpp
close(new_socket);  // Close the socket for this client
```
The server may continue to listen for new connections by looping over the `accept()` and communication steps.

<br>
<br>

</details>

### 🗿 Basic Server Flow:
- create a socket with `socket()` <br>
- identify the socket with `bind()` <br>
- wait for a connection with `listen()` and `accept()` <br>
- send and receive messages with `read()` and `write()` *(or send() and recv())* <br>
- close the socket with `close()` <br> <br>


## 🐢Request & Response

### 🗿 Request:
The process that is responsible for interpreting and extracting information from HTTP requests. <br>
Receives an incoming request, parses it, and extracts the relevant information <br>
such as the method, path, headers, and message body *(if present)*. <br>
If any syntax error was found in the request during parsing, error flags are set and parsing stops.<br> <br>
```script

				         	         HTTP request:
		    ---------------------|-------------------------------------------|---------------------

	
🟥 Request line:			    🟨 Headers:			        	🟩 Optional message body:		
consists of				    contain additional 				- GET method usually doesn't
- the method, 				    information about the request		  include any body, most servers
- the path, 				    - the hostname of the server, 		  respond with information
- the HTTP version.			    - the type of browser being used.		  from the URI/URL requested.
------------------------------------------|-------------------------------------------|--------------------------------------	
`The method` specifies the action
that the client wants to perform,
such as GET (to retrieve a resource) 
or POST (to submit data to the server)
------------------------------------------|-------------------------------------------|--------------------------------------
`The path` or URI specifies the location 
of the resource on the server.
------------------------------------------|-------------------------------------------|--------------------------------------
`The HTTP version` indicates the version
of the HTTP protocol being used.
------------------------------------------|-------------------------------------------|--------------------------------------

```
<br>

### 🗿 Response:
The response is responsible for constructing and formatting the HTTP responses <br>
that are sent back to clients in response to their requests. <br>
Building and storing the HTTP response, including the status line, headers, and message body. <br>
The response builder may also perform tasks such as setting the appropriate status code and <br>
reason phrase based on the result of the request, <br>
adding headers to the response to provide additional information about the content or the server, <br>
and formatting the message body according to the content type and encoding of the response. <br> <br>
```script

							 HTTP response:
	        --------------------------|---------------------------------------------|--------------------------


🔴 Status line:				     🟡 Headers:			         🟢 Optional message body:
consists of				     contain additional 			 - The message body contains the actual	
- the HTTP version, 			     information about the response		   content of the response,
- the status code, 			     - the type and size			   such as the HTML code for a webpage.
					       of the content being returned.
-------------------------------------------|---------------------------------------------|------------------------------------------				 	
The status code indicates 
the result of the request,
such as 200 OK (successful) 
or 404 Not Found (resource not found). 
The reason phrase is 
a short description of the status code.
   ----------------------------
Responses are grouped in five classes:
(https://developer.mozilla.org/en-US/docs/Web/HTTP/Status)
Informational responses (100 – 199)
Successful responses (200 – 299)
Redirection messages (300 – 399)
Client error responses (400 – 499)
Server error responses (500 – 599)
-------------------------------------------|---------------------------------------------|------------------------------------------

```
<br>

## 🦜CGI

CGI *(Common Gateway Interface)* is a standard for running external programs from a web server.<br> 
When a user requests a web page that should be handled by a CGI program, <br>
the web server executes the program and returns the output to the user's web browser.<br>
CGI programs are simply scripts that can be written in any programming language, <br>
such as Perl, Python, or bash, <br>
and are typically used to process data submitted by a user through a web browser, <br>
or to generate dynamic content on a web page.<br> <br>


<details>
  <summary><strong>How CGI Works:</strong></summary>

### 🧭 How CGI Works:
When a web browser requests a web page that involves a CGI script: <br> <br>

**1. User Requests a Web Page:** <br>
- The user requests a web page that requires the execution of a CGI program *(e.g., a form submission or dynamic content)*. <br>

**2. Server Identifies CGI Request:** <br>
- The web server identifies that the request is associated with a CGI script, <br>
based on the URL pattern, typically the `/cgi-bin/` directory or a file with an executable permission. <br> 

**3. Server Executes the CGI Script:** <br>
- The server runs the CGI program, which is typically a script written in Python, Perl, bash, etc. <br>

**4. Script Processes Input Data:** <br>
- The CGI program processes any user input data *(from forms or query parameters)* or generates content dynamically. <br>

**5. Script Outputs Content:** <br>
- The CGI program outputs the generated HTML or other content. <br>
This output is sent back to the web server with a header *(e.g., Content-Type: text/html)*. <br>

**6. Server Sends the Output to the Browser:** <br>
- The web server captures the output from the CGI script and forwards it as the HTTP response to the user's web browser. <br>

<br>
<br>

</details>

<details>
  <summary><strong>How Web Server Handles CGI Requests:</strong></summary>

### 🧭 How Web Server Handles CGI Requests:
Web servers like Apache or Nginx must be configured to recognize and handle CGI scripts. <br>
Typically, the server is configured to treat files in a specific directory *(e.g., /cgi-bin/)* <br>
as executable scripts rather than static files. <br> <br>

**1. Sets Up Environment Variables:** <br>
- The server prepares several environment variables like `QUERY_STRING`, `REQUEST_METHOD`, `CONTENT_LENGTH`, etc., <br>
which provide information to the CGI script about the request. <br>

**2. Executes the Script:** <br>
- The server runs the CGI script, providing input data *(if any)* via stdin and setting the environment variables. <br>

**3. Captures the Script Output:** <br>
- The server captures the output from the script, ensuring that it adheres to the correct HTTP response format. <br>

<br>
<br>

</details>
<br> <br>

## 🪲Configuration File

Configuration file is a text file that contains various settings and directives that dictate how the web server should operate.  <br>
Could use **nginx** as example for testing. <br> <br>

#### 🗿 Nginx (engine x):
NGINX is open source software for web serving, reverse proxying, caching, load balancing, media streaming, and more. <br>
It started out as a web server designed for maximum performance and stability. <br>
In addition to its HTTP server capabilities, NGINX can also function as a proxy server for email *(IMAP, POP3, and SMTP)* <br>
and a reverse proxy and load balancer for HTTP, TCP, and UDP servers.<br> <br>


<details>
  <summary><strong>Default server block:</strong></summary>

## Default server block:
```bash

# --- All variables ---
server {
	server_name		=	W3bMasters			# Name of server
	port 			=	8080				# Listen on port for incoming connections
	host			=	127.0.0.1			# Server block will respond to requests for localhost (or (127.0.0.1))
	root			=	/www/html			# Sets the root directory for this server block
	max_client_size =	5000000					# File upload limit is 5MB -> 5000000
	index			=	/index.html		 	# File to serve when a server running (Main web page)
	error_page 400 	=	/config/error_page/400.html		# Files to serve when a error occurs
	error_page 403 	=	/config/error_page/403.html 
	error_page 404 	=	/config/error_page/404.html
	error_page 405 	=	/config/error_page/405.html
	error_page 406 	=	/config/error_page/406.html
	error_page 409 	=	/config/error_page/409.html
	error_page 410 	=	/config/error_page/410.html
	error_page 500 	=	/config/error_page/500.html
	error_page 505 	=	/config/error_page/505.html

	# Handles requests to the root URL '/cgi-bin' -> first location block context 
	location /cgi-bin {
		allowed_methods = POST GET DELETE			# Allowed_methods for that location Block
		root			= /www
	}
	
	# Handles requests to the root URL '/' -> second location block context
	location / {
		index 	= /index.html					# Specifies the default file to serve in this location
		return	= 301 <URL>					# For redirecting this to a specific page <URL> can be any http-page (https://en.wikipedia.org/wiki/42_(number))
	}

	# Handles requests to the root URL '/img' -> third location block context
	location /img {
		root		= /www/html				# Local pathing to what directory should be served in this location Block
		autoindex	= on 					# Turns off/on directory listing. -> showes/listing of directories
	}
}
```
<br>
<br>

</details>

---

<details>
  <summary><strong>Test Server blocks:</strong></summary>

## Test Server blocks:

### Test 1 : Location Block with a valid autoindex (list of directories)
```bash
server {
	server_name		=	TestServer
	port 			=	8080
	host			=	localhost
	root			=	/www/html
	index			=	/index.html

	location /img {
		root		= /www/html
		autoindex	= on 
	}
}
```
⭕ ***=> How to test:*** Web browser URL ->  localhost:8080/img/
<br> <br>

### Test 2 : Location Block with a invalid root (output '404 error')
```bash
server {
	server_name		=	TestServer
	port 			=	8080
	host			=	localhost
	root			=	/www/html

	location /cgi-bin {
		root 			= /test
	}
}
```
⭕ ***=> How to test:*** Web browser URL -> localhost:8080
<br> <br>

### Test 3 : different index & No Location block (should display index1.html)
```bash
server {
	server_name		=	TestServer
	port 			=	7070
	host			=	localhost
	index			=	/index1.html
	root			=	/www/html1
}
```
⭕ ***=> How to test:*** Web browser URL -> localhost:7070
<br> <br>

### Test 4 : Test Redirects
```bash
server {
	server_name		=	TestServer
	port 			=	6060
	host			=	localhost
	root			=	/www/html1
	index			=	/index1.html

	location /img {
		root		= /www/html
		return		= 301 https://en.wikipedia.org/wiki/42_(number)
	}
}
```
⭕ ***=> How to test:*** Web browser URL -> localhost:6060 <br>
*(should go to the redirect URL, in this case to 'https://en.wikipedia.org/wiki/42_(number)')*
<br> <br>

### Test 5 : Test server_name 
```bash
server {
	server_name		=	TestServer
	port 			=	8080
	host			=	localhost
	root			=	/www/html
	index			=	/index.html
}
```
⭕ ***=> How to test:*** open second terminal *(same folder as currecnt one)* -> `$ curl --resolve TestServer:8080:127.0.0.1 http://TestServer:8080/index.html -v`

<br>

**Output example:** <br>
```bash
1		* Added TestServer:8080:127.0.0.1 to DNS cache
2		* Hostname TestServer was found in DNS cache
3		*   Trying 127.0.0.1:8080...
4		* Connected to exampletest (127.0.0.1) port 8080 (#0)
5		> GET /index.html HTTP/1.1
6		> Host: TestServer:8080
7		> User-Agent: curl/7.81.0
8		> Accept: */*
9		> 
10		* Mark bundle as not supporting multiuse
11		< HTTP/1.1 200 Found
12		< Content-Type: text/html
13		< Content-Length: 1139
14		< 
15		<!DOCTYPE html>
16		<head>
17		    <meta charset="UTF-8">
18		    <meta name="viewport" content="width=device-width, initial-scale=1.0">
19		    <title>404 Not Found</title>
20		    <style>
21		        body {
22		            font-family: Arial, sans-serif;
23		            display: flex;
24		            justify-content: center;
25		            align-items: center;
26		            height: 100vh;
27		            margin: 0;
28		            background-color: #fff;
29		        }
30		        .container {
31		            text-align: center;
32		            padding: 20px;
33		            background-color: #fff;
34		        }
35		        .error-code {
36		            font-size: 9em;
37		            font-weight: bold;
38		            margin-bottom: -0.1em;
39		            color: #333;
40		        }
41		        h1 {
42		            font-size: 2em;
43		            margin-bottom: 0.1em;
44		            color: #333;
45		        }
46		        p {
47		            font-size: 1em;
48		            margin-bottom: 1em;
49		            color: #666;
50		        }
51		    </style>
52		</head>
53		<body>
54		    <div class="container">
55		        <div class="error-code">404</div>
56		        <h1>Not Found</h1>
57		        <p>The requested resource could not be found on the server!</p>
58		    </div>
59		</body>
60		* Connection #0 to host exampletest left intact
61		</html>% 
```
**Line 11 &nbsp;&nbsp; : &nbsp;&nbsp;** `< HTTP/1.1 200 Found` <br>
- displayes the responds code, so also coud be `< HTTP/1.1 404 Not Found` <br>

**Line 15+ &nbsp;&nbsp; : &nbsp;&nbsp;** `<!DOCTYPE html>` <br>
- Onward is the index.html file that the browser is displaying, so might look different <br>
- `< HTTP/1.1 404 Not Found` will display the 404_ErrorPage.html file <br> <br>

<br>
<br>

</details>
<br> <br>

## 🐸Configuration Parser

- [Server Name;](#Server-Name)
- [Host;](#Host)
- [Ports;](#Ports)
- [Root;](#Root)
- [Limit Client Body;](#Limit-Client-Body)
- [Index;](#Index)
- [Location Blocks;](#Location-Blocks)

<br> <br>

### 🗿Server Name
You need the server name for testing: <br>
```bash
$ curl --resolve TestServer:8080:127.0.0.1 http://TestServer:8080/index.html -v
```
<br>

The **server name** and **host** is not the same thing. <br> <br>

---

### 🗿Host
If the Host is `localhost` or `127.0.0.1`, both need to return 127.0.0.1, <br>
because for seting up your sever you need numbers, so convert `localhost` into ***127.0.0.1***. <br> <br>

---

### 🗿Ports
Ports, in the context of computer networking, are identified by port numbers, which range from 0 to 65535. <br>
This range means that port numbers can be up to five digits long. <br>
A breakdown of the port number ranges:<br>

- `Well-Known Ports (0 to 1023):`  <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; These ports are reserved for system or well-known services and protocols *(e.g., HTTP uses port 80, FTP uses port 21)*.
- `Registered Ports (1024 to 49151):` <br> 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; These ports are assigned by the Internet Assigned Numbers Authority *(IANA)* for user processes or applications.
- `Dynamic or Private Ports (49152 to 65535):`  <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; These ports are used for private or temporary purposes, often assigned dynamically to client applications when they connect to a network service. <br> <br>

---

### 🗿Root
Check if the Root given exist: <br>

```C
// If the path type is a directory -> path needs to be a valid directory
if (Config::getPathType(newRoot) == FOLDER)
	return newRoot;
else
{
	// Concatenates the current working directory (dir) with the given NewRoot path, resulting in a full path
	char dir[1024];
	getcwd(dir, 1024);
	NewFullRoot = dir + newRoot;
	if (Config::getPathType(NewFullRoot) != FOLDER) // If the path type is not a directory
		throw Exception_Config("Invalid Root");
}
return NewFullRoot;
```
<br>

---

### 🗿Limit Client Body
You need a limit client body for later in the code. <br>
If the user uploads something, lets say a `PNG`. <br>
If the `PNG` is bigger than the limit client body, it should throw an error. <br> 
You could set yor limit to **3MB** or **5MB**. <br> <br>

Decimal **(Base-10)**: <br>
- **1 Megabyte (MB)** = 1,000,000 bytes. <br>

***So:*** <br>
- **3 MB** = 3×1,000,000 = 3,000,000 bytes <br>
***Therefore***, 3 MB = 3,000,000 bytes in the decimal definition. <br> <br>

---

### 🗿Index
The `index.html` file is typically the **default landing page** of a website. <br>
When a user visits a website in a browser, the web server automatically looks for the index.html file and serves it. <br>
It serves as the homepage or the entry point for the website. <br> 
The index page provides an overview of the website and directs users to other sections. <br> <br>

---

### 🗿Location Blocks
A location block in a web server configuration defines how to handle specific URLs. <br>
It sets rules for things like serving static files, setting up redirects,<br>
enabling directory browsing, or rewriting URLs. <br>

**For example:** <br>
```nginx
location /img {
	root		= /www/html
	autoindex	= on
}
```

<img src="https://github.com/user-attachments/assets/c43d1d6d-c2d7-4486-91b5-1cc61e7f4a12" alt="Images_examples" width="500">

<br> <br>

<details>
  <summary><strong>How Location Block Works</strong></summary>
#### 🧭How Location Block Works:

**1. Pattern Matching:** <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; - The location block is responsible for matching URL patterns and defining how the server responds to those patterns. <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; - For example: <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ○ The first `location /` block handles requests to the root of the site **(/)**. <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ○ The second `location /images/` block handles requests that start with **/images/** <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; and serves them from a specific directory. <br>

**2. Serving Files:** <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; - Inside the location block, you can define how files are served, where they are located, <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; or what should happen when a request is made to that path. <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; - For example: <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; *Here, requests to `/` are served from `/var/www/html` and the default file to be served is `index.html.`*
```nginx
location / {
    root /var/www/html;
    index index.html;
}
```

<br>
<br>

</details>

<details>
  <summary><strong>Common Directives Used in Location Blocks</strong></summary>

#### 🧭 Common Directives Used in Location Blocks:

**1. Redirects (return):** <br>
*Redirects are used to send users from one URL to another.* <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; - **return Directive:** <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ○ The return directive sends a specific HTTP status code along with the URL to which the client should be redirected. <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; - For example: <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; *This creates a 301 (Permanent) Redirect, sending users from `/old-path/` to `/new-path/`.*
```nginx
location /old-path/ {
    return 301 /new-path/;		// or a URL <https://en.wikipedia.org/wiki/42_(number)>
}
```
**2. Autoindex (Directory Listing):**<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; - The autoindex directive enables or disables directory listing when the requested URL <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; corresponds to a directory rather than a file and no index file (like index.html) is found.<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; - When autoindex is enabled, the server will show a list of files and directories within the requested directory.<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; - For example:<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; *In this case, if you visit `http://example.com/files/` and no index file exists in `/var/www/data/`, a directory listing is displayed, allowing users to browse files.*<br>
```nginx
location /files/ {
    root /var/www/data/;
    autoindex on;
}
```
**3. Index:**<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; - The index directive defines the default file that is served when a request is made to a directory.<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; *In this case, if a user visits `http://example.com/`, the server will look for **index.html** or **index.htm** in `/var/www/html.`*<br>
```nginx
location / {
    root /var/www/html;
    index index.html index.htm;
}
```
<br>
<br>

</details>
<br> <br>

## 🦚Installation
1. Compile the program by running the following command:
```bash
$ make
```
2. Execute the program using the following command:
```bash
$ ./webserv
```
or
```bash
$ ./webserv <configuration file your choice>
```
3. The program is now running. Go to your web browser:

<img src="https://github.com/user-attachments/assets/80b7ff38-5fa4-43b3-986d-9523e5136810" alt="Image_browser" width="100">
 <br><br>


## 🦠Resources

- NGINX Resources
	- [Official NGINX Website](https://nginx.org/en/)
	- [NGINX Blog: HTTP/2 Theory and Practice](https://blog.nginx.org/blog/http2-theory-and-practice-in-nginx-stable-13)
	- [NGINX Configuration Examples](https://kbeezie.com/nginx-configuration-examples/)

- Node.js and HTTP Servers
	- [Node.js HTTP Module Documentation](https://nodejs.org/api/http.html)

- HTTP Protocol
	- [TechTarget: What is an RFC?](https://www.techtarget.com/whatis/definition/Request-for-Comments-RFC)
	- [Linode Guide: HTTP GET Request](https://www.linode.com/docs/guides/http-get-request/)
	- [MDN: HTTP Status Codes](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status)
