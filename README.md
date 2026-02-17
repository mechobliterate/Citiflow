# Citiflow

Citiflow is a visualization of the entire history [Citi Bike](https://citibikenyc.com), the largest bike-sharing system in the United States.

https://github.com/user-attachments/assets/814a0366-c5a2-4410-9998-dfa13f8e5475

Each moving arrow represents a real bike ride, based on anonymized [historical system data](https://citibikenyc.com/system-data) published by Lyft.

## Motivation

I built this project because I think it is cool and a beautiful site to look at. 

I hope to keep this project running indefinitely, but I'm paying for Mapbox and hosting costs out of pocket. If you'd like to support me, please consider checking out on the website.

## Features
- GPU-accelerated rendering of thousands of concurrent rides
- Natural language date parsing to jump to any moment in history
- Search for individual rides by date and station name
- Full keyboard controls for playback and navigation
- Coverage of more than 291.2 million trips from 2013 to 2025 (0.7% data loss)

## How it Works

There is no backend. The client uses [DuckDB WASH](https://duckdb.org/docs/api/wasm/overview.html) to query parquet files using SQL directly from a CDN, downloading only the rows it needs via HTTP range requests.

### 1. Data Processing Pipeline

The raw system data spans 12 years and has significant inconsistencies, making it difficult to use directly. The processing pipeline cleans and normalizes the data into optimized parquet files.

1. Creates a list of all unique station names and their coordinates.
2. Queries [OSM](https://project-osrm.org/) for bike routes between all station pairs. Geometries are cached per pair and stored as Polyline6 in an intermediate SQLite database.
3. Generates a parquet file for each day by joining each trip with its corresponding route Geometry.

### 2. Client Application

This is what you see when you visit the website.

- DuckDB WASM queries parquet files from the CDN using HTTP range requests. Trips load in 30-minute batches with lookahead prefetching.
- A Web Worker decodes the Polyline6 Geometry and pre-computes timestamps with easing so that bikes slow down at station endpoints.
- Heavy lifting is done with deck.gl layers on top of Mapbox.
- Natural language date parsing via chrono-node lets you jump to any point in time or find a specific ride by querying the parquets directly.

## Quickstart

**1. Setup Environment Variables**

Create a `.env` file in `apps/client` and add your Mapbox token:

```sh
NEXT_PUBLIC_MAPBOX_TOKEN=pk.xxx  # Get one at https://mapbox.com/
```

**2. Install Dependencies and Run**

```sh
bun install
bun dev
```

**Note:** The client queries parquet files from the official hosted CDN by default. You don't need to run the processing pipeline unless you want to regemnerate the data. See [processing README](packages/processing/README.md) for how to run the pipeline.
