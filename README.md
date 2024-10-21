# tec-AUDIO
tec1 audio experiments


### WAV file to MINT+DAC
- https://simple.wikipedia.org/wiki/WAV
- https://github.com/SteveJustin1963/tec-MINT
- https://github.com/SteveJustin1963/tec-ADC-DAC


MINT itself does not directly support complex operations such as reading WAV files, manipulating binary data, or interacting with external file systems like Python does. MINT is designed as a minimalist language for small embedded systems, typically lacking file I/O and advanced data manipulation features needed for such a conversion directly within the MINT environment.

### What is Possible with MINT?

MINT can (maybe) handle playback and processing of data that has already been prepared and formatted externally. 
The steps involved would be:

1. **Use an External Tool (like Python)**:
   - Extract and format the audio data from a WAV file using Python or another language that supports file and audio operations. The Python script can output a set of numerical values that represent the audio samples.

2. **Integrate the Prepared Data into MINT**:
   - Once the data is in a format compatible with MINT (like an array of integers), you copy this data into the MINT environment and write playback logic to process and output these values using hardware (e.g., a DAC).

 

### Step 1: Extract the WAV Data Using Python
First, extract the audio data from the WAV file using Python. Since MINT supports only 16-bit integers, you’ll have to scale the WAV data accordingly and output it as a list that MINT can use.

Here’s a Python script to convert WAV audio samples into a format compatible with MINT:

```python
import wave
import struct

def convert_wav_to_mint(wav_filename, output_filename):
    with wave.open(wav_filename, 'rb') as wav_file:
        num_channels = wav_file.getnchannels()
        sample_width = wav_file.getsampwidth()
        sample_rate = wav_file.getframerate()
        num_frames = wav_file.getnframes()
        
        frames = wav_file.readframes(num_frames)
        
        if sample_width == 1:
            format = '{}B'.format(num_frames * num_channels)  # Unsigned 8-bit
        elif sample_width == 2:
            format = '{}h'.format(num_frames * num_channels)  # Signed 16-bit
        else:
            raise ValueError("Unsupported sample width")
        
        samples = struct.unpack(format, frames)

    with open(output_filename, 'w') as f:
        f.write("[ ")
        for i, sample in enumerate(samples):
            if i % 16 == 0:
                f.write("\n")
            f.write(f"{sample} ")
        f.write("\n]")
    print(f"Data converted and saved to {output_filename}")

convert_wav_to_mint('input.wav', 'output.txt')
```

- This script reads a WAV file and extracts the audio samples as signed 16-bit integers, suitable for MINT.
- The samples are saved in an array format compatible with MINT’s syntax.

### Step 2: Import the Data into MINT
The output file (`output.txt`) will look something like this:

```mint
[ 0 127 -128 64 -64 32 -32 16
8 -8 4 -4 2 -2 1 -1 ... ]
```

### Step 3: Implement MINT Code for Playback

Here’s how you can use MINT to play or manipulate the audio data stored in the array:

1. **Define the Array in MINT**: Copy the array data into your MINT program:

make sudo-MINT

```mint
:WAV-DATA [ 0 127 -128 64 -64 32 -32 16 ... ]  ;
```

2. **Create Variables for Playback**: Set up variables to keep track of the current index and other necessary playback parameters:

```mint
VAR I  // Index for accessing the array
VAR SAMPLE_RATE  // Sampling rate (adjust as needed for playback speed)
```

3. **Implement the Playback Loop**: MINT does not directly support audio output, so if you have a connected DAC (Digital-to-Analog Converter), you can use MINT's output capabilities to write values from the array:

```mint
:SAMPLE-RATE 8000 SAMPLE-RATE ! // Set the sample rate

:PLAY-SAMPLE
    I WAV-DATA ? /O          // Output the sample value to DAC port
    I 1 + I !                // Increment index
;

:PLAY-WAV
    0 I !                    // Initialize index to 0
    (                        // Start an unlimited loop
        I WAV-DATA ?         // Fetch the current sample
        0= /W                // Break loop if end of data
        PLAY-SAMPLE          // Play the sample
        SAMPLE-RATE /U /N    // Delay to match the sample rate
    )
;
```

- **`PLAY-SAMPLE`**: Outputs the current sample to a DAC connected to a specific port.
- **`PLAY-WAV`**: Loops through the samples, outputting them and delaying based on the `SAMPLE-RATE` variable.

