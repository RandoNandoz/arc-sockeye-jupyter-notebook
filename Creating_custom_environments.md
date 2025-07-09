# Creating custom envs on `conda` for Jupyter Notebook

You might limited by the packages inside the default environment, so you can create new envs and add your own packages like so:

Inside `/arc/project/<allocation>/jupyter/jupyter-datascience.sif` is an image that defines an environment (essentially another OS/docker image) that contains the necessary files to Jupyter Notebook.

We will modify it using this command:


First run:

```
module load gcc
module load apptainer
```

Create a place to store your environments
```
mkdir /arc/project/<allocation>/<cwl>/jupyter
```

```
apptainer shell --home /scratch/<allocation>/<cwl>/my_jupyter/ --env XDG_CACHE_HOME=/scratch/<allocation>/<cwl>/my_jupyter /arc/project/<allocation>/<cwl>/images/jupyter-datascience.sif
```
Once you run this, you should see a prompt like so:

```
Apptainer>
```

Inside the prompt, run:

```
conda create --prefix /arc/project/<allocation>/<cwl>/jupyter/<your_environment_name>
```

And then run:

```
conda install -y ipykernel --prefix /arc/project/<allocation>/<cwl>/jupyter/<your_environment_name>
```

and for the R kernel:
```
conda install -c conda-forge r-irkernel /arc/project/<allocation>/<cwl>/jupyter/<your_environment_name>
```

To install packages:



**WARNING: All R packages must be installed using conda-forge. Fortunately, almost every package can be installed this way. Do not try installing using `install.packages()`. You will break things.**

Activate the conda environment we created in a previous step with:

```
source activate /arc/project/<allocation>/<cwl>/jupyter/<your_environment_name>
```

Then use `conda` and `pip` as normal.



Once you are satisfied with the packages you installed, package your environment into a kernel image like so:


```
python -m ipykernel install --user --name <your_environment_name>
```

<details>
<summary>If you installed R and related R packages:</summary>

Initialize the R kernel using:

```
R
```

```
IRkernel::installspec(name = "my_r_env", displayname = "R (My Environment)")
```

</details>


Once you're done, run `exit` in the shell, and then go to [Initializing a Jupyter Notebook Instance](Initializing%20a%20Jupyter%20Notebook%20Instance.md)