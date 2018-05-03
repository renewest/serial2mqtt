<!DOCTYPE html>
<html>

<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Serial2Mqtt</title>
  <link rel="stylesheet" href="https://stackedit.io/style.css" />
</head>

<body class="stackedit">
  <div class="stackedit__left">
    <div class="stackedit__toc">
      
<ul>
<li><a href="#serial2mqtt">serial2mqtt</a></li>
<li><a href="#work-in-progress--major-rework">WORK IN PROGRESS !! Major rework</a>
<ul>
<li><a href="#working-assumptions">Working assumptions</a></li>
</ul>
</li>
<li><a href="#programming-through-serial2mqtt">Programming through serial2mqtt</a></li>
<li><a href="#logging-through-serial2mqtt">Logging through serial2mqtt</a></li>
</ul>

    </div>
  </div>
  <div class="stackedit__right">
    <div class="stackedit__html">
      <h1 id="serial2mqtt">serial2mqtt</h1>
<h1 id="work-in-progress--major-rework">WORK IN PROGRESS !! Major rework</h1>
<p>MQTT for all micro-controllers ! The purpose is to offer MQTT publisher/subscriber functionality to all small micro controllers. Those with just a UART or USB interface.<br>
Example : a cheap STM32 board on ebay.<br>
<img src="http://www.rogerclark.net/wp-content/uploads/2014/11/STM32Mini-300x300.jpg" alt="Afbeeldingsresultaat voor ebay stm32"><br>
This program will act as a full MQTT Client gateway and make integration as simple as possible.<br>
This was created because Ethernet or WiFi is still absent in most ( cheap ) controllers .<br>
Also the concept behind is that a central PC or Raspberry PI can act as the intelligent mind behind commodity components.<br>
![enter image description here](./images/serial2mqtt concept.jpg)</p>
<p>Arduino Sample program to communicate with the serial2mqtt  gateway</p>
<pre><code>class Mqtt {
  public:
    static void publish(int qos, bool retain, String topic, String message ) {
      Serial.printf("M,%d,%d,%s,%s\n", qos, retain, topic.c_str(), message.c_str());
    }
};
int i = 0;
void loop() {
  delay(100);                       // wait for a 0.1 second
  String  message=String(i++);
  Mqtt::publish(0, false, "src/arduino1/system/upTime", message);
  Mqtt::publish(0,false,"src/arduino1/system/host","arduino1");
}
</code></pre>
<h2 id="working-assumptions">Working assumptions</h2>
<ul>
<li>Topic Names<br>
–The design will take into account some assumptions about topic names and tree-structure to make it simple to use.<br>
Structure topic to and from  device :<br>
– dst/DEVICE/SERVICE/PROPERTY<br>
– src/DEVICE/SERVICE/PROPERTY<br>
– if DEVICE is not known yet the serial2mqtt will subscribe to the dst/PORT/prog , where PORT is for example ttyUSB0</li>
<li>Serial messages will be <strong>BINARY</strong> or <strong>ASCII</strong><br>
– BINARY format will be CBOR encoded in a SLIP envelope<br>
– ASCII will be text delimited by newlines</li>
<li>The serial2mqtt should be able to reset the device ( hard reset )</li>
<li>The serial2mqtt should be able to program new code into the device</li>
<li>Through the same communication, debugging logs can be handled without disturbing the mqtt flow</li>
<li>the serial2mqtt establishes the client MQTT link and subscribes to dst/DEVICE/#</li>
<li>when there is a big delay on the serial2mqtt serial input, it will stop the TCP MQTT connection</li>
<li>serial2mqtt is event driven and not blocking</li>
<li>one instance of serial2mqtt should be able to handle different serial ports</li>
<li>USB devices coming and going should be tracked by serial2mqtt</li>
<li>Configuration can be command line and config file driven ( JSON )</li>
<li>serial2mqtt should be able to program the device through the serial interface, for this purpose a third party app will be launched with the concerned serial port as argument.</li>
</ul>
<div class="mermaid"><svg xmlns="http://www.w3.org/2000/svg" id="mermaid-svg-nlODJXV7G1stxLWf" height="100%" width="100%" style="max-width:650px;" viewBox="-50 -10 650 643.984375"><g></g><g><line id="actor35" x1="75" y1="5" x2="75" y2="632.984375" class="actor-line" stroke-width="0.5px" stroke="#999"></line><rect x="0" y="0" fill="#eaeaea" stroke="#666" width="150" height="65" rx="3" ry="3" class="actor"></rect><text x="75" y="32.5" dominant-baseline="central" alignment-baseline="central" class="actor" style="text-anchor: middle;"><tspan x="75" dy="0">µC</tspan></text></g><g><line id="actor36" x1="275" y1="5" x2="275" y2="632.984375" class="actor-line" stroke-width="0.5px" stroke="#999"></line><rect x="200" y="0" fill="#eaeaea" stroke="#666" width="150" height="65" rx="3" ry="3" class="actor"></rect><text x="275" y="32.5" dominant-baseline="central" alignment-baseline="central" class="actor" style="text-anchor: middle;"><tspan x="275" dy="0">serial2mqtt</tspan></text></g><g><line id="actor37" x1="475" y1="5" x2="475" y2="632.984375" class="actor-line" stroke-width="0.5px" stroke="#999"></line><rect x="400" y="0" fill="#eaeaea" stroke="#666" width="150" height="65" rx="3" ry="3" class="actor"></rect><text x="475" y="32.5" dominant-baseline="central" alignment-baseline="central" class="actor" style="text-anchor: middle;"><tspan x="475" dy="0">MQTT Broker</tspan></text></g><defs><marker id="arrowhead" refX="5" refY="2" markerWidth="6" markerHeight="4" orient="auto"><path d="M 0,0 V 4 L6,2 Z"></path></marker></defs><defs><marker id="crosshead" markerWidth="15" markerHeight="8" orient="auto" refX="16" refY="4"><path fill="black" stroke="#000000" stroke-width="1px" d="M 9,2 V 6 L16,4 Z" style="stroke-dasharray: 0, 0;"></path><path fill="none" stroke="#000000" stroke-width="1px" d="M 0,1 L 6,7 M 6,1 L 0,7" style="stroke-dasharray: 0, 0;"></path></marker></defs><g><text x="175" y="93" class="messageText" style="text-anchor: middle;">M,PUB,0,0,src/drive/motor/rpm,1234</text><line x1="75" y1="100" x2="275" y2="100" class="messageLine0" stroke-width="2" stroke="black" marker-end="url(#arrowhead)" style="fill: none;"></line></g><g><rect x="300" y="110" fill="#EDF2AE" stroke="#666" width="150" height="80.390625" rx="0" ry="0" class="note"></rect><text x="316" y="140" fill="black" class="noteText"><tspan x="316">connects to mqtt</tspan><tspan dy="23" x="316">broker at first</tspan><tspan dy="23" x="316">message.</tspan></text></g><g><text x="375" y="218.390625" class="messageText" style="text-anchor: middle;">CONNECT(broker,port)</text><line x1="275" y1="225.390625" x2="475" y2="225.390625" class="messageLine1" stroke-width="2" stroke="black" marker-end="url(#arrowhead)" style="stroke-dasharray: 3, 3; fill: none;"></line></g><g><text x="375" y="253.390625" class="messageText" style="text-anchor: middle;">CONNACK</text><line x1="475" y1="260.390625" x2="275" y2="260.390625" class="messageLine1" stroke-width="2" stroke="black" marker-end="url(#crosshead)" style="stroke-dasharray: 3, 3; fill: none;"></line></g><g><text x="375" y="288.390625" class="messageText" style="text-anchor: middle;">SUBSCRIBE("dst/drive/</text><line x1="275" y1="295.390625" x2="475" y2="295.390625" class="messageLine1" stroke-width="2" stroke="black" marker-end="url(#arrowhead)" style="stroke-dasharray: 3, 3; fill: none;"></line></g><g><text x="375" y="323.390625" class="messageText" style="text-anchor: middle;">PUBLISH("src/drive/motor/rpm",1000,0,false)</text><line x1="275" y1="330.390625" x2="475" y2="330.390625" class="messageLine1" stroke-width="2" stroke="black" marker-end="url(#arrowhead)" style="stroke-dasharray: 3, 3; fill: none;"></line></g><g><text x="175" y="358.390625" class="messageText" style="text-anchor: middle;">M,PUB,0,0,src/drive/motor/temp,20</text><line x1="75" y1="365.390625" x2="275" y2="365.390625" class="messageLine0" stroke-width="2" stroke="black" marker-end="url(#arrowhead)" style="fill: none;"></line></g><g><text x="375" y="393.390625" class="messageText" style="text-anchor: middle;">PUBLISH("src/drive/motor/temp",20,0,false)</text><line x1="275" y1="400.390625" x2="475" y2="400.390625" class="messageLine1" stroke-width="2" stroke="black" marker-end="url(#arrowhead)" style="stroke-dasharray: 3, 3; fill: none;"></line></g><g><rect x="100" y="410.390625" fill="#EDF2AE" stroke="#666" width="150" height="102.59375" rx="0" ry="0" class="note"></rect><text x="116" y="440" fill="black" class="noteText"><tspan x="116">no more messages</tspan><tspan dy="23" x="116">after 5 sec,</tspan><tspan dy="23" x="116">serial2mqtt</tspan><tspan dy="23" x="116">disconnects.</tspan></text></g><g><text x="375" y="540.984375" class="messageText" style="text-anchor: middle;">DISCONNECT</text><line x1="275" y1="547.984375" x2="475" y2="547.984375" class="messageLine1" stroke-width="2" stroke="black" marker-end="url(#arrowhead)" style="stroke-dasharray: 3, 3; fill: none;"></line></g><g><rect x="0" y="567.984375" fill="#eaeaea" stroke="#666" width="150" height="65" rx="3" ry="3" class="actor"></rect><text x="75" y="600.484375" dominant-baseline="central" alignment-baseline="central" class="actor" style="text-anchor: middle;"><tspan x="75" dy="0">µC</tspan></text></g><g><rect x="200" y="567.984375" fill="#eaeaea" stroke="#666" width="150" height="65" rx="3" ry="3" class="actor"></rect><text x="275" y="600.484375" dominant-baseline="central" alignment-baseline="central" class="actor" style="text-anchor: middle;"><tspan x="275" dy="0">serial2mqtt</tspan></text></g><g><rect x="400" y="567.984375" fill="#eaeaea" stroke="#666" width="150" height="65" rx="3" ry="3" class="actor"></rect><text x="475" y="600.484375" dominant-baseline="central" alignment-baseline="central" class="actor" style="text-anchor: middle;"><tspan x="475" dy="0">MQTT Broker</tspan></text></g></svg></div>
<h1 id="programming-through-serial2mqtt">Programming through serial2mqtt</h1>
<p>A command line utility will send a single mqtt request to the serial2mqtt gateway to program the microcontroller.</p>
<div class="mermaid"><svg xmlns="http://www.w3.org/2000/svg" id="mermaid-svg-tHiothPWrZDteOBQ" height="100%" width="100%" style="max-width:850px;" viewBox="-50 -10 850 336"><g></g><g><line id="actor38" x1="75" y1="5" x2="75" y2="325" class="actor-line" stroke-width="0.5px" stroke="#999"></line><rect x="0" y="0" fill="#eaeaea" stroke="#666" width="150" height="65" rx="3" ry="3" class="actor"></rect><text x="75" y="32.5" dominant-baseline="central" alignment-baseline="central" class="actor" style="text-anchor: middle;"><tspan x="75" dy="0">µC</tspan></text></g><g><line id="actor39" x1="275" y1="5" x2="275" y2="325" class="actor-line" stroke-width="0.5px" stroke="#999"></line><rect x="200" y="0" fill="#eaeaea" stroke="#666" width="150" height="65" rx="3" ry="3" class="actor"></rect><text x="275" y="32.5" dominant-baseline="central" alignment-baseline="central" class="actor" style="text-anchor: middle;"><tspan x="275" dy="0">serial2mqtt</tspan></text></g><g><line id="actor40" x1="475" y1="5" x2="475" y2="325" class="actor-line" stroke-width="0.5px" stroke="#999"></line><rect x="400" y="0" fill="#eaeaea" stroke="#666" width="150" height="65" rx="3" ry="3" class="actor"></rect><text x="475" y="32.5" dominant-baseline="central" alignment-baseline="central" class="actor" style="text-anchor: middle;"><tspan x="475" dy="0">MQTT Broker</tspan></text></g><g><line id="actor41" x1="675" y1="5" x2="675" y2="325" class="actor-line" stroke-width="0.5px" stroke="#999"></line><rect x="600" y="0" fill="#eaeaea" stroke="#666" width="150" height="65" rx="3" ry="3" class="actor"></rect><text x="675" y="32.5" dominant-baseline="central" alignment-baseline="central" class="actor" style="text-anchor: middle;"><tspan x="675" dy="0">programmer CLI</tspan></text></g><defs><marker id="arrowhead" refX="5" refY="2" markerWidth="6" markerHeight="4" orient="auto"><path d="M 0,0 V 4 L6,2 Z"></path></marker></defs><defs><marker id="crosshead" markerWidth="15" markerHeight="8" orient="auto" refX="16" refY="4"><path fill="black" stroke="#000000" stroke-width="1px" d="M 9,2 V 6 L16,4 Z" style="stroke-dasharray: 0, 0;"></path><path fill="none" stroke="#000000" stroke-width="1px" d="M 0,1 L 6,7 M 6,1 L 0,7" style="stroke-dasharray: 0, 0;"></path></marker></defs><g><text x="175" y="93" class="messageText" style="text-anchor: middle;">M,PUB,0,0,src/drive/motor/rpm,1234</text><line x1="75" y1="100" x2="275" y2="100" class="messageLine0" stroke-width="2" stroke="black" marker-end="url(#arrowhead)" style="fill: none;"></line></g><g><text x="375" y="128" class="messageText" style="text-anchor: middle;">PUBLISH("src/drive/motor/temp",20,0,false)</text><line x1="275" y1="135" x2="475" y2="135" class="messageLine0" stroke-width="2" stroke="black" marker-end="url(#arrowhead)" style="fill: none;"></line></g><g><text x="575" y="163" class="messageText" style="text-anchor: middle;">PUBLISH("dst/drive/prog/flash",flash image binary)</text><line x1="675" y1="170" x2="475" y2="170" class="messageLine0" stroke-width="2" stroke="black" marker-end="url(#crosshead)" style="fill: none;"></line></g><g><text x="375" y="198" class="messageText" style="text-anchor: middle;">PUBLISH</text><line x1="475" y1="205" x2="275" y2="205" class="messageLine0" stroke-width="2" stroke="black" marker-end="url(#crosshead)" style="fill: none;"></line></g><g><rect x="270" y="207" fill="#f4f4f4" stroke="#666" width="10" height="33" rx="0" ry="0"></rect></g><g><text x="172.5" y="233" class="messageText" style="text-anchor: middle;">program flash image</text><line x1="270" y1="240" x2="75" y2="240" class="messageLine0" stroke-width="2" stroke="black" marker-end="url(#crosshead)" style="fill: none;"></line></g><g><rect x="0" y="260" fill="#eaeaea" stroke="#666" width="150" height="65" rx="3" ry="3" class="actor"></rect><text x="75" y="292.5" dominant-baseline="central" alignment-baseline="central" class="actor" style="text-anchor: middle;"><tspan x="75" dy="0">µC</tspan></text></g><g><rect x="200" y="260" fill="#eaeaea" stroke="#666" width="150" height="65" rx="3" ry="3" class="actor"></rect><text x="275" y="292.5" dominant-baseline="central" alignment-baseline="central" class="actor" style="text-anchor: middle;"><tspan x="275" dy="0">serial2mqtt</tspan></text></g><g><rect x="400" y="260" fill="#eaeaea" stroke="#666" width="150" height="65" rx="3" ry="3" class="actor"></rect><text x="475" y="292.5" dominant-baseline="central" alignment-baseline="central" class="actor" style="text-anchor: middle;"><tspan x="475" dy="0">MQTT Broker</tspan></text></g><g><rect x="600" y="260" fill="#eaeaea" stroke="#666" width="150" height="65" rx="3" ry="3" class="actor"></rect><text x="675" y="292.5" dominant-baseline="central" alignment-baseline="central" class="actor" style="text-anchor: middle;"><tspan x="675" dy="0">programmer CLI</tspan></text></g></svg></div>
<h1 id="logging-through-serial2mqtt">Logging through serial2mqtt</h1>
<p>The micrcontroller will also log to the central logging system</p>

    </div>
  </div>
</body>

</html>