# streamvideo_with_blur
Anonymize faces in video stream using Kafka, PySpark, CV Model

# How it's works?
video_recoder.py: record video to Kafka as stream

video_reader.py: read video from Kafka as stream

video_stream_job.py: PySpark stream job to process frames, detect face, blur and then record frame to Kafka.

Video Recorder -> Kafka(topic videostream_in) -> Video Stream Job -> Kafka(topic videostream_out) -> Video Reader

# Reproduce
- Up Kafka Brokers docker-compose up -d
- Create kafka topics (make sure set max.message.bytes property with value=1246000 or more):
    - VideoStream-in: kafka-topics --create --topic videostream_in --partitions 3 --bootstrap-server localhost:9092,localhost:9093,localhost:9094 --config max.message.bytes=1246000
    - VideoStream-out: kafka-topics --create --topic videostream_out --partitions 3 --bootstrap-server localhost:9092,localhost:9093,localhost:9094 --config max.message.bytes=1246000
- Submit Video Stream Job spark-submit --py-files app/utils.py, app/face_detector.py, app/video_stream_job.py
- Run python app/video_reader.py
- Run python app/video_recorder.py
