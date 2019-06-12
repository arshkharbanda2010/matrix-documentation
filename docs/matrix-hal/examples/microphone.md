<h2 style="padding-top:0">Microphone Array</h2>
<h4 style="padding-top:0">HAL Example</h4>

### Device Compatibility

<img class="creator-compatibility-icon" src="../../img/creator-icon.svg">
<img class="voice-compatibility-icon" src="../../img/voice-icon.svg">

## Overview

The microphone array interface supports:

- Accepting input from individual microphones
- Accepting input from beamformed microphone

**Device Pinouts**:

- [MATRIX Creator](/matrix-creator/resources/pinout.md)
- [MATRIX Voice](/matrix-voice/resources/pinout.md)

## Code Examples

Below are examples of how to interface with the microphone array in MATRIX HAL.

Microphone array function references can be found [here](/matrix-hal/reference/microphone).

The command below will compile each example. Be sure to pass in your C++ file and desired output file.

```cpp
g++ -o YOUR_OUTPUT_FILE YOUR_CPP_FILE -std=c++11 -lmatrix_creator_hal -lgflags
```

<details>
<summary style="font-size: 1.75rem; font-weight: 300;">Microphone Array Record to File</summary>
The following section shows how to record data from the microphone array to a file. You can download this example <a href="https://github.com/matrix-io/matrix-hal-examples/blob/master/microphone_array/mic_record_file.cpp" target="_blank">here</a>.

> Beamformed microphone is channel 8

To convert the `.raw` files outputted by this example to playable `.wav` files run these commands, replacing `16000` with selected sampling rate.

```bash
sudo apt-get install sox alsa-utils
sox -r 16000 -c 1 -e signed -c 1 -e signed -b 16 mic_16000_s16le_channel_0.raw channel_0.wav
sox -r 16000 -c 1 -e signed -c 1 -e signed -b 16 mic_16000_s16le_channel_1.raw channel_1.wav
sox -r 16000 -c 1 -e signed -c 1 -e signed -b 16 mic_16000_s16le_channel_2.raw channel_2.wav
sox -r 16000 -c 1 -e signed -c 1 -e signed -b 16 mic_16000_s16le_channel_3.raw channel_3.wav
sox -r 16000 -c 1 -e signed -c 1 -e signed -b 16 mic_16000_s16le_channel_4.raw channel_4.wav
sox -r 16000 -c 1 -e signed -c 1 -e signed -b 16 mic_16000_s16le_channel_5.raw channel_5.wav
sox -r 16000 -c 1 -e signed -c 1 -e signed -b 16 mic_16000_s16le_channel_6.raw channel_6.wav
sox -r 16000 -c 1 -e signed -c 1 -e signed -b 16 mic_16000_s16le_channel_7.raw channel_7.wav
sox -r 16000 -c 1 -e signed -c 1 -e signed -b 16 mic_16000_s16le_channel_8.raw channel_8.wav
```

<details open>
<summary style="font-size: 1.5rem; font-weight: 300;">Include Statements</summary>
To begin working with the Microphone Array you need to include these header files.

```cpp
// Google gflags parser
#include <gflags/gflags.h>
// Communicating with Pi GPIO
#include <wiringPi.h>
// Input/output stream class to operate on files
#include <fstream>
// Input/output streams and functions
#include <iostream>
// Use strings
#include <string>
// Arrays for math operations
#include <valarray>

// Communicates with MATRIX device
#include "matrix_hal/matrixio_bus.h"
// Interfaces with microphone array
#include "matrix_hal/microphone_array.h"
// Enables using FIR filter with microphone array
#include "matrix_hal/microphone_core.h"
```

</details>

<details open>
<summary style="font-size: 1.5rem; font-weight: 300;">Initial Variables</summary>
These initial variables are used in the example.

