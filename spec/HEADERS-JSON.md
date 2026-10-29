# WFDB JSON Header Files

A WFDB header file can be written in one of two syntaxes:

- The **classic** syntax, described in [HEADERS.md](./HEADERS.md).
- The **JSON** syntax, described in this document.

Both syntaxes describe the same record: the same fields, with the same
meaning and the same defaults. The JSON syntax names every field, so
that a header can be read without knowing the position and delimiter
of each field, and can be parsed and validated with standard tools. A
header in either syntax can be converted to the other without losing
information (see [Conversion](#conversion-between-syntaxes)).

The classic syntax remains fully supported. It is more compact, and is
understood by all existing WFDB software.

## File Version History

> This draft specification, called **WFDB JSON Header version 1.0**,
> is under discussion in
> [issue #22](https://github.com/wfdb/wfdb-spec/issues/22).

The version of the JSON syntax is independent of the version of the
classic syntax ("v10").

## File Name and Detection

A JSON header file has the same name as a classic header file:
`<record_name>.hea`. Software finds the header of a record in the same
way, whichever syntax it uses.

To tell the two syntaxes apart, a reader skips an optional UTF-8 byte
order mark (U+FEFF) and any whitespace (space, tab, carriage return
and line feed) at the start of the file. If the next character is `{`,
the file is a JSON header. Otherwise, it is a classic header. A
classic header can never start with `{`, because its first non-empty
line is either a comment (starting with `#`) or a record line
(starting with the record name).

A record's header must use one syntax only. The segments of a
multi-segment record are separate headers, so each may use either
syntax.

## General Rules

- A JSON header file is a single JSON object, as defined by
  [RFC 8259](https://www.rfc-editor.org/rfc/rfc8259), encoded as
  UTF-8. Writers must not write a byte order mark.
- Keys within an object must be unique. Readers may reject a file that
  repeats a key.
- Keys are case-sensitive. The order of keys within an object has no
  meaning. The order of items within an array does: signals are
  numbered, and segments are read, in the order in which they appear.
- An optional field that is missing takes its default value, as given
  in the tables below. A field must not be set to `null` unless this
  document says so.
- The [JSON Schema](https://json-schema.org/) in
  [schemas/header-1.0.schema.json](./schemas/header-1.0.schema.json)
  describes the structure of a version 1.0 header. A header that does
  not match the schema is invalid. Some rules, such as the agreement
  between a multi-segment header and its segments, cannot be expressed
  in the schema and are given only in this document.

### Versioning

Every JSON header contains a `wfdb_header_version` key, which gives
the version of this specification that the file follows, as a string
of the form `"MAJOR.MINOR"`.

- A new **minor** version may add optional keys. Their meaning will
  not change how the keys of earlier minor versions are read. A reader
  that supports version 1.0 can therefore read any version 1.x header,
  and must ignore keys that it does not know.
- A new **major** version may make any change. A reader must refuse a
  header whose major version it does not support.

Each version has its own schema, which accepts only headers of that
version.

### Extension Keys

Any object in a JSON header may contain keys that start with `x-` (for
example, `x-device-serial`). This specification will never define such
keys. They are available for experiments and for data that is specific
to one application. Readers must ignore extension keys that they do
not understand.

Extension keys should not be used for information about the subject
(such as age, sex or diagnosis). That information belongs in a
separate file of the database, where it can be maintained and
de-identified in one place.

## Record Fields

The top-level object describes the record. It contains the following
keys:

| Key                   | Type    | Default              | Notes                                     |
| --------------------- | ------- | -------------------- | ----------------------------------------- |
| `wfdb_header_version` | string  | (required)           | `"1.0"` for this version.                 |
| `record_name`         | string  | (required)           | See below.                                |
| `sampling_frequency`  | number  | 250                  | Samples per second per signal. Above 0.   |
| `counter_frequency`   | number  | `sampling_frequency` | Ticks per second. Above 0.                |
| `base_counter`        | number  | 0                    | Counter value of sample 0.                |
| `samples_per_signal`  | integer | not specified        | See below.                                |
| `start_time`          | string  | not specified        | Time of day of sample 0. See below.       |
| `start_date`          | string  | not specified        | Date of sample 0. See below.              |
| `files`               | array   | `[]`                 | Single-segment records only. See below.   |
| `segments`            | array   | —                    | Multi-segment records only. See below.    |
| `number_of_signals`   | integer | —                    | Multi-segment records only, and required. |
| `comments`            | array   | `[]`                 | Strings. See [Comments](#comments).       |

- **`record_name`** identifies the record, and must match the name of
  the header file without its `.hea` suffix. It contains 1 to 50
  letters, digits, underscores (`_`) and hyphens (`-`).
- **`samples_per_signal`** is the length of the record in frames (the
  number of samples of a signal whose `samples_per_frame` is 1). If it
  is missing or 0, the length is not specified, and signal checksums
  are not verified.
- **`start_time`** has the form `HH:MM:SS`, optionally followed by a
  decimal point and 1 to 6 digits of fractional seconds, using a
  24-hour clock (for example `"13:05:00"` or `"00:31:25.894"`). Hours,
  minutes and seconds always have two digits.
- **`start_date`** has the form `YYYY-MM-DD` (for example
  `"1989-04-25"`). It can be present only if `start_time` is present.

The fields `sampling_frequency`, `counter_frequency`, `base_counter`
and `samples_per_signal` have the same meaning as the corresponding
fields of the classic record line. Unlike the classic syntax, any of
them may be given without the others.

A header contains either `files` (a single-segment record) or
`segments` (a multi-segment record), never both. A header that
contains neither describes a single-segment record with no signals.

## Single-segment Headers

### Files

`files` is an array of signal files. Each item is an object with the
following keys:

| Key           | Type           | Default    | Notes                              |
| ------------- | -------------- | ---------- | ---------------------------------- |
| `name`        | string or null | (required) | Name of the signal file.           |
| `format`      | integer        | (required) | Storage format. See below.         |
| `byte_offset` | integer        | 0          | Offset of sample 0, in bytes.      |
| `block_size`  | integer        | 0          | Block size, in bytes.              |
| `signals`     | array          | (required) | The signals in the file, in order. |

- **`name`** is the name of the file that contains the samples, in the
  same directory as the header. It starts with a letter, digit or
  underscore, and contains only letters, digits, underscores, hyphens
  and periods. It is `null` only in the layout segment of a
  variable-layout record (written `~` in the classic syntax), which
  has no signal files.
- **`format`** is one of the formats described in
  [SIGNALS.md](./SIGNALS.md). All signals in a file are stored in the
  same format.
- **`byte_offset`** and **`block_size`** have the same meaning as the
  corresponding fields of a classic signal line. In the classic
  syntax, they are repeated for every signal in a file and must agree;
  here they are given once per file.
- **`signals`** lists the signals stored in the file, with at least
  one signal. When a file contains several signals, their samples are
  interleaved in this order (see [SIGNALS.md](./SIGNALS.md)).

Each file name appears at most once in `files`, except `null`. Signals
are numbered from 0, in the order in which they appear, starting with
the signals of the first file.

### Signals

Each signal is an object with the following keys, all optional:

| Key                 | Type    | Default       | Notes                               |
| ------------------- | ------- | ------------- | ----------------------------------- |
| `description`       | string  | not specified | Signal name, such as `"MLII"`.      |
| `samples_per_frame` | integer | 1             | At least 1.                         |
| `skew`              | integer | 0             | At least 0.                         |
| `adc_gain`          | number  | uncalibrated  | ADC units per physical unit. Not 0. |
| `baseline`          | integer | `adc_zero`    | ADC units.                          |
| `units`             | string  | `"mV"`        | Physical units, without whitespace. |
| `adc_resolution`    | integer | see below     | Bits. At least 1.                   |
| `adc_zero`          | integer | 0             | ADC units.                          |
| `initial_value`     | integer | `adc_zero`    | Used only by difference formats.    |
| `checksum`          | integer | not specified | 16-bit signed: -32768 to 32767.     |

These keys have the same meaning as the corresponding fields of a
classic signal line, with the following notes:

- **`description`** may contain spaces, but no control characters.
- **`adc_gain`**: if it is missing, the signal is uncalibrated, and
  readers that need a gain use 200 (as for a gain of 0 in the classic
  syntax).
- **`adc_resolution`**: if it is missing, it is 12 bits for amplitude
  formats and 10 bits for difference formats, unless the format allows
  fewer bits.

Unlike the classic syntax, any of these keys may be given without the
others.

## Multi-segment Headers

A multi-segment header contains `segments` and `number_of_signals`,
and no `files`. The meaning of a multi-segment record, and the rules
that its segments must follow, are the same as for the classic syntax
(see [Multi-Segment Headers](./HEADERS.md#multi-segment-headers)).

`number_of_signals` is the number of signals in the record. It must be
the same as in each segment (for a fixed-layout record) or as in the
layout segment (for a variable-layout record).

`segments` is an array of at least one segment. Each segment is an
object with the following keys:

| Key      | Type           | Default    | Notes                                     |
| -------- | -------------- | ---------- | ----------------------------------------- |
| `record` | string or null | (required) | Name of the record of this segment.       |
| `length` | integer        | (required) | Number of samples per signal. At least 0. |

- **`record`** is the name of a single-segment record in the same
  directory, whose header may use either syntax. It is `null` for a
  null segment (written `~` in the classic syntax), which has no
  header or signal files, and whose samples are read as missing
  values.
- **`length`** must match `samples_per_signal` in the header of the
  segment's record. A segment with a length of 0 is the layout segment
  of a variable-layout record, and may only be the first segment.

If `samples_per_signal` is present, it must equal the sum of the
segment lengths.

## Comments

`comments` is an array of strings, each holding the text of one
comment, without the leading `#`. A comment cannot contain line breaks
or other control characters. This is where the information that
classic headers keep in "info strings" is stored. As in the classic
syntax, the content of comments is not defined by this specification.

## Conversion Between Syntaxes

Converting a header from one syntax to the other must give a header
that describes the same record. In particular:

- Every field keeps its value. A field that is missing in one syntax
  is missing (or written with its default value) in the other. A
  converter may leave out a field whose value equals its default.
- Signals keep their order. Consecutive classic signal lines with the
  same file name become one item of `files`. Converting to classic
  writes one signal line per signal, repeating the file's `format`,
  `byte_offset` and `block_size`.
- The number of signals of a single-segment record is not stored in
  the JSON syntax, since it is the total number of signals in `files`.
  Signal or segment lines beyond the number given in a classic record
  line are ignored, and are not converted.
- Times and dates are written in the form required by each syntax. For
  example, the classic time `13:5:0` becomes `"13:05:00"`, and the
  classic date `25/4/1989` becomes `"1989-04-25"`.
- A classic gain of 0 means that the signal is uncalibrated, and
  becomes a missing `adc_gain`. A classic ADC resolution of 0 becomes
  a missing `adc_resolution`. A classic number of samples of 0 becomes
  a missing `samples_per_signal`, except in a layout segment, where it
  is written as 0. In a record whose number of samples is not
  specified, a classic checksum of 0 is a placeholder, and becomes a
  missing `checksum`.
- The classic file name and record name `~` become `null`, and the
  other way around.
- When converting to classic, fields that the classic syntax cannot
  omit (because a later field is present) are written with their
  default values.
- Comments become items of `comments`, in the order in which they
  appear in the file. A converter removes the leading `#` and any
  whitespace that follows it. When converting to classic, each comment
  is written after the signal (or segment) lines, as `#` followed by a
  space and the comment. The position of comments placed between
  classic header lines is not kept.
- Extension keys (`x-...`) cannot be stored in the classic syntax, and
  are lost when converting to it.

## Examples

The [examples](./examples) directory contains each of the following
records in both syntaxes, in the `classic` and `json` directories.

### Example 1: MIT DB Record 100

This is the record described in
[Example 1](./HEADERS.md#example-1-mit-db-record-100) of the classic
header specification.

```json
{
  "wfdb_header_version": "1.0",
  "record_name": "100",
  "sampling_frequency": 360,
  "samples_per_signal": 650000,
  "files": [
    {
      "name": "100.dat",
      "format": 212,
      "signals": [
        {
          "description": "MLII",
          "adc_gain": 200,
          "adc_resolution": 11,
          "adc_zero": 1024,
          "initial_value": 995,
          "checksum": -22131
        },
        {
          "description": "V5",
          "adc_gain": 200,
          "adc_resolution": 11,
          "adc_zero": 1024,
          "initial_value": 1011,
          "checksum": 20052
        }
      ]
    }
  ],
  "comments": ["69 M 1085 1629 x1", "Aldomet, Inderal"]
}
```

The units are not given, and are therefore millivolts. The baselines
are not given, and are therefore equal to the ADC zero (1024).

### Example 2: Record with a Counter Frequency and Several Units

```json
{
  "wfdb_header_version": "1.0",
  "record_name": "12726",
  "sampling_frequency": 250,
  "counter_frequency": 24000,
  "samples_per_signal": 825000,
  "start_time": "15:08:24",
  "files": [
    {
      "name": "12726.dat",
      "format": 16,
      "signals": [
        {
          "description": "ABP",
          "adc_gain": 64.02,
          "baseline": 4,
          "units": "mmHg",
          "adc_resolution": 16,
          "initial_value": 4004,
          "checksum": -26056
        },
        {
          "description": "ECG",
          "adc_gain": 6554,
          "units": "mV",
          "adc_resolution": 16,
          "initial_value": -12996,
          "checksum": -24904
        },
        {
          "description": "Angle",
          "adc_gain": 174.83,
          "baseline": 11204,
          "units": "degrees",
          "adc_resolution": 16,
          "initial_value": 11196,
          "checksum": 11564
        }
      ]
    }
  ],
  "comments": [
    "<age>: 28  <sex>: M  <Height>: 170  <Weight>: 64",
    "Produced by xform from record 12726_orig, beginning at 0:0"
  ]
}
```

The start time is known, but the start date is not.

### Example 3: Variable-layout Multi-segment Record

The top-level header, `multi_var.hea`:

```json
{
  "wfdb_header_version": "1.0",
  "record_name": "multi_var",
  "sampling_frequency": 125,
  "samples_per_signal": 3000,
  "start_time": "00:31:25.894",
  "start_date": "2023-10-10",
  "number_of_signals": 2,
  "segments": [
    { "record": "multi_var_layout", "length": 0 },
    { "record": "multi_var_0001", "length": 1000 },
    { "record": null, "length": 500 },
    { "record": "multi_var_0002", "length": 1500 }
  ]
}
```

The header of its layout segment, `multi_var_layout.hea`:

```json
{
  "wfdb_header_version": "1.0",
  "record_name": "multi_var_layout",
  "sampling_frequency": 125,
  "samples_per_signal": 0,
  "files": [
    {
      "name": null,
      "format": 0,
      "signals": [
        {
          "description": "II",
          "adc_gain": 83,
          "units": "mV",
          "adc_resolution": 15,
          "adc_zero": 0,
          "initial_value": -16384
        },
        {
          "description": "ABP",
          "adc_gain": 1,
          "units": "mmHg",
          "adc_resolution": 9,
          "adc_zero": 0,
          "initial_value": -256
        }
      ]
    }
  ]
}
```

The first segment has a length of 0, so it is the layout segment, and
the record has a variable layout. The third segment is a null segment.
The headers of segments `multi_var_0001` and `multi_var_0002` (not
shown) are ordinary single-segment headers, in either syntax.
