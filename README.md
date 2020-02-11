# Pedestrian detector
This project looks at fine-tuning a pre-trained object detection model so that it performs better at the task of pedestrian detection.

The repo makes heavy use of Tensorflow's [Object Detection API](https://github.com/tensorflow/models/tree/master/research/object_detection). Set-up instructions are included in the quick start guide, but feel free to refer to the official API documentation if things don't make sense.

Provided below is a quick start guide on how to set things up and start inference asap.

There is also an extended guide discussing the fine-tuning pipeline, in the hope that results can be reproduced.

## Quick start inference
Start with the Python notebook [`model_inference.ipynb`](https://github.com/conorg000/ped-detector/blob/master/model_inference.ipynb). Use it in [Google Colab](https://colab.research.google.com/) to use their [free GPU](https://colab.research.google.com/notebooks/gpu.ipynb).

The notebook contains setup instructions for Tensorflow Object Detection API, loads the fine-tuned model, and performs inference on a test image (or any image you upload to Colab).

## Fine-tuning pipeline
A summary of the method:
- Get a pre-trained object detection model from [here](https://github.com/tensorflow/models/blob/master/research/object_detection/g3doc/detection_model_zoo.md).
- Get training/test data from [MOT](https://motchallenge.net/data/MOT16/).
- Pre-process detection annotations.
- Pre-process training/test data.
- Configure pre-trained model.
- Start training with new data from pre-trained model's checkpoint.
- Export the fine-tuned model.
- Perform inference on test data.
- Evaluate pre-trained model vs fine-tuned model.

### Pre-trained model
We use `ssd_mobilenet_v2_coco`, available for download [here](https://github.com/tensorflow/models/blob/master/research/object_detection/g3doc/detection_model_zoo.md).

It's MobileNetV2 trained on Microsoft's COCO dataset, topped with a Single Shot MultiBox Detector.

On a GPU, it's supposed to achieve 22 mAP (on COCO test set) and 31 ms inference speed on a single image.

### Data
We use Motion Object Tracking Benchmark challenge data, specifically from 2016 ([MOT16](https://motchallenge.net/data/MOT16/)).

Download the data (1.9GB) and then run the pre-processing scripts below. **Just extract the MOT16 zip file, no need to move the files anywhere yet. The scripts below will deal with that.**

### Pre-process detection annotations
MOT16's data has each video's annotations stored in separate files in different directories. Run [`make_labels.py`](https://github.com/conorg000/ped-detector/blob/master/scripts/make_labels.py) to end up with one file for training annotations (`train_det.txt`) and one file for test annotations (`test_det.txt`). **Change paths in script as needed**. The script also adjusts bounding box coordinates to cater for the upcoming image resizing.

We only use three directories of training data from MOT16 (02, 04, 09) and one directory of test data (01).

### Pre-processing images
We now move and resize the images. Run [`move_images.py`](https://github.com/conorg000/ped-detector/blob/master/scripts/move_images.py) to resize the necessary images (to 600 x 337 pixels) and move them within this repo. The images will end up in `ped-detector/images` directory, in their respective `/train` and `/test` directories.

Then make TFRecords using [`train_tf_records.py`](https://github.com/conorg000/ped-detector/blob/master/scripts/train_tf_records.py) and [`test_tf_records.py`](https://github.com/conorg000/ped-detector/blob/master/scripts/test_tf_records.py). These scripts convert the images and annotations to a data format required by the Object Detection API.

The command for the training data is: `python ped-detector/scripts/train_tf_records.py --output_path=ped-detector/annotations/train.record`

The command for the test data is: `python ped-detector/scripts/test_tf_records.py --output_path=ped-detector/annotations/test.record`

### Fine-tuning
Relevant scripts and commands

###
