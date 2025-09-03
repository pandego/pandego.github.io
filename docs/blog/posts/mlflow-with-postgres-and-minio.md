---
date: 2023-03-05
authors:
  - migueldias
categories:
  - Tools
  - Python
  - Development
description: MCP is a way to help your LLM-powered apps interact with the outside world — tools, databases, documents, APIs — in a standard and structured way.
---

# Red Wine and Dockerized MLflow: A Love Story Between DevOps and Data Science

Are you tired of messy machine learning pipelines? Do you struggle to keep track of your experiments and their results? Fear not, because **MLflow and Docker Compose are here to save the day!**

<!-- more -->

---
> *Condensed mini‑blog from my piece on self-hosting an MLflow instance.*

## Why this stack works

If you’re wrangling ML experiments, **MLflow** is your control tower. Pair it with **PostgreSQL** for the tracking backend and **MinIO** (S3‑compatible) for artifact storage, then wrap the whole thing in **Docker Compose** so it’s reproducible and easy to boot up anywhere. Result: clean experiment tracking, model registry, and artifact management without yak‑shaving.

**Ingredients**

* **MLflow**: experiment tracking, model registry, model serving
* **Postgres**: durable metadata (experiments, runs, metrics)
* **MinIO**: object store for artifacts/models (S3 API)
* **Docker Compose**: one command to run them all

---

## Prereqs

* Ubuntu 20.04 LTS (WSL2 on Windows works too)
* Miniconda, Docker, Docker Compose
* Optional but handy: **pyenv** to keep Python versions isolated

```bash
# Install pyenv (Ubuntu)
sudo apt-get update -y && \
  sudo apt-get install -y make build-essential libssl-dev zlib1g-dev \
  libbz2-dev libreadline-dev libsqlite3-dev wget curl llvm \
  libncursesw5-dev xz-utils tk-dev libxml2-dev libxmlsec1-dev \
  libffi-dev liblzma-dev
curl https://pyenv.run | bash
# Add to ~/.bashrc
export PYENV_ROOT="$HOME/.pyenv"
export PATH="$PYENV_ROOT/bin:$PATH"
eval "$(pyenv init --path)"
source ~/.bashrc
```

Create a working env and set S3/MLflow env vars:

```bash
conda create -n mlflow_env python=3.11 -y
conda activate mlflow_env
pip install pandas scikit-learn mlflow[extras]

export AWS_ACCESS_KEY_ID=minio
export AWS_SECRET_ACCESS_KEY=minio123   # change in .env
export MLFLOW_S3_ENDPOINT_URL=http://localhost:9000
```

---

## Compose it all together

Clone the template project:

```bash
git clone https://github.com/pandego/mlflow-postgres-minio.git
cd mlflow-postgres-minio
cp default.env .env   # edit secrets/ports as needed
```

**Key services (trimmed):**

```yaml
services:
  db:
    image: postgres:${PG_VERSION}
    environment:
      - POSTGRES_USER=${PG_USER}
      - POSTGRES_PASSWORD=${PG_PASSWORD}
      - POSTGRES_DATABASE=${PG_DATABASE}
    healthcheck:
      test: ["CMD", "pg_isready", "-p", "${PG_PORT_SERVICE}", "-U", "${PG_USER}"]

  s3:
    image: minio/minio:${MINIO_VERSION}
    command: server /data --console-address ":9001" --address ":9000"
    environment:
      - MINIO_ROOT_USER=${MINIO_ROOT_USER}
      - MINIO_ROOT_PASSWORD=${MINIO_ROOT_PASSWORD}

  create_buckets:
    image: minio/mc:${MINIO_VERSION}
    depends_on: { s3: { condition: service_healthy } }
    entrypoint: >
      /bin/sh -c '
      sleep 5;
      mc config host add s3 http://s3:${MINIO_PORT_API} ${MINIO_ROOT_USER} ${MINIO_ROOT_PASSWORD} --api S3v4;
      mc mb s3/${MLFLOW_BUCKET_NAME} || true;
      mc policy download s3/${MLFLOW_BUCKET_NAME};
      mc mb s3/${DATA_REPO_BUCKET_NAME} || true;
      mc policy download s3/${DATA_REPO_BUCKET_NAME};
      '

  tracking_server:
    build: ./mlflow
    command: >
      mlflow server \
        --backend-store-uri postgresql://${PG_USER}:${PG_PASSWORD}@db:${PG_PORT_SERVICE}/${PG_DATABASE} \
        --host 0.0.0.0 \
        --port ${MLFLOW_PORT_SERVICE} \
        --default-artifact-root s3://mlflow/
    environment:
      - AWS_ACCESS_KEY_ID=${MINIO_ROOT_USER}
      - AWS_SECRET_ACCESS_KEY=${MINIO_ROOT_PASSWORD}
      - MLFLOW_S3_ENDPOINT_URL=http://s3:${MINIO_PORT_API}
```

Bring it up:

```bash
docker-compose --env-file .env up -d --build
docker ps -a   # all services should be healthy
```

**UIs**

