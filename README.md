# Spark + Kafka + MinIO DataLake

Environnement de developpement Big Data pour apprendre et prototyper des pipelines modernes avec Spark, Kafka, MinIO et PostgreSQL.

## Description

Cette stack Docker fournit une infrastructure data lake prete a l'emploi :

- Apache Spark 4.0 via JupyterLab pour le traitement distribue
- Apache Kafka pour le streaming en temps reel
- MinIO comme stockage objet compatible S3
- PostgreSQL avec la base Northwind pour les exercices SQL

## Demarrage rapide

```bash
docker compose up -d
docker compose ps
docker compose logs -f
```

## Arret

```bash
docker compose down
docker compose down -v
```

## Services

| Service | URL | Credentials |
|---------|-----|-------------|
| JupyterLab | http://localhost:8888 | Acces direct |
| Kafka UI | http://localhost:7080 | - |
| Adminer | http://localhost:9080 | `postgres` / `postgres` |
| MinIO Console | http://localhost:9001 | `minioadmin` / `minioadmin123` |
| MinIO API (S3) | http://localhost:9000 | - |
| Kafka Broker | localhost:9092 | - |
| PostgreSQL | localhost:5433 | `postgres` / `postgres` / DB: `app` |

## Structure du projet

```text
docker-compose.yml          Configuration des services
notebooks/                  Notebooks Jupyter
  exemples/                 Exemples de demarrage
  exercices/                Exercices pratiques
  elk/                       Integration ELK
vol/
  jupyter/                  Config Jupyter
  postgresql/               Scripts SQL (Northwind)
.docker/
  jupyter-spark/            Dockerfile Spark custom
```

## Notebooks disponibles

| Notebook | Description |
|----------|-------------|
| `1_test_demarrage.ipynb` | Verification de l'environnement Spark |
| `2_simple_python_producer.ipynb` | Producer Kafka en Python |
| `3_pyspark_consumer.ipynb` | Consumer Kafka avec PySpark |
| `4_pyspark_stream_consumer.ipynb` | Streaming Spark + Kafka |
| `5_gen_data.ipynb` | Generation de donnees de test |

## Configuration

### Variables d'environnement optionnelles

```env
MINIO_ROOT_USER=minioadmin
MINIO_ROOT_PASSWORD=minioadmin123
MINIO_VERSION=latest
```

### Connexion Spark -> MinIO

```python
spark = SparkSession.builder \
    .appName("MinIO") \
    .config("spark.hadoop.fs.s3a.endpoint", "http://minio:9000") \
    .config("spark.hadoop.fs.s3a.access.key", "minioadmin") \
    .config("spark.hadoop.fs.s3a.secret.key", "minioadmin123") \
    .config("spark.hadoop.fs.s3a.path.style.access", "true") \
    .getOrCreate()
```

### Connexion Spark -> Kafka

```python
df = spark.readStream \
    .format("kafka") \
    .option("kafka.bootstrap.servers", "broker:29092") \
    .option("subscribe", "mon-topic") \
    .load()
```

### Connexion Spark -> PostgreSQL

```python
df = spark.read \
    .format("jdbc") \
    .option("url", "jdbc:postgresql://postgres:5432/app") \
    .option("user", "postgres") \
    .option("password", "postgres") \
    .option("dbtable", "customers") \
    .load()
```

## Depannage

### Container qui ne demarre pas

```bash
docker compose logs postgres
docker compose build --no-cache jupyter-spark
```

### Port deja utilise

```bash
netstat -ano | findstr :8888
```

### Reinitialisation complete

```bash
docker compose down -v
docker system prune -f
docker compose up -d
```

## Documentation

- [Apache Spark](https://spark.apache.org/docs/latest/)
- [Apache Kafka](https://kafka.apache.org/documentation/)
- [MinIO](https://min.io/docs/minio/container/index.html)
- [PySpark](https://spark.apache.org/docs/latest/api/python/)
