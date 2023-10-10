Summarization
Many detection use cases don't require each individual log record. This summarization runs an hourly playbook against ADX that aggregates entity data from an external table in ADLS. Doing so, only the summarized data is ingested into compute ready (Sentinel) storage while the raw data is kept in ADLS for ad hoc searches or as other analytics use cases are developed.
