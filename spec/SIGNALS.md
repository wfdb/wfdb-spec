# WFDB Signals

A WFDB signal file stores consecutive samples of digital signal values.

## Storage

A signal file can contain one or more channels of digital data. When a multi-channel signal is multiplexed into a single signal file, the samples for each channel are stored alternately. ie. For a 3 channel signal with 1000 samples per channel, the samples would be stored in the order: V<sub>0,0</sub>, V<sub>0,1</sub>, V<sub>0,2</sub>, V<sub>1,0</sub>, ... , V<sub>999,0</sub>, V<sub>999,1</sub>, V<sub>999,2</sub>

Because of this deterministic storage mechanism, libraries can efficiently seek and read only the relevant file sections, in order to return a desired signal range.

Note: There is a slight caveat for [file formats](#signal-file-formats), such as 212, in which sample values don't fully align on byte boundaries. However, the samples are still stored in the order that they are decoded from the file.

### Multiple Samples/Frame

In certain cases, a channel can contain more than one sample per frame. In this case, the samples within a frame for a single channel are stored consecutively, before the samples for the next channel.

ie. For a two-channel signal, where channel 0 has one sample/frame, and channel 1 has two samples/frame, the values would be stored: V<sub>0,1a</sub>, V<sub>0,1b</sub>, V<sub>0,2</sub>, V<sub>1,1a</sub>, etc.

## File Names

Signal files should, have the `.dat` file extension. Though technically, they are not required to, as long as the file name matches the dat file name specified in the [header file](./HEADERS.md) identifying it.

Signal file names are case sensitive, and should only contain alphanumerics, hyphens, and underscores only.

## Signal File Formats

There are several WFDB signal file formats, and all of them support multiplexed signals.

### Format 8

Each sample is represented as an 8-bit first difference; i.e., to get the value of sample n, sum the first n bytes of the sample data file together with the initial value from the header file. When format 8 files are created, first differences which cannot be represented in 8 bits are represented instead by the largest difference of the appropriate sign (-128 or +127), and subsequent differences are adjusted such that the correct amplitude is obtained as quickly as possible. Thus there may be loss of information if signals in another of the formats listed below are converted to format 8. Note that the first differences stored in multiplexed format 8 files are always determined by subtraction of successive samples from the same signal (otherwise signals with baselines which differ by 128 units or more could not be represented this way).

### Format 16

Each sample is represented by a 16-bit two’s complement amplitude stored least significant byte first. Any unused high-order bits are sign-extended from the most significant bit.

### Format 24

Each sample is represented by a 24-bit two’s complement amplitude stored least significant byte first.

### Format 32

Each sample is represented by a 32-bit two’s complement amplitude stored least significant byte first.

### Format 61

Each sample is represented by a 16-bit two’s complement amplitude stored most significant byte first.

### Format 80

Each sample is represented by an 8-bit amplitude in offset binary form (i.e., 128 must be subtracted from each unsigned byte to obtain a signed 8-bit amplitude).

### Format 160

Each sample is represented by a 16-bit amplitude in offset binary form (i.e., 32,768 must be subtracted from each unsigned byte pair to obtain a signed 16-bit amplitude). As for format 16, the least significant byte of each pair is first.

### Format 212

Each sample is represented by a 12-bit two’s complement amplitude. The first sample is obtained from the 12 least significant bits of the first byte pair (stored least significant byte first). The second sample is formed from the 4 remaining bits of the first byte pair (which are the 4 high bits of the 12-bit sample) and the next byte (which contains the remaining 8 bits of the second sample). The process is repeated for each successive pair of samples.

### Format 310

Each sample is represented by a 10-bit two’s-complement amplitude. The first sample is obtained from the 11 least significant bits of the first byte pair (stored least significant byte first), with the low bit discarded. The second sample comes from the 11 least significant bits of the second byte pair, in the same way as the first. The third sample is formed from the 5 most significant bits of each of the first two byte pairs (those from the first byte pair are the least significant bits of the third sample).

### Format 311

Each sample is represented by a 10-bit two’s-complement amplitude. Three samples are bit-packed into a 32-bit integer as for format 310, but the layout is different. Each set of four bytes is stored in little-endian order (least significant byte first, most significant byte last). The first sample is obtained from the 10 least significant bits of the 32-bit integer, the second is obtained from the next 10 bits, the third from the next 10 bits, and the two most significant bits are unused. This process is repeated for each successive set of three samples.

## Missing Values

TODO(bemoody): Add invalid sample representation for all file formats.
