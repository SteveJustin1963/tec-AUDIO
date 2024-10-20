# tec-AUDIO
tec1 audio experiments


### WAV file to MINT+DAC io
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

### 
The conversion of WAV files into MINT-compatible data must be done outside of MINT (e.g., using Python). MINT can then be used to process or play the pre-formatted data, but it does not have the capabilities to extract or manipulate the WAV file directly.
//////////////

To convert a WAV file into MINT code, you'll need to transform the audio data into a format that MINT can handle since it is a minimalist language built for small systems (like the Z80 microprocessor). Here's a step-by-step guide on how you can achieve this:

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

```mint
:WAV-DATA [ 0 127 -128 64 -64 32 -32 16 ... ] ;
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

By following this process, you can convert a WAV file into a MINT-compatible format and write MINT code to manipulate and play the audio data. Let me know if you need any further assistance with specific parts!


  
//////////////////////



### From https://create.arduino.cc/projecthub/projects/tags/audio
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
