# 🚀 Simple Example: AWS Lambda with S3 Trigger
## 📂 Automatically Trigger a Lambda Function When a File is Uploaded to S3

This is a simple example demonstrating how to use an **AWS Lambda function** that gets triggered whenever a file is uploaded to an **S3 bucket**, all running locally using **LocalStack**.

---

## 🔧 What It Does

1. 🐳 **Starts a LocalStack Docker container** to emulate AWS services.
2. 🛢️ **Creates an S3 bucket** named `tinyflix`.
3. 🧾 **Creates an IAM role** that allows Lambda to assume execution permissions.
4. 📦 **Packages the Lambda function code** into a zip file for deployment.
5. 🚀 **Deploys the Lambda function** using the zipped package and assigned role.
6. 🔐 **Grants S3 permission** to invoke the Lambda function.
7. 🔁 **Configures S3 event notifications** to trigger the Lambda on file upload.
8. ✅ **Tests the setup** by uploading a file to the S3 bucket and checking the Lambda logs via Docker.

---

## 📌 Requirements

- 🐳 **Docker** installed and running on your machine  
- 🔁 **LocalStack** Docker image pulled (`localstack/localstack`)  
- 🧰 **AWS CLI** and `awslocal` installed (`pip install awscli-local`)  
- 📄 Your Lambda function code (e.g., `main.py`) in a local folder  
- 💻 Windows PowerShell (or an equivalent terminal that supports `Compress-Archive`, `Out-File`, etc.)

---

✨ Looks good to go!
