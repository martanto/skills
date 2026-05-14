# Reading, Writing & Downloading — I/O and FDSN web service access

## Overview

ObsPy can read and write 25+ seismic file formats (MiniSEED, SAC, SEG-Y, StationXML, QuakeML, …) with automatic format detection. The `obspy.clients.fdsn.Client` provides a unified interface to download waveforms, station metadata, and earthquake catalogs from IRIS/EarthScope, GEOFON, ORFEUS, and other FDSN-compliant data centers.

## Quickstart

```python
from obspy import read, UTCDateTime
from obspy.clients.fdsn import Client

# Local file
st = read("data/example.mseed")

# From FDSN web service
client = Client("IRIS")
t1 = UTCDateTime("2023-02-06T01:17:35")
st  = client.get_waveforms("IU", "ANMO", "00", "BH?", t1, t1 + 600)
inv = client.get_stations("IU", "ANMO", location="00", channel="BH?",
                          level="response", starttime=t1, endtime=t1 + 600)
```

## When to Use

- Reading local seismic files in any supported format.
- Writing processed waveforms back to MiniSEED, SAC, or other formats.
- Downloading waveforms, station metadata, or event catalogs from IRIS/GEOFON/ORFEUS.
- Building an end-to-end download → process pipeline.

## How to Use

### Reading and Writing Local Files

```python
from obspy import read

# Format is auto-detected from file contents
st = read("data/example.mseed")
st = read("https://examples.obspy.org/RJOB_061005_072159.ehz.new")   # remote URL

# Write Stream to MiniSEED
st.write("output.mseed", format="MSEED")

# Write one SAC file per Trace
for tr in st:
    tr.write(f"{tr.id}.sac", format="SAC")
```

Common format strings: `"MSEED"`, `"SAC"`, `"SEGY"`, `"GSE2"`, `"WAV"`, `"ASCII"`.

### Downloading from FDSN Web Services

```python
from obspy import UTCDateTime
from obspy.clients.fdsn import Client

client = Client("IRIS")   # other centers: "GEOFON", "ORFEUS", "ETH", "BGSU", …

t1 = UTCDateTime("2023-02-06T01:17:35")
t2 = t1 + 600   # 10-minute window

# Waveforms
st = client.get_waveforms(
    network="IU", station="ANMO", location="00", channel="BH?",
    starttime=t1, endtime=t2
)

# Station metadata — use level="response" for instrument correction
inv = client.get_stations(
    network="IU", station="ANMO", location="00", channel="BH?",
    level="response", starttime=t1, endtime=t2
)

# Earthquake catalog
cat = client.get_events(
    starttime=UTCDateTime("2023-02-06"),
    endtime=UTCDateTime("2023-02-07"),
    minmagnitude=6.0
)
print(cat)
```

Wildcards: `?` matches one character, `*` matches any sequence. `channel="BH?"` retrieves all three broadband components.

### Handling Gaps

```python
print(st.get_gaps())                      # inspect gaps/overlaps
st.merge(fill_value="interpolate")        # fill gaps by interpolation
st.merge(fill_value=0)                    # fill with zeros
```

### Full Processing Pipeline

```python
from obspy import UTCDateTime
from obspy.clients.fdsn import Client

client = Client("IRIS")
t1 = UTCDateTime("2023-02-06T01:17:35")
t2 = t1 + 600

st  = client.get_waveforms("IU", "ANMO", "00", "BH?", t1, t2)
inv = client.get_stations("IU", "ANMO", location="00", channel="BH?",
                          level="response", starttime=t1, endtime=t2)

st.detrend("demean")
st.detrend("linear")
st.taper(max_percentage=0.05)
st.remove_response(inventory=inv, output="VEL",
                   pre_filt=(0.005, 0.006, 30.0, 35.0))
st.filter("bandpass", freqmin=0.05, freqmax=0.5, zerophase=True)
st.plot()
```

### Trimming a Window Around an Arrival

```python
origin_time = UTCDateTime("2023-02-06T01:17:35")
st.trim(origin_time - 60, origin_time + 300)
```

## References

- [obspy.read](https://docs.obspy.org/packages/autogen/obspy.core.stream.read.html)
- [FDSN Client API](https://docs.obspy.org/packages/obspy.clients.fdsn.html)
- [Supported formats](https://docs.obspy.org/packages/autogen/obspy.core.stream.read.html#supported-formats)
