import json
import os
import boto3
from datetime import datetime, timezone

s3 = boto3.client("s3")
sns = boto3.client("sns")

BUCKET = os.environ["BUCKET_NAME"]
KEY = os.environ.get("EVENTS_KEY", "events.json")
TOPIC_ARN = os.environ["TOPIC_ARN"]

def resp(status, body, origin="*"):
    return {
        "statusCode": status,
        "headers": {
            "Content-Type": "application/json",
            "Access-Control-Allow-Origin": origin,
            "Access-Control-Allow-Methods": "OPTIONS,POST",
            "Access-Control-Allow-Headers": "Content-Type",
        },
        "body": json.dumps(body, ensure_ascii=False),
    }

def load_events():
    obj = s3.get_object(Bucket=BUCKET, Key=KEY)
    data = json.loads(obj["Body"].read().decode("utf-8"))
    if "events" not in data or not isinstance(data["events"], list):
        data = {"events": []}
    return data

def save_events(data):
    s3.put_object(
        Bucket=BUCKET,
        Key=KEY,
        Body=json.dumps(data, ensure_ascii=False, indent=2).encode("utf-8"),
        ContentType="application/json",
        CacheControl="no-store"
    )

def lambda_handler(event, context):
  
    method = event.get("requestContext", {}).get("http", {}).get("method")
    if method == "OPTIONS":
        return resp(200, {"ok": True})

    try:
        body = json.loads(event.get("body") or "{}")

        title = (body.get("title") or "").strip()
        date = (body.get("date") or "").strip()  # YYYY-MM-DD
        location = (body.get("location") or "").strip()
        description = (body.get("description") or "").strip()

        if not title or not date:
            return resp(400, {"error": "Missing required fields: title, date"})

        new_event = {
            "id": f"evt_{int(datetime.now(timezone.utc).timestamp())}",
            "title": title,
            "date": date,
            "location": location,
            "description": description,
            "createdAt": datetime.now(timezone.utc).isoformat()
        }

        data = load_events()
        data["events"].append(new_event)
        save_events(data)

        
        message = "\n".join([
            f"New event created: {title}",
            f"Date: {date}",
            f"Location: {location}",
            "",
            description
        ]).strip()

        sns.publish(
            TopicArn=TOPIC_ARN,
            Subject=f"New Event: {title}"[:100],
            Message=message
        )

        return resp(200, {"message": "Event created", "event": new_event})

    except Exception as e:
        return resp(500, {"error": str(e)})
