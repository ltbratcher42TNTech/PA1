# PA1 Design — ETP/1 Client and Server

_One to two pages. Short, specific answers beat long general ones. Every section
should be true of the code you submit. Delete these italic prompts when you write._

## 1. Architecture

_Your modules and what calls what. A small diagram (ASCII is fine) or a list:
which file holds the protocol (encode/decode), which holds the transport, which
holds the server's session logic, which holds the client._

Our project separates the protocol from the transport from the client from the server via different python files. Etp contains the message types, constants, and functions to both encode and decode. Transport provides an interface to transport_tcp which handles tcp connections and data transferring. Client manages the client sessions as well as the HELLO, LIST, GET, PUT, AND BYE. Finally, server handles incoming sessions, processes it, and returns a response. 

Both the client and server use etp for protocol messages and transport for communication.

## 2. The transport seam

_Which file is your transport? List the six operations as you implemented them.
Confirm that no other file calls the socket API. In two or three sentences: what
would you have to change, and what would stay the same, when you swap TCP for your
PA2 transport in the final project?_

Our transport is in transport_tcp and accessed via transport. It provides six operations: listne, accept, connect, send_all, recv_exact, and close. No other files calls socket.

I'm not sure what we'll convert to with PA2, but the important thing is that regardless, as long as it uses the same six functions, the client, server, and etp should require very little changes to continue functioning.


## 3. Framing

_How does your code read one ETP message from a byte stream? Where and how do you
handle a header that arrives in pieces, or two messages that arrive in one read?
Where do you check VERSION and LENGTH, and why before reading the payload?_

Etp uses an 8 byte header that has the version, message type, status, reserved field, and payload len. Our read message function calls thee recv_exact function to read the header, unpack it, and checks the version as well as the length before reading it.

The recv_exact function continues reading until it receivs the EXACT number of bytes. This handles the issue of fragmentaion. It also prevents multiple messages being read as one, since each read only uses the required bits. 

The version and length checks happen before reading the payload so invalid protocol versions and oversized messages can be rejected without reading or allocating the advertised payload.


## 4. Concurrency

_Threads, processes or an event loop? What happens to everyone else when one
client stalls half-way through a header? Is any state shared between sessions
(for example, two clients PUT and GET the same file at once), and how is it
protected?_

Our server uses threads. It creates a new thread for every client that connects. If one client freezes halfway through a header, only that specific client is stuck. Other clients can continue communicating with a server.

The clients share access to root, meaning two clients could theoretically GET or PUT the same file at the same time, since we dont lock that. But partial writes to a temp file before using os.replace to replace the original. This prevents the client from reading an incomplete file during a put. There's also LOG_LOCL to prevent different threads from printing over one another.


## 5. Errors and disconnects

_How do fatal and non-fatal errors differ in your code? What happens when a client
disconnects in the middle of a GET of a large file? (C: how did you deal with
SIGPIPE?) How does your client choose its exit code?_

Fatal errors like oversized message or request before HELLO causes the server to send an error and end the entire session. Non-fatal errors, such as requesting a file that doesn't exist, meanwhile, return an error BUT allow the session to continue. 

If a client disconnects during a GET, the server notices and closes the session without killing the other clients.

Our client has four exit codes: 0 for success, 1 for server error, 2 for other fails, and 3 when the digest from SHA doesn't match the file


## 6. Threat model

_One paragraph. Who could attack an ETP/1 server, and how? Name one attack your
server resists (and how), and one it does not. Chapter 5 §5.9 and the
specification's Security Considerations are good starting points._

A bad actor could connect to the server and attemp to access files, send malformed messages, or try to overwhelm it by opening a bunch of connections. Our server prevents basc traversal by validating file names, which rejects names with path separators and/or invalid characters. Additionally, it rejects payload lengths larger than 16 MiB. HOWEVER, our server doesn't limit how many threads can be created, so we still have the risk of resource exhaustion if they open a ton of connections


## 7. Testing

_What did you test beyond `etp_selfcheck.py`? How? Name one bug your testing
found, how you found it, and the fix._

Beyond the provided self_check file, I utilized chatgpt to generate a mock server to temporarily test the HELLO, LIST, GET, and BYE requests (hadn't implemented PUT, and by the time I did, Tyler had a server ready). This allowed me to test basic comms and error handling before the actual server was built.

One bug we encountered was that the PUT command returned before sending BYE. The self-check revealed this when C03 reported that the connection closed before the BYE message. We fixed it by moving the PUT result handling to right after the BYE, which also correct a different test. All six client self-checks passed after that.