# Event Announcement Website (AWS)

A simple serverless event announcement system using:
- S3 Static Website Hosting (frontend + events.json)
- API Gateway (HTTP API)
- AWS Lambda (2 functions)
- SNS (email notifications)

## Architecture
- Frontend hosted on S3
- `POST /subscribe` -> Lambda -> SNS subscribe (email confirmation)
- `POST /new-event` -> Lambda -> update `events.json` in S3 + SNS publish

## Repo structure
- `frontend/` - Static website files uploaded to S3
- `lambda/subscribe/` - Lambda for SNS email subscription
- `lambda/create_event/` - Lambda for creating events and sending notifications

## Deployment (manual)
### 1) Frontend (S3)
Upload contents of `frontend/` to your S3 bucket and enable Static Website Hosting.

### 2) SNS
Create a Standard SNS Topic and keep the Topic ARN.

### 3) Lambda env vars
**subscribe Lambda**
- `TOPIC_ARN`

**create_event Lambda**
- `BUCKET_NAME`
- `EVENTS_KEY` (default: events.json)
- `TOPIC_ARN`

### 4) API Gateway (HTTP API)
Create routes:
- `POST /subscribe` -> subscribe Lambda
- `POST /new-event` -> create_event Lambda

Enable CORS:
- origins: `*` (or your S3 website URL)
- methods: `POST`, `OPTIONS`
- headers: `content-type`

### 5) Frontend config
Set API base URL in `frontend/config.js`:
```js
window.API_BASE = "https://<api-id>.execute-api.<region>.amazonaws.com";
