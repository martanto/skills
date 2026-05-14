# Stream, Inventory & Catalog — Container types for seismic metadata and events

## Overview

Beyond raw waveforms, ObsPy provides `Inventory` (station metadata / instrument response from StationXML) and `Catalog` / `Event` (earthquake metadata from QuakeML). This document also covers the full set of `Stream` methods for merging, trimming, and writing data, and ObsPy's built-in visualization tools.

## Quickstart

```python
from obspy import read, read_inventory, read_events

st  = read("waveform.mseed")
inv = read_inventory("station.xml")
cat = read_events("catalog.xml")

st.merge()
st.trim(st[0].stats.starttime, st[0].stats.starttime + 300)
st.plot()

for event in cat:
    origin = event.preferred_origin() or event.origins[0]
    print(origin.time, origin.latitude, origin.longitude)
```

## When to Use

- Calling `Stream` methods to merge gaps, trim windows, or write files.
- Reading or iterating over station inventories (coordinates, response info).
- Attaching instrument response to a `Stream` before `remove_response()`.
- Reading earthquake catalogs and accessing origin/magnitude information.
- Plotting waveforms, spectrograms, beachballs, or ray paths.

## How to Use

### Stream Methods

| Method | Purpose |
|---|---|
| `st.merge()` | Merge traces with the same SEED ID |
| `st.trim(t1, t2)` | Cut all traces to a time window (in-place) |
| `st.slice(t1, t2)` | Return new Stream within window (non-destructive) |
| `st.split()` | Split masked/gapped traces into contiguous segments |
| `st.get_gaps()` | List gaps and overlaps |
| `st.copy()` | Deep copy of the Stream |
| `st.write("out.mseed", format="MSEED")` | Write to file |
| `st.plot()` | Quick waveform plot |
| `st.select(network, station, location, channel)` | Filter traces by SEED identifier |

### Working with Inventories

```python
from obspy import read_inventory, UTCDateTime

inv = read_inventory("station.xml")
print(inv)

# Iterate hierarchy
for network in inv:
    for station in network:
        print(station.code, station.latitude, station.longitude)

# Select subset
inv_sel = inv.select(network="IU", station="ANMO", channel="BHZ")

# Get response object for a specific channel and time
response = inv.get_response("IU.ANMO.00.BHZ", UTCDateTime("2023-02-06"))

# Attach response to Stream (required before remove_response)
st.attach_response(inv)
```

### Working with Event Catalogs

```python
from obspy import read_events

cat = read_events("catalog.xml")   # read QuakeML
print(cat)

for event in cat:
    origin = event.preferred_origin() or event.origins[0]
    mag    = event.preferred_magnitude() or event.magnitudes[0]
    print(origin.time, origin.latitude, origin.longitude, mag.mag)
```

### Visualization

```python
# Waveform plot to screen or file
st.plot()
st.plot(outfile="waveform.png")

# Spectrogram of a single Trace
tr.spectrogram()

# Beachball (focal mechanism)
from obspy.imaging.beachball import beachball
beachball([235, 80, 35], size=200, linewidth=2, facecolor="b")

# Travel-time ray path plot (requires TauP arrivals — see processing.md)
arrivals.plot_rays()
```

## References

- [Stream API](https://docs.obspy.org/packages/autogen/obspy.core.stream.Stream.html)
- [Inventory API](https://docs.obspy.org/packages/autogen/obspy.core.inventory.inventory.Inventory.html)
- [Catalog API](https://docs.obspy.org/packages/autogen/obspy.core.event.catalog.Catalog.html)
- [obspy.imaging](https://docs.obspy.org/packages/obspy.imaging.html)
