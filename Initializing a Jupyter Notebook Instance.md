## Initializing a Jupyter Notebook Instance

On ARC Sockeye, you have to initiate compute jobs using `slurm`, which is a command-line job manager.

Jobs are started with `sbatch`, monitored using `squeue`, and stopped using `scancel`.

You can view your jobs with `squeue --user=$USER`.

To start a job, run `sbatch <script file>`, which is typically a bash script.

For Jupyter Notebooks, I have copied a script from the ARC Confluence documentation. It lives at `/arc/project/<allocation>/<CWL>/jupyter-datascience.sh`.

What this script does is that it creates a new Jupyter Notebook instance on an ARC Sockeye compute node on an available port, then it produces a file with instructions on how to connect to that using your computer.

To run it and start a new Jupyter Notebook instance on ARC Sockeye, run:

```
sbatch /arc/project/<allocation>/<CWL>/jupyter-datascience.sh
```

What this will do is start a new job on ARC Sockeye, with the script that starts a new Jupyter Notebook instance.

Once this is run, it will produce a text file at:
```
connection.txt
```
This text document contains the previously mentioned instructions. Here's an **example**, your port **will** be different.

```
1. Create an SSH tunnel to Jupyter Notebooks from your local workstation using the following command:

ssh -N -L 8888:se238:52409 <CWL>@sockeye.arc.ubc.ca

2. Point your web browser to http://localhost:8888

3. Login to Jupyter Notebooks using the following token (password):

password

When done using Jupyter Notebooks, terminate the job by:

1. Quit or Logout of Jupyter Notebooks
2. Issue the following command on the login node (if you did Logout instead of Quit):

scancel 4889894
```

Here's a breakdown of the instructions:
- For point number one, this command should be run on your OWN laptop inside of a terminal window. **DO NOT CLOSE THIS WINDOW AS IT ACTIVELY PROVIDES THE CONNECTION!**

-  Once this is run, go to your browser of choice, and navigate to http://localhost:8888

<!-- 1. For number 3, just quit the instance once you're done. -->