* MLflow Tracking: [http://localhost:5000](http://localhost:5000)
* MinIO Console: [http://localhost:9001](http://localhost:9001) (API is :9000)

---

## Train a model (ElasticNet on red wine)

Inside `./wine_quality_example/` you’ll find `wine_quality_data.csv` and `train.py`. The script trains a **scikit‑learn ElasticNet** regressor to predict wine quality and logs everything to MLflow.

```python
# metrics helper
def eval_metrics(y_true, y_pred):
    rmse = np.sqrt(mean_squared_error(y_true, y_pred))
    mae = mean_absolute_error(y_true, y_pred)
    r2 = r2_score(y_true, y_pred)
    return rmse, mae, r2

# point MLflow at your server
mlflow.set_tracking_uri("http://localhost:5000")
mlflow.set_experiment("red-wine-elasticnet")

with mlflow.start_run():
    model = ElasticNet(alpha=alpha, l1_ratio=l1_ratio, random_state=42)
    model.fit(train_x, train_y)
    preds = model.predict(test_x)
    rmse, mae, r2 = eval_metrics(test_y, preds)

    mlflow.log_param("alpha", alpha)
    mlflow.log_param("l1_ratio", l1_ratio)
    mlflow.log_metric("rmse", rmse)
    mlflow.log_metric("mae", mae)
    mlflow.log_metric("r2", r2)
    mlflow.sklearn.log_model(model, "model")
```

Run it:

```bash
cd wine_quality_example
python train.py
```

You should see a new experiment/run in the MLflow UI with params, metrics, and a logged model artifact stored in MinIO.

---

## Serve the model

Serve any logged run locally with MLflow’s lightweight server:

```bash
mlflow models serve \
  -m s3://mlflow/1/<run_id>/artifacts/model \
  -p 1234 --timeout 0
```

Smoke‑test with `curl` (JSON `dataframe_split`):

```bash
curl -X POST \
  -H "Content-Type: application/json" \
  --data '{
    "dataframe_split": {
      "data": [[7.4,0.7,0,1.9,0.076,11,34,0.9978,3.51,0.56,9.4]],
      "columns": [
        "fixed acidity","volatile acidity","citric acid","residual sugar",
        "chlorides","free sulfur dioxide","total sulfur dioxide","density",
        "pH","sulphates","alcohol"
      ]
    }
  }' http://127.0.0.1:1234/invocations
# → {"predictions": [5.576883967129616]}
```

---

## Tips & gotchas

* Keep `.env` in sync with your conda env variables (keys/ports/bucket names).
* MinIO exposes **API on :9000** and **Console on :9001** by default—both must be reachable from the tracking server.
* Healthchecks in Compose catch misconfig early; don’t disable them.
* For production, put Postgres/MinIO on persistent volumes you back up.

---

## Wrap‑up

MLflow + Postgres + MinIO, dockerized with Compose, gives you a **reproducible, portable MLOps base** in minutes. Log parameters and metrics, register and serve models, and keep artifacts tidy—all while avoiding snowflake setups. Pour yourself a glass; your experiments are finally organized.

---

## 📖 Read the Full Article

<div class="medium-card" style="border: 1px solid var(--md-default-fg-color--lightest, #e1e5e9); border-radius: 8px; padding: 16px; margin: 20px 0; background: var(--md-code-bg-color, #f8f9fa); transition: all 0.2s ease;">
  <div style="display: flex; align-items: center; gap: 12px;">
    <div style="flex-shrink: 0;">
      <!-- Replace with your article image -->
      <img src="https://miro.medium.com/v2/resize:fit:1400/format:webp/1*jB4nybWEiBnxLmIFPvv4Ng.png" alt="Article Preview" style="width: 80px; height: 80px; border-radius: 8px; object-fit: cover; border: 1px solid var(--md-default-fg-color--lightest, #e1e5e9);">
    </div>
    <div style="flex: 1;">
      <h4 style="margin: 0 0 8px 0; font-size: 16px; color: var(--md-default-fg-color, #1a1a1a);">Red Wine, ElasticNet, and Dockerized MLflow with Postgres and MinIO: A Love Story Between DevOps and Data Science</h4>
      <!-- Replace with your custom description -->
      <p style="margin: 0 0 12px 0; font-size: 14px; color: var(--md-default-fg-color--light, #6b7280); line-height: 1.4;">A comprehensive guide on setting up MLflow with Docker, using Postgres for tracking and MinIO for artifact storage, demonstrated through a wine quality prediction model.</p>
      <a href="https://medium.com/@pandego/red-wine-elasticnet-and-dockerized-mlflow-with-postgres-and-minio-d5aee144d1df" target="_blank" style="display: inline-flex; align-items: center; gap: 4px; color: var(--md-accent-fg-color, #059669); text-decoration: none; font-weight: 500; font-size: 14px; transition: color 0.2s ease;">
        📖 Full article available on Medium
        <svg width="12" height="12" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
          <path d="M7 17L17 7M17 7H7M17 7V17"/>
        </svg>
      </a>
    </div>
  </div>
</div>
