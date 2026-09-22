# Cloudfront

CloudFront reduces application load by caching static and dynamic content at edge locations, preventing repeated requests from reaching the origin. It further reduces traffic via persistent connections to the backend, DDoS protection through AWS Shield, and by restricting traffic access to ensure only valid, authorized requests reach the backend.

Key ways CloudFront prevents application load:

- Caching Content: Edge nodes serve cacheable content (images, JS, CSS) directly, which dramatically reduces the number of requests the Application Load Balancer (ALB) needs to process.

- Request Collapsing: If multiple users request the same expired object simultaneously, CloudFront sends only one request to the origin, using it to serve all users.

- Persistent Connections: Maintains a pool of connections between CloudFront and the origin, reducing the overhead of establishing new TCP connections for every request.

- DDoS Mitigation (AWS Shield): CloudFront provides built-in, no-cost protection from Layer 3/4 DDoS attacks, absorbing high-volume traffic at edge locations.

- Origin Shield: An additional caching layer that further consolidates requests before they reach the origin, enhancing the cache hit ratio.

- Filtering Unauthorized Traffic: By using AWS WAF (Web Application Firewall) and custom headers, CloudFront stops malicious traffic (such as bots or invalid requests) from reaching the application.

- Geoblocking: Blocks traffic from specific geographic regions at the edge, reducing unnecessary load. 

For maximum benefit, the ALB should be configured to accept traffic only from CloudFront to prevent users from bypassing the cache.


# The 7 OSI Layers (Top to Bottom) (APSTNDP):
Layer 7: Application Layer – Closest to the user, providing network services to applications like HTTP, FTP, and SMTP.

Layer 6: Presentation Layer – Translates, compresses, or encrypts data for the application layer, ensuring it is in a readable format.

Layer 5: Session Layer – Manages sessions, maintaining, opening, and closing connections between applications.

Layer 4: Transport Layer – Handles transmission of data, managing flow control, segmentation, and error control using TCP/UDP.

Layer 3: Network Layer – Routes data packets across networks using logical IP addressing and routers.

Layer 2: Data Link Layer – Manages node-to-node data transfer and switches, using MAC addresses to handle physical addressing.

Layer 1: Physical Layer – Handles physical transmission of raw data, transmitting bits over cables, hubs, or wireless.

# Pipeda, Soc2, ISO27001

| Framework | Type                   | Focus                          | Mandatory?                       |
| --------- | ---------------------- | ------------------------------ | -------------------------------- |
| PIPEDA    | Law                    | Personal data privacy (Canada) | Yes (if applicable)              |
| SOC 2     | Audit standard/framewrk| Data security & trust controls | Contractual/business req/ SAAS   |
| ISO 27001 | International standard | InfoSec management system      | Certification-based              |

How This Connects to Shared Responsibility

Even though:
- AWS is SOC 2 compliant
- GCP is ISO 27001 certified

👉 You are still responsible for:

- Configuring IAM correctly
- Not exposing S3 publicly
- Encrypting data
- Managing access properly

Cloud providers give you compliant infrastructure. You must configure it securely.