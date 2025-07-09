### GPU Sockeye Instances w/ PyTorch

To give our container GPU support, we need to fix two problems in our previous guides:
1. There is no physical GPU device passed to the container itself.
2. No CUDA drivers are available in the `quay.io/jupyter/datascience-notebook` image.

#### Adding a GPU:

We'll have to patch some lines in `jupyter-datascience.sh`. Recall we put it in `/arc/project/<allocation>/<cwl>`, so let's work there.

Run:

```
cd /arc/project/<allocation>/<cwl>
```

Create a new script with GPU support with this command:

```
cp ./jupyter-datascience.sh jupyter-pytorch-gpu.sh
```

Now, using your editor of choice (mine is `vim`), patch these lines:

- `#SBATCH --account=<allocation>` should now be `#SBATCH --account=<allocation>-gpu`
  - For instance, if your account was originally `st-researcher-1`, it should be `st-researcher-1-gpu`.
- `#SBATCH --partition=interactive_cpu` should now be `#SBATCH --partition=interactive_gpu`
- In the last line of the script, you should have command that looks like `apptainer exec [...]`, where the elided part runs `jupyter` notebook with some custom environment variables.
  - Append `--nv` after `apptainer exec`, so your new command looks like `apptainer exec --nv [...]` (elided part stays the same)

`jupyter-pytorch-gpu.sh` is not ready to be run with `sbatch` yet - make sure to follow the second part of this guide.

#### Downloading the new container image

We mentioned that there are no CUDA drivers with the CPU image, so we'll have to pull a new one.
Recall we store our images in `/arc/project/<allocation>/<cwl>/images`, so let's `cd` to it:

```
cd /arc/project/<allocation>/jupyter
```

As of 2025-06-19, the standard version of CUDA that PyTorch uses is 12.6, so we'll pull an image with that version of CUDA.

```
apptainer pull --force --name jupyter-pytorch-cuda.sif docker://quay.io/jupyter/pytorch-notebook:cuda12-latest
```

Now amend the `jupyter-pytorch-gpu.sh` file we created in the previous section to point it at our new image:

When you edit the file, the last line should look something like this:

```
apptainer exec --nv --home /scratch/<allocation>/<cwl>/my_jupyter --env XDG_CACHE_HOME=/scratch/<allocation>/<cwl>/my_jupyter /arc/project/<allocation>/jupyter/jupyter-datascience.sif jupyter notebook --no-browser --port=${PORT} --ip=0.0.0.0 --notebook-dir=$SLURM_SUBMIT_DIR
```

Change the `/arc/project/<allocation>/<cwl>/images/jupuyter-datascience.sif` line, which is before the invocation of `jupyter notebook` to `/arc/project/<allocation>/<cwl>/images/jupyter-pytorch-cuda.sif`. Now our slurm script uses a CUDA-enabled container.

I highly recommend you create a new environment for GPU-related work. See [Creating Custom Environments](./Creating_custom_environments.md) for more details.

**Installing PyTorch in new environment**

Do not install PyTorch via conda as it's no longer supported by the PyTorch team, instead, install it using `pip`.

```
pip3 install torch torchvision torchaudio
```

(this should be done in your new conda environment!)

