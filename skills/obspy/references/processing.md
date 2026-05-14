# Signal Processing & Travel Times — Filtering, response removal, and TauP

## Overview

ObsPy provides a complete seismological signal-processing pipeline: detrending, tapering, filtering, resampling, and instrument response removal via deconvolution. For travel-time calculation and ray paths, the built-in TauP wrapper supports standard Earth models (IASP91, AK135, PREM).

## Quickstart

```python
# Basic processing pipeline
st.detrend("demean")
st.taper(max_percentage=0.05)
st.filter("bandpass", freqmin=1.0, freqmax=10.0, corners=4, zerophase=True)

# Remove instrument response
pre_filt = (0.005, 0.006, 30.0, 35.0)
st.remove_response(inventory=inv, output="VEL", pre_filt=pre_filt)

# Travel times
from obspy.taup import TauPyModel
model = TauPyModel(model="iasp91")
arrivals = model.get_travel_times(source_depth_in_km=35.0, distance_in_degree=45.0)
for arr in arrivals:
    print(arr.name, arr.time)
```

## When to Use

- Removing the mean, trend, or linear drift from seismograms.
- Applying cosine tapers before FFT-based operations.
- Bandpass, lowpass, or highpass filtering waveforms.
- Resampling or decimating data to a target sample rate.
- Removing instrument response to get displacement, velocity, or acceleration.
- Simulating a target seismometer response using poles-and-zeros (PAZ).
- Computing theoretical phase arrival times or ray paths with TauP.

## How to Use

| Subsection | Description |
|---|---|
| [Detrending and Tapering](#detrending-and-tapering) | Remove mean, linear trend, and apply cosine tapers before filtering or response removal. |
| [Filtering](#filtering) | Bandpass, lowpass, and highpass filtering with zero-phase support. |
| [Resampling](#resampling) | Change sample rate via Fourier resampling or integer decimation. |
| [Instrument Response Removal](#instrument-response-removal) | Deconvolve instrument response to displacement, velocity, or acceleration. |
| [Simulation with Poles and Zeros (PAZ)](#simulation-with-poles-and-zeros-paz) | Simulate a target seismometer response using poles-and-zeros dictionaries. |
| [Travel Times with TauP](#travel-times-with-taup) | Compute theoretical phase arrival times and ray paths with standard Earth models. |

### Detrending and Tapering

Always detrend and taper **before** filtering or response removal.

```python
st.detrend("demean")           # remove mean value
st.detrend("linear")           # remove linear trend
st.taper(max_percentage=0.05)  # 5% cosine taper on each end
```

### Filtering

```python
tr_filt = tr.copy()
tr_filt.filter("bandpass", freqmin=1.0, freqmax=10.0, corners=4, zerophase=True)
tr_filt.filter("lowpass",  freq=1.0,  corners=2, zerophase=True)
tr_filt.filter("highpass", freq=0.1,  corners=2, zerophase=True)

# Apply to entire Stream
st.filter("bandpass", freqmin=0.05, freqmax=0.5, zerophase=True)
```

Use `zerophase=True` to avoid phase distortion. `corners` is the filter order per pass (doubled with zero-phase).

### Resampling

```python
st.resample(20.0)       # resample to 20 Hz (Fourier method, any ratio)
st.decimate(factor=2)   # downsample by integer factor (faster, anti-alias filter applied)
```

### Instrument Response Removal

Requires an `Inventory` with `level="response"` (see `read-seismic-data.md` for download).

```python
pre_filt = (0.005, 0.006, 30.0, 35.0)   # (f1, f2, f3, f4) Hz — cosine taper corners

st.remove_response(
    inventory=inv,
    output="VEL",       # "DISP" (m), "VEL" (m/s), "ACC" (m/s²)
    pre_filt=pre_filt,
    water_level=60      # stabilizes deconvolution at low frequencies
)
```

`pre_filt` is applied before deconvolution to suppress low-frequency noise amplification. `water_level` (dB) sets the noise floor for the spectral division.

### Simulation with Poles and Zeros (PAZ)

```python
paz_sts2 = {
    "poles": [-0.037004 + 0.037016j, -0.037004 - 0.037016j,
              -251.33 + 0j, -131.04 - 467.29j, -131.04 + 467.29j],
    "zeros": [0j, 0j],
    "gain": 60077000.0,
    "sensitivity": 2516778400.0,
}
paz_target = { ... }   # poles/zeros of target instrument

st.simulate(paz_remove=paz_sts2, paz_simulate=paz_target)
```

### Travel Times with TauP

```python
from obspy.taup import TauPyModel

model = TauPyModel(model="iasp91")   # or "ak135", "prem"

arrivals = model.get_travel_times(
    source_depth_in_km=35.0,
    distance_in_degree=45.0,
    phase_list=["P", "S", "PP", "PKP"]   # omit for all phases
)

for arr in arrivals:
    print(arr.name, arr.time)   # phase name, travel time in seconds

# Ray path visualization
arrivals.plot_rays()
```

## References

- [Stream.filter](https://docs.obspy.org/packages/autogen/obspy.core.stream.Stream.filter.html)
- [Stream.remove_response](https://docs.obspy.org/packages/autogen/obspy.core.stream.Stream.remove_response.html)
- [Stream.simulate](https://docs.obspy.org/packages/autogen/obspy.core.stream.Stream.simulate.html)
- [obspy.taup](https://docs.obspy.org/packages/obspy.taup.html)
