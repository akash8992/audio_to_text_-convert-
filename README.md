# Convert Audio to Text Using AWS Transcribe and Python (Mac & Windows Guide)

## Introduction
AWS Transcribe is a powerful tool that allows you to convert speech into text using machine learning. In this guide, we will walk through the process of setting up and running a Python script to transcribe audio files on both **Mac** and **Windows** systems.

## Prerequisites
Before proceeding, ensure you have the following:

### Required Accounts & Tools
- An **AWS Account** (Sign up at [AWS](https://aws.amazon.com/))
- An **IAM User** with permissions to use S3 and Transcribe
- **Python 3.x** installed
- **AWS CLI** installed and configured

## Step 1: Install Required Software

### Mac Users
1. Install **Homebrew** (if not installed):
   ```sh
   /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
   ```
2. Install Python 3 (if not installed):
   ```sh
   brew install python3
   ```
3. Install AWS CLI:
   ```sh
   brew install awscli
   ```
4. Verify installations:
   ```sh
   python3 --version
   aws --version
   ```

### Windows Users
1. Install **Python 3** from [Python's official website](https://www.python.org/downloads/)
2. Install **AWS CLI** from [AWS CLI installer](https://aws.amazon.com/cli/)
3. Add Python and AWS CLI to system PATH
4. Verify installations:
   ```powershell
   python --version
   aws --version
   ```

## Step 2: Configure AWS CLI
Run the following command to set up AWS credentials:
```sh
aws configure
```
It will prompt you for:
- **AWS Access Key ID**
- **AWS Secret Access Key**
- **Region** (e.g., `ap-south-1` for India)
- **Output format** (leave blank or type `json`)

## Step 3: Install Required Python Libraries
Run the following command to install the required Python packages:
```sh
pip install boto3 requests
```

## Step 4: Create an S3 Bucket
1. Go to the AWS **S3 Console**: [https://s3.console.aws.amazon.com](https://s3.console.aws.amazon.com)
2. Click **Create Bucket**
3. Choose a unique name (e.g., `my-transcribe-bucket-123`)
4. Select a region
5. Make sure "Block all public access" is unchecked
6. Click **Create**

## Step 5: Create the Python Script
Create a file named **audiototext.py** and copy the following code:

```python
import os
import time
import boto3
import requests

# AWS Configuration
AWS_REGION = "ap-south-1"  # Change if needed
BUCKET_NAME = "your-s3-bucket-name"  # Replace with your S3 bucket name

# Local paths
INPUT_FOLDER = "./recordings"  # Update with the correct path
OUTPUT_FOLDER = "./transcriptions"  # Update with the correct path

# Ensure output folder exists
os.makedirs(OUTPUT_FOLDER, exist_ok=True)

# Initialize AWS clients
s3 = boto3.client("s3")
transcribe = boto3.client("transcribe", region_name=AWS_REGION)

def upload_to_s3(file_path, bucket_name):
    file_name = os.path.basename(file_path)
    s3.upload_file(file_path, bucket_name, file_name)
    return f"s3://{bucket_name}/{file_name}"

def start_transcription(job_name, file_url):
    response = transcribe.start_transcription_job(
        TranscriptionJobName=job_name,
        Media={"MediaFileUri": file_url},
        MediaFormat="mp3",
        LanguageCode="en-US",
        OutputBucketName=BUCKET_NAME
    )
    print(f"Started transcription job: {job_name}")

def wait_for_transcription(job_name):
    while True:
        status = transcribe.get_transcription_job(TranscriptionJobName=job_name)
        job_status = status["TranscriptionJob"]["TranscriptionJobStatus"]
        print(f"Job status: {job_status}")
        if job_status in ["COMPLETED", "FAILED"]:
            break
        time.sleep(5)
    if job_status == "COMPLETED":
        return status["TranscriptionJob"]["Transcript"]["TranscriptFileUri"]
    return None

def download_transcript(transcript_url, output_file):
    response = requests.get(transcript_url)
    if response.status_code == 200:
        text = response.json()["results"]["transcripts"][0]["transcript"]
        with open(output_file, "w", encoding="utf-8") as f:
            f.write(text)
        print(f"Saved transcription: {output_file}")
    else:
        print(f"Failed to download transcript. Status code: {response.status_code}")

def process_audio_files():
    for file_name in os.listdir(INPUT_FOLDER):
        if file_name.endswith(".mp3"):
            file_path = os.path.join(INPUT_FOLDER, file_name)
            print(f"Processing {file_name}...")
            file_url = upload_to_s3(file_path, BUCKET_NAME)
            job_name = f"transcribe_{file_name.replace('.mp3', '').replace(' ', '_')}"
            start_transcription(job_name, file_url)
            transcript_url = wait_for_transcription(job_name)
            if transcript_url:
                output_file = os.path.join(OUTPUT_FOLDER, f"{file_name.replace('.mp3', '.txt')}")
                download_transcript(transcript_url, output_file)
            else:
                print(f"Transcription failed for {file_name}")

process_audio_files()
```

## Step 6: Run the Script
Make sure your `.mp3` files are placed in the `recordings` folder. Then, execute the script:

### Mac:
```sh
python3 audiototext.py
```

### Windows:
```powershell
python audiototext.py
```

## Step 7: Access the Transcription
The text output will be saved in the `transcriptions` folder.

## Troubleshooting
- **403 Error When Downloading Transcript**: Ensure your S3 bucket policy allows public access or modify the ACL for the object.
- **AWS Credentials Not Found**: Reconfigure using `aws configure`.
- **Invalid Bucket Name**: Ensure your bucket name matches the one in AWS S3.

## Conclusion
By following these steps, you can efficiently convert speech into text using AWS Transcribe. This automation can be useful for interviews, meetings, podcasts, and more!

Happy coding!

