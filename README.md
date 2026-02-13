# IoT Event Data Generation

Generate synthetic IoT-style event data with **fixed columns** and a **dynamic JSON column**. Use for demos, testing ETL/streaming pipelines, or feeding Azure Event Hub → Delta Lake / Iceberg.

## Features

- **Parameterised**: All settings in one **Config** cell at the top (event count, paths, keys range, streaming, Delta/Iceberg).
- **Batch or stream**: Generate a one-off DataFrame/JSON file, or stream events to Azure Event Hub (e.g. from Databricks).
- **Flexible schema**: 23 fixed columns (device_type, sensor_id, temperature, etc.) plus one `json_column` with 5–25 random keys (including nested objects: `location`, `diagnostics`, `config`).
- **Reproducible**: Optional `RANDOM_SEED` for identical runs.
- **Reusable**: Change catalogs (device types, status values, regions) in a single **Data catalogs** cell; generator functions accept overrides.

## Requirements

- **Batch (local or any Python):**  
  `pandas`
- **Streaming (Databricks):**  
  PySpark, Spark session, Azure Event Hub connection, and (for Delta/Iceberg) Databricks runtime with Delta/Iceberg support.

## Quick Start

1. Open `iot_data_generation.ipynb`.
2. Edit **Cell 1 (Config)**:
   - `NUM_EVENTS`, `MIN_KEYS`, `MAX_KEYS` for batch size and JSON richness.
   - `OUTPUT_DIR`, `SAVE_JSON`, `OUTPUT_FILENAME_PREFIX` for file output.
   - For streaming: `EVENTHUB_NAMESPACE`, `EVENTHUB_NAME`, `EVENTHUB_CONNECTION_STRING`, `BATCH_SIZE`, `SLEEP_INTERVAL_SEC`.
   - For Delta: `DELTA_TABLE_NAME`, `CHECKPOINT_DIR`.
3. Run cells in order:
   - **Batch only:** Run up to and including the **Preview** cell. Optionally run **Generate batch and save** to write JSON.
   - **Streaming:** Set `EVENTHUB_CONNECTION_STRING` (or env var / Databricks secret), then run from **Event Hub connection** through **Stream to Event Hub** and/or **Read stream** → **Parse JSON** → **Write to Delta**.

## How to Use

### Batch generation (no Databricks)

- Set `NUM_EVENTS`, `OUTPUT_DIR`, `SAVE_JSON = True`.
- Run: Imports → Data catalogs → Generator functions → **Generate batch and (optional) save** → Preview.
- Output: in-memory `df` and, if `SAVE_JSON` is True, a timestamped JSON file under `OUTPUT_DIR`.

### Streaming to Event Hub (Databricks)

- Set `EVENTHUB_CONNECTION_STRING` in Config (or `os.environ["EVENTHUB_CONNECTION_STRING"]` / Databricks secret).
- Set `CATALOG_VOLUME_PATH` to a Volume path for checkpoints (e.g. `/Volumes/<catalog>/<schema>/<volume>`).
- Run the Event Hub connection cell, then **Stream IoT data to Event Hub**. Stop the stream with the notebook **Stop** button.

### Writing to Delta / Iceberg

- After **Parse JSON and convert json_column to VARIANT**, use the **Write stream to Delta table** cell (or the equivalent Iceberg cell if you add it). Set `DELTA_TABLE_NAME` and `CHECKPOINT_DIR` in Config.

### Customising data

- **Config:** `BASE_TIME_DAYS_AGO`, `TIME_SPREAD_HOURS` control the time window of generated events.
- **Data catalogs cell:** Edit `DEVICE_TYPES`, `STATUS_VALUES`, `REGIONS`, `SOURCES`, `ENVS`, `IOT_KEY_POOL`, or `NESTED_KEYS` to change allowed values and dynamic keys.
- **Generator functions:** You can pass overrides into `generate_fixed_row` and `generate_dynamic_json` (e.g. custom device types or key pool) for programmatic reuse.

## Best practices

- **Secrets:** Do not commit connection strings. Use environment variables or Databricks secrets and reference them in the Config cell.
- **Reproducibility:** Set `RANDOM_SEED` (e.g. `42`) when you need the same dataset across runs.
- **Paths:** Use a dedicated Volume or directory for `OUTPUT_DIR` and checkpoints so multiple runs or users don’t overwrite each other.
- **Streaming rate:** Keep `BATCH_SIZE / SLEEP_INTERVAL_SEC` within Event Hub throttling limits; tune in Config.

## Output schema (batch)

| Concept        | Columns |
|----------------|--------|
| Fixed (23)     | `event_id`, `timestamp`, `device_type`, `firmware_version`, `battery_level`, `sensor_id`, `temperature`, `humidity`, `pressure`, `status`, `region`, `zone`, `model`, `version`, `signal_strength`, `uptime_hours`, `fault_count`, `last_calibration`, `maintenance_due`, `created_at`, `updated_at`, `source`, `env` |
| Dynamic (1)    | `json_column` (object with 5–25 keys from `IOT_KEY_POOL`; may contain nested `location`, `diagnostics`, `config`) |

## File layout (suggested)

```
.
├── README.md
├── iot_data_generation.ipynb   # Modular notebook (use this)
├── Data generation.ipynb       # Optional: original notebook
└── output/                     # Or use a Databricks Volume path
    └── iot_events_YYYYMMDD_HHMMSS.json
```

## License

Use and adapt as needed for your organisation. If you publish a derivative, consider keeping attribution to the original data generation logic.
