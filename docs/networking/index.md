# __:material-ip-network-outline:{.lg .top} Networking__


## Distinction between DNS redirection and proxing

### __:material-dns-outline:{.lg .top} DNS Redirection__

This is a technique where the Domain Name System (DNS) is used to redirect a user from one IP address to another. This is often used for load balancing, failover, or to direct users to geographically close servers to reduce latency. DNS redirection doesn't involve any intermediate server to handle the traffic between the client and the server. However, it doesn't provide any control over the actual data being transferred.


### __:simple-envoyproxy:{.lg .top} Proxying__

A proxy server acts as an intermediary for requests from clients seeking resources from other servers. It allows a client to make an indirect network connection to other network services. A client connects to the proxy server, requesting some service, such as a file, connection, web page, or other resources available from a different server and the proxy server evaluates the request as a way to simplify and control its complexity.

Key features of a proxy server include:

- Controlling and optimizing internet usage by caching web pages and files.
- Blocking access to certain web pages, acting as a web filter.
- Improving security by hiding the client's IP address from the web, and by encrypting connections.


### __:simple-awselasticloadbalancing:{.lg .top} Load Balancer__

A load balancer distributes network traffic across multiple servers to ensure that no single server bears too much demand. This improves the overall performance, availability, and reliability of applications. Load balancers can operate at both the transport level (Layer 4 - TCP/UDP) and at the application level (Layer 7 - HTTP).

Key features of a load balancer include:

- Distributing client requests or network load efficiently across multiple servers.
- Ensuring high availability and reliability by sending requests only to servers that are online.
- Providing the flexibility to add or subtract servers as demand dictates.