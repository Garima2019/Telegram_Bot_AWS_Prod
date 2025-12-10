# 🚀 Telegram Smart Storage Bot (AWS + Terraform)

An advanced Telegram bot built on AWS Lambda, API Gateway v2, DynamoDB, and S3, deployed using Terraform.
The bot can:
- Store user text messages
- Accept & store photos, documents, and voice messages
- Provide AI responses (OpenAI / Gemini)
- Save key-value data
- Search through history
- Provide personal analytics
- Maintain file metadata
- Support webhook via API Gateway


The full bot logic is implemented in ```handler.py```

## 📌 Features
### 📝 Text & Commands
- ```/start, /help, /menu```
- ```/echo <text>```
- ```/save <key> <value>``` & ```/get <key>```
- ```/list``` – list saved custom keys
- ```/search <keyword>``` – search messages
- ```/latest, /history```
- ```/stats``` – personal analytics
- ```/ask``` – ask OpenAI or Gemini
- ```/summarize``` – summarise previous notes


### 📁 File Storage (S3)
- Photos
- Documents (PDF, Word, etc.)
- Voice messages

Each file is:

- Downloaded from Telegram servers
- Uploaded to AWS S3
- Stored with unique S3 keys
- Logged in DynamoDB with metadata

(See ```handle_photo, handle_document, handle_voice``` in code.)

### 🔐 AWS Services Used
- AWS Lambda (Python handler)
- Amazon API Gateway HTTP API v2 (webhook endpoint)
- Amazon DynamoDB (user messages, files, metadata)
- Amazon S3 (file storage)
- CloudWatch Logs (logging)
- IAM (Lambda permissions)

### 🧱 Architecture Overview
```
Telegram  →  API Gateway (HTTP API v2)
                  ↓
              AWS Lambda
                  ↓
       ┌──────────────┬──────────────┐
       │              │              │
     DynamoDB         S3           OpenAI/Gemini
   (messages +      (file          (optional AI
     metadata)      storage)         replies)
```

### 📂 Folder Structure
``` bash
.
├── handler.py               # Lambda function code
├── main.tf                  # Full AWS infrastructure
├── variables.tf             # Input variables
├── outputs.tf               # Outputs (Webhook URL, ARNs, etc.)
└── terraform.tfvars         # Bot token + secrets (not committed)
```

### ⚙️ Requirements

- AWS account
- Terraform ≥ 1.7
- Python 3.11 runtime (AWS Lambda)
- Telegram bot token (from @BotFather)

## 🔧 Terraform Setup & Deployment
### 1️⃣ Clone the repo
```bash
git clone <repo-url>
cd your-repo
```
### 2️⃣ Create ```terraform.tfvars```
```bash
telegram_bot_token = "YOUR_TELEGRAM_BOT_TOKEN"
openai_api_key     = "YOUR_OPENAI_API_KEY"     # optional
gemini_api_key     = "YOUR_GEMINI_API_KEY"     # optional
s3_bucket_name     = "telegram-bot-files-UNIQUE-NAME"
environment        = "dev"
```
### 3️⃣ Initialize Terraform
```bash
terraform init
```
### 4️⃣ Review planned resources
```bash
terraform plan
```
### 5️⃣ Deploy the entire stack
```bash
terraform apply
```
### 🔗 Configure Telegram Webhook

After Terraform finishes:
```
terraform output webhook_url
```

Set the webhook:
```
$BotToken   = "YOUR_BOT_TOKEN"
$WebhookUrl = "<output from terraform>"

$uri = "https://api.telegram.org/bot$BotToken/setWebhook?url=$([uri]::EscapeDataString($WebhookUrl))"
Invoke-WebRequest -Uri $uri -Method Get | Select-Object -ExpandProperty Content
```

Verify:
```
Invoke-WebRequest -Uri "https://api.telegram.org/bot$BotToken/getWebhookInfo" -Method Get
```
### 🎤 Supported User Actions
Users can simply:
- Send text
- Send photos
- Send documents
- Send voice notes

The bot automatically:
- Downloads file from Telegram
- Uploads it to S3
- Creates metadata in DynamoDB
- Sends confirmation

### 🗂 DynamoDB Schema

Partition key: ```user_id```
Sort key prefixes:

```msg#123``` → stored messages

```kv#key``` → key-value store

```file#file_id``` → file metadata
### 📁 S3 Storage Layout
```
s3://<bucket-name>/
   └── <user_id>/
         ├── photos/
         ├── documents/
         └── voice/
```
### 🧪 Testing
1. Send text → bot should store & reply
2. Send photo → bot saves to S3, logs to DynamoDB
3. Send a document → stored properly
4. Send voice note → saved in S3
5. Run commands:
```
  /myfiles
  /fileinfo <id>
  /search hello
  /stats
```
6. Tail Lambda logs
   
``` aws logs tail "/aws/lambda/telegram-bot-lambda-v2" --follow --since 15m ```

### 🛑 Cleanup

``` terraform destroy ```

### 📜 License

MIT License — you're free to use, modify, or extend the bot.
