# Module 1 Assignment — Protocol Comparison Report

**Student Name:** Farzana Islam
**Student ID:**  101044158
**Date:**       25/05/26

\---

## 5.1 QoS Comparison Results Table

> Run `pytest tests/mqtt/test\\\\\\\\\\\\\\\_qos\\\\\\\\\\\\\\\_loss.py -v -s` and paste the output table here.



|Protocol / QoS|Sent|Received|Lost (%)|Duplicates|Avg Latency (ms)|
|-|-|-|-|-|-|
|MQTT QoS 0|100|100|0.0%|0|1.8|
|MQTT QoS 1|100|100|0.0%|0|1.5|
|MQTT QoS 2|100|100|0.0%|0|3.9|
|CoAP NON|N/A|N/A|N/A|N/A|N/A|
|CoAP CON|N/A|N/A|N/A|N/A|N/A|
|AMQP (confirms off)|Not implemented|Not implemented|Not implemented|Not implemented|Not implemented|

**Analysis Questions:**

1. **Why does QoS 0 lose messages while QoS 1 and 2 do not?** 

> QoS 0 uses a fire-and-forget delivery model and does not require acknowledgements from the receiver. If a packet is lost during transmission, the sender does not retransmit it. QoS 1 and QoS 2 use acknowledgement mechanisms that allow lost packets to be retransmitted, providing reliable delivery.

2. **QoS 1 may show duplicates. Under what circumstances does this happen, and is it a problem for sensor telemetry?** 

> QoS 1 guarantees at-least-once delivery. If the sender does not receive a PUBACK acknowledgement in time, it retransmits the message. The receiver may therefore receive the same message more than once. For sensor telemetry this is usually acceptable because duplicate readings can be filtered by timestamp or message ID.

3\. **QoS 2 has higher latency than QoS 1. What causes this, and when is the trade-off worth it?** 

> QoS 2 requires a four-step handshake (PUBLISH, PUBREC, PUBREL, PUBCOMP) while QoS 1 only requires a PUBACK acknowledgement. The additional control packets increase transmission time and processing overhead. QoS 2 is worth using when duplicate messages are unacceptable and exactly-once delivery is required.

5.2 CoAP–HTTP Proxy Mapping

> Run `pytest tests/coap/test\\\\\\\\\\\\\\\_proxy.py -v -s` and record the observed HTTP headers.

|HTTP Header|CoAP Option|Your Observed Value|
|-|-|-|
|Content-Type|Content-Format|application/json|
|Cache-Control: max-age|Max-Age|Default|
|ETag|ETag|Not Observed|
|Location|Location-Path|Not Observed|



## 5.3 Protocol Selection Recommendation



### Data Path Recommendations

|Data Path|Recommended Protocol|Justification|
|-|-|-|
|Sensor → Cloud (high frequency, <100 ms latency)|MQTT QoS 1|Reliable delivery with low latency and low overhead|
|Actuator commands (safety-critical, exactly-once)|MQTT QoS 2|Exactly-once delivery prevents duplicate command execution|
|Backend service-to-service routing|MQTT|Publish/subscribe model simplifies routing between services|
|OTA firmware delivery to constrained MCU (Class 2)|CoAP Block2|Efficient segmented transfer designed for constrained devices|

### Detailed Justification



MQTT is the most suitable protocol for high-frequency sensor-to-cloud communication because it provides a good balance between reliability, low latency and minimal protocol overhead. During testing, MQTT QoS 0, QoS 1 and QoS 2 all successfully delivered 100 messages, with measured average latencies of approximately 1.8 ms, 1.5 ms and 3.9 ms respectively. These results demonstrate that MQTT can easily satisfy the requirement of less than 100 ms latency for real-time IoT telemetry. Packet captures also showed that MQTT messages contain relatively small headers compared to traditional HTTP requests, reducing bandwidth consumption. In addition, the publish/subscribe architecture allows sensors to send data to a broker without needing to know which applications consume the data. This decoupling improves scalability because new analytics services can subscribe to existing topics without modifying sensor devices.



For actuator commands that require safety-critical operation and exactly-once execution, MQTT QoS 2 is the recommended choice. Packet analysis revealed that QoS 2 uses a four-step acknowledgement sequence consisting of PUBLISH, PUBREC, PUBREL and PUBCOMP messages. Although this introduces additional protocol overhead and increased latency compared to QoS 1, it guarantees exactly-once delivery. This prevents situations where a command such as opening a valve, activating a motor or shutting down equipment could be executed multiple times because of duplicate message delivery. The slightly higher latency observed during testing is acceptable because reliability and correctness are more important than speed for critical control operations.



Backend service-to-service routing is also well suited to MQTT. Modern IoT systems often include multiple consumers such as dashboards, analytics engines, alerting systems and databases. MQTT topics provide flexible routing and filtering capabilities that allow services to subscribe only to relevant information. During implementation, publishers and subscribers could be added independently without changing existing code. This loose coupling simplifies system maintenance and expansion. Packet captures further demonstrated that MQTT exchanges require relatively few protocol messages, making communication efficient while supporting many subscribers simultaneously.



For over-the-air (OTA) firmware delivery to constrained microcontrollers, CoAP with Block2 transfer is the preferred protocol. Unlike MQTT, which is optimized for telemetry streams, CoAP provides mechanisms specifically designed for resource-oriented communication. The implementation included Block2 transfers that divided larger payloads into multiple manageable blocks. Testing confirmed that firmware manifest data could be transferred successfully using segmented CoAP responses. Because CoAP operates over UDP rather than TCP, protocol overhead is reduced, making it suitable for constrained devices with limited memory, processing power and network bandwidth. The packet captures also showed that CoAP headers are extremely compact, often requiring only a few bytes before payload data.



Considering implementation effort, packet captures and measured performance, MQTT and CoAP serve complementary purposes within an IoT architecture. MQTT is the preferred solution for real-time telemetry, event distribution and service integration because of its low latency, scalability and reliable QoS options. CoAP is more appropriate for device management, firmware distribution and constrained-device interactions because of its lightweight UDP-based design and support for blockwise transfers. Together, these protocols provide an effective communication foundation for modern IoT systems while addressing different operational requirements and resource constraints.



## 5.4 Reflection



### Technical Challenge



One technical challenge encountered during the assignment was configuring the CoAP server correctly on Windows. Initial test execution produced asyncio event loop errors and socket binding conflicts because multiple processes attempted to use the same port. The problem was resolved by configuring the correct event loop policy and ensuring that only one server instance was running during testing. After applying these changes, all CoAP tests passed successfully.



### Most Surprising Protocol Difference



The most surprising difference was the amount of protocol overhead required by MQTT QoS 2 compared to QoS 1. Packet captures showed that QoS 2 requires a four-step handshake while QoS 1 only requires a PUBACK acknowledgement. Although QoS 2 provides stronger delivery guarantees, it also increases latency and protocol complexity.



### Most Complex Protocol to Implement



CoAP was the most complex protocol to implement correctly. The implementation required asynchronous programming, resource registration, handling of GET and PUT requests, blockwise transfers and observable resources. Debugging asynchronous event loop issues and ensuring compatibility with the testing framework required significantly more effort than implementing MQTT publishers and subscribers.



*Module 1 Assignment — Real-Time Data Analytics for IoT*