```cpp
// Defines variables from user arguments using gflags utility
// (https://gflags.github.io/gflags/)

// Grabs sampling frequency input from user
DEFINE_int32(sampling_frequency, 16000, "Sampling Frequency");  // Argument example: "--sampling_frequency 48000"
// Grabs duration input from user
DEFINE_int32(duration, 5, "Interrupt after N seconds"); // Argument example: "--duration 10"
// Grabs gain input from user
DEFINE_int32(gain, -1, "Microphone Gain"); // Argument example: "--gain 5"
```

</details>

<details open>
<summary style="font-size: 1.5rem; font-weight: 300;">Initial Setup</summary>
You'll then need to setup `MatrixIOBus` in order to communicate with the hardware on your MATRIX device. Also, parse command line flags and set user flags as variables.

```cpp
int main(int argc, char *agrv[]) {
  // Parse command line flags with gflags utility
  // (https://gflags.github.io/gflags/)
  google::ParseCommandLineFlags(&argc, &agrv, true);

  // Create MatrixIOBus object for hardware communication
  matrix_hal::MatrixIOBus bus;
  // Initialize bus and exit program if error occurs
  if (!bus.Init()) return false;

  // Set user flags from gflags as variables
  int sampling_rate = FLAGS_sampling_frequency;
  int seconds_to_record = FLAGS_duration;
  int gain = FLAGS_gain;
```

</details>

<details open>
<summary style="font-size: 1.5rem; font-weight: 300;">Main Setup</summary>
Now we will create our `MicrophoneArray` object and use it to interface with the microphone array.

```cpp
  // The following code is part of main()

  // Create MicrophoneArray object
  matrix_hal::MicrophoneArray microphone_array;
  // Set microphone_array to use MatrixIOBus bus
  microphone_array.Setup(&bus);
  // Set microphone sampling rate
  microphone_array.SetSamplingRate(sampling_rate);
  // If gain is positive, set the gain
  if (gain > 0) microphone_array.SetGain(gain);

  // Log gain_ and sampling_frequency_ variables
  microphone_array.ShowConfiguration();
  // Log recording duration variable
  std::cout << "Duration : " << seconds_to_record << "s" << std::endl;

  // Calculate and set up beamforming delays for beamforming
  microphone_array.CalculateDelays(0, 0, 1000, 320 * 1000);  // These are default values
```

</details>

<details open>
<summary style="font-size: 1.5rem; font-weight: 300;">Fir Filter Setup</summary>
Now we will create our `MicrophoneCore` object and use it to enable the FIR filter.

```cpp
  // The following code is part of main()

  // Create MicrophoneCore object
  matrix_hal::MicrophoneCore microphone_core(microphone_array);
  // Set microphone_core to use MatrixIOBus bus
  microphone_core.Setup(&bus);
```

</details>

<details open>
<summary style="font-size: 1.5rem; font-weight: 300;">Microphone Input</summary>
Now we will read microphone array data, send to a buffer, and write to file.

```cpp
  // The following code is part of main()

  // Create a buffer array for microphone input
  int16_t buffer[microphone_array.Channels() + 1]
                [microphone_array.SamplingRate() +
                 microphone_array.NumberOfSamples()];

  // Create an array of streams to write microphone data to files
  std::ofstream os[microphone_array.Channels() + 1];

  // For each microphone channel (+1 for beamforming), make a file and open it
  for (uint16_t c = 0; c < microphone_array.Channels() + 1; c++) {
    // Set filename for microphone output
    std::string filename = "mic_" +
                           std::to_string(microphone_array.SamplingRate()) +
                           "_s16le_channel_" + std::to_string(c) + ".raw";
    // Create and open file
    os[c].open(filename, std::ofstream::binary);
  }

  // Counter variable for tracking recording time
  uint32_t samples = 0;
  // For recording duration
  for (int s = 0; s < seconds_to_record; s++) {
    // Endless loop
    while (true) {
      // Read microphone stream data
      microphone_array.Read();

      // For number of samples
      for (uint32_t s = 0; s < microphone_array.NumberOfSamples(); s++) {
        // For each microphone
        for (uint16_t c = 0; c < microphone_array.Channels(); c++) {
          // Send microphone data to buffer
          buffer[c][samples] = microphone_array.At(s, c);
        }
        // Writes beamformed microphone data into buffer
        buffer[microphone_array.Channels()][samples] = microphone_array.Beam(s);
        // Increment samples for buffer write
        samples++;
      }

      // Once number of samples is >= sampling rate
      if (samples >= microphone_array.SamplingRate()) {
        // For each microphone channel
        for (uint16_t c = 0; c < microphone_array.Channels() + 1; c++) {
          // Write to recording file
          os[c].write((const char *)buffer[c], samples * sizeof(int16_t));
        }
        // Set samples to zero for loop to fill buffer
        samples = 0;
        break;
      }
    }
  }

  return 0;
}
```

