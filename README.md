# 🛡️ SYN Flood Attack Incident Report

*A cybersecurity trainee's analysis of a SYN flood attack, using a Wireshark log.*

---

## 📋 Scenario Overview

I work as a security analyst for a travel agency that advertises sales and promotions on the company's website. Employees regularly visit the sales webpage to search for vacation packages their customers might like.

One afternoon, I received an automated alert from the monitoring system indicating a problem with the web server. When I tried to visit the company website, my browser returned a connection timeout error.

I used a packet sniffer to capture data packets going to and from the web server. I noticed a large number of TCP SYN requests coming from an unfamiliar IP address. The server appeared overwhelmed by the volume of incoming traffic and was losing its ability to respond, so I suspected a malicious actor was attacking it.

To respond, I took the server offline temporarily so it could recover and return to normal operation. I also configured the company firewall to block the IP address sending the abnormal number of SYN requests. This block will not last long, because an attacker can spoof other IP addresses to get around it. I need to alert my manager quickly to discuss next steps to stop this attacker and prevent the problem from happening again, and I must be prepared to explain the type of attack, how it affected the web server, and how it affected employees.

---

## 🔍 Log Analysis & Data Interpretation
<img width="497" height="166" alt="Color Coded TCP Log" src="https://github.com/user-attachments/assets/07879897-3317-4cc1-bdf5-dbd4576a7e78" />
<img width="1216" height="475" alt="Color Coded TCP Log 3" src="https://github.com/user-attachments/assets/7c70b568-ae4f-44c7-8978-248b195fe6f2" />
<img width="1127" height="762" alt="Color Coded TCP Log 2" src="https://github.com/user-attachments/assets/eee452ec-e21b-44d0-a158-4cdf44ff5aea" />
*Wireshark log: Provided by Google*

**Key Observations:**

- The capture covers packets 47 to 214, spanning about 48 seconds of traffic, all directed at the web server (192.0.2.1) on port 443.
- At the start of the log, traffic looks normal. For example, 198.51.100.23 completes the full three-way handshake (SYN, SYN-ACK, ACK) and then receives a "200 OK" response for `/sales.html`.
- Starting around packet 52, the IP address 203.0.113.0 begins sending SYN packets to the server, and it keeps doing so from the same source port (54770).
- In total, 203.0.113.0 sent 139 SYN packets in the capture, far more than any other visitor. Every other IP address sent a single SYN request.
- Several legitimate visitors (198.51.100.16, 198.51.100.7, 198.51.100.22, and 198.51.100.9) received RST, ACK packets from the server instead of a completed connection, which means their requests were rejected or reset.
- One visitor (198.51.100.5) received an "HTTP/1.1 504 Gateway Time-out" response, showing that the server could no longer answer requests in time.
- From roughly 17 seconds onward, almost all traffic in the log comes from 203.0.113.0, and no legitimate visitors appear to complete a connection.

---

## 🚨 Incident Report

### Summary of the problem found in the Wireshark log

The log shows that the web server stopped responding to legitimate visitors after it was flooded with TCP SYN requests from a single IP address, 203.0.113.0. At first, visitors were able to connect normally and load the sales page. As the capture progresses, the same IP address sends SYN packet after SYN packet, and the server begins sending reset responses and a gateway time-out error to other visitors. Because of this, employees and customers received a connection timeout error when they tried to reach the website. This pattern is consistent with a type of denial of service (DoS) attack called SYN flooding.

### Analysis of the data and cause of the incident

## Type of attack that may have caused this network interruption
One potential explanation for the website’s connection timeout error message is a DoS attack. The logs show that the web server stops responding after it is overloaded with SYN packet requests. This event could be a type of DoS attack called SYN flooding.

## How the attack is causing the website malfunction
When the website visitors try to establish a connection with the web server, a three-way handshake occurs using the TCP protocol. The handshake consists of three steps: 

1.	A SYN packet is sent from the source to the destination, requesting to connect.
2.	The destination replies to the source with a SYN-ACK packet to accept the connection request. The destination will reserve resources for the source to connect.
3.	A final ACK packet is sent from the source to the destination acknowledging the permission to connect. 

In the case of a SYN flood attack, a malicious actor will send a large number of SYN packets all at once, which overwhelms the server’s available resources to reserve for the connection. When this happens, there are no server resources left for legitimate TCP connection requests. 

The logs indicate that the web server has become overwhelmed and is unable to process the visitors’ SYN requests. The server is unable to open a new connection to new visitors who receive a connection timeout message.
