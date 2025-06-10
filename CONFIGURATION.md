# Configuration File Format (.cfg)

This document describes the text based configuration files parsed by
`ConfigFileParser_createModelFromConfigFileEx`. These files define the
IED data model that the server loads at runtime.

Each configuration file contains nested blocks that create logical
nodes, data objects, data attributes, data sets and report control
blocks. The syntax closely matches the model hierarchy and is usually
generated from SCL (ICD) files using `genconfig.jar`.

## General Structure

```
MODEL(<ied-name>) {
    LD(<ld-name>) {
        LN(<ln-name>) {
            ...
        }
    }
}
```

Within a logical node the following blocks can appear:

- `DO` – Data object
- `DA` – Data attribute
- `DS` – Data set
- `RC` – Report control block

Data sets contain `DE` entries referring to FCDAs.

### Number Codes

Several parameters use numeric codes. The values are defined by the
library headers.

**DataAttributeType** (excerpt from
`iec61850_model.h`):
```c
89  IEC61850_BOOLEAN = 0
90  IEC61850_INT8 = 1
... (see file for full list)
112 IEC61850_UNICODE_STRING_255 = 21
113 IEC61850_TIMESTAMP = 22
114 IEC61850_QUALITY = 23
```
See lines 90‑123 of `src/iec61850/inc/iec61850_model.h` for all codes.

**FunctionalConstraint** values (from
`iec61850_common.h`):
```c
266 IEC61850_FC_ST = 0  // status information
268 IEC61850_FC_MX = 1  // measurands
270 IEC61850_FC_SP = 2  // setpoint
... (see file for rest)
302 IEC61850_FC_GO = 18
```
See lines 264‑306 of
`src/iec61850/inc/iec61850_common.h` for the enumeration.

**Trigger options** used by `DA` and `RC` (from `iec61850_common.h`):
```c
97  TRG_OPT_DATA_CHANGED   = 1
100 TRG_OPT_QUALITY_CHANGED = 2
103 TRG_OPT_DATA_UPDATE    = 4
106 TRG_OPT_INTEGRITY      = 8
109 TRG_OPT_GI            = 16
112 TRG_OPT_TRANSIENT    = 128
```

**Report options** (bit mask for the `options` field of `RC`):
```c
124 RPT_OPT_SEQ_NUM         = 1
127 RPT_OPT_TIME_STAMP      = 2
130 RPT_OPT_REASON_FOR_INCLUSION = 4
133 RPT_OPT_DATA_SET        = 8
136 RPT_OPT_DATA_REFERENCE  = 16
139 RPT_OPT_BUFFER_OVERFLOW = 32
142 RPT_OPT_ENTRY_ID        = 64
145 RPT_OPT_CONF_REV        = 128
```
See lines 120‑146 of `src/iec61850/inc/iec61850_common.h`.

## Block Parameters

### LN – Logical Node
```
LN(<ln-name>) {
    ...
}
```
Creates a logical node with the given name. No further parameters.

### DO – Data Object
```
DO(<name> <arrayCount>) {
    ...
}
```
`arrayCount` specifies the number of elements if the data object is an
array (0 for single objects).

### DA – Data Attribute
```
DA(<name> <arrayCount> <type> <FC> <trgOps> <sAddr>) [=value];
```
Parameters:
- **name** – attribute name.
- **arrayCount** – number of elements for array attributes (0 for scalar).
- **type** – numeric `DataAttributeType` value.
- **FC** – numeric `FunctionalConstraint`.
- **trgOps** – trigger options bit mask (`TRG_OPT_*`).
- **sAddr** – short address (0 if unused).

An optional default value can follow after `=` (for primitive types or
strings). When the attribute type is a constructed type, nested `DA`
blocks can be placed inside.

### DS – Data Set
```
DS(<name>) {
    DE(...)
    ...
}
```
Creates a data set under the current logical node. The block contains
one or more `DE` entries.

### DE – Data Set Entry
```
DE(<variable> [index [component]]);
```
Adds an FCDA reference to the current data set. `variable` uses `$` as
separator (and may optionally include the logical device name followed
by `/`). `index` specifies the array element if the target is an array
(-1 or omitted for none). `component` allows addressing a subelement of
an array item.

Example:
```
DE(GGIO1$ST$SPCSO1$stVal);
DE(LD1/LLN0$MX$TmpSv 0 instMag);
```

### RC – Report Control Block
```
RC(<name> <rptId> <buffered> <dataSet> <confRev> <trgOps> <options> <bufTm> <intgPd>);
```
Parameters:
- **name** – object name of the RCB.
- **rptId** – value for `RptID` (use `-` if not set).
- **buffered** – `1` for buffered, `0` for unbuffered.
- **dataSet** – name of the data set (use `-` if none).
- **confRev** – configuration revision.
- **trgOps** – trigger options bit mask.
- **options** – report inclusion options (`RPT_OPT_*`).
- **bufTm** – buffering time in milliseconds.
- **intgPd** – integrity period in milliseconds.

## Example Snippet
From `examples/server_example_config_file/model.cfg`:
```
RC(EventsRCB01 Events 0 Events 1 24 175 50 1000);
```
Here `trgOps` is `24` (`TRG_OPT_INTEGRITY | TRG_OPT_GI`) and
`options` is `175` (e.g. sequence number, timestamp, dataset, entry id and
confRev).

This document should help interpreting the numeric parameters when
manually editing configuration files.
