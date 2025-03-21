# AWS Machine Learning Security System

## Overview
This project builds an intruder detection system using machine learning with AWS services. It captures video from a laptop webcam, processes frames in real time, and detects intruders using Amazon Rekognition. The system then stores data in Amazon S3 and DynamoDB and sends alerts via Amazon SNS.

## Tech Stack
- **Laptop Webcam**: Captures video frames
- **Amazon Kinesis Video Stream**: Streams video for processing
- **Amazon EC2 & Amazon S3**: Stores backup data
- **Amazon Rekognition**: Detects people in the video
- **Amazon DynamoDB**: Stores detected persons' information
- **Amazon SNS**: Sends alerts upon intruder detection

## Prerequisites
Ensure you have the following installed:
- **Ubuntu Linux**
- **AWS CLI configured**
- **Python 3**

## Setup
### Install Required Dependencies
```sh
sudo apt update
git clone https://github.com/awslabs/amazon-kinesis-video-streams-producer-sdk-cpp.git
mkdir -p amazon-kinesis-video-streams-producer-sdk-cpp/build
cd amazon-kinesis-video-streams-producer-sdk-cpp/build
sudo apt-get install libssl-dev libcurl4-openssl-dev liblog4cplus-dev libgstreamer1.0-dev \
  libgstreamer-plugins-base1.0-dev gstreamer1.0-plugins-base-apps gstreamer1.0-plugins-bad \
  gstreamer1.0-plugins-good gstreamer1.0-plugins-ugly gstreamer1.0-tools
sudo apt install cmake
gsudo apt-get install g++ build-essential
cmake .. -DBUILD_DEPENDENCIES=OFF -DBUILD_GSTREAMER_PLUGIN=ON
make
sudo make install
cd ..
export GST_PLUGIN_PATH=`pwd`/build
export LD_LIBRARY_PATH=`pwd`/open-source/local/lib
```

### Setting Up AWS Resources
#### 1. Create a Kinesis Video Stream
- Go to AWS Kinesis Video Streams
- Create a new stream

#### 2. Configure IAM User
- Go to AWS IAM
- Create a new user with `AmazonKinesisVideoStreamsFullAccess` permissions
- Generate access keys

#### 3. Start Video Streaming
```sh
gst-launch-1.0 v4l2src do-timestamp=TRUE device=/dev/video0 ! videoconvert ! \
  video/x-raw,format=I420,width=640,height=480,framerate=30/1 ! x264enc bframes=0 \
  key-int-max=45 bitrate=500 ! video/x-h264,stream-format=avc,alignment=au,profile=baseline \
  ! kvssink stream-name='Intruder-detection-video-stream' storage-size=512 \
  access-key='YOUR_ACCESS_KEY' secret-key='YOUR_SECRET_KEY' aws-region='us-east-1'
```

## Setting Up Consumers
### Consumer #1: Backup
1. **Create an S3 Bucket**
   - Name: `intruder-detection-bucket`
   - Create a `backup` folder
2. **Launch an EC2 Instance**
   - Instance Type: `t2.medium`
   - Storage: `8GB`
   - AMI: Ubuntu Linux
3. **Assign IAM Role**
   - Permissions: `AmazonKinesisVideoStreamsFullAccess`, `AmazonS3FullAccess`
4. **SSH into EC2**
```sh
ssh -i ~/ssh/aws-ft-key-pair.pem ubuntu@<EC2-PUBLIC-IP>
```
5. **Install and Setup Python Environment**
```sh
sudo apt update
sudo apt install python3-virtualenv
virtualenv venv --python=python3
source venv/bin/activate
git clone https://github.com/aws-samples/amazon-kinesis-video-streams-consumer-library-for-python.git
cd amazon-kinesis-video-streams-consumer-library-for-python
pip install -r requirements.txt
```
6. **Run Backup Process**
```sh
python kvs_consumer_library_example.py
```

### Consumer #2: Intruder Detection
1. **Create an S3 Bucket**
   - Add a `detections` folder
2. **Create a DynamoDB Table**
   - Store detected person information
3. **Setup SNS for Alerts**
   - Create a new SNS topic
   - Create an email subscription
4. **Create S3 Event Notification**
5. **Launch an EC2 Instance**
   - Instance Type: `t2.medium`
   - Storage: `8GB`
   - AMI: Ubuntu Linux
6. **Assign IAM Role**
   - Permissions: `AmazonKinesisVideoStreamsFullAccess`, `AmazonDynamoDBFullAccess`, `AmazonS3FullAccess`, `AmazonRekognitionFullAccess`
7. **SSH into EC2**
```sh
ssh -i ~/ssh/aws-ft-key-pair.pem ubuntu@<EC2-PUBLIC-IP>
```
8. **Install and Setup Python Environment**
```sh
sudo apt update
sudo apt install python3-virtualenv
virtualenv venv --python=python3
source venv/bin/activate
git clone https://github.com/aws-samples/amazon-kinesis-video-streams-consumer-library-for-python.git
cd amazon-kinesis-video-streams-consumer-library-for-python
pip install -r requirements.txt
sudo apt-get update && sudo apt-get install ffmpeg libsm6 libxext6 -y
```
9. **Run Intruder Detection**
```sh
python kvs_consumer_library_example.py
```

## Conclusion
This project successfully sets up an AWS-based intruder detection system, utilizing real-time video streaming, machine learning, and cloud services for security automation.
