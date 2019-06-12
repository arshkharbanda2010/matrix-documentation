<h2 style="padding-top:0">Inertial Measurement Unit (IMU)</h2>
<h4 style="padding-top:0">Javascript Example</h4>

### Device Compatibility
<img class="creator-compatibility-icon" src="../../img/creator-icon.svg">

## Overview

The IMU driver reports values for:

* Yaw, Pitch, and Roll
* Acceleration for **x**, **y**, **z** axes
* Gyroscope for **x**, **y**, **z** axes
* Magnetometer for **x**, **y**, **z** axes

<h3 style="padding-top:0">Available ZeroMQ Ports</h3>
* `Base port`: 20013
* `Keep-alive port`: 20014
* `Error port`: 20015
* `Data Update port`: 20016

## Code Example
The following sections show how to implement a connection to each of the IMU driver's ports. You can download this example <a href="https://github.com/matrix-io/matrix-core-examples/blob/master/javascript/imu.js" target="_blank">here</a>.

<!-- Initial Variables -->
<details open>
<summary style="font-size: 1.75rem; font-weight: 300;">Initial Variables</summary>
Before we go into connecting to each port, the variables defined below are needed in order to access the ZeroMQ and MATRIX Protocol Buffer libraries for Javascript. We also define a few helpful variables for easy references.
```javascript
var zmq = require('zeromq');// Asynchronous Messaging Framework
var matrix_io = require('matrix-protos').matrix_io;// Protocol Buffers for MATRIX function
var matrix_ip = '127.0.0.1';// Local IP
var matrix_imu_base_port = 20013;// Port for IMU driver
```
</details>

<!-- Base PORT -->
<details open>
<summary style="font-size: 1.75rem; font-weight: 300;">Base Port</summary>
Here is where the configuration for our IMU example goes. Once we connect to the **Base Port**, we will pass a configuration to the IMU driver. With this we can set the update rate and timeout configuration.
```javascript
// Create a Pusher socket
var configSocket = zmq.socket('push');
// Connect Pusher to Base port
configSocket.connect('tcp://' + matrix_ip + ':' + matrix_imu_base_port);
// Create driver configuration
var config = matrix_io.malos.v1.driver.DriverConfig.create({
  // Update rate configuration
  delayBetweenUpdates: 2.0,// 2 seconds between updates
  timeoutAfterLastPing: 6.0,// Stop sending updates 6 seconds after pings.
});
// Send driver configuration
configSocket.send(matrix_io.malos.v1.driver.DriverConfig.encode(config).finish());
```
</details>

<!-- Keep-alive PORT -->
<details open>
<summary style="font-size: 1.75rem; font-weight: 300;">Keep-alive Port</summary>
The next step is to connect and send a message to the **Keep-alive Port**. That message, an empty string, will grant us a response from the **Data Update Port** for the current IMU values. An interval for pinging is then set to continuously obtain that data.
```javascript
// Create a Pusher socket
var pingSocket = zmq.socket('push');
// Connect Pusher to Keep-alive port
pingSocket.connect('tcp://' + matrix_ip + ':' + (matrix_imu_base_port + 1));
// Send initial ping
pingSocket.send('');
// Send ping every 5 seconds
setInterval(function(){
  pingSocket.send('');
}, 5000);
```
</details>

<!-- Error PORT -->
<details open>
<summary style="font-size: 1.75rem; font-weight: 300;">Error Port</summary>
Connecting to the **Error Port** is optional, but highly recommended if you want to log any errors that occur within MATRIX CORE.
```javascript
// Create a Subscriber socket
var errorSocket = zmq.socket('sub');
// Connect Subscriber to Error port
errorSocket.connect('tcp://' + matrix_ip + ':' + (matrix_imu_base_port + 2));
// Connect Subscriber to Error port
errorSocket.subscribe('');
// On Message
errorSocket.on('message', function(error_message){
  console.log('Error received: ' + error_message.toString('utf8'));// Log error
});
```
</details>

<!-- Data Update PORT -->
<details open>
<summary style="font-size: 1.75rem; font-weight: 300;">Data Update Port</summary>
A connection to the **Data Update Port** is then made to allow us to receive the current IMU data we want.

```javascript
// Create a Subscriber socket
var updateSocket = zmq.socket('sub');
// Connect Subscriber to Data Update port
updateSocket.connect('tcp://' + matrix_ip + ':' + (matrix_imu_base_port + 3));
// Subscribe to messages
updateSocket.subscribe('');
// On Message
updateSocket.on('message', function(buffer){
  var data = matrix_io.malos.v1.sense.Imu.decode(buffer);// Extract message
    console.log(data);// Log new IMU data
});
```
<h4>Data Output</h4>
The javascript object below is an example output you'll receive from the **Data Update Port**.
```javascript
{
  yaw: 29.820072174072266,
  pitch: 9.994316101074219,
  roll: -179.4230194091797,
  accelX: -0.17499999701976776,
  accelY: -0.009999999776482582,
  accelZ: -0.9929999709129333,
  gyroX: 2.871000051498413,
  gyroY: 0.3059999942779541,
  gyroZ: 0.8069999814033508,
  magX: -0.0820000022649765,
  magY: 0.04699999839067459,
  magZ: 0.11299999803304672
}
```
</details>