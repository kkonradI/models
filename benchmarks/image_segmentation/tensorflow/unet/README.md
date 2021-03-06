# UNet

This document has instructions for how to run UNet for the following
modes/precisions:
* [FP32 inference](#fp32-inference-instructions)

## FP32 Inference Instructions

1. Clone this [intelai/models](https://github.com/IntelAI/models)
   repository:
   ```
   $ git clone git@github.com:IntelAI/models.git
   ```
   This repository includes launch scripts for running benchmarks.

2. A link to download the pre-trained model is coming soon.

3. Clone the [tf_unet](https://github.com/jakeret/tf_unet) repository,
   and then get [PR #202](https://github.com/jakeret/tf_unet/pull/202)
   to get cpu optimizations:

   ```
   $ git clone git@github.com:jakeret/tf_unet.git

   $ cd tf_unet/

   $ git fetch origin pull/202/head:cpu_optimized
   From github.com:jakeret/tf_unet
    * [new ref]         refs/pull/202/head -> cpu_optimized

   $ git checkout cpu_optimized
   Switched to branch 'cpu_optimized'
   ```

4. Navigate to the `benchmarks` directory in your local clone of the
   [intelai/models](https://github.com/IntelAI/models) repo from step 1.
   The `launch_benchmark.py` script in the `benchmarks` directory is
   used for starting a benchmarking run in a optimized TensorFlow docker
   container. It has arguments to specify which model, framework, mode,
   precision, and docker image to use, along with the checkpoint files
   that were downloaded in step 2 and the path to the UNet model
   repository that you cloned in step 3.

   UNet benchmarking can be run to test throughput and latency using the
   following command with your checkpoint and model-source-dir paths:

   ```
   python launch_benchmark.py \
       --model-name unet \
       --precision fp32 \
       --mode inference \
       --framework tensorflow \
       --benchmark-only \
       --batch-size 1 \
       --socket-id 0 \
       --docker-image intelaipg/intel-optimized-tensorflow:latest-devel-mkl \
       --checkpoint /home/myuser/unet_trained \
       --model-source-dir /home/myuser/tf_unet \
       -- checkpoint_name=model.cpkt
   ```

   Note that the `--verbose` or `--output-dir` flag can be added to the above
   command to get additional debug output or change the default output location.

6. The log file is saved to the value of `--output-dir`.

   Below is an example of what the log file tail:

   ```
   Time spent per BATCH: 1.1043 ms
   Total samples/sec: 905.5344 samples/s
   lscpu_path_cmd = command -v lscpu
   lscpu located here: /usr/bin/lscpu
   Ran inference with batch size 1
   Log location outside container: {--output-dir value}/benchmark_unet_inference_fp32_20190201_205601.log
   ```