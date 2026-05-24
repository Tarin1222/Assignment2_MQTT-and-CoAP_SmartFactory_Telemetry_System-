# Module 1 Assignment — Packet Analysis

## Task 4: Wire-Level Protocol Annotation



Name: Farzana Islam
ID: 101044158



## 4.2 MQTT Packet Annotations

### CONNECT Packet

|Field|Offset (bytes)|Raw Hex|Decoded Value|
|-|-|-|-|
|Frame type + flags (byte 1)|0|10|Type=CONNECT (1), flags=0x10|
|Remaining length (byte 2)|1|20|32 bytes|
|Protocol name length|2–3|00 04|4|
|Protocol name|4–7|4D 51 54 54|"MQTT"|
|Protocol version|8|04|4 (MQTT 3.1.1)|
|Connect flags|9|02|See breakdown below|
|Keep-alive|10–11|00 3C|60 seconds|
|Client ID length|12–13|00 15|21|
|Client ID|14–…|73 6D 61 72 74 66 61 63 74 6F 72 79 2D 70 75 62 6C 69 73 68 65 72|Client ID = "smartfactory-publisher"|

**Connect Flags byte breakdown:**

|Bit|Name|Value|Meaning|
|-|-|-|-|
|7|Username flag|0|Username not present|
|6|Password flag|0|Password not present|
|5|Will retain|0|Will message not retained|
|4–3|Will QoS|00|QoS 0 (not used because Will disabled)|
|2|Will flag|0|No Will message|
|1|Clean session|0|Persistent session / clean session disabled|
|0|Reserved|0|Must be 0|

\---

### QoS 1 PUBLISH Packet

|Field|Offset (bytes)|Raw Hex|Decoded Value|
|-|-|-|-|
|Fixed header byte 1|0|32|Type=PUBLISH(3), DUP=0, QoS=1, RETAIN=0|
|Remaining length|1|A2|162 bytes|
|Topic length|2–3|00 19|25|
|Topic string|4–…|factory/line2/temperature|"factory/line2/temperature"|
|Packet Identifier|Offset not shown|04 92| 1170|
|Payload|…|JSON payload| sensor telemetry message|

**Fixed header byte 1 bit expansion:**

|Bits 7–4 (packet type)|Bit 3 (DUP)|Bits 2–1 (QoS)|Bit 0 (RETAIN)|
|-|-|-|-|
|0011 = PUBLISH (3)|0 = Not set|01 = QoS 1|0 = Not set|
|||||
|||||
|||||

\---

### PUBACK Packet

|Field|Offset|Raw Hex|Decoded Value|
|-|-|-|-|
|Fixed header|0|40|Type=PUBACK (0100)|
|Remaining length|1|02|2 bytes|
|Packet Identifier|2–3|2F E4|12260|

**Packet Identifier match:** PUBLISH PKT ID = 12260 ; PUBACK PKT ID = 12260 ; **Match? YES**



## 4.3 CoAP Packet Annotations

### CON GET Request



Bytes: \[ Header ] \[ Token ] \[ Uri-Host ] \[ Uri-Path: factory ] \[ Uri-Path: line1 ] \[ Uri-Path: temperature ]



|Field|Bits/Bytes|Raw Value|Decoded Value|
|-|-|-|-|
|Version (bits 7–6)|2 bits|01|1 (always 1)|
|Type (bits 5–4)|2 bits|00| 00 = CON|
|TKL (bits 3–0)|4 bits|0010|Token length = 2|
|Code (byte 1)|8 bits|01|*0.01* = GET|
|Message ID (bytes 2–3)|16 bits|22 DA|8922|
|Token (bytes 4–TKL+3)|TKL bytes|eedd|0xEEDD|
|Option Delta|4 bits|8 (Uri-Path)|Delta = 8 **, Option# = 11 (**Uri-Path)|
|Option Length|4 bits|variable|Length depends on Uri-Path segment|
|Option Value|variable bytes|factory / line1 / temperature|"factory/line1/temperature" (Uri-Path)|

**Byte 0 full expansion:**

|Bit 7|Bit 6|Bit 5|Bit 4|Bit 3|Bit 2|Bit 1|Bit 0|
|-|-|-|-|-|-|-|-|
|Ver|Ver|T|T|TKL|TKL|TKL|TKL|
|0|1|0|0|0|0|1|0|



### ACK 2.05 Content Response

|Field|Bytes|Raw Hex|Decoded Value|
|-|-|-|-|
|Fixed header byte 0|0|62|Ver=01, T=10 (ACK), TKL=2|
|Code byte 1|1|45|2.05 = Content|
|Message ID|2–3|22 DA|8922 (matches request? YES)|
|Token|4–5|eedd|0xEEDD (matches request? YES)|
|Option: Content-Format|…|C1 32|Option# = 12, Value = 50 (application/json)|
|Payload Marker|…|FF|0xFF|
|Payload|…|JSON payload|Sensor data in JSON format|



### Observe Notification

|Field|Value|
|-|-|
|Observe option number|6|
|Observe sequence value|Not observed in capture|
|Message type|CON|
|Response code|2.05 Content|

\---

## 4.4 AMQP Frame Annotations

### basic.publish Method Frame





AMQP protocol analysis was not performed because the assignment instructions specified that AMQP should be ignored.





|Field|Bytes|Raw Hex|Decoded Value|
|-|-|-|-|
|Frame Type|0|N/A|N/A — AMQP ignored per instructor instruction|
|Channel|1–2|N/A|N/A|
|Payload Size|3–6|N/A|N/A|
|Class ID|7–8|N/A|N/A — basic.publish not analyzed|
|Method ID|9–10|N/A|N/A — basic.publish not analyzed|
|Reserved (ticket)|11–12|N/A|N/A|
|Exchange name length|13|N/A|N/A|
|Exchange name|14–…|N/A|N/A|
|Routing key length|…|N/A|N/A|
|Routing key|…|N/A|N/A|
|Mandatory + Immediate|…|N/A|N/A|
|Frame End|last|N/A|N/A|

\---

### Content Header Frame

|Field|Bytes|Raw Hex|Decoded Value|
|-|-|-|-|
|Frame Type|0|N/A|N/A|
|Channel|1–2|N/A|N/A|
|Payload Size|3–6|N/A|N/A|
|Class ID|7–8|N/A|N/A|
|Weight|9–10|N/A|N/A|
|Body Size|11–18|N/A|N/A|
|Property Flags|19–20|N/A|N/A|
|delivery\_mode|…|N/A|N/A|
|content\_type length|…|N/A|N/A|
|content\_type|…|N/A|N/A|
|Frame End|last|N/A|N/A|



AMQP content header and heartbeat frames were not analyzed because AMQP was excluded from the assignment scope by the instructor.

### Heartbeat Frame

|Field|Value|
|-|-|
|Frame Type|N/A|
|Channel|N/A|
|Payload Size|N/A|
|Payload|N/A|
|Frame End|N/A|

**Why is the Heartbeat payload empty?**



N/A — AMQP heartbeat frames were not analyzed because AMQP was excluded from the assignment scope by the instructor.



*Module 1 Assignment — Real-Time Data Analytics for IoT*

