import json
import os
import boto3

sns = boto3.client("sns")
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

def lambda_handler(event, context):
    method = event.get("requestContext", {}).get("http", {}).get("method")
    if method == "OPTIONS":
        return resp(200, {"ok": True})

    try:
        body = json.loads(event.get("body") or "{}")
        email = (body.get("email") or "").strip()

        if not email or "@" not in email:
            return resp(400, {"error": "Invalid email"})

        out = sns.subscribe(
            TopicArn=TOPIC_ARN,
            Protocol="email",
            Endpoint=email,
            ReturnSubscriptionArn=True
        )

        
        return resp(200, {
            "message": "Confirmation email sent. Please confirm to start receiving notifications.",
            "subscriptionArn": out.get("SubscriptionArn")
        })

    except Exception as e:
        return resp(500, {"error": str(e)})
