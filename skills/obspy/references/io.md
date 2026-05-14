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

| Subsection | Description |
|---|---|
| [Reading and Writing Local Files](#reading-and-writing-local-files) | Read any supported format from disk or URL; write back to MiniSEED, SAC, and others. |
| [Downloading from FDSN Web Services](#downloading-from-fdsn-web-services) | Fetch waveforms, station metadata, and earthquake catalogs from IRIS, GEOFON, ORFEUS, and other FDSN centers. |
| [Handling Gaps](#handling-gaps) | Inspect, merge, or fill data gaps and overlaps in a Stream. |
| [Full Processing Pipeline](#full-processing-pipeline) | End-to-end example: download → detrend → taper → remove response → filter → plot. |
| [Trimming a Window Around an Arrival](#trimming-a-window-around-an-arrival) | Cut a Stream to a time window relative to an origin or arrival time. |
| [Downloading from Earthworm Wave Server](#downloading-from-earthworm-wave-server) | Retrieve near-real-time waveforms directly from an Earthworm Wave Server by host and port. |
| [Reading from a SeisComP Data Structure (SDS) Archive](#reading-from-a-seiscomp-data-structure-sds-archive) | Read waveforms from a local SDS archive (SeisComP layout) with availability and latency helpers. |

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

### Downloading from Earthworm Wave Server

The `obspy.clients.earthworm.Client` connects to an Earthworm Wave Server (Winston or standard) to retrieve real-time or archived waveform data via the Winston Wave Server protocol.

```python
from obspy import UTCDateTime
from obspy.clients.earthworm import Client

client = Client("pubavo1.wr.usgs.gov", 16022)

t1 = UTCDateTime("2010-03-01T00:00:00")
t2 = t1 + 30   # 30-second window

# Get waveforms — same signature as FDSN client
st = client.get_waveforms("AV", "ACH", "", "EHZ", t1, t2)
st.plot()
```

Available methods mirror the FDSN client:

```python
# List available streams on the server
streams = client.get_availability()
for s in streams:
    print(s)   # (network, station, location, channel, start, end)

# Save to MiniSEED
st.write("output.mseed", format="MSEED")
```

Key differences from the FDSN client:
- Connects directly to a Wave Server host and port (no data center name string).
- No `get_stations()` or `get_events()` — metadata must come from another source (e.g., StationXML via FDSN).
- Suited for near-real-time acquisition from Earthworm-based networks (volcanoes, regional arrays).

### Reading from a SeisComP Data Structure (SDS) Archive

`obspy.clients.filesystem.sds.Client` reads waveforms directly from a local SDS archive — the on-disk layout used by SeisComP and many regional networks.

**Directory layout:**

```
<sds_root>/
  YEAR/
    NET/
      STA/
        CHAN.TYPE/
          NET.STA.LOC.CHAN.TYPE.YEAR.DAY
```

Example path: `/archive/2023/IU/ANMO/BHZ.D/IU.ANMO.00.BHZ.D.2023.037`

Type codes: `D`=Waveform data (default), `E`=Detection, `L`=Log, `T`=Timing, `C`=Calibration, `R`=Response, `O`=Opaque.

**Constructor:**

```python
from obspy.clients.filesystem.sds import Client

client = Client(
    sds_root="/path/to/sds/archive",
    sds_type="D",        # data type; supports wildcards '?' and '*'
    format="MSEED",      # set to None for auto-detection
    fileborder_seconds=30,    # check adjacent day files for spillover
    fileborder_samples=5000,
)
```

**Retrieve waveforms:**

```python
from obspy import UTCDateTime

t1 = UTCDateTime("2023-02-06T01:17:35")
t2 = t1 + 600

st = client.get_waveforms("IU", "ANMO", "00", "BH?", t1, t2)
st.plot()
```

Wildcards (`?`, `*`) are supported in all NSLC fields.

**Bulk retrieval:**

```python
requests = [
    ("IU", "ANMO", "00", "BHZ", t1, t2),
    ("IU", "COLA", "00", "BHZ", t1, t2),
]
st = client.get_waveforms_bulk(requests)
```

**Inspect archive contents:**

```python
# All available (network, station, location, channel) tuples
nslc = client.get_all_nslc()
for n, s, l, c in nslc:
    print(n, s, l, c)

# All available (network, station) tuples
stations = client.get_all_stations()

# Check whether data exists for a stream
exists = client.has_data("IU", "ANMO", "00", "BHZ", t1, t2)

# Data availability fraction (0.0–1.0) and gap count
fraction, ngaps = client.get_availability_percentage("IU", "ANMO", "00", "BHZ", t1, t2)

# Latency — seconds since the most recent sample
latency = client.get_latency("IU", "ANMO", "00", "BHZ")
```

Key differences from the FDSN client:
- Reads from a **local directory tree** — no network access required.
- No `get_stations()` or `get_events()` — use an FDSN client or local StationXML for metadata.
- Suited for processing archived data from SeisComP-managed networks.

## References

- [obspy.read](https://docs.obspy.org/packages/autogen/obspy.core.stream.read.html)
- [FDSN Client API](https://docs.obspy.org/packages/obspy.clients.fdsn.html)
- [Earthworm Wave Server Client](https://docs.obspy.org/packages/obspy.clients.earthworm.html)
- [SDS Client API](https://docs.obspy.org/packages/obspy.clients.filesystem.sds.html)
- [Supported formats](https://docs.obspy.org/packages/autogen/obspy.core.stream.read.html#supported-formats)
