# Fabric Weather Warehouse Pipeline

A PySpark notebook in Microsoft Fabric that pulls hourly weather forecasts from the Open-Meteo API into a Fabric warehouse, scheduled to run every hour. The warehouse is connected to Power BI Desktop for reporting.

**Status: in progress.** The data model and the Power BI report are not finished yet.

## How it works
1. **Extract:** calls the Open-Meteo forecast API (Hyderabad, 7-day hourly forecast, 168 rows per pull) with retry handling.
2. **Validate:** checks required columns, duplicate keys (`location` + `weather_time`), null keys, row count and value ranges. Any failed check stops the run.
3. **Stage:** writes the latest pull to a staging table in the warehouse.
4. **Upsert:** keeps existing rows whose keys are not in the new pull, replaces or adds the rest, and verifies the row count after writing. A one-time `INITIAL_LOAD` flag creates the target table on the first run.
5. **Schedule:** the notebook runs every hour through a Fabric schedule.

## Stack
Python, PySpark, Microsoft Fabric (Warehouse, Notebooks), Open-Meteo API, Power BI Desktop

## Data
Open-Meteo hourly forecast variables: temperature, apparent temperature, humidity, precipitation, rain, weather code, cloud cover, pressure, wind speed, wind direction and wind gusts. No API key is needed.

## Known limitations
- The API returns **forecast** data, not observations, so each hour's values are overwritten by the latest forecast.
- Single location. The target table is one flat table, not yet a star schema.
- The upsert rewrites the target table rather than using an atomic MERGE, so a failure mid-write could leave it in a bad state.
- The Power BI report is still to be built.

## Next steps
- Model the warehouse as a star schema (fact and dimension tables) and add more locations.
- Build the Power BI report on top of the warehouse.

## Running it
Import the notebook into a Fabric workspace, attach it to a warehouse, update the `WAREHOUSE` name in the configuration cell, set `INITIAL_LOAD = True` for the first run only, then schedule it.# fabric-weather-warehouse-pipeline
