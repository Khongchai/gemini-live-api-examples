# Deploying to Google Cloud Run

This guide explains how to deploy the Gemini Live demo application to Google Cloud Run.

## Prerequisites

1.  **Google Cloud Project**: Have an active Google Cloud project.
2.  **Google Cloud CLI**: Install and initialize the [gcloud CLI](https://cloud.google.com/sdk/docs/install).
3.  **Gemini API Key**: Obtain an API key from [Google AI Studio](https://aistudio.google.com/).

## Setup Instructions

### 1. Enable APIs

Enable the Cloud Run and Cloud Build APIs in your project:

```bash
gcloud services enable run.googleapis.com cloudbuild.googleapis.com
```

### 2. Configure Environment Variables

The application requires the following environment variables:

- `GEMINI_API_KEY`: Your Gemini API key.
- `MODEL`: (Optional) The model name to use. Defaults to `gemini-3.1-flash-live-preview`.

### 3. Deploy to Cloud Run

Run the following command from the root of the repository to build and deploy the application:

```bash
gcloud run deploy gemini-live-demo \
    --source . \
    --env-vars-file .env.yaml \
    --allow-unauthenticated \
    --region us-central1
```

> [!TIP]
> Instead of using `--env-vars-file`, you can pass variables directly using `--set-env-vars GEMINI_API_KEY=YOUR_KEY`.

#### Create `.env.yaml` (Recommended)

To avoid exposing your API key in the command history, create a `.env.yaml` file:

```yaml
GEMINI_API_KEY: "YOUR_GEMINI_API_KEY"
MODEL: "gemini-3.1-flash-live-preview"
```

### 4. Access the Application

Once the deployment completes, the gcloud CLI will provide a Service URL (e.g., `https://gemini-live-demo-xxxx-uc.a.run.app`). Open this URL in your browser to interact with the demo.

## Twilio Integration (Optional)

If you are using the Twilio integration, store credentials in **Secret Manager** rather than `.env.yaml`:

```bash
# Enable the Secret Manager API
gcloud services enable secretmanager.googleapis.com

# Create secrets (reads values from your .env file)
echo -n "$(grep TWILIO_ACCOUNT_SID .env | cut -d '=' -f2)" | gcloud secrets create TWILIO_ACCOUNT_SID --data-file=-
echo -n "$(grep TWILIO_AUTH_TOKEN .env | cut -d '=' -f2)" | gcloud secrets create TWILIO_AUTH_TOKEN --data-file=-
```

Then deploy with `--set-secrets` to mount them as environment variables:

```bash
gcloud run deploy gemini-live-demo \
    --source . \
    --env-vars-file .env.yaml \
    --set-secrets TWILIO_ACCOUNT_SID=TWILIO_ACCOUNT_SID:latest,TWILIO_AUTH_TOKEN=TWILIO_AUTH_TOKEN:latest \
    --set-env-vars TWILIO_APP_HOST=your-cloud-run-url.run.app \
    --allow-unauthenticated \
    --region us-central1
```

After deploying, update your Twilio Webhook URL in the Twilio Console to point to `https://YOUR_CLOUD_RUN_URL/twilio/inbound`.

## Local Testing with Docker

Before deploying, you can test the container locally:

```bash
# Build the image
docker build -t gemini-live-demo .

# Run the container
docker run -p 8080:8080 \
    -e GEMINI_API_KEY=YOUR_API_KEY \
    -e PORT=8080 \
    gemini-live-demo
```
