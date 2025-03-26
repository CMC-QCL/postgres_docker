# Using NVIDIA GPUs on Single Node Laguna Cluster
To start an interactive single-node session on the Laguna Cluster, open a terminal through the web interface or by ssh and run `salloc --partition=gpu --gres=gpu:l40s:2 --nodes=1 --ntasks=1 --cpus-per-task=2 --time=2:00:00 --mem=0`. This allocates one node with two GPUs for 2 hours with the maximum memory allowed. Scripts can then be run inside this terminal as they would on the QCL GPU machine.

To submit a single-node GPU job, first create a new job in the Job Composer interface. This will create a dedicated directory in your user folder for the job where all assets for the job should be kept and where all job output will be placed. It is not important for this example that a certain job template be used; we will overwrite the files it creates.

The following template for `job.sh` (which should be placed in the job directory) will create a 36 hour job on a single node with 2 GPUs using all the node's available memory that runs `my_script.py` using the `<your_environment>` in conda. `my_script.py` should be in the job directory.
```
#!/bin/bash

#SBATCH --partition=gpu
#SBATCH --gres=gpu:l40s:2
#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=2
#SBATCH --mem=0
#SBATCH --time=36:00:00

module purge
module load gcc/13.3.0
module load openmpi/5.0.5
module load cuda/12.6.3

~/.conda/envs/<your_environment>/bin/python3 my_script.py
```

```python
# example my_script.py

import tensorflow as tf

strategy = tf.distribute.MirroredStrategy()

with strategy.scope():
    a = tf.constant([1.0, 2.0, 3.0])
    b = tf.constant([4.0, 5.0, 6.0])
    c = a + b
    print(c.numpy()) # should print [5. 7. 9.]

    with open("./outfile.txt", "w") as doc:
        doc.write(str(c.numpy())) # should output to <your_job_folder>/outfile.txt

```

# Using NVIDIA GPUs on Multiple Nodes Laguna Cluster

