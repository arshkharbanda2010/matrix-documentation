<h2 style="padding-top:0">Wakeword</h2>
<h4 style="padding-top:0">Python Example</h4>

### Device Compatibility
<img class="creator-compatibility-icon" src="../../img/creator-icon.svg">
<img class="voice-compatibility-icon" src="../../img/voice-icon.svg">

## Overview

The Wakeword driver allows for:

* Reading custom wakewords created with <a href="http://www.speech.cs.cmu.edu/tools/lmtool-new.html" target="_blank">Sphinx Knowledge Base</a>.
* Notifications on which wakewords are heard.

<h3 style="padding-top:0">Available ZeroMQ Ports</h3>

* `Base port`: 60001
* `Error port`: 60003
* `Data update port`: 60004

## Code Example
The following sections show how to implement a connection to each of the IMU driver's ports. You can download this example <a href="https://github.com/matrix-io/matrix-core-examples/blob/master/python/wakeword.py" target="_blank">here</a>.

<!-- Setup -->
> Before moving on, please take a look at the Wakeword driver's protocol page and follow the [Installation](./../protocols/wakeword#installation) & [Creating Custom Phrases](./../protocols/wakeword#installation#creating-custom-phrases) sections.

<!-- Initial Variables -->
<details markdown="1" open>
<summary style="font-size: 1.75rem; font-weight: 300;">Initial Variables</summary>
Before we go into connecting to each port, the variables defined below are needed in order to access the ZeroMQ and MATRIX Protocol Buffer libraries for Python. We also define a few helpful variables and the path for our <a href="http://www.speech.cs.cmu.edu/tools/lmtool-new.html" target="_blank">Sphinx Knowledge Base</a> files.
```python
import os # Miscellaneous operating system interface
import zmq # Asynchronous messaging framework
import time # Time access and conversions
import sys # System-specific parameters and functions
from matrix_io.proto.malos.v1 import driver_pb2 # MATRIX Protocol Buffer driver library
from matrix_io.proto.malos.v1 import io_pb2 # MATRIX Protocol Buffer io library
from multiprocessing import Process # Allow for multiple processes at once
from zmq.eventloop import ioloop # Asynchronous events through ZMQ
matrix_ip = '127.0.0.1' # Local device ip
wakeword_port = 60001 # Driver Base port
# Handy functions for connecting to the Data Update, & Error port 
from utils import register_data_callback, register_error_callback
# Sphinx Knowledge Base files
LM_PATH = 'INSERT_PATH_TO_YOUR_FILE.lm'# Language Model File
DIC_PATH = 'INSERT_PATH_TO_YOUR_FILE.dic'# Dictation File
```
</details>

<!-- Base PORT -->
<details markdown="1" open>
<summary style="font-size: 1.75rem; font-weight: 300;">Base Port</summary>
Here is where the configuration for our wakeword example goes. Once we connect to the **Base Port**, we will pass a configuration to the Wakeword driver. With this we can set our wakeword configurations.
```python
def config_socket():
    # Define zmq socket
    context = zmq.Context()
    # Create a Pusher socket
    socket = context.socket(zmq.PUSH)
    # Connect Pusher to configuration socket
    socket.connect('tcp://{0}:{1}'.format(matrix_ip, wakeword_port))
    
    # Create a new driver config
    config = driver_pb2.DriverConfig()
    # Language Model File
    config.wakeword.lm_path = LM_PATH
    # Dictation File
    config.wakeword.dic_path = DIC_PATH
    # Desired MATRIX microphone
    config.wakeword.channel = 8
    # Enable verbose option
    config.wakeword.enable_verbose = False

    # Send driver configuration through ZMQ socket
    socket.send(config.SerializeToString())
    print ('Listening for wakewords')
```
</details>

<!-- Keep-alive PORT -->
<details markdown="1" open>
<summary style="font-size: 1.75rem; font-weight: 300;">Keep-alive Port</summary>
The next step is to connect and send a message to the **Keep-alive Port**. That message will grant us a response from the Data Update Port with the wake words that were understood.
```python
def ping_socket():
    # Define zmq socket
    context = zmq.Context()
    # Create a Pusher socket
    ping_socket = context.socket(zmq.PUSH)
    # Connect to the socket
    ping_socket.connect('tcp://{0}:{1}'.format(matrix_ip, wakeword_port+1))
    # Send a ping every 2 seconds
    while True:
        ping_socket.send_string('')
        time.sleep(2)
```
</details>

<!-- Error PORT -->
<details markdown="1" open>
<summary style="font-size: 1.75rem; font-weight: 300;">Error Port</summary>
The **Error Port** connection is also taken care of by the `utils import`. Below we define a function to be called and given any error messages that occur within MATRIX CORE.
> The Error Port is currently reporting false errors. 
```python
def wakeword_error_callback(error):
    # Log error
    print('{0}'.format(error))
```
</details>

<!-- Data Update PORT -->
<details markdown="1" open>
<summary style="font-size: 1.75rem; font-weight: 300;">Data Update Port</summary>
A connection to the **Data Update Port** is then made to allow us to receive each custom phrase the Wakeword driver picks up.

```python
def wakeword_data_callback(data):
    # Extract data
    data = io_pb2.WakeWordParams().FromString(data[0])
    # Log data 
    print('{0}'.format(data))
    # Run actions based on the phrase heard
    # CHANGE TO YOUR PHRASE
    if data.wake_word == 'MATRIX START':
        print ('I HEARD MATRIX START!\n')
    # CHANGE TO YOUR PHRASE
    elif data.wake_word == 'MATRIX STOP':
        print ('I HEARD MATRIX STOP!\n')
```
<h4>Data Output</h4>
The Python object below is an example output you'll receive from the **Data Update Port**. All wakeword strings are capitalized.
```python
wake_word: "MATRIX START"
```
</details>

<!-- Start Process -->
<details markdown="1" open>
<summary style="font-size: 1.75rem; font-weight: 300;">Start Processes</summary>
This is where we begin the asynchronous events for each of the driver ports and where we define the functions we want to use for each port.

```python
if __name__ == '__main__':
    # Initiate asynchronous events
    ioloop.install()
    # Send Base Port configuration 
    config_socket()
    # Start Error Port connection
    # Process(target=register_error_callback, args=(wakeword_error_callback, matrix_ip, wakeword_port)).start()
    # Start Data Update Port connection
    Process(target=register_data_callback, args=(wakeword_data_callback, matrix_ip, wakeword_port)).start()
    # Start Keep-alive Port connection
    Process(target=ping_socket).start()
```
</details>