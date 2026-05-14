---
name: obspy
description: Write, debug, and explain Python code using the ObsPy seismology framework. Use whenever the user asks to read/write seismic waveforms, process seismograms, remove instrument response, query FDSN/IRIS web services, work with earthquakes catalogs or station inventories, or perform any seismological signal processing with ObsPy.
metadata:
  skill-author: Martanto
---

# ObsPy Skill

Write correct, idiomatic Python code using [ObsPy](https://docs.obspy.org/) — the open-source framework for processing seismological data.

## Overview

ObsPy provides:

- **Core data types**: `Stream`, `Trace`, `Inventory`, `Catalog`, `Event`, `UTCDateTime`
- **File I/O**: read/write 25+ seismic formats (MiniSEED, SAC, SEG-Y, GSE2, StationXML, QuakeML, …)
- **Web clients**: FDSN, IRIS, SeedLink, Earthworm, Syngine
- **Signal processing**: filtering, tapering, resampling, instrument correction, triggering, beamforming
- **Visualization**: waveform plots, spectrograms, beachballs, ray paths

## When to Use This Skill

Use this skill when the user is:

- Asking to use ObsPy or run seismic-related data processing
- Reading, writing, or converting seismic waveform files
- Querying FDSN/IRIS/GEOFON web services for waveforms, stations, or events
- Filtering, resampling, tapering, or detrending seismograms
- Removing instrument response or simulating seismometers
- Working with `Stream`, `Trace`, `Inventory`, `Catalog`, or `UTCDateTime` objects
- Computing travel times with TauP
- Plotting waveforms, spectrograms, or beachballs

Do not use this skill when the user is working with non-seismological geophysical data (GPS, gravity, MT) or when they need a general signal-processing library without seismological context.

## Requirements

- Python **3.8+**
- Core dependencies (installed automatically with ObsPy):
  - `numpy`, `scipy`, `matplotlib` — numerics and plotting
  - `lxml` — XML parsing for StationXML / QuakeML
  - `requests` — HTTP for FDSN web clients
  - `decorator`, `sqlalchemy` — internal utilities
- Optional but recommended:
  - `cartopy` — geographic map plots (ray paths, station maps)

## Installation

**uv** (recommended):

```bash
uv add obspy
```

**pip**:

```bash
pip install obspy
```

**conda** (conda-forge channel):

```bash
conda install -c conda-forge obspy
```

**Important:** `conda-forge` is the only maintained conda channel for ObsPy. Do not use the default Anaconda channel.

## Quick Start

```python
from obspy import UTCDateTime
from obspy.clients.fdsn import Client

client = Client("IRIS")
t1 = UTCDateTime("2023-02-06T01:17:35")
t2 = t1 + 600  # 10-minute window

st  = client.get_waveforms("IU", "ANMO", "00", "BH?", t1, t2)
inv = client.get_stations("IU", "ANMO", location="00", channel="BH?",
                          level="response", starttime=t1, endtime=t2)

st.detrend("demean")
st.taper(max_percentage=0.05)
st.remove_response(inventory=inv, output="VEL",
                   pre_filt=(0.005, 0.006, 30.0, 35.0))
st.filter("bandpass", freqmin=0.05, freqmax=0.5, zerophase=True)
st.plot()
```

## Progressive Disclosure

- Start with this file for installation, a working example, and trigger guidance.
- Use `references/trace.md` when working with `UTCDateTime`, `Trace`, or `Stream` attributes and time arithmetic.
- Use `references/stream.md` for Stream methods, `Inventory`, `Catalog`, or visualization.
- Use `references/processing.md` for filtering, response removal, resampling, PAZ simulation, or TauP.
- Use `references/io.md` for local file I/O, FDSN downloads, gap handling, or the full pipeline pattern.

## Reference Files

Detailed information available in `references/`:

- `trace.md`: `UTCDateTime`, `Trace`, and `Stream` — core data types and time arithmetic.
- `stream.md`: Stream methods, `Inventory`, `Catalog`, and visualization.
- `processing.md`: Filtering, tapering, detrending, resampling, instrument response removal, PAZ simulation, and TauP travel times.
- `io.md`: Local file I/O, FDSN web services, gap handling, and full pipeline pattern.

## Additional Resources

- Full API: https://docs.obspy.org/packages/index.html
- Tutorial: https://docs.obspy.org/tutorial/index.html
