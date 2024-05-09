provider "aws" {
  region = "us-east-1" 
}

# Create an S3 bucket using the AWS CLI command
resource "aws_s3_bucket" "my_bucket" {
  bucket = "terraform-vutivi-bck"  
  acl    = "private"  
}

# Wait for S3 bucket creation to complete
resource "null_resource" "create_s3_buckets" {
  depends_on = [aws_s3_bucket.my_bucket]

  provisioner "local-exec" {
    command = "sleep 10"  # Wait for 10 seconds to allow the bucket creation to propagate
  }
}

# Create an S3 bucket resource
resource "aws_s3_bucket" "terraform_vutivi_bck" {
  bucket = "terraform-vutivi-bck"  # Declare the name of the S3 bucket
}

resource "aws_s3_bucket_notification" "terraform_vutivi_bck" {
  bucket = aws_s3_bucket.terraform_vutivi_bck.id  # Specify the S3 bucket to send notifications from
}

# Create an SNS topic
resource "aws_sns_topic" "my_topic" {
  name = "terraform_topic"  # Update with your desired topic name
}

# create a dynamodb table
resource "aws_dynamodb_table" "my_table" {
  name           = "terraform"  # Update with your desired table name
  billing_mode   = "PAY_PER_REQUEST"
  hash_key       = "objectId"   # Define hash key
  range_key      = "timestamp"  # Define range key

  attribute {
    name = "objectId"
    type = "S"
  }

  attribute {
    name = "objectKey"
    type = "S"
  }
  attribute {
    name = "timestamp"
    type = "S"
  }

  global_secondary_index {
    name               = "ObjectKeyIndex"
    hash_key           = "objectKey"
    projection_type    = "ALL"  # Adjust based on your query requirements

    write_capacity     = 5     # Adjust based on your workload
    read_capacity      = 5     # Adjust based on your workload
  }
}

# Create a Lambda function
resource "aws_lambda_function" "my_function" {
  function_name    = "terraform_func"  # Update with your desired function name
  filename         = "lambda-function.zip"  # Path to your Lambda function code
  source_code_hash = filebase64sha256("lambda-function.zip")
  handler          = "lambda-function.lambda_handler"  # Update with your Lambda handler function
  runtime          = "python3.8"  # Update with your Lambda runtime
  role             = aws_iam_role.lambda_role.arn  # Update with your IAM role ARN

  environment {
    variables = {
      DYNAMODB_TABLE_NAME = aws_dynamodb_table.my_table.name
      SNS_TOPIC_ARN       = aws_sns_topic.my_topic.arn
    }
  }
}

# IAM role for Lambda execution
resource "aws_iam_role" "lambda_role" {
  name = "lambda_execution_role"

  assume_role_policy = jsonencode({
    "Version": "2012-10-17",
    "Statement": [{
      "Effect": "Allow",
      "Principal": {
        "Service": "lambda.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }]
  })
}

# IAM policy for DynamoDB access
resource "aws_iam_policy" "dynamodb_policy" {
  name        = "lambda_dynamodb_policy"
  description = "Allows Lambda function to access DynamoDB"

  policy = jsonencode({
    "Version": "2012-10-17",
    "Statement": [{
      "Effect": "Allow",
      "Action": [
        "dynamodb:PutItem",
        "dynamodb:GetItem"
      ],
      "Resource": aws_dynamodb_table.my_table.arn
    }]
  })
}

# Attach IAM policy to Lambda execution role
resource "aws_iam_role_policy_attachment" "lambda_dynamodb_attachment" {
  role       = aws_iam_role.lambda_role.name
  policy_arn = aws_iam_policy.dynamodb_policy.arn
}

# IAM policy for SNS access
resource "aws_iam_policy" "sns_policy" {
  name        = "lambda_sns_policy"
  description = "Allows Lambda function to publish to SNS"

  policy = jsonencode({
    "Version": "2012-10-17",
    "Statement": [{
      "Effect": "Allow",
      "Action": "sns:Publish",
      "Resource": aws_sns_topic.my_topic.arn
    }]
  })
}

# Attach IAM policy to Lambda execution role
resource "aws_iam_role_policy_attachment" "lambda_sns_attachment" {
  role       = aws_iam_role.lambda_role.name
  policy_arn = aws_iam_policy.sns_policy.arn
}
