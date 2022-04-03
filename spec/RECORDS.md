# WFDB Records

A WFDB record contains a continuous recording from a single subject.

## Naming

Records are identified by record names of up to 50 characters. A record name is case-sensitive, and must only contain letters, digits, and underscores.

## Content

There is no single 'record file'. A record is comprised of potentially several files:

- One header file. The `record name` field is specified in the header file, and the file itself must be named: `<record_name>.hea`. See the [headers page](./HEADERS.md) for details.
- Zero or more signal files. The names of the signal files are specified in the header file. The first part of each file name is not required to, but does conventionally match the record name. ie. `<record_name>.dat` or `<record_name>_1.dat`. See the [signals page](./SIGNALS.md) for details.

A record may also have zero or more associated annotation files. However, the header file does not have any structured fields which specify associated annotation files, therefore the annotations are not considered _part of_ the record. Annotation files are identified to be associated with a record primarily by having the first parts of their file names fully matching that of the record. ie. `<record_name>.qrs`. See the [annotations page](./ANNOTATIONS) for details.

## Examples

Record names in the [MIT DB](https://physionet.org/content/mitdb/1.0.0/#files) are three-digit numbers. For instance, record `100` has a header file: [`100.hea`](https://physionet.org/content/mitdb/1.0.0/100.hea). The header specifies that the dat file `100.dat` contains the data of its two signal channels.

The annotation files, `100.xws` and `100.atr` contain annotations of the record.
