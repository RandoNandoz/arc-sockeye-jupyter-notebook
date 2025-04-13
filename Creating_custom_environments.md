# Creating custom envs on `conda` for Jupyter Notebook

You might limited by the packages inside the default environment, so you can create new envs and add your own packages like so:

Inside `/arc/project/<allocation>/jupyter/jupyter-datascience.sif` is an image that defines an environment (essentially another OS/docker image) that contains the necessary files to Jupyter Notebook.

We will modify it using this command:

First run:

```
module load gcc
module load apptainer
```

```
apptainer shell --home /scratch/<allocation>/<CWL>>/my_jupyter/ --env XDG_CACHE_HOME=/scratch/<allocation>/<CWL>/my_jupyter /arc/project/<allocation>/jupyter/jupyter-datascience.sif
```
Once you run this, you should see a prompt like so:

```
Apptainer>
```

Inside the prompt, run:

```
 conda create --prefix /arc/project/<allocation>/jupyter/<your_environment_name>
```

And then run:

```
conda install -y ipykernel --prefix /arc/project/<allocation>/jupyter/<your_environment_name>
```

and for the R kernel:
```
conda install -c conda-forge r-irkernel /arc/project/<allocation>/jupyter/<your_environment_name>
```

To install packages, run:

```
conda install -y <your package(s)> --prefix /arc/project/<allocation>/jupyter/<your_environment_name>
```

> [!WARNING]
> All R packages must be installed using conda-forge. Fortunately, almost every package can be installed this way. Do not try installing using `install.packages()`. You will break things.

Then, you must package your environment into a kernel image like so:

```
source activate /arc/project/<allocation>/jupyter/<your_environment_name>
```

```
python -m ipykernel install --user --name <your_environment_name>
```

and initialize the R kernel using:

```
R
```

```
IRkernel::installspec(name = "my_r_env", displayname = "R (My Environment)")
```

Once you're done, run `exit` in the shell, and then go follow the instructions at the top to run the image you just created.