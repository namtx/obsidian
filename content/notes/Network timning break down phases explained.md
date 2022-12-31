---
title: "Network timning break down phases explained"
tags:
  - web
  - http
  - networking
---

![[notes/images/Pasted image 20221231172352.png]]
Here's more information about each of the phases you may see in the Timing tab:
- **Queuing**: The browser queues requests when:
	- There a higher priority requests
	- There are already six TCP connections open for this origin, which is the limit. Applies to HTTP/1.0 and HTTP/1.1 only.
	- The browser is briefly allocating space in the disk cache

- **Stalled**: The request could be stalled for any of the reasons described in the **Queueing**.

- **DNS Lookup**: The browser is resolving the request's IP address.

- **Initial Connection**: The browser is establishing a connection, including TPC handshakes/retries and negotiating an SSL.

- **Proxy negotiation**: The browser is negotiating the request with proxy server.

- **Request sent**:  The request is being sent

- **ServiceWorker Preparation**: The browser is starting up t he service worker.

- **Request to ServiceWorker**: The request is being sent to the service worker.

- **Waiting (TTFB)**: The browser is waiting for the first byte of a response. TTFB stands for Time for First Byte. This timing includes 1 round trip of latency and the time the server took to prepare the response.

- **Content Download**: The browser is receiving the response, either directly from the network or from a service worker. This value is the total amount of time spent reading the response body. Larger than expected values could indicate a slow network, or that the browser is busy performing other work which delays the response from being read.

- **Receiving Push**: The browser is receiving data for this response via HTTP/2 Server Push.

- **Reading Push**: The browser is reading the local data previously received.