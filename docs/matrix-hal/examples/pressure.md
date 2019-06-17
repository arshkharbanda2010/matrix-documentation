<h2 style="padding-top:0">Pressure</h2>
<h4 style="padding-top:0">HAL Example</h4>

### Device Compatibility
<img class="creator-compatibility-icon" src="../../img/creator-icon.svg">

## Overview

The pressure sensor reports values for:

* Pressure
* Altitude
* Temperature

## Code Example

Below is an example of how to interface with the pressure sensor in MATRIX HAL.

Pressure sensor function references can be found [here](/matrix-hal/reference/pressure).

The following section shows how to receive data from the pressure sensor. You can download this example <a href="https://github.com/matrix-io/matrix-hal-examples/blob/master/sensors/pressure_sensor.cpp" target="_blank">here</a>.

The command below will compile the example. Be sure to pass in your C++ file and desired output file.

```c++
g++ -o YOUR_OUTPUT_FILE YOUR_CPP_FILE -std=c++11 -lmatrix_creator_hal
```

<details markdown="1" open>
<summary style="font-size: 1.5rem; font-weight: 300;">Include Statements</summary>
To begin working with the pressure sensor you need to include these header files.

```c++
// System calls
#include <unistd.h>
// Input/output streams and functions
#include <iostream>

// Interfaces with pressure sensor
#include "matrix_hal/pressure_sensor.h"
// Holds data from pressure sensor
#include "matrix_hal/pressure_data.h"
// Communicates with MATRIX device
#include "matrix_hal/matrixio_bus.h"
```

</details>

<details markdown="1" open>
<summary style="font-size: 1.5rem; font-weight: 300;">Initial Setup</summary>
You'll then need to setup `MatrixIOBus` in order to communicate with the hardware on your MATRIX device.

```c++
int main() {
  // Create MatrixIOBus object for hardware communication
  matrix_hal::MatrixIOBus bus;
  // Initialize bus and exit program if error occurs
  if (!bus.Init()) return false;
```

</details>

<details markdown="1" open>
<summary style="font-size: 1.5rem; font-weight: 300;">Main Setup</summary>
Now we will create our `PressureData` and `PressureSensor` object and use it to receive data from the pressure sensor.

```c++
  // The following code is part of main()

  // Create PressureData object
  matrix_hal::PressureData pressure_data;
  // Create PressureSensor object
  matrix_hal::PressureSensor pressure_sensor;
  // Set pressure_sensor to use MatrixIOBus bus
  pressure_sensor.Setup(&bus);

  // Endless loop
  while (true) {
    // Overwrites pressure_data with new data from pressure sensor
    pressure_sensor.Read(&pressure_data);
    // Altitude output is represented in meters
    float altitude = pressure_data.altitude;
    // Pressure output is represented in Pa
    float pressure = pressure_data.pressure;
    // Temperature output is represented in Celsius
    float temperature = pressure_data.temperature;
    // Clear console
    std::system("clear");
    // Output sensor data to console
    std::cout << " [ Pressure Sensor Output ]" << std::endl;
    std::cout << " [ Altitude (m) : " << altitude
              << " ] [ Pressure (Pa) : " << pressure
              << " ] [ Temperature (Celsius) : " << temperature << " ]" << std::endl;

    // Sleep for 20000 microseconds
    usleep(20000);
  }

  return 0;
}
```

</details>