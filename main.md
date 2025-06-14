# LocalStack Setup for TinyFlix Project

This document explains the commands used to set up a local AWS environment using LocalStack for the TinyFlix project.

## 1. Starting LocalStack Container

```docker
docker run --rm -it -p 4566:4566 -p 4571:4571 -v //var/run/docker.sock:/var/run/docker.sock localstack/localstack
```

- **`docker run`**: Creates and starts a new container
- **`--rm`**: Automatically removes the container when it exits
- **`-it`**: Runs the container in interactive mode with a pseudo-TTY (terminal)
- **`-p 4566:4566 -p 4571:4571`**: Maps container ports to host ports:
  - `4566`: Default port for most LocalStack services
  - `4571`: Port for Simple Notification Service (SNS)
- **`-v //var/run/docker.sock:/var/run/docker.sock`**: Mounts the Docker socket from the host into the container, allowing LocalStack to manage Docker containers (used for Lambda execution)

## 2. Creating an S3 Bucket

```bash
awslocal s3 mb s3://tinyflix
```

- **`awslocal`**: CLI tool for interacting with LocalStack (equivalent to AWS CLI)
- **`s3 mb`**: Creates (makes) a new S3 bucket
- **`s3://tinyflix`**: Name of the bucket to create

## 3. Creating IAM Role for Lambda

```powershell
$assumeRolePolicy = @'
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "lambda.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
'@

$assumeRolePolicy | Out-File -Encoding ascii -FilePath .\assume-role-policy.json

awslocal iam create-role --role-name myTinyflixRole --assume-role-policy-document file://assume-role-policy.json
```

- Creates an IAM role that Lambda can assume
- The JSON policy document allows Lambda service to assume this role
- Saves the policy to a file and creates the role using `awslocal`

## 4. Packaging Lambda Function

```powershell
Compress-Archive -Path "path" -DestinationPath "path" -Force
```

- Zips the Lambda function code (`main.py`) into a deployment package
- `-Force` overwrites if the zip file already exists

## 5. Creating Lambda Function

```bash
awslocal lambda create-function \
  --function-name my-local-lambda \
  --runtime python3.9 \
  --handler main.lambda_handler \
  --role arn:aws:iam::000000000000:role/myTinyflixRole \
  --zip-file fileb://path
```

- Creates a Lambda function with:
  - Name: `my-local-lambda`
  - Python 3.9 runtime
  - Handler function `lambda_handler` in `main.py`
  - IAM role created earlier
  - Code from the zipped package

## 6. Adding S3 Invoke Permission

```bash
awslocal lambda add-permission \
  --function-name my-local-lambda \
  --statement-id s3invoke1 \
  --action "lambda:InvokeFunction" \
  --principal s3.amazonaws.com \
  --source-arn arn:aws:s3:::tinyflix
```

- Grants S3 permission to invoke the Lambda function
- `--statement-id`: Unique identifier for the permission
- `--principal`: Service being granted permission (S3)
- `--source-arn`: Specific S3 bucket that can trigger the Lambda

## 7. Configuring S3 Event Notifications

```powershell
@'
{
  "LambdaFunctionConfigurations": [
    {
      "LambdaFunctionArn": "arn:aws:lambda:us-east-1:000000000000:function:my-local-lambda",
      "Events": [
        "s3:ObjectCreated:*"
      ]
    }
  ]
}
'@ | Out-File -Encoding ascii -FilePath .\notification.json

awslocal s3api put-bucket-notification-configuration \
  --bucket tinyflix \
  --notification-configuration file://notification.json
```

- Creates a notification configuration that triggers the Lambda when:
  - Any object is created in the bucket (`s3:ObjectCreated:*`)
- Applies this configuration to the `tinyflix` bucket

## For test
### upload a file on S3
```docker
awslocal s3 path s3://tinyflix/
```
### Check Lambda logs
```docker
docker logs <container_id_of_localstack>
```
### somthing like this 
```bash
docker logs distracted-leakey-lambda-my-local-lambda-8706ecba95c15cccdf8ec923dafc4086 
```
