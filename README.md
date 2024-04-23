import json
import boto3
import urllib.parse
from datetime import datetime

dynamodb = boto3.resource('dynamodb')
sns = boto3.client('sns')
s3 = boto3.client('s3')

dynamodb_table_name = 'Name'
sns_topic_arn = 'arn:aws:sns:topic_name'
bucket_name_arn ='arn:aws:s3:::bucket_name




def lambda_handler(event, context):
    try:
        # Extract values from the event
        bucket_name = event.get('key1')
        object_key = event.get('key2')
        if not bucket_name or not object_key:
         raise ValueError("Missing required fields in the event")

        object_id = f"{bucket_name}:{object_key}"

        # Save object details to DynamoDB
        save_to_dynamodb(object_id, object_key)

        # Trigger SNS topic to send notification via email
        send_sns_notification(object_id, object_key)

        return {
            'statusCode': 200,
            'body': json.dumps('Object details saved to DynamoDB and SNS notification sent')
        }
    except Exception as e:
        return {
            'statusCode': 500,
            'body': json.dumps(f'Error: {str(e)}')
        }


def save_to_dynamodb(object_id, object_key):
    timestamp = datetime.utcnow().isoformat()  # Generate current timestamp
    table = dynamodb.Table(dynamodb_table_name)
    table.put_item(
        Item={
            'objectId': object_id,
            'objectKey': object_key,
            'timestamp': timestamp  # Include the timestamp attribute
        }
    )
    print('Object details saved to DynamoDB:', {'objectId': object_id, 'objectKey': object_key, 'timestamp': timestamp})


def send_sns_notification(object_id, object_key):
    message = f'New object uploaded:\nObject ID: {object_id}\nObject Key: {object_key}'
    sns.publish(
        TopicArn=sns_topic_arn,
        Message=message,
        Subject='New Object Uploaded'
    )
    print('SNS notification sent:', message)
