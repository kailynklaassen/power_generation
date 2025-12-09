# power_generation

Energy Plant Operations – Getting Started
This project demonstrates an end‑to‑end data contract–driven pipeline for renewable energy plant operations, from raw telemetry (Bronze) through cleaned/conformed (Silver) to plant‑level KPIs (Gold).

1. Sample CSV datasets
Three CSV files are used as the starting point for the POC. They represent plant master data and high‑frequency telemetry from battery, solar, and wind assets at a renewable plant.

1.1 plant_details.csv
Static master data for each energy plant.

Key columns (examples):

plant_id: Unique identifier for the plant.

plant_name: Human‑readable plant name.

plant_type: Plant category, e.g. Battery, Solar, Wind.

regulatory_category: Regulatory classification, e.g. NERC or FERC.

internal_regulatory_category: Internal classification, e.g. NEER or FPL.

latitude, longitude: Plant coordinates.

capacity_mw: Nameplate capacity in megawatts.

commissioning_date: Commercial operation date.

status: Operational status, e.g. Active, Maintenance, Offline.

This file is used to populate the plant_details_bronze / plant_details_silver tables.

1.2 battery_sensor_feed.csv
Time‑series sensor data from battery energy storage systems.

Representative columns:

reading_id: Unique ID for each sensor reading.

plant_id, component_id, sensor_id, sensor_type: Keys that locate the reading (plant, asset, measurement type).

timestamp: When the reading was captured.

value, unit_of_measure: Raw sensor value and its unit (e.g. MW, MWh, V, A, C).

state_of_charge_pct, state_of_health_pct: Battery SOC/SOH percentages.

charge_discharge_cycles: Cumulative cycle count.

power_mw, energy_mwh, voltage_v, current_a, temperature_c: Core electrical and thermal metrics.

quality_flag, alarm_status: Data quality and alarm indicators.

ingestion_datetime: When the reading landed in the platform.

This file feeds the battery_sensor_feed_bronze / battery_sensor_feed_silver tables.

1.3 solar_sensor_feed.csv and wind_sensor_feed.csv
Time‑series sensor data from solar PV and wind turbine assets.

Typical solar columns:

reading_id, plant_id, component_id, sensor_id, sensor_type

timestamp, value, unit_of_measure

irradiance_wm2, panel_temperature_c, ambient_temperature_c

power_mw, energy_mwh

dc_voltage_v, dc_current_a, ac_voltage_v, ac_current_a

inverter_efficiency_pct, performance_ratio, tracker_angle_degrees

quality_flag, alarm_status, ingestion_datetime

Typical wind columns:

reading_id, plant_id, component_id, sensor_id, sensor_type

timestamp, value, unit_of_measure

wind_speed_ms, wind_direction_degrees

power_mw, energy_mwh

rotor_speed_rpm, generator_speed_rpm

blade_pitch_angle_degrees, nacelle_yaw_angle_degrees

gearbox_oil_temperature_c, generator_bearing_temperature_c, nacelle_temperature_c, ambient_temperature_c

vibration_mms, power_coefficient, tip_speed_ratio

quality_flag, alarm_status, turbine_status, ingestion_datetime

These files populate the solar_sensor_feed_* and wind_sensor_feed_* tables.

2. Uploading CSVs into a Unity Catalog volume
Follow these steps once per environment to stage the CSVs where the notebooks expect them.

Create / locate a volume

In Databricks, create (or reuse) a catalog and schema, for example:

Catalog: kailyn_klaassen

Schema: odcs

Under that schema, create a Volume, for example ingest_volume.

Upload the CSVs

In the Databricks UI, open the Volume (e.g. kailyn_klaassen.odcs.ingest_volume).

Use the “Upload” button to upload:

plant_details.csv

battery_sensor_feed.csv

solar_sensor_feed.csv

wind_sensor_feed.csv

Confirm the files appear at the root of the volume (or note any subfolder you use).

Reference the volume from notebooks

In your notebooks, point the input paths at the volume, for example:

python
catalog = "kailyn_klaassen"
schema = "odcs"
volume = "ingest_volume"

base_path = f"/Volumes/{catalog}/{schema}/{volume}"

plant_details_path = f"{base_path}/plant_details.csv"
battery_feed_path = f"{base_path}/battery_sensor_feed.csv"
solar_feed_path = f"{base_path}/solar_sensor_feed.csv"
wind_feed_path = f"{base_path}/wind_sensor_feed.csv"
These paths should match what the Bronze ingestion code in your repo expects.

3. Next steps: run Bronze → Silver → Gold
Once the CSVs are in the volume and the paths are set:

Bronze layer

Run the Bronze notebooks/scripts in your repo.

These should:

Read the CSVs from the volume.

Apply minimal parsing and landing logic.

Create the *_bronze tables (e.g. plant_details_bronze, battery_sensor_feed_bronze, solar_sensor_feed_bronze, wind_sensor_feed_bronze).

Silver layer

Run the Silver notebooks/scripts.

These typically:

Read from the Bronze tables.

Apply cleaning, conforming, and data quality rules.

Write the *_silver tables (e.g. plant_details_silver, battery_sensor_feed_silver, etc.).

Gold layer

Run the Gold notebook/script.

This step:

Reads the Silver tables.

Aggregates metrics into plant‑level KPIs on an hourly window.

Writes the plant_operations_kpi_gold table, which is the primary analytics output.

After this sequence, the environment has a complete medallion pipeline driven by your data contract, and consumers can start querying the Gold table for plant KPIs, or the Silver layer for detailed operational analysis.
