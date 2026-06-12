# Access USDF EFD Tutorial

```{abstract}
A short tutorial showing how people can query EFD data from USDF , including required packages and a notebook as an example.
```


## Introduction

This note provides a minimal example of how to access EFD telemetry from a local machine using the Rubin Science Pipelines environment and the `lsst_efd_client` package.


## Environment Setup

Follow the Rubin Science Pipelines installation instructions:

https://pipelines.lsst.io/install/index.html

A typical installation requires approximately 10 GB of available disk space.

When reaching Step 4, repeat the procedure using:

`lsst_sitcom`

instead of:

`lsst_distrib`

Later, when configuring the environment, use:

`setup summit_utils`

instead of:

`setup lsst_distrib`

After the installation is complete, install the EFD client:

```bash
conda install -c lsstts lsst-efd-client
```


## Starting the Environment

Open a terminal and configure the environment:

```bash
cd ~/lsst_stack

source loadLSST.bash

setup lsst_sitcom

setup summit_utils
```

Start Jupyter Lab:

```bash
jupyter lab
```


### Example 1: Retrieve the Latest Available Value

In the examples below, `topic` identifies the telemetry stream and `fields` specifies which variables should be returned.

```python
from lsst_efd_client import EfdClient

client = EfdClient("usdf_efd")

topic = "lsst.sal.ESS.airFlow"
fields = ["speed"]

df = await client.select_top_n(
    topic,
    fields,
    num=1
)

last = df.iloc[0]

print(last)
```

### Example 2: Retrieve Data Over a Time Range

```python
from lsst_efd_client import EfdClient
from astropy.time import Time

client = EfdClient("usdf_efd")

topic = "lsst.sal.ESS.airFlow"
fields = ["speed"]

start = Time("2026-06-11T12:00:00", scale="utc")
end   = Time("2026-06-11T13:00:00", scale="utc")

df = await client.select_time_series(
    topic,
    fields,
    start,
    end
)

print(df)
```

### Example 3: Averaging Data into Time Bins

```python
from lsst_efd_client import EfdClient
from astropy.time import Time

client = EfdClient("usdf_efd")

topic = "lsst.sal.ESS.airFlow"
fields = ["speed"]

start = Time("2026-06-11T12:00:00", scale="utc")
end   = Time("2026-06-11T13:00:00", scale="utc")

df = await client.select_time_series(
    topic,
    fields,
    start,
    end
)

df_5min = df.resample("5min").mean()

print(df_5min)
```

## Discovering Available Topics

To list available topics:

```python
from lsst_efd_client import EfdClient

client = EfdClient("usdf_efd")

topics = await client.get_topics()

for topic in topics:
    print(topic)
```

To inspect the fields available within a topic:

```python
from lsst_efd_client import EfdClient

client = EfdClient("usdf_efd")

fields = await client.get_fields(
    "lsst.sal.ESS.airFlow"
)

print(fields)
```

## Exploring New Telemetry Streams

Most EFD queries follow the same structure. Typically only the `topic` and field names need to be changed:

```python
topic = "REPLACE_TOPIC"
fields = ["REPLACE_FIELD"]
```

A typical workflow is:

1. Find the topic of interest using `get_topics()`.
2. Inspect the available fields using `get_fields()`.
3. Select the desired topic, field, and time range.
4. Query the EFD.

## Gemini Telemetry

A parallel effort is underway to provide access to selected Gemini telemetry from the Rubin environment. Current data sources include the Gemini weather database and the RINGSS seeing monitor database.

An example output is shown below:

```text
=== ringss ===
time: 2026-06-11 23:58:22
see: 0.605
wind: 13.16
tau0: 3.892
theta0: 2.157

=== weather_data ===
time: 2026-06-11 23:58:14
temp: 7.59
wind: 7.7
humidity: 28.9
press: 727.06
dewPoint: -9.37994
```

A set of Python scripts developed for this effort currently provides access to the Gemini Weather Station (GWS) and RINGSS telemetry through channel-based queries, telemetry snapshots around a given timestamp, and scheduled execution through `cron`. The current approach being evaluated is to provide Rubin with database credentials and access from a dedicated Rubin-side host capable of executing these queries against Gemini telemetry databases.

Gemini currently uses `TED` (Telescope Engineering Data), a Grafana-based interface for engineering and environmental telemetry. In parallel, work is underway to standardize and consolidate Gemini telemetry databases behind `TED`, providing a more uniform source of telemetry from multiple Gemini systems in the future (not only weather data). This should provide a natural path for sharing telemetry with Rubin before pursuing additional integration work.

An ITOps ticket (`ITOPS-20756`) has been opened to evaluate infrastructure options for an initial access path from the Rubin side.




See the [Documenteer documentation](https://documenteer.lsst.io/technotes/index.html) for tips on how to write and configure your new technote.

