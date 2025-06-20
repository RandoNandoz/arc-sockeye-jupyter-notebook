# Setup and Apptainer

## Initial setup

Before we get started with running notebooks on Sockeye, we have to pull the requisite container images, and set up some scripts.



| Parameter      | Description        |
| -------------- | ------------------ |
| \<allocation\> | Sockeye allocation |
| \<CWL\>        | Your CWL           |

### Pulling the container

> [!NOTE]
> - We'll be storing all of our container images inside `/arc/project/<allocation>/jupyter` as per UBC Confluence convention. They can be stored elsewhere, but this document uses the UBC Confluence convention, so you will have to adapt the paths if you store them elsewhere.


#### Instructions

1. ##### Create the directory in `/arc/project/<allocation>` to store your images.

    ```bash
    mkdir /arc/project/<allocation>/jupyter
    ```

2. ###### Pull the `jupyter/datascience-notebook` container from `quay.io` into your image folder.
   
   ```bash
    module load gcc apptainer
    cd /arc/project/<allocation>/jupyter
    apptainer pull --name jupyter-datascience.sif docker://quay.io/jupyter/datascience-notebook
   ```
    ###### Updating your container
    Many times, containers have to be updated to bring in the latest Python/R/compiler versions. You can overwrite the current file with a new version by running:
    ```bash
    apptainer pull --force --name jupyter-datascience.sif docker://quay.io/datascience-notebook
    ```

3. ##### Set up the Slurm job.

    First, create a job directory in `/scratch` for your personal Jupyter Notebooks to use as scratch space - ARC Sockeye has a file count quota on top of a file size quota, and files produced by Jupyter can cause you to hit this limit. Run this command to do so:

    ```bash
    mkdir -p /scratch/<allocation>/<cwl>/jupyter
    ```

    Now, put this script wherever you would like. A good spot could be in your home folder, but in this guide, we'll use `/arc/project/<allocation>/<cwl>/jupyter-datascience.sh`.
    
<details>
<summary>Job script</summary>

    ```bash
    #!/bin/bash
    
    #SBATCH --job-name=my_jupyter_notebook
    #SBATCH --account=st-lknelson-1
    #SBATCH --time=03:00:00
    #SBATCH --nodes=1
    #SBATCH --ntasks=1
    #SBATCH --mem=10G
    #SBATCH --partition=interactive_cpu
    
    ################################################################################
    
    # Change directory into the job dir
    cd $SLURM_SUBMIT_DIR
    
    # Load software environment
    module load gcc
    module load apptainer
    
    # Set RANDFILE location to writeable dir
    export RANDFILE=$TMPDIR/.rnd
    
    # Generate a unique token (password) for Jupyter Notebooks
    export APPTAINERENV_JUPYTER_TOKEN=password

    # Find a unique port for Jupyter Notebooks to listen on
    readonly PORT=$(python -c 'import socket; s=socket.socket(); s.bind(("", 0)); print(s.getsockname()[1]); s.close()')
    
    # Print connection details to file
    cat > connection.txt <<END
    
    1. Create an SSH tunnel to Jupyter Notebooks from your local workstation using the following command:
    
    ssh -N -L 8888:${HOSTNAME}:${PORT} ${USER}@sockeye.arc.ubc.ca
    
    2. Point your web browser to http://localhost:8888
    
    3. Login to Jupyter Notebooks using the following token (password):
    
    ${APPTAINERENV_JUPYTER_TOKEN}
    
    When done using Jupyter Notebooks, terminate the job by:
    
    4. Quit or Logout of Jupyter Notebooks
    5. Issue the following command on the login node (if you did Logout instead of Quit):
    
    scancel ${SLURM_JOB_ID}
    
    END
    
    # Execute jupyter within the Apptainer container
    apptainer exec --home /scratch/st-lknelson-1/iberez01/my_jupyter --env XDG_CACHE_HOME=/scratch/st-lknelson-1/iberez01/my_jupyter /arc/project/st-lknelson-1/jupyter/jupyter-datascience.sif jupyter notebook --no-browser --port=${PORT} --ip=0.0.0.0 --notebook-dir=$SLURM_SUBMIT_DIR
    ```
    
</details>


### Next steps

Now that you're "done" setting up, here are some next steps.

#### Happy with packages

If you *are* happy with the packages that are installed in the `jupyter-datascience` image, then you can proceed to [Initializing a Jupyter Notebook Instance](Initializing%20a%20Jupyter%20Notebook%20Instance.md).

#### Need GPUs?

See [Adding GPU Support](./Adding%20GPU%20Support.md). Do this before you create new environments.

#### Need more packages?

If you (suspect) you require more packages in your notebook's environment, have a look at [Creating Custom Environments](Creating_custom_environments.md).