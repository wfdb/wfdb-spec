# WFDB Header Files

A WFDB record is composed of a header file, and zero or more signal
files. A header file is composed of Unicode-encoded text, and contains
information about the subject (person), the recording, and the signals
recorded.

In some sense, the header file is what defines the record. When
processing a record, the first step is to read the header file and
parse the information.

## File Version History

A specification, called **WFDB Header File version 10**, was
formalized in May 2022. This is based on the information set in the
[WFDB Application Guide](https://www.physionet.org/physiotools/wag/header-5.htm),
and the behavior of the original WFDB software package.

This complete format was first specified in and supported by WFDB
v10.4.0, which was released on 2006-03-02. This format extended the
format released in WFDB v10.0.0, in a backwards compatible manner, by
adding new fields. Thus, a header file adhering to the WFDB v10.0.0
format, which was released on 2001-01-15, also adheres to this new
format.

Therefore, the specification version is named: "v10". In effect, a
vast majority of existing WFDB header files (all those created using
WFDB v10.0.0 and later,) conform to this standard. This document
describes header files according to this standard. Prior formats are
now considered obsolete.

## Header File Types

There are two types records with their own respective header file
types:

1. Ordinary/single-segment records. Their header files contain:
   - A **record line**.
   - A **signal specification line** for each signal.
   - Zero or more **comment lines**.
2. Multi-segment records. Their header files contain:
   - A **record line** (same as above).
   - A **segment specification line** for each segment.

See the section on multi-segment records below for details.

## Single-segment Headers

### Record Line

The first non-empty, non-comment line is the record line. It contains
information applicable to all signals in the record. Its fields are,
from left to right:

- **Record name**:
  - A string of characters that identify the record. The record name
    may include letters, digits and underscores ('\_') only.
  - This is the first field in the record line.
- **Number of segments** [optional]:
  - If the field is present, it indicates that the record is a
    multi-segment record containing the specified number of segments,
    and that the header file contains segment specification lines
    rather than signal specification lines. The number of segments
    must be greater than zero. A value of 1 in this field is legal,
    though unlikely to be useful.
  - This field is separated from the record name field by a '/', not
    by whitespace.
- **Number of signals**:
  - Note that this is not necessarily equal to the number of signal
    files, since two or more signals can share a signal file. This
    number must not be negative; a value of zero is legal, however.
  - This field is separated from the previous field by whitespace.
- **Sampling frequency** (in samples per second per signal)
  [optional]:
  - This number can be expressed in any format legal for scanf(3)
    input of floating point numbers (thus '360', '360.', '360.0', and
    '3.6e2' are all legal and equivalent). The sampling frequency must
    be greater than zero; if it is missing, a value of 250 is assumed.
  - This field is separated from the previous field by whitespace.
- **Counter frequency** (in ticks per second) [optional]:
  - This field is a floating-point number, in the same format as the
    sampling frequency. The sampling and counter frequencies are used
    by strtim to convert strings beginning with ‘c’ into sample
    intervals. Typically, the counter frequency may be derived from an
    analog tape counter, or from page numbers in a chart recording. If
    the counter frequency is absent or not positive, it is assumed to
    be equal to the sampling frequency.
  - It can be present only if the sampling frequency is also present.
    This field is separated from the sampling frequency by a '/', not
    by whitespace.
- **Base counter value** [optional]:
  - A floating-point number that specifies the counter value
    corresponding to sample 0. If absent, it is taken to be zero.
  - This field can be present only if the counter frequency is also
    present. This field is surrounded by parentheses: '()', and is not
    separated from the previous field by whitespace.
- **Number of samples per signal** [optional]:
  - AKA, the signal length, in samples. If it is zero or missing, the
    number of samples is unspecified and checksum verification of the
    signals is disabled.
  - This field can be present only if the sampling frequency is also
    present. This field is separated from the previous field by
    whitespace.
- **Base time** [optional]:
  - Gives the time of day that corresponds to the beginning of the
    record, in `HH:MM:SS` format, using a 24-hour clock. eg.
    `13:05:00` and `13:5:0` both represent 1:05 pm.
  - This field can be present only if the number of samples is also
    present. This field is separated from the previous field by
    whitespace.
- **Base date** [optional]:
  - It contains the date that corresponds to the beginning of the
    record, in `DD/MM/YYYY` format (e.g., `25/4/1989` is 25 April
    1989).
  - This field can be present only if the base time is also present.
    This field is separated from the previous field by whitespace.

## Signal Lines

Each non-empty, non-comment line following the record line in a
single-segment record contains specifications for one signal (aka.
channel), beginning with signal 0. Header files must contain valid
signal specification lines for at least as many signals as were
indicated in the record line. Any extra signal specification lines are
to be disregarded.

From left to right in each line, the fields are:

- **File name**:
  - The name of the file in which samples of the signal are stored.
    Several signals can share the same file. Entries for signals
    contained in a given file must be consecutive. The sum of the
    lengths of the file name and description fields (see below) is
    limited to 80 characters.
  - This is the first field in the signal line.
- **Format**:
  - An integer that specifies the storage format of the signal. All
    signals in a given file are stored in the same format. The
    following three optional fields, if present, are bound to the
    format field (i.e., not separated from it by whitespace); they may
    be considered as format modifiers, since they further describe the
    encoding of samples within the signal file. See the
    [signals documentation](./SIGNALS.md) for the set of WFDB formats.
  - This field is separated from the previous field by whitespace.
- **Samples per frame** [optional]:
  - Normally, all signals in a given record are sampled at the (base)
    sampling frequency as specified in the record line. In this case,
    the number of samples per frame is 1 for all signals, and this
    field is conventionally omitted (which is equivalent to setting it
    to a value of 1). If the signal was sampled at some integer
    multiple, n, of the base sampling frequency, however, each frame
    (set of samples) contains n samples of the signal, and the value
    specified in this field is also n. Non-integer multiples of the
    base sampling frequency are not supported. A common use for this
    field is to enable associating signals with different sampling
    frequencies, with the same record.
  - This field is separated from the previous field by an 'x'.
- **Skew** [optional]:
  - Ideally, within a given record, samples of different signals with
    the same sample number are simultaneous (within one sampling
    interval). If this is not the case (as, for example, when a
    multitrack analog tape recording is digitized and the azimuth of
    the playback head does not match that of the recording head), the
    skew between signals can sometimes determined (for example, by
    locating recorded waveform features with known time relationships,
    such as calibration signals). If this has been done, the skew
    field may be inserted into the header file to indicate the
    (positive) number of samples of the signal that are considered to
    precede sample 0. These samples, if any, are included in the
    checksum, but cannot be returned by getvec or getframe (thus the
    checksum need not be changed if the skew field is inserted or
    modified). WFDB library versions 9.1 and earlier ignore this field
    if it is present; later versions correctly deskew signals in
    accordance with the contents of this field.
  - This field is separated from the previous field by a ':'.
- **Byte offset** [optional]:
  - Normally, signal files include only sample data. If a signal file
    includes a preamble, however, this field specifies the offset in
    bytes, from the beginning of the signal file, to sample 0 (i.e.,
    the length of the preamble). Data within the preamble is not
    included in the signal checksum. Note that the byte offset must be
    the same for all signals within a given group (use the skew field
    to correct for intersignal skew). This feature is provided only to
    simplify the task of reading non-WFDB signal files.
  - This field is separated from the previous field by a '+'.
- **ADC gain** (ADC units per physical unit) [optional]:
  - A floating-point number that specifies the difference in sample
    values that would be observed if a step of one physical unit
    occurred in the original analog signal. For ECGs, the gain is
    usually roughly equal to the R-wave amplitude in a lead that is
    roughly parallel to the mean cardiac electrical axis. If the gain
    is zero or missing, this indicates that the signal amplitude is
    uncalibrated; in such cases, a value of 200 ADC units per physical
    unit is assumed.
  - This field is separated from the previous field by whitespace.
- **Baseline** (ADC units) [optional]:
  - An integer that specifies the sample value corresponding to 0
    physical units. If absent, the baseline is taken to be equal to
    the ADC zero. Note that the baseline need not be a value within
    the ADC range; for example, if the ADC input range corresponds to
    200-300 degrees Kelvin, the baseline is the (extended precision)
    value that would map to 0 degrees Kelvin. WFDB library versions
    5.0 and earlier ignore baseline fields.
  - This field can be present only if the ADC gain is also present.
    This field is surrounded by parentheses: '()', and is not
    separated from the previous field by whitespace.
- **Units** [optional]:
  - A character string without embedded whitespace or ASCII control
    characters, that specifies the type of physical unit. If this
    field is absent, the physical unit may be assumed to be one
    millivolt.
  - This field can be present only if the ADC gain is also present. It
    follows the baseline field if that field is present, or the gain
    field if the baseline field is absent. This field is separated
    from the previous field by a '/', not by whitespace.
- **ADC resolution** (bits) [optional]:
  - It specifies the resolution of the analog-to-digital converter
    used to digitize the signal. Typical ADCs have resolutions between
    8 and 16 bits. If this field is missing or zero, the default value
    is 12 bits for amplitude-format signals, or 10 bits for
    difference-format signals (unless a lower value is specified by
    the format field).
  - This field can be present only if the ADC gain is also present.
    This field is separated from the previous field by whitespace.
- **ADC zero** [optional]:
  - An integer that represents the amplitude (sample value) that would
    be observed if the analog signal present at the ADC inputs had a
    level that fell exactly in the middle of the input range of the
    ADC. For a bipolar ADC, this value is usually zero, but a unipolar
    (offset binary) ADC usually produces a non-zero value in the
    middle of its range. Together with the ADC resolution, the
    contents of this field can be used to determine the range of
    possible sample values. If this field is missing, a value of zero
    is assumed.
  - This field can be present only if the ADC resolution is also
    present. It is separated from the previous field by whitespace.
- Initial value [optional]:
  - Specifies the value of the first sample in the signal, but is used
    only if the signal is stored in difference format. If this field
    is missing, a value equal to the ADC zero is assumed.
  - This field can be present only if the ADC zero is also present.
- **Checksum** [optional]:
  - It is a 16-bit signed checksum of all samples in the signal. (Thus
    the checksum is independent of the storage format.) If the entire
    record is read without skipping samples, and the header’s record
    line specifies the correct number of samples per signal, this
    field is compared against a computed checksum to verify that the
    signal file has not been corrupted. A value of zero may be used as
    a field placeholder if the number of samples is unspecified.
  - This field can be present only if the initial value is also
    present. It is separated from the previous field by whitespace.
- **Block size** [optional]:
  - This field is an integer and is usually zero. If the signal is
    stored in a file that must be read in blocks of a specific size,
    however, this field specifies the block size in bytes. (On UNIX
    systems, this is the case only for character special files,
    corresponding to certain tape and raw disk files. If necessary,
    the block size may be given as a negative number to indicate that
    the associated file lacks I/O driver support for fseek(3)
    operations.) All signals belonging to the same signal group have
    the same block size.
  - This field can be present only if the checksum is present. It is
    separated from the previous field by whitespace.
- **Description** [optional]:
  - AKA, the signal name. Any text, excluding ASCII control
    characters, between the block size field and the end of the line
    is taken to be a description of the signal. Unlike the other
    fields in the header file, the description may include embedded
    spaces; note that whitespace between the block size and
    description fields is not considered to be part of the
    description, however.
  - This field can be present only if the block size is present.

### Comment Lines

Comment lines may appear anywhere in a header file. The first printing
character in a comment line must be '#'. The content of comment lines
(excluding the initial #) that follow the last signal specification
line, are called 'info strings'.

There is no common standard/enforcement of specific data fields in
info strings. Header files in one, or several data sets may share the
similar conventions, but there are no guarantees as comment lines are
optional.

## Multi-Segment Headers

A multi-segment header must begin with a record line, in the same
[format](#record-lines) as that of a single-segment record. The
"number of segments" field must be present and greater than zero.

Each non-empty, non-comment line following the record line in the
top-level header file of a multi-segment record contains
specifications for one segment, beginning with segment 0. Info strings
cannot be used in the top-level header file of a multi-segment record.
Top-level header files must contain valid segment specification lines
for at least as many segments as were indicated in the record line.
Any extra segment specification lines are to be ignored.

A segment is simply an ordinary (single-segment) record, with its own
header and signal files. By including segments in a multi-segment
record, the signals within them can be read by WFDB applications as if
they were continuous signals, beginning with those in segment 0 and
continuing with those in segment 1, with no need for the applications
to do anything special to move from one segment to another. The only
restrictions are that segments cannot themselves contain other
segments (they must be single-segment records), the sampling
frequencies must not change from segment to segment, and the number of
samples per signal must be defined for each segment in the record line
of the segment’s own header file.

Two types of multi-segment records are defined.

1. In a **fixed-layout** record, the arrangement of signals is
   constant across all segments, and the signal gain, baseline, units,
   ADC resolution and zero, and description match for corresponding
   signals in all segments. Note, however, that it is not necessary to
   use the same signal storage format in all segments, and significant
   space savings may be possible in some cases by selecting an optimal
   format for each segment. Each segment of a fixed-layout record is
   an ordinary record containing one or more samples.
2. In a **variable-layout** record, the arrangement of signals may
   vary, signals may be absent in some segments, and the gains and
   baselines may change between segments. A variable-layout record can
   be identified by the presence of a layout segment, which must be
   segment 0 and must have a length of 0 samples. The layout segment
   has no associated signal files; its header file specifies the
   desired arrangement of signals and their gains and baselines.
   Signal file names in a layout segment header are recorded as '~'.

### Segment Lines

Each segment specification line contains the following fields:

- **Record name**:
  - A string of characters identifying the single-segment record that
    comprises the segment. As in the record line, the record name may
    include letters, digits, and underscores ('\_') only.
- **Number of samples per signal**:
  - This number must match the number specified in the header file for
    the single-segment record that comprises the segment.
    Variable-layout records may contain null segments, which can be
    identified if the record name given in the segment specification
    line is '~'. The number of samples per signal indicates the length
    of the null segment; when read, these samples have a NaN value.
    Null segments do not have associated header or signal files.
  - This field is separated from the previous field by whitespace.

## Example Header Files

### Example 1: MIT DB Record 100

```txt
100 2 360 650000
100.dat 212 200 11 1024 995 -22131 0 MLII

100.dat 212 200 11 1024 1011 20052 0 V5

# 69 M 1085 1629 x1
# Aldomet, Inderal
```

This header specifies 2 signals each sampled at 360 Hz, each 650000
samples (slightly over 30 minutes) long. The starting time and date
were not recorded.

Both signals are stored in the same format 212 signal file: 100.dat.
The gain for each signal was the (default) 200 ADC units per millivolt
(the default physical unit), and the ADC had 11-bit resolution and an
offset such that its output was 1024 ADC units given an input exactly
in the middle of its range.

The baseline is not given explicitly, but may be assumed to be equal
to the ADC zero value of 1024. The first samples acquired had values
of 995 and 1011 (i.e., both signals began slightly below 0 VDC). The
checksums of the 650000 samples are -22131 and 20052, and I/O may be
performed in blocks of any desired size, since the block size fields
are zero. The signal descriptions specify which leads were used: MLII
and V5.

Finally, the last two lines contain info strings. The first info
string specifies the sex and age of the subject and data about the
recording, and the second lists the subject’s medications.

### Example 2: Modified AHA DB Record 7001

```txt
7001 2 250 525000
d0.7001 8 100 10 0 -53 -1279 0 ECG signal 0
d1.7001 8 100 10 0 -69 15626 0 ECG signal 1
```

Each signal is kept in its own signal file, specified by its filename.
The signals are kept in 8-bit first difference format, but the
sampling rate requires that the signals be scaled down (from 12-bit to
10-bit ADC resolution) to stay within the slew rate limits imposed by
the format. Note that signal checksums (-1279 and 15626 in this
example) are derived from the reconstructed sample values, and not
from the first differences; thus they should not change if the signals
are reformatted.

### Example 3: Multi-segment Header File

```txt
multi/3 2 360 45000
100s 21600
null 1800
100s 21600
```

This header file is a sample of a multi-segment record. The first line
contains the record name ("multi"), the number of segments (there are
3), the number of signals (2; this must be the same in each segment),
the sampling frequency (360), and the total length of the record in
sample intervals (45000; this must be the sum of the segment lengths).

The second line contains the record name ("100s") of the first segment
of the record, and its length in sample intervals (21600). The third
and fourth lines contain the record names and lengths of the remaining
segments.

Note that a segment may appear more than once in a multi-segment
record, as in this sample, and that storage formats may vary between
segments. The second segment is a null segment of length 1800.

## FAQs

### General

Q: Can a record have zero signals?

A: Yes. For example, there may be a WFDB header file with zero
signals, and a set of associated annotation files.

### Multi-segment Records

Q: What does having a "number of segments" field value of 1 mean?

A: It means that the header file is a multi-segment header, with one
segment. And as a multi-segment header, the file would have to contain
one segment specification line, and no signal specification lines.
This would not be very useful, but it is valid. A single-segment
header file should NOT set this field, even to 1.

### Multiple sampling frequencies

Q: How can one represent a record with multiple sampling frequencies?

A: Use the frames per sample field. For example, to associate two
signals together, with individual sampling frequences of 50Hz and
100Hz, set the sampling frequency of the record to 50, and set the
frames per sample field of the second signal, to 2.

In the case where two signals have an inconvenient lowest common
multiple, such as a signal with fs=100, and another with fs=101, WFDB
files currently do not offer a good solution to associate such signals
with the same record/header.