</details>

</details>

<details>
<summary style="font-size: 1.75rem; font-weight: 300;">Microphone Array Record to Pipe</summary>
The following section shows how to record data from the microphone array to a linux FIFO pipe. You can download this example <a href="https://github.com/matrix-io/matrix-hal-examples/blob/master/microphone_array/mic_record_pipe.cpp" target="_blank">here</a>.

> When beamformed microphone (channel 8) is read from a FIFO pipe distortion may occur.

The following commands copy a modified `asound.conf` file into `/etc/`, which allows `arecord` to record from the pipe.

```bash
wget https://github.com/matrix-io/matrix-hal-examples/blob/master/microphone_array/asound.conf
sudo mv -f /etc/asound.conf /etc/asound.conf_old
sudo mv -f ./asound.conf /etc/
```

To record from microphone channel 0 for 5 seconds at 16KHz using `arecord`, run these commands. 

```bash
rm -rf /tmp/matrix_micarray_channel_*
./mic_record_pipe --sampling_frequency 16000 &
arecord channel0.wav -f S16_LE -r 16000 -d 5 --device=mic_channel0
```

To stop the example from running, run this command.

```bash
killall mic_record_pipe
```

<details open>
<summary style="font-size: 1.5rem; font-weight: 300;">Include Statements</summary>
To begin working with the Microphone Array you need to include these header files.

```cpp
// Imports FIFO pipe support (https://en.wikipedia.org/wiki/Named_pipe)
#include <sys/stat.h>
// Linux file control options
#include <fcntl.h>
// System calls
#include <unistd.h>
// Google gflags parser
#include <gflags/gflags.h>
// Communicating with Pi GPIO
#include <wiringPi.h>
// Input/output stream class to operate on files
#include <fstream>
// Input/output streams and functions
#include <iostream>
// Use strings
#include <string>
// Arrays for math operations
#include <valarray>

// Communicates with MATRIX device
#include "matrix_hal/matrixio_bus.h"
// Interfaces with microphone array
#include "matrix_hal/microphone_array.h"
// Enables using FIR filter with microphone array
#include "matrix_hal/microphone_core.h"
```

</details>

<details open>
<summary style="font-size: 1.5rem; font-weight: 300;">Initial Variables</summary>
These initial variables are used in the example.

```cpp
// Defines variables from user arguments using gflags utility
// (https://gflags.github.io/gflags/)

// Grabs sampling frequency input from user
DEFINE_int32(sampling_frequency, 16000, "Sampling Frequency");  // Argument example: "--sampling_frequency 48000"
// Grabs gain input from user
DEFINE_int32(gain, -1, "Microphone Gain"); // Argument example: "--gain 5"
```

</details>

<details open>
<summary style="font-size: 1.5rem; font-weight: 300;">Initial Setup</summary>
You'll then need to setup `MatrixIOBus` in order to communicate with the hardware on your MATRIX device. Also, parse command line flags and set user flags as variables.

