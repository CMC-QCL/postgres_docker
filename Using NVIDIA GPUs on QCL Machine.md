# Using NVIDIA GPUs on QCL Machine
First, verify that GPUs are available by running `nvidia-smi`.

Second, ensure you have a compatible Python environment. You can verify this for TensorFlow by running `python3 -c 'import tensorflow as tf; print("GPUs:", tf.config.list_physical_devices("GPU"))'`. If you do not have a compatible environment, a Miniconda environment will work. Miniconda can be installed by running 
```
mkdir -p ~/miniconda3
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh -O ~/miniconda3/miniconda.sh
bash ~/miniconda3/miniconda.sh -b -u -p ~/miniconda3
rm ~/miniconda3/miniconda.sh
```
(see the [Anaconda website](https://www.anaconda.com/docs/getting-started/miniconda/install) for more information). To install TensorFlow and activate the environment, run `conda create -n tf tensorflow -y && conda activate tf`.

To use the GPUs in a TensorFlow script, wrap your model creation and all computation or fit calls with the following lines:
```
strategy = tf.distribute.MirroredStrategy()
with strategy.scope():
    ...
```
To use specific GPUs, change the first line to `strategy = tf.distribute.MirroredStrategy(["/GPU:0", "/GPU:1"])`, specifying the indices of the specific GPUs instead of `0` or `1` (valid values for the QCL GPU machine are `"/GPU:0"`, `"/GPU:1"`, `"/GPU:2"`, `"/GPU:3"`.
