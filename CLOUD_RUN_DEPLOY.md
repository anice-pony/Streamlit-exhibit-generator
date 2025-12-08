# Google Cloud Run Deployment Guide

## Prerequisites

1. **Google Cloud SDK** installed and authenticated
2. **Docker** installed (for local testing)
3. **Google Cloud Project** with:
   - Cloud Run API enabled
   - Container Registry API enabled
   - Appropriate permissions (Cloud Run Admin, Service Account User)

## Step 1: Build and Push the Docker Image

Navigate to the `streamlit-exhibit-generator` directory:

```bash
cd streamlit-exhibit-generator
```

Build and push the image to Google Container Registry:

```bash
gcloud builds submit --tag gcr.io/exhibits-480112/streamlit-app:latest
```

**OR** if you're in the root directory, specify the path:

```bash
gcloud builds submit --tag gcr.io/exhibits-480112/streamlit-app:latest streamlit-exhibit-generator/
```

## Step 2: Deploy to Cloud Run

Once the image is built and pushed, deploy it:

```bash
gcloud run deploy streamlit \
  --image gcr.io/exhibits-480112/streamlit-app:latest \
  --platform managed \
  --region us-east1 \
  --allow-unauthenticated \
  --port 8080 \
  --memory 2Gi \
  --cpu 2 \
  --timeout 3600 \
  --max-instances 10
```

## Important Notes

### Build Context
- The build must run from the `streamlit-exhibit-generator` directory OR
- You must specify the directory path in the build command
- The Dockerfile must be named `Dockerfile` (capital D)

### Resource Requirements
- **Memory**: Set to 2Gi (or higher) for PDF processing
- **CPU**: Set to 2 for better performance
- **Timeout**: Set to 3600s (1 hour) for large file processing
- **Port**: Must be 8080 (Cloud Run default)

### Environment Variables (if needed)
If you need to set environment variables:

```bash
gcloud run deploy streamlit \
  --image gcr.io/exhibits-480112/streamlit-app:latest \
  --platform managed \
  --region us-east1 \
  --allow-unauthenticated \
  --set-env-vars "KEY1=value1,KEY2=value2"
```

### Troubleshooting

**Issue: "Dockerfile not found"**
- Make sure you're in the `streamlit-exhibit-generator` directory
- Or specify the path: `gcloud builds submit --tag ... streamlit-exhibit-generator/`

**Issue: "Permission denied"**
- Run: `gcloud auth login`
- Check project: `gcloud config get-value project`
- Set project: `gcloud config set project exhibits-480112`

**Issue: "Build fails"**
- Check the build logs: `gcloud builds list`
- View specific build: `gcloud builds log [BUILD_ID]`

**Issue: "Container crashes"**
- Check logs: `gcloud run services logs read streamlit --region us-east1`
- Ensure PORT environment variable is set to 8080
- Verify Streamlit is configured for headless mode

## Testing Locally

Before deploying, test the Docker image locally:

```bash
# Build locally
docker build -t streamlit-app:local .

# Run locally
docker run -p 8080:8080 streamlit-app:local

# Test at http://localhost:8080
```
