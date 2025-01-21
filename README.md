# Steps to Build, Create, Deploy, and Execute GCP Workflows

## 1. **Set Up Your Environment**

### Prerequisites:

- A Google Cloud account.
- A GCP project with billing enabled.
- The `gcloud` CLI installed and configured.

### Steps:

1. Authenticate to GCP:

   ```bash
   gcloud auth login
   ```

2. Set your project:

   ```bash
   gcloud config set project [PROJECT_ID]
   ```

3. Enable the Workflows API:

   ```bash
   gcloud services enable workflows.googleapis.com
   ```

4. Optional: Create a Cloud Storage bucket for logs:

   ```bash
   gsutil mb -p [PROJECT_ID] -l [REGION] gs://[BUCKET_NAME]/
   ```

---

## 2. **Build a Workflow**

### Example Workflow Definition (`workflow.yaml`):

```yaml
main:
  steps:
    - logStart:
        call: sys.log
        args:
          text: "Workflow started"
    - fetchWeather:
        call: http.get
        args:
          url: "https://api.open-meteo.com/v1/forecast?latitude=35&longitude=139&hourly=temperature_2m"
    - logResponse:
        call: sys.log
        args:
          text: ${"Weather API response: " + fetchWeather.body}
    - workflowEnd:
        return: "Workflow finished"
```

Save this file as `workflow.yaml`.

---

## 3. **Create and Deploy the Workflow**

1. Deploy the workflow:

   ```bash
   gcloud workflows deploy [WORKFLOW_NAME] --source=workflow.yaml --location=[REGION]
   ```

   Replace:

   - `[WORKFLOW_NAME]` with your desired workflow name.
   - `[REGION]` with your preferred region (e.g., `us-central1`).

2. Confirm successful deployment:

   ```bash
   gcloud workflows list
   ```

---

## 4. **Execute the Workflow**

1. Run the workflow:

   ```bash
   gcloud workflows execute [WORKFLOW_NAME] --location=[REGION]
   ```

2. View the execution details in the terminal.

3. To see logs:

   ```bash
   gcloud workflows executions describe [EXECUTION_ID] --workflow=[WORKFLOW_NAME] --location=[REGION]
   ```

   Replace `[EXECUTION_ID]` with the ID returned during execution.

---

## 5. **Debugging and Logs**

1. View logs in the Cloud Console:

   - Navigate to [Logs Explorer](https://console.cloud.google.com/logs) and filter by "Workflow Executions."

2. Add debug steps to your workflow to troubleshoot:

   ```yaml
   - debugLog:
       call: sys.log
       args:
         text: ${"Debugging: " + fetchWeather}
   ```

---

## 6. **Best Practices**

- **Indentation**: Ensure proper YAML formatting (two spaces per level).
- **Validation**: Use online YAML validators to check for syntax errors.
- **Error Handling**: Add error handling in workflows using conditional steps and retries.
- **Testing**: Test workflows with minimal configurations before adding complexity.

---

## 7. **Additional Resources**

- [GCP Workflows Documentation](https://cloud.google.com/workflows/docs)
- [gcloud CLI Reference](https://cloud.google.com/sdk/gcloud)

Emi Roberti - Happy coding