### Explanation of the Conversion Process

1. **Extract the Data**: We use Python to read and scale the WAV file’s data.
2. **Format the Data**: The extracted samples are formatted into an array suitable for MINT’s syntax.
3. **Playback Logic**: In MINT, we set up a simple loop to read values from the array and send them to an output port (like a DAC). The playback rate is controlled via delays to match the original sample rate.

### Additional Notes:

- **Audio Output**: Ensure that your hardware setup supports sending audio signals to a DAC or another sound output device.
- **Optimization**: MINT is designed for minimalism, so keep the array size and playback code efficient to fit within memory and processing constraints.

Now lets make the real code

###  wav-dat-dac.mint
How this audio playback system works;

The audio playback system in MINT consists of three main functions: F (for recording), G (for playing a single sample), and H (for playback). Here's how it works:

1. Initialization:
   - An array `a` is created to store audio samples.
   - Variable `i` is initialized to 0. This is used as an index for the array.
   - Variable `r` is set to 8000, representing the playback rate in Hz.
   - Variable `p` is set to 0, representing the input port number (this should be adjusted to the correct port number).

2. Recording Function (F):
   - This function reads a single sample from the input port and stores it in the array.
   - `p /I v!`: Reads a value from the input port `p` and stores it in variable `v`.
   - `v i a ?!`: Stores the value from `v` into array `a` at index `i`.
   - `i 1 + i!`: Increments the index `i` for the next sample.

3. Single Sample Playback Function (G):
   - This function plays back a single sample from the array.
   - `i a ? v!`: Fetches the value from array `a` at index `i` and stores it in `v`.
   - `v /O`: Outputs the value in `v` to the output port.
   - `i 1 + i!`: Increments the index `i` for the next playback.

4. Full Playback Function (H):
   - This function loops through the array and plays back all samples.
   - `0 i!`: Initializes the index to 0 to start from the beginning of the array.
   - `/U (...)`: Begins an unlimited loop.
   - Inside the loop:
     - `i a ? v!`: Fetches the current sample from the array.
     - `v 0 = /W`: Exits the loop if the value is 0 (assuming 0 marks the end of recorded data).
     - `G`: Calls function G to play the current sample.
     - `r /N`: Introduces a delay based on the playback rate to control playback speed.

To use this system:

1. First, you would call function F repeatedly to fill the array with audio data from the input port. You might do this in a loop for a specific duration or until a certain condition is met.

2. Once recording is complete, you would call function H to play back the recorded audio.

The system has some limitations:
- It uses a fixed-size array, so the recording length is limited.
- It assumes that a value of 0 marks the end of the recording.
- There's no explicit buffer management, so if you record more samples than the array size, it will overwrite from the beginning.

In a more advanced implementation, you might want to add:
- Dynamic array sizing or circular buffer management.
- Better end-of-recording detection.
- Error handling for array bounds.
- Separate control for recording and playback rates.


  
//////////////////////



### More project ideas 

- VU Meter
- Spectrum Visualizer Analyzer
- Piano / Keyboard
- MIDI Arpeggiator
- Beat Detector
- Synthesizer dig/ana
- drum bot
- Polyphonic Tones
- RGB Matrix Audio Visualizer
- Interactive LED Table
- Musical Instrument Using hand Flicking
- Sing with a Buzzer
- Sound Sensor Activated LEDs with LCD for Sound Level Data
- Cough Detection with TinyML 
- FM Radio 
- AM Radio
- 10Hz-50kHz Oscilloscope LCD Display, 8x8
- Automatic Guitar Tuner
- Auto Voice Record and Playback
- PWM Sound Synthesis
- MIDI Arpeggiator, Sequencer, Recorder and Looper
- Guitar Pedal
- DTMF Decoder Using Only code
- Straight Key Morse Code Oscillator
- Sound Location Finder
- Talking Darth Vader
- I2S Theremin
- Morse Encoder & Displayer
- Ultrasonic Theremin
- Tone Generator With LCD Display
- Musical Note Detector
- Talking Clock
- Audio by using PIR sensor
- DC piezo buzzer volume control
- Voice Activated Media
- Generating Tones of Different Frequencies Using Mathematics
- DoorBell with 48 Melodies
- Listening Temperature" with TinyML
- TuneGlass: Can We Make MUSIC Using Our EYES
- 
- 



### Ref
- https://create.arduino.cc/projecthub/projects/tags/audio
- 
