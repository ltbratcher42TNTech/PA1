# PA1 Design — ETP/1 Client and Server

## 1. Architecture

Our project separates the protocol from the transport from the client from the server via different python files. `etp.py` contains the message types, constants, and functions to both encode and decode. `transport.py` provides an interface to `transport_tcp.py` which handles tcp connections and data transferring. `client.py` manages the client sessions as well as the HELLO, LIST, GET, PUT, AND BYE. Finally, `server.py` handles incoming sessions, processes it, and returns a response.

Both the client and server use `etp.py` for protocol messages and `transport.py` for communication.

## 2. The transport seam

Our transport is in `transport_tcp.py` and accessed via `transport.py`. It provides six operations: listen, accept, connect, send_all, recv_exact, and close. No other file calls the socket API.

I'm not sure what we'll convert to with PA2, but the important thing is that regardless, as long as it uses the same six functions, the client, server, and etp should require very little changes to continue functioning.


## 3. Framing

ETP uses an 8 byte header that has the version, message type, status, reserved field, and payload len. Our read message function calls the recv_exact function to read the header, unpack it, and checks the version as well as the length before reading it.

Etp uses an 8 byte header that has the version, message type, status, reserved field, and payload len. Our read message function calls thee recv_exact function to read the header, unpack it, and checks the version as well as the length before reading it.

The recv_exact function continues reading until it receivs the EXACT number of bytes. This handles the issue of fragmentaion. It also prevents multiple messages being read as one, since each read only uses the required bytes. 

The version and length checks happen before reading the payload so invalid protocol versions and oversized messages can be rejected without reading or allocating the advertised payload.


## 4. Concurrency

Our server uses threads. It creates a new thread for every client that connects. If one client freezes halfway through a header, only that specific client is stuck. Other clients can continue communicating with a server.

The clients share access to the root directory, meaning two clients could theoretically GET or PUT the same file at the same time, since we don't use file-level locks. Each PUT creates its own temporary file using mkstemp, then uses os.replace to atomically replace the original. This prevents a GET from reading a partially uploaded file. If two clients PUT the same filename, the last replacement wins. There's also LOG_LOCK to prevent different threads from printing over one another.


## 5. Errors and disconnects

Fatal errors like oversized message or request before HELLO causes the server to send an error and end the entire session. Non-fatal errors, such as requesting a file that doesn't exist, meanwhile, return an error BUT allow the session to continue.

If a client disconnects during a GET, the server notices and closes the session without killing the other clients.

Our client has four exit codes: 0 for success, 1 for server error, 2 for other fails, and 3 when the digest from SHA doesn't match the file


## 6. Threat model

A bad actor could connect to the server and attempt to access files, send malformed messages, or try to overwhelm it by opening a bunch of connections. Our server prevents basc traversal by validating file names, which rejects names with path separators and/or invalid characters. Additionally, it rejects payload lengths larger than 16 MiB. HOWEVER, our server doesn't limit how many threads can be created, so we still have the risk of resource exhaustion if they open a ton of connections. Additionally, we also don't have an idle timeout, meaning someone could connect and leave their thread waiting indefinitely without finishing a request.


## 7. Testing

Beyond the provided self_check file, I utilized chatgpt to generate a mock server to temporarily test the HELLO, LIST, GET, and BYE requests (hadn't implemented PUT, and by the time I did, Tyler had a server ready). This allowed me to test basic comms and error handling before the actual server was built.

One bug we encountered was that the PUT command returned before sending BYE. The self-check revealed this when C03 reported that the connection closed before the BYE message. We fixed it by moving the PUT result handling to right after the BYE, which also corrected a different test. All six client self-checks passed after that.