```cpp
int main(int argc, char *agrv[]) {
  // Parse command line flags with gflags utility
  // (https://gflags.github.io/gflags/)
  google::ParseCommandLineFlags(&argc, &agrv, true);

  // Create MatrixIOBus object for hardware communication
  matrix_hal::MatrixIOBus bus;
  // Initialize bus and exit program if error occurs
  if (!bus.Init()) return false;

  // Set user flags from gflags as variables
  int sampling_rate = FLAGS_sampling_frequency;
  int gain = FLAGS_gain;
```

</details>

<details open>
<summary style="font-size: 1.5rem; font-weight: 300;">Main Setup</summary>
Now we will create our `MicrophoneArray` object and use it to interface with the microphone array.

```cpp
  // The following code is part of main()

  // Create MicrophoneArray object
  matrix_hal::MicrophoneArray microphone_array;
  // Set microphone_array to use MatrixIOBus bus
  microphone_array.Setup(&bus);
  // Set microphone sampling rate
  microphone_array.SetSamplingRate(sampling_rate);
  // If gain is positive, set the gain
  if (gain > 0) microphone_array.SetGain(gain);

  // Log gain_ and sampling_frequency_ variables
  microphone_array.ShowConfiguration();
  // Log recording duration variable
  std::cout << "Duration : " << seconds_to_record << "s" << std::endl;

  // Calculate and set up beamforming delays for beamforming
  microphone_array.CalculateDelays(0, 0, 1000, 320 * 1000);  // These are default values
```

</details>

<details open>
<summary style="font-size: 1.5rem; font-weight: 300;">Fir Filter Setup</summary>
Now we will create our `MicrophoneCore` object and use it to enable the FIR filter.

```cpp
  // The following code is part of main()

  // Create MicrophoneCore object
  matrix_hal::MicrophoneCore microphone_core(microphone_array);
  // Set microphone_core to use MatrixIOBus bus
  microphone_core.Setup(&bus);
```

</details>

<details open>
<summary style="font-size: 1.5rem; font-weight: 300;">Microphone Input</summary>
Now we will read microphone array data, send to a buffer, and write to a FIFO pipe.

```cpp
  // The following code is part of main()

  // Create a buffer array for microphone input
  int16_t buffer[microphone_array.Channels() + 1]
                [microphone_array.SamplingRate() +
                 microphone_array.NumberOfSamples()];

  // For each channel plus the beamforming channel
  for (uint16_t c = 0; c < microphone_array.Channels() + 1; c++) {
    // Name for the FIFO pipe
    std::string name = "/tmp/matrix_micarray_channel_" + std::to_string(c);

    // Create the FIFO pipe
    if (mkfifo(name.c_str(), 0666) != 0) {
      // Output error if mkfifo fails
      std::cerr << "unable to create " << name << " FIFO." << std::endl;
    }
  }

  // For pipe operations
  int named_pipe_handle;
  // Endless loop
  while (true) {
    // Read microphone stream data
    microphone_array.Read();

    // Bool to flag when beamformed written
    bool beam_write = false;

    // For each microphone
    for (uint16_t c = 0; c < microphone_array.Channels() + 1; c++) {
      // Open pipe
      std::string name = "/tmp/matrix_micarray_channel_" + std::to_string(c);
      named_pipe_handle = open(name.c_str(), O_WRONLY | O_NONBLOCK);

      // For number of samples
      for (uint32_t s = 0; s < microphone_array.NumberOfSamples(); s++) {
        buffer[c][s] = microphone_array.At(s, c);
        // If beamformed data was not sent to buffer, send it
        if (!beam_write) {
          // Send beamformed data to buffer
          buffer[microphone_array.Channels()][s] = microphone_array.Beam(s);
        }
      }
      // Flag that beamforming data is in buffer
      beam_write = true;

      // Write to each pipe
      write(named_pipe_handle, &buffer[c][0],
            sizeof(int16_t) * microphone_array.NumberOfSamples());
      // Close pipe after write
      close(named_pipe_handle);
    }
  }

  return 0;
}
```

</details>

</details>