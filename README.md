# Waveform Database Specifications

This repository is used to document specifications for Waveform Database (WFDB) files and concepts. See the [spec](./spec) directory for the formal documentation.

Go to <https://wfdb.io> for an overview of WFDB, along with a list of available WFDB software packages.

## WFDB File Specifications

The specification details a set of [signal](./spec/SIGNALS.md) and [annotation](./spec/ANNOTATIONS.md) file formats, known as "WFDB format."

These file formats were invented along with, and used by, the original WFDB software package. Traditionally, the software package and the specifications were inextricably linked, which has led to ambiguity about what "WFDB format" actually means. Historically, these file formats have also been referred to as "MIT format." This term should NOT be further used.

There are also other file formats that the original WFDB software package supports reading, including [EDF](https://www.edfplus.info/) and AHA. These file formats are NOT part of the WFDB specification, despite being frequently referred to as "WFDB compatible." This term should also NOT be further used.

> Currently, the WFDB specifications are mostly located in the WFDB [Programmer's](https://physionet.org/physiotools/wpg/)/[Application](https://physionet.org/physiotools/wag/wag.htm) guides. We are working to migrate the documentation to this repository in order to separate the spec from the implementation.

## Discussions and Issues

We strongly encourage WFDB users to participate in the [discussions](https://github.com/wfdb/wfdb-spec/discussions). Please use the discussions section for:

- Discussing new ideas and feature requests
- General questions and comments

Examples:

- "WFDB is very difficult to understand as a beginner"
- "Exploration of a new WFDB annotation file format"
- "Exploration of a new WFDB record format"
- "Should I worry about the endianness of my machine when choosing a file format?"

Please use [issues](https://github.com/wfdb/wfdb-spec/issues) for:

- Identifying and tracking bugs in this repository
- Tracking specific tasks and accepted feature requests

Examples:

- "Typo in X README file"
- "Ambiguous description of file format X"
- "Adding description of FLAC format dat file"

We aim to keep issues clean in order to facilitate better task tracking. Please raise issues/discussions about specific implementation packages in their respective repositories rather than here.
