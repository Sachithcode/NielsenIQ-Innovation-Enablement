# NielsenIQ-Innovation-Enablement

# NIQ Innovation Enablement - Challenge 1 (Object Counting)

The goal of this repo is demonstrate how to apply Hexagonal Architecture in a ML based system.

This application consists in a Flask API that receives an image and a threshold and returns the number of objects detected in the image.

The application is composed by 3 layers:

- **entrypoints**: This layer is responsible for exposing the API and receiving the requests. It is also responsible for validating the requests and returning the responses.

- **adapters**: This layer is responsible for the communication with the external services. It is responsible for translating the domain objects to the external services objects and vice-versa.

- **domain**: This layer is responsible for the business logic. It is responsible for orchestrating the calls to the external services and for applying the business rules.

The model used in this example has been taken from 
[IntelAI](https://github.com/IntelAI/models/blob/master/docs/object_detection/tensorflow_serving/Tutorial.md)


## Instructions to configure this project
```
# Download the rfcn model 
wget https://storage.googleapis.com/intel-optimized-tensorflow/models/v1_8/rfcn_resnet101_fp32_coco_pretrained_model.tar.gz
tar -xzvf rfcn_resnet101_fp32_coco_pretrained_model.tar.gz -C tmp
rm rfcn_resnet101_fp32_coco_pretrained_model.tar.gz
chmod -R 777 tmp/rfcn_resnet101_coco_2018_01_28
mkdir -p tmp/model/rfcn/1
mv tmp/rfcn_resnet101_coco_2018_01_28/saved_model/saved_model.pb tmp/model/rfcn/1
rm -rf tmp/rfcn_resnet101_coco_2018_01_28
```

## Setup and run Tensorflow Serving

```

# For unix systems
cores_per_socket=`lscpu | grep "Core(s) per socket" | cut -d':' -f2 | xargs`
num_sockets=`lscpu | grep "Socket(s)" | cut -d':' -f2 | xargs`
num_physical_cores=$((cores_per_socket * num_sockets))

docker rm -f tfserving
docker run \
    --name=tfserving \
    -p 8500:8500 \
    -p 8501:8501 \
    -v "$(pwd)\tmp\model:/models" \
    -e OMP_NUM_THREADS=$num_physical_cores \
    -e TENSORFLOW_INTER_OP_PARALLELISM=2 \
    -e TENSORFLOW_INTRA_OP_PARALLELISM=$num_physical_cores \
    intel/intel-optimized-tensorflow-serving:2.8.0 \
    --model_config_file=/models/model_config.config

# For Windows (Powershell)
$num_physical_cores=(Get-WmiObject Win32_Processor | Select-Object NumberOfCores).NumberOfCores
echo $num_physical_cores

docker rm -f tfserving
docker run `
    --name=tfserving `
    -p 8500:8500 `
    -p 8501:8501 `
    -v "$pwd\tmp\model:/models" `
    -e OMP_NUM_THREADS=$num_physical_cores `
    -e TENSORFLOW_INTER_OP_PARALLELISM=2 `
    -e TENSORFLOW_INTRA_OP_PARALLELISM=$num_physical_cores `
    intel/intel-optimized-tensorflow-serving:2.8.0 `
    --model_config_file=/models/model_config.config
```


## Run mongo 

```bash
docker rm -f test-mongo
docker run --name test-mongo --rm -p 27017:27017 -d mongo:latest
```


## Setup virtualenv

```bash
# Python >= 3.0
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

## Run the application

### Using fakes
```
python -m counter.entrypoints.webapp
```

### Using real services in docker containers

```
# Unix
ENV=prod python -m counter.entrypoints.webapp

# Powershell
$env:ENV = "prod"
python -m counter.entrypoints.webapp
```

## Call the service

```shell script
 curl -F "threshold=0.9" -F "file=@resources/images/boy.jpg" http://0.0.0.0:5000/object-count
 curl -F "threshold=0.9" -F "file=@resources/images/cat.jpg" http://0.0.0.0:5000/object-count
 curl -F "threshold=0.9" -F "file=@resources/images/food.jpg" http://0.0.0.0:5000/object-count 
```

## Run the tests

```
pytest
```

# Object Detection API

This project provides an object detection API using FastAPI, Flask, and PostgreSQL. It enables users to upload images and detect objects using different models like YOLO, SSD, and Faster R-CNN. The results are stored in a PostgreSQL database for future reference.

## Features
- **Image Upload**: Users can upload images for object detection.
- **Object Detection**: Supports multiple detection models (YOLO, SSD, Faster R-CNN).
- **Storage**: Detected objects and metadata are stored in PostgreSQL.
- **Multi-Model Support**: Users can choose different detection models.
- **API Endpoints**: Exposes RESTful APIs using FastAPI.

## Tech Stack
- **Backend**: FastAPI (Python)
- **Object Detection**: Flask (Python) with YOLO, SSD, Faster R-CNN
- **Database**: PostgreSQL
- **Storage**: Local filesystem for images

## Installation

### Prerequisites
- Python 3.8+
- PostgreSQL
- pip, virtualenv

### Steps
1. Clone the repository:
   ```sh
   git clone https://github.com/your-repo/object-detection-api.git
   cd object-detection-api
   ```
2. Set up a virtual environment:
   ```sh
   python -m venv venv
   source venv/bin/activate  # On Windows use `venv\Scripts\activate`
   ```
3. Install dependencies:
   ```sh
   pip install -r requirements.txt
   ```
4. Set up PostgreSQL and create a database:
   ```sql
   CREATE DATABASE object_detection;
   ```
   Update `.env` with database credentials:
   ```env
   DATABASE_URL=postgresql://user:password@localhost/object_detection
   ```
5. Apply database migrations:
   ```sh
   alembic upgrade head
   ```
6. Start the API server:
   ```sh
   uvicorn main:app --reload
   ```
7. Start the Flask object detection service:
   ```sh
   python detect_service.py
   ```

## API Endpoints
### Image Upload
```
POST /upload/
```
- Uploads an image for object detection.
- Parameters: `file` (image file)

### Retrieve Detected Objects
```
GET /detections/
```
- Returns a list of detected objects.

### Retrieve Detection by Image ID
```
GET /detections/{image_id}
```
- Retrieves objects detected in a specific image.

### Choose Detection Model
```
POST /set_model/
```
- Allows the user to select a different object detection model.
- Parameters: `model_name` (YOLO, SSD, Faster R-CNN)

## Future Enhancements
- Support for cloud storage (AWS S3, Google Cloud Storage)
- Model optimization for faster inference
- Frontend for image upload and visualization


