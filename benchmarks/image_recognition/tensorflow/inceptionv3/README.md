# Inception V3

This document has instructions for how to run Inception V3 for the
following modes/precisions:
* [Int8 inference](#int8-inference-instructions)
* [FP32 inference](#fp32-inference-instructions)

Benchmarking instructions and scripts for model training and inference
other precisions are coming later.

## Int8 Inference Instructions

1. Clone this [intelai/models](https://github.com/IntelAI/models)
repository:

```
$ git clone https://github.com/IntelAI/models.git
```

This repository includes launch scripts for running benchmarks and the
an optimized version of the inceptionv3 model code.

2. Clone the [tensorflow/models](https://github.com/tensorflow/models)
repository:

```
git clone https://github.com/tensorflow/models.git
```

This repository is used for dependencies that the Inception V3 model
requires.

3. Download the pre-trained model.
```
$ wget https://storage.googleapis.com/intel-optimized-tensorflow/models/inceptionv3_int8_pretrained_model.pb
```

4. If you would like to run Inception V3 inference and test for
accuracy, you will need the ImageNet dataset. Benchmarking for latency
and throughput do not require the ImageNet dataset.

Register and download the
[ImageNet dataset](http://image-net.org/download-images).

Once you have the raw ImageNet dataset downloaded, we need to convert
it to the TFRecord format. This is done using the
[build_imagenet_data.py](https://github.com/tensorflow/models/blob/master/research/inception/inception/data/build_imagenet_data.py)
script. There are instructions in the header of the script explaining
its usage.

After the script has completed, you should have a directory with the
sharded dataset something like:

```
$ ll /home/myuser/datasets/ImageNet_TFRecords
-rw-r--r--. 1 user 143009929 Jun 20 14:53 train-00000-of-01024
-rw-r--r--. 1 user 144699468 Jun 20 14:53 train-00001-of-01024
-rw-r--r--. 1 user 138428833 Jun 20 14:53 train-00002-of-01024
...
-rw-r--r--. 1 user 143137777 Jun 20 15:08 train-01022-of-01024
-rw-r--r--. 1 user 143315487 Jun 20 15:08 train-01023-of-01024
-rw-r--r--. 1 user  52223858 Jun 20 15:08 validation-00000-of-00128
-rw-r--r--. 1 user  51019711 Jun 20 15:08 validation-00001-of-00128
-rw-r--r--. 1 user  51520046 Jun 20 15:08 validation-00002-of-00128
...
-rw-r--r--. 1 user  52508270 Jun 20 15:09 validation-00126-of-00128
-rw-r--r--. 1 user  55292089 Jun 20 15:09 validation-00127-of-00128
```

5. Next, navigate to the `benchmarks` directory in your local clone of
the [intelai/models](https://github.com/IntelAI/models) repo from step 1.
The `launch_benchmark.py` script in the `benchmarks` directory is
used for starting a benchmarking run in a optimized TensorFlow docker
container. It has arguments to specify which model, framework, mode,
precision, and docker image to use, along with your path to the ImageNet
TF Records that you generated in step 4.

Substitute in your own `--data-location` (from step 4, for accuracy
only), `--in-graph` pretrained model file path (from step 3) and
`--model-source-dir` for the location where you cloned the
[tensorflow/models](https://github.com/tensorflow/models) repo
(from step 2).

Inception V3 can be run for accuracy, latency benchmarking, or throughput
benchmarking. Use one of the following examples below, depending on
your use case.

For accuracy (using your `--data-location`, `--accuracy-only` and
`--batch-size 100`):

```
python launch_benchmark.py \
    --model-name inceptionv3 \
    --precision int8 \
    --mode inference \
    --framework tensorflow \
    --accuracy-only \
    --batch-size 100 \
    --docker-image intelaipg/intel-optimized-tensorflow:PR25765-devel-mkl \
    --in-graph /home/myuser/inceptionv3_int8_pretrained_model.pb \
    --data-location /home/myuser/datasets/ImageNet_TFRecords \
    -- input_height=299 input_width=299
```

When running performance benchmarking, it is optional to specify the
number of `warmup_steps` and `steps` as extra args, as shown in the
commands below. If these values are not specified, the script will
default to use `warmup_steps=10` and `steps=50`.

For latency (using `--benchmark-only`, `--socket-id 0` and `--batch-size 1`):

```
python launch_benchmark.py \
    --model-name inceptionv3 \
    --precision int8 \
    --mode inference \
    --framework tensorflow \
    --benchmark-only \
    --batch-size 1 \
    --socket-id 0 \
    --docker-image intelaipg/intel-optimized-tensorflow:PR25765-devel-mkl \
    --in-graph /home/myuser/inceptionv3_int8_pretrained_model.pb \
    --data-location /home/myuser/datasets/ImageNet_TFRecords \
    -- input_height=299 input_width=299 warmup_steps=50 steps=500
```

For throughput (using `--benchmark-only`, `--socket-id 0` and `--batch-size 128`):

```
python launch_benchmark.py \
    --model-name inceptionv3 \
    --precision int8 \
    --mode inference \
    --framework tensorflow \
    --benchmark-only \
    --batch-size 128 \
    --socket-id 0 \
    --docker-image intelaipg/intel-optimized-tensorflow:PR25765-devel-mkl \
    --in-graph /home/myuser/inceptionv3_int8_pretrained_model.pb \
    --data-location /home/myuser/datasets/ImageNet_TFRecords \
    -- input_height=299 input_width=299 warmup_steps=50 steps=500
```

The docker image (`intelaipg/intel-optimized-tensorflow:PR25765-devel-mkl`)
used in the commands above were built using
[TensorFlow](git@github.com:tensorflow/tensorflow.git) master
([e889ea1](https://github.com/tensorflow/tensorflow/commit/e889ea1dd965c31c391106aa3518fc23d2689954)) and
[PR #25765](https://github.com/tensorflow/tensorflow/pull/25765).

Note that the `--verbose` or `--output-dir` flag can be added to any of the above commands
to get additional debug output or change the default output location..

7. The log file is saved to the value
of `--output-dir`. Below are
examples of what the tail of your log file should look like for the
different configs.

Example log tail when running for accuracy:

```
Processed 50000 images. (Top1 accuracy, Top5 accuracy) = (0.7666, 0.9333)
lscpu_path_cmd = command -v lscpu
lscpu located here: /usr/bin/lscpu
Executing command: python /workspace/intelai_models/int8/accuracy.py --input_height=299 --input_width=299 --num_intra_threads=56 --num_inter_threads=2 --batch_size=100 --input_graph=/in_graph/inceptionv3_int8_pretrained_model.pb --data_location=/dataset
Ran inference with batch size 100
Log location outside container: {--output-dir value}/benchmark_inceptionv3_inference_int8_20190104_013246.log
```

Example log tail when benchmarking for latency:
```
...
steps = 470, 53.7256017113 images/sec
steps = 480, 52.5430812016 images/sec
steps = 490, 52.9076139058 images/sec
steps = 500, 53.5021876395 images/sec
lscpu_path_cmd = command -v lscpu
lscpu located here: /usr/bin/lscpu
Ran inference with batch size 1
Log location outside container: {--output-dir value}/benchmark_inceptionv3_inference_int8_20190223_194002.log
```

Example log tail when benchmarking for throughput:
```
...
steps = 470, 370.435654276 images/sec
steps = 480, 369.710160177 images/sec
steps = 490, 369.083388904 images/sec
steps = 500, 370.287978128 images/sec
lscpu_path_cmd = command -v lscpu
lscpu located here: /usr/bin/lscpu
Ran inference with batch size 128
Log location outside container: {--output-dir value}/benchmark_inceptionv3_inference_int8_20190223_194314.log
```

## FP32 Inference Instructions

1. Clone this [intelai/models](https://github.com/IntelAI/models)
repository:

```
$ git clone https://github.com/IntelAI/models.git
```

2. Download the pre-trained model.
```
$ wget https://storage.googleapis.com/intel-optimized-tensorflow/models/inceptionv3_fp32_pretrained_model.pb
```

3. If you would like to run Inception V3 FP32 inference and test for
accuracy, you will need the ImageNet dataset. Benchmarking for latency
and throughput do not require the ImageNet dataset. Instructions for
downloading the dataset and converting it to the TF Records format can
be found in the TensorFlow documentation
[here](https://github.com/tensorflow/models/tree/master/research/slim#an-automated-script-for-processing-imagenet-data).

4. Navigate to the `benchmarks` directory in your local clone of
the [intelai/models](https://github.com/IntelAI/models) repo from step 1.
The `launch_benchmark.py` script in the `benchmarks` directory is
used for starting a benchmarking run in a optimized TensorFlow docker
container. It has arguments to specify which model, framework, mode,
precision, and docker image.

Substitute in your own `--in-graph` pretrained model file path (from step 2).

Inception V3 can be run for latency benchmarking, throughput
benchmarking, or accuracy. Use one of the following examples below,
depending on your use case.

* For latency (using `--batch-size 1`):

```
python launch_benchmark.py \
    --model-name inceptionv3 \
    --precision fp32 \
    --mode inference \
    --framework tensorflow \
    --batch-size 1 \
    --socket-id 0 \
    --docker-image intelaipg/intel-optimized-tensorflow:latest-devel-mkl \
    --in-graph /home/myuser/inceptionv3_fp32_pretrained_model.pb
```
Example log tail when benchmarking for latency:
```
Inference with dummy data.
Iteration 1: 1.075 sec
Iteration 2: 0.023 sec
Iteration 3: 0.016 sec
...
Iteration 38: 0.014 sec
Iteration 39: 0.014 sec
Iteration 40: 0.014 sec
Average time: 0.014 sec
Batch size = 1
Latency: 14.442 ms
Throughput: 69.243 images/sec
lscpu_path_cmd = command -v lscpu
lscpu located here: /usr/bin/lscpu
Ran inference with batch size 1
Log location outside container: {--output-dir value}/benchmark_inceptionv3_inference_fp32_20190104_025220.log
```

* For throughput (using `--batch-size 128`):

```
python launch_benchmark.py \
    --model-name inceptionv3 \
    --precision fp32 \
    --mode inference \
    --framework tensorflow \
    --batch-size 128 \
    --socket-id 0 \
    --docker-image intelaipg/intel-optimized-tensorflow:latest-devel-mkl \
    --in-graph /home/myuser/inceptionv3_fp32_pretrained_model.pb
```
Example log tail when benchmarking for throughput:
```
Inference with dummy data.
Iteration 1: 2.024 sec
Iteration 2: 0.765 sec
Iteration 3: 0.781 sec
...
Iteration 38: 0.756 sec
Iteration 39: 0.760 sec
Iteration 40: 0.757 sec
Average time: 0.760 sec
Batch size = 128
Throughput: 168.431 images/sec
lscpu_path_cmd = command -v lscpu
lscpu located here: /usr/bin/lscpu
Ran inference with batch size 128
Log location outside container: {--output-dir value}/benchmark_inceptionv3_inference_fp32_20190104_024842.log
```

* For accuracy (using `--accuracy-only`, `--batch-size 100`, and
`--data-location` with the path to the ImageNet dataset from step 3):

```
python launch_benchmark.py \
    --model-name inceptionv3 \
    --precision fp32 \
    --mode inference \
    --framework tensorflow \
    --accuracy-only \
    --batch-size 100 \
    --data-location /dataset/Imagenet_Validation \
    --docker-image intelaipg/intel-optimized-tensorflow:latest-devel-mkl \
    --in-graph /home/myuser/inceptionv3_fp32_pretrained_model.pb
```
Example log tail when benchmarking for accuracy:
```
Processed 49800 images. (Top1 accuracy, Top5 accuracy) = (0.7673, 0.9341)
Processed 49900 images. (Top1 accuracy, Top5 accuracy) = (0.7674, 0.9341)
Processed 50000 images. (Top1 accuracy, Top5 accuracy) = (0.7675, 0.9342)
lscpu_path_cmd = command -v lscpu
lscpu located here: /usr/bin/lscpu
Ran inference with batch size 100
Log location outside container: {--output-dir value}/benchmark_inceptionv3_inference_fp32_20190104_023816.log
```

Note that the `--verbose` or `--output-dir` flag can be added to any of the above commands
to get additional debug output or change the default output location..