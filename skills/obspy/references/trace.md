# Trace & UTCDateTime — Core time-series data types in ObsPy

## Overview

`UTCDateTime` is ObsPy's universal timestamp type — always use it instead of Python's `datetime` for seismic data. `Trace` is a single contiguous time series paired with a `stats` header (network, station, location, channel, sampling rate, start time). `Stream` is a list-like container of `Trace` objects returned by most read and download operations.

## Quickstart

```python
from obspy import read, UTCDateTime

st = read("waveform.mseed")   # Stream
tr = st[0]                    # first Trace

print(tr.stats.network, tr.stats.station, tr.stats.channel)
print(tr.stats.starttime)     # UTCDateTime
print(tr.data)                # numpy array

t = UTCDateTime("2023-02-06T01:17:35")
t2 = t + 3600                 # add seconds → new UTCDateTime
```

## When to Use

- Creating or parsing timestamps for seismic data.
- Accessing waveform samples and header metadata from a `Trace`.
- Selecting specific traces from a `Stream` by network/station/channel.
- Doing time arithmetic (offsets, durations, comparisons) on seismic timestamps.

## How to Use

### UTCDateTime

All ObsPy times are `UTCDateTime`. Never use Python `datetime` for seismic timestamps.

```python
from obspy import UTCDateTime

t = UTCDateTime("2023-02-06T01:17:35")   # ISO 8601 string
t = UTCDateTime(2023, 2, 6, 1, 17, 35)  # positional args
t = UTCDateTime(1675645055.0)            # POSIX timestamp float
t = UTCDateTime()                        # current UTC time

# Attributes
t.year       # 2023
t.julday     # day-of-year (37)
t.timestamp  # float seconds since epoch

# Arithmetic
t2 = t + 3600       # add seconds → new UTCDateTime
dt = t2 - t         # subtract → float seconds (3600.0)
```

### Stream

`Stream` is a list-like container of `Trace` objects.

```python
from obspy import read

st = read("waveform.mseed")    # returns Stream
print(st)                      # shows SEED IDs and trace count

tr = st[0]                     # index to get a Trace

# Select by SEED identifier
st_z = st.select(network="IU", station="ANMO", channel="BHZ")
```

### Trace

A `Trace` holds a numpy array of samples and a `stats` header.

```python
tr.stats                   # Stats object
tr.stats.network           # 'IU'
tr.stats.station           # 'ANMO'
tr.stats.location          # '00'
tr.stats.channel           # 'BHZ'
tr.stats.sampling_rate     # float Hz
tr.stats.delta             # sample interval in seconds
tr.stats.npts              # number of samples
tr.stats.starttime         # UTCDateTime
tr.stats.endtime           # UTCDateTime (derived)
tr.data                    # numpy.ndarray of sample values
```

Always `copy()` a trace before destructive operations if you need the original:

```python
tr_work = tr.copy()
tr_work.filter("bandpass", freqmin=1.0, freqmax=10.0)
```

## References

- [UTCDateTime API](https://docs.obspy.org/packages/autogen/obspy.core.utcdatetime.UTCDateTime.html)
- [Trace API](https://docs.obspy.org/packages/autogen/obspy.core.trace.Trace.html)
- [Stream API](https://docs.obspy.org/packages/autogen/obspy.core.stream.Stream.html)
