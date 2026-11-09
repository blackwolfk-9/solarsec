# Local Energy Display

Owners of solar systems usually see their data through the manufacturer's cloud and app. That means the data leaves the house and the security depends on the manufacturer keeping the service safe.

**Goal:** A display device that sits in the building, reads the solar system's values directly over the local network, stores them locally and shows them on a touchscreen. No manufacturer cloud is needed to read the data.

**Target group** B2B: The service company that installs the system can sell the panel as an optional accessory. The device is placed at the enduser’s site and shows that customer's system. 

Target group B2C: Customers of Hoymiles or Sigenergy Systems, who want to be as independent as possible can use our display instead of the manufacturers cloud application.

### Functionality

The system reads live values from a Sigenergy or a Hoymiles inverter and stores the local history on the device. It shows live values on a screen and hides fields that the connected system does not provide. We need to be able to do remote software updates for a fleet of devices, without opening any inbound port.

### Constraints

- Runs on a single small computer at the customer's site on the customers local network
- Updates must be possible for many devices without on-site visits

| Manufacturer | How we read it |
| --- | --- |
| Sigenergy | Directly, Modbus TCP |
| Hoymiles | Via an additional OpenDTU radio module, MQTT |

### System overview

It is important to recognize, that our System needs to be working with the systems of both manufacturers. The Data Model needs to have one format so the incoming Data needs to be transformed in the driver before it’s getting stored.

Sigenergy

![image.png](Local%20Energy%20Display/image.png)

Hoymiles

![image.png](Local%20Energy%20Display/image%201.png)

Data

|  | Sigenergy | Hoymiles |
| --- | --- | --- |
| Protocol | Modbus TCP, port 502 | MQTT (OpenDTU publishes to a broker) |
| Who starts the transfer | We are polling information | OpenDTU pushes and we listen |
| Data format | 16-bit register values, scaled by a gain (e.g. 523 ÷ 10 = 52.3) | One MQTT topic per value, plain text |
| Access protection | none on the protocol: any device in the home network can read and write | Reading open by default; settings and control require login |

### Sigenergy

The Energy Controller speaks Modbus TCP itself. The collector connects to port 502, reads a set of registers at a fixed interval and converts the raw numbers with the gain from the manufacturer's protocol.

### Hoymiles via OpenDTU

Hoymiles microinverters have no network port. They send data by radio. **OpenDTU** is open-source firmware for ESP32 microcontrollers. It receives the radio data and publishes it on the home network. The job of our system is to get the data from the home network, bring it into the right format and display it on the screen.

### Software architecture

Solar System → Driver → Data Model → Core → Display

| Station | Job |
| --- | --- |
| **System** | The  solar system: the source of values. |
| **Driver** | Speaks one manufacturer's language and returns values in the common format. |
| **Data model** | Needs to be working with both systems. Missing values stay empty. |
| **Core** | Gets a value from the driver, stores it, sends it to the display. No manufacturer logic. |
| **Display** | A plain web page in full screen. Shows what the core sends, hides empty fields. |

### Data Model

We need to know, what Data we get from each the Hoymiles and Sigenergy Panel, so we can create a Data Model. Metrics that are only provided by one of the panel should not be shown on the display when there is no data.

### Possible Security Issues

1. Someone gets into the home network and fakes or listens to MQTT messages.
2. Faulty or fake values breake the programm.
3. Third party systems have a vulnerability