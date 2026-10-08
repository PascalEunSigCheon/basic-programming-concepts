# What happens when you type a URL in a browser?

Let's say you typed `google.com` in your browser.

## 1. Browser parsing

The browser interprets it as a website address and thus it becomes `https://www.google.com/`.
Then, the browser asks "What server should I connect to?"

## 2. IP address

To connect to the server, the browser must know the IP address, but `google.com` is just a domain name. We do not yet have the IP address. So the browser looks it up on DNS (Domain name system). This may involve several layer of caching. Using DNS, the browser can translate the domain name to the IP address.

## 3. Network connection

Now the browser has the address, it needs to establish a network connection to the server.

## 4. TCP

The browser tries to establish a TCP connection with the server. This involves the famous three-way handshake.

```mermaid
sequenceDiagram
    participant Client
    participant Server
    Client->>Server: SYN
    Server-->>Client: SYN+ACK
    Client->>Server: ACK
```

In this diagram, the client initiates the TCP connection by sending a SYN packet to the server.
The server responds with a SYN+ACK packet, indicating its willingness to establish a connection.
Finally, the client sends an ACK packet to acknowledge the server's response, completing the three-way handshake process and establishing a TCP connection between the client and the server.

TCP connection ensures reliable, ordered-byte stream between two endpoints.
