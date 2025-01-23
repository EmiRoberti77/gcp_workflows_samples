# GCP Serverless Orchestration with Workflows

## 1. Workflows Overview

A workflow consists of a series of steps described in YAML. Each step represents a function call or operation. Workflows enable automation by connecting various services.

### Enable Necessary Services

Before proceeding, enable the required services on Google Cloud:

```bash
gcloud services enable \
  cloudfunctions.googleapis.com \
  run.googleapis.com \
  workflows.googleapis.com \
  cloudbuild.googleapis.com \
  storage.googleapis.com
```

## 2. Deploy Your First Cloud Function

### Create a Cloud Function for Random Number Generation

1. Create a directory for the function code:

   ```bash
   mkdir ~/randomgen
   cd ~/randomgen
   ```

2. Add the following Python code in a `main.py` file:

   ```python
   import random, json
   from flask import jsonify

   def randomgen(request):
       randomNum = random.randint(1, 100)
       output = {"random": randomNum}
       return jsonify(output)
   ```

3. Create a `requirements.txt` file with the following content:

   ```
   flask>=1.0.2
   ```

4. Deploy the Cloud Function with the following command:

   ```bash
   gcloud functions deploy randomgen \
     --runtime python37 \
     --trigger-http \
     --allow-unauthenticated
   ```

5. Test the function using:
   ```bash
   curl $(gcloud functions describe randomgen --format='value(httpsTrigger.url)')
   ```

## 3. Create a Cloud Function to Multiply Input

1. Create a new directory and add the following Python code in `main.py`:

   ```python
   import json
   from flask import jsonify

   def multiply(request):
       request_json = request.get_json()
       output = {"multiplied": 2 * request_json["input"]}
       return jsonify(output)
   ```

2. Use the same `requirements.txt` as before. Deploy the function:

   ```bash
   gcloud functions deploy multiply \
     --runtime python37 \
     --trigger-http \
     --allow-unauthenticated
   ```

3. Test the function with:
   ```bash
   curl $(gcloud functions describe multiply --format='value(httpsTrigger.url)') \
   -X POST \
   -H "content-type: application/json" \
   -d '{"input": 5}'
   ```

## 4. Set Up Your Workflow

1. Create a `workflow.yaml` file:

   ```yaml
   - randomgenFunction:
       call: http.get
       args:
         url: https://<region>-<project-id>.cloudfunctions.net/randomgen
         result: randomgenResult
   - multiplyFunction:
       call: http.post
       args:
         url: https://<region>-<project-id>.cloudfunctions.net/multiply
         body:
           input: ${randomgenResult.body.random}
         result: multiplyResult
   - returnResult:
       return: ${multiplyResult}
   ```

2. Deploy the workflow:

   ```bash
   gcloud workflows deploy workflow --source=workflow.yaml
   ```

3. Execute the workflow:

   ```bash
   gcloud workflows execute workflow
   ```

4. Check the result of the workflow:
   ```bash
   gcloud workflows executions describe <your-execution-id> --workflow workflow
   ```

## 5. Deploy a Cloud Run Service

1. Create a Cloud Run service to compute `math.floor` of a number. Add the following Python code in `app.py`:

   ```python
   import json, math
   from flask import Flask, request

   app = Flask(__name__)

   @app.route('/', methods=['POST'])
   def handle_post():
       content = json.loads(request.data)
       input = float(content['input'])
       return f"math.floor({input})", 200

   if __name__ == '__main__':
       app.run(host='0.0.0.0', port=8080)
   ```

2. Create a `Dockerfile` with:

   ```
   FROM python:3.7-slim
   RUN pip install Flask gunicorn
   WORKDIR /app
   COPY . .
   CMD exec gunicorn --bind 0.0.0.0:8080 --workers 1 --threads 8 app:app
   ```

3. Build and deploy the service:
   ```bash
   export SERVICE_NAME=floor
   gcloud builds submit --tag gcr.io/$(gcloud config get-value project)/$SERVICE_NAME
   gcloud run deploy $SERVICE_NAME \
     --image gcr.io/$(gcloud config get-value project)/$SERVICE_NAME \
     --platform managed \
     --no-allow-unauthenticated
   ```

### Set Up IAM Roles for Cloud Run

1. Create a service account:

   ```bash
   export SERVICE_ACCOUNT=my-service-account
   gcloud iam service-accounts create ${SERVICE_ACCOUNT}
   ```

2. Grant the service account permissions to invoke the Cloud Run service:
   ```bash
   gcloud projects add-iam-policy-binding $(gcloud config get-value project) \
     --member "serviceAccount:${SERVICE_ACCOUNT}@$(gcloud config get-value project).iam.gserviceaccount.com" \
     --role "roles/run.invoker"
   ```

## 6. Update the Workflow

Update `workflow.yaml` to include the Cloud Run service:

```yaml
- floorFunction:
    call: http.post
    args:
      url: https://<cloud-run-url>
      auth:
        type: OIDC
      body:
        input: ${logResult.body}
    result: floorResult
- returnResult:
    return: ${floorResult}
```

Deploy the updated workflow:

```bash
gcloud workflows deploy workflow --source=workflow.yaml --service-account=${SERVICE_ACCOUNT}@$(gcloud config get-value project).iam.gserviceaccount.com
```

Execute and test the workflow as before.

## Conclusion

Emi Roberti - Happy coding
