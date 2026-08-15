# Real-Time Media Stream Processing Pipeline (Kafka ➔ PySpark Streaming ➔ OpenCV)

## 📌 Overview
A high-throughput, low-latency stream processing pipeline designed to handle live video data feeds. The system captures frames in real-time, publishes them to a distributed message broker, runs real-time machine learning inference for face detection via optimized Spark vector operations, anonymizes sensitive visual data, and pipes the result back to an output stream.

## 🛠️ Tech Stack
* **Streaming & Messaging:** Apache Kafka (Kafka Clusters, Producers, Consumers)
* **Stream Processing:** PySpark (Structured Streaming)
* **Computer Vision & ML:** OpenCV, Haar Cascades / DNN face detection models
* **Performance Optimization:** PyArrow, Spark Pandas UDF (User Defined Functions)
* **Deployment:** Docker

## 🚀 Key Implementation Features
1. **Distributed Frame Ingestion:** OpenCV captures live video frames, serializes them into bytes, and transmits them to an Apache Kafka cluster with optimal partitioning to handle high-frequency write throughput.
2. **Vectorized ML Inference:** Integrated the computer vision model inside a **Pandas UDF**, enabling PySpark to leverage Apache Arrow for zero-serialization data transfer and execute vectorized ML inference on batch arrays instead of row-by-row processing.
3. **Real-Time Data Masking:** Faces are detected and dynamically anonymized (Gaussian blur applied) on the fly with sub-second latency, adhering to strict data privacy regulations (GDPR/Compliance).

## ⚙️ How it's works?
app/video_recoder.py: record video to Kafka as stream

app/video_reader.py: read video from Kafka as stream

app/video_stream_job.py: PySpark stream job to process frames, detect face, blur and then record frame to Kafka.

```text
[ Camera / Video Source ] 
       │
       ▼ (OpenCV Frame Capture)
[ Apache Kafka Producer ] ➔ Topic: `raw-video-frames`
                                      │
                                      ▼
[ PySpark Structured Streaming ] (Consumes stream)
       │
       ▼ (ML Inference via Vectorized Pandas UDF)
[ Face Detection & Blurring Engine ]
       │
       ▼
[ Apache Kafka Consumer ] ➔ Topic: `processed-video-frames`
```

## ▶ Reproduce
- Up Kafka Brokers
 ```docker-compose up -d```
- Create kafka topics (make sure set max.message.bytes property with value=1246000 or more):
```bash
# VideoStream-in:
docker-compose exec kafka1 kafka-topics --create --topic videostream_in --partitions 2 --replication-factor 2 --bootstrap-server kafka1:19092,kafka2:19093 --config max.message.bytes=1246000
# VideoStream-out:
docker-compose exec kafka1 kafka-topics --create --topic videostream_out --partitions 2 --replication-factor 2 --bootstrap-server kafka1:19092,kafka2:19093 --config max.message.bytes=1246000
```
- Submit Video Stream Job: 
 ```bash
  # In cluster mode:
  spark-submit --py-files app/utils.py,app/face_detector.py app/video_stream_job.py
  # Or locally:
  python app/video_stream_job.py
  ```
- Run ```python app/video_reader.py```
- Run ```python app/video_recorder.py```

