# Using NVIDIA GPUs on QCL Machine
First, verify that GPUs are available by running `nvidia-smi`.

Second, ensure you have a compatible Python environment. You can verify this for TensorFlow by running `python3 -c 'import tensorflow as tf; print("GPUs:", tf.config.list_physical_devices("GPU"))'`. If you do not have a compatible environment, a Miniconda environment will work. Miniconda can be installed by running 
```sh
mkdir -p ~/miniconda3
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh -O ~/miniconda3/miniconda.sh
bash ~/miniconda3/miniconda.sh -b -u -p ~/miniconda3
rm ~/miniconda3/miniconda.sh
```
(see the [Anaconda website](https://www.anaconda.com/docs/getting-started/miniconda/install) for more information). To install TensorFlow and activate the environment, run `conda create -n tf tensorflow -y && conda activate tf`.

To use the GPUs in a TensorFlow script, wrap your model creation and all computation or fit calls with the following lines (minimum working example at the end of this file):
```python
strategy = tf.distribute.MirroredStrategy()
with strategy.scope():
    ...
```
To use specific GPUs, change the first line to `strategy = tf.distribute.MirroredStrategy(["/GPU:0", "/GPU:1"])`, specifying the indices of the specific GPUs instead of `0` or `1` (valid values for the QCL GPU machine are `"/GPU:0"`, `"/GPU:1"`, `"/GPU:2"`, `"/GPU:3"`).

### Minimum Working Example
```python
import tensorflow as tf

strategy = tf.distribute.MirroredStrategy()

with strategy.scope():
    a = tf.constant([1.0, 2.0, 3.0])
    b = tf.constant([4.0, 5.0, 6.0])
    c = a + b
    print(c.numpy()) # should print [5.0 7.0 9.0]
```

### Larger Test Example
```python
import tensorflow as tf

BATCH_SIZE = 2048

# Create distribution strategy
strategy = tf.distribute.MirroredStrategy()

# Load and preprocess MNIST dataset
(x_train, y_train), (x_test, y_test) = tf.keras.datasets.mnist.load_data()
x_train, x_test = x_train / 255.0, x_test / 255.0  # Normalize

train_ds = tf.data.Dataset.from_tensor_slices((x_train, y_train)).shuffle(10000).batch(BATCH_SIZE)
test_ds = tf.data.Dataset.from_tensor_slices((x_test, y_test)).batch(BATCH_SIZE)

with strategy.scope():
    # Build the model
    model = tf.keras.Sequential([
        tf.keras.layers.Flatten(input_shape=(28, 28)),
        tf.keras.layers.Dense(128, activation='relu'),
        tf.keras.layers.Dense(10, activation='softmax')
    ])

    # Compile the model
    model.compile(optimizer='adam',
                  loss='sparse_categorical_crossentropy',
                  metrics=['accuracy'])

    # Train the model
    model.fit(train_ds, epochs=5, validation_data=test_ds)

    # Evaluate the model
    test_loss, test_acc = model.evaluate(x_test, y_test)
print(f"Test accuracy: {test_acc:.4f}")
```
