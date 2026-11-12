# Harbor – Flask auth app (Cloud Run ready)

## Run locally
    python -m venv venv && source venv/bin/activate
    pip install -r requirements.txt
    python app.py            # http://localhost:8080

## Run with Docker
    docker build -t flask-auth-app .
    docker run -p 8080:8080 -e SECRET_KEY=change-me flask-auth-app

## Deploy to Cloud Run (from source, no local Docker needed)
    gcloud config set project YOUR_PROJECT_ID
    gcloud run deploy flask-auth-app \
      --source . \
      --region asia-southeast1 \
      --allow-unauthenticated \
      --set-env-vars SECRET_KEY=$(openssl rand -hex 32),COOKIE_SECURE=1

## Or build & push the image yourself
    gcloud artifacts repositories create apps --repository-format=docker --location=asia-southeast1
    docker build -t asia-southeast1-docker.pkg.dev/YOUR_PROJECT_ID/apps/flask-auth-app:v1 .
    docker push asia-southeast1-docker.pkg.dev/YOUR_PROJECT_ID/apps/flask-auth-app:v1
    gcloud run deploy flask-auth-app \
      --image asia-southeast1-docker.pkg.dev/YOUR_PROJECT_ID/apps/flask-auth-app:v1 \
      --region asia-southeast1 --allow-unauthenticated \
      --set-env-vars SECRET_KEY=...,COOKIE_SECURE=1

## IMPORTANT: database
Default SQLite lives in /tmp and is wiped when Cloud Run restarts or scales,
so users would disappear. For real use, create a Cloud SQL (PostgreSQL)
instance and set DATABASE_URL, e.g.
    postgresql+psycopg2://USER:PASS@/DB?host=/cloudsql/PROJECT:REGION:INSTANCE
and deploy with --add-cloudsql-instances PROJECT:REGION:INSTANCE
(store SECRET_KEY / DB password in Secret Manager with --set-secrets).
