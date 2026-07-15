# Spark Cluster

Standalone Apache Spark cluster (master + worker) for distributed writes from the Credit API and other PySpark jobs. Shares object storage and Hive Metastore with the Trino lakehouse stack.

## Services

| Service | Host ports | Internal | Purpose |
|---------|------------|----------|---------|
| spark-master | 8083 (UI), 7077 (RPC) | `spark-master` | Standalone master |
| spark-worker | 8089 (UI) | `spark-worker` | Worker (2 cores, 4 GiB) |

## Prerequisites

External Docker networks must exist before starting (created by `make local-up` or the credit stack):

- `minio_minio`
- `credit_risk_shared`

## Quick start

```bash
# After credit_risk_forecast and trino stacks are up
docker compose up -d
```

Or from repo root: `make local-up`.

## Configuration

Hive and S3 settings live in per-node config:

| File | Key settings |
|------|--------------|
| `spark-master/conf/spark-defaults.conf` | `spark.hadoop.hive.metastore.uris`, `spark.sql.warehouse.dir` |
| `spark-master/conf/hive-site.xml` | Metastore thrift URI |
| `spark-worker/conf/` | Same layout as master |

**Defaults:**

- **Metastore:** `thrift://hive-metastore:9083`
- **Warehouse:** `s3a://lakehouse/` on MinIO (`http://minio:9000`, keys `minio123`)

## How Credit API uses Spark

[`credit_risk_forecast/prod/lgbm_prod.py`](../credit_risk_forecast/prod/lgbm_prod.py) runs a PySpark driver inside the `credit-scoring-api` container and connects executors to `spark://spark-master:7077`. Prediction events are appended to Iceberg tables (e.g. `forecast.prediction_events`) via the `iceberg_hms` catalog.

**Relevant env vars (Credit API):**

| Variable | Typical value |
|----------|---------------|
| `SPARK_MASTER_URL` | `spark://spark-master:7077` |
| `SPARK_HIVE_METASTORE_URIS` | `thrift://hive-metastore:9083` |
| `SPARK_SQL_WAREHOUSE_DIR` | `s3a://lakehouse/` |
| `SPARK_DRIVER_HOST` | `credit-scoring-api` (Docker hostname) |
| `SPARK_S3A_ENDPOINT` | `http://minio:9000` |

## Layout

```
spark-cluster/
├── docker-compose.yml
├── spark-master/
│   ├── Dockerfile
│   └── conf/
│       ├── spark-defaults.conf
│       └── hive-site.xml
└── spark-worker/
    ├── Dockerfile
    └── conf/
```

## Kubernetes

Equivalent manifests: [`k8s/spark/`](../k8s/spark/). Images: `local/spark-master:latest`, `local/spark-worker:latest` (built by `k8s/scripts/build-images.sh`).
