# London Bioinformatics Frontiers Hackathon Tutorial
Tutorial for The [London Bioinformatics Frontiers](http://bioinformatics-frontiers.com) Hackathon 2019.

![lbf_banner](https://raw.githubusercontent.com/PhilPalmer/lbf-hack-tutorial/master/images/lbf_banner.png)

In this tutorial you will learn:
- [Nextflow](https://www.nextflow.io/) - how to build parallelisable & scalable computational pipelines
- [Docker](https://www.docker.com/) - how to build & run containers to bundle dependencies
- [FlowCraft](https://flowcraft.readthedocs.io/en/latest/) - how to build & use modular, extensible and flexible components for Nextflow pipelines
- [Deploit](https://lifebit.ai/deploit) - how to scale your analyses over the cloud

## Agenda

The following is a very rough agenda for the day. The coffee & lunch breaks will definitely be at the times stated here. However, we may find that different parts of the tutorial take us more or less time to get through the content than the times stated here:

![agenda](https://raw.githubusercontent.com/lifebit-ai/lbf-hack-tutorial/master/images/agenda.png)

## Prerequisites
The following are required for the hackathon:
- Java 8 or later
- Docker engine 1.10.x (or higher)
- Git
- Python3

*If you have them installed that's great! Don't worry if not we will help you to install them & other software throughout the tutorial*

## Contents

- [Session 1: Nextflow](#session-1-nextflow)
    - [a) Installation](#a-installation)
    - [b) Parameters](#b-parameters)
    - [c) Processes (inputs, outputs & scripts)](#c-processes-inputs-outputs--scripts)
    - [d) Channels](#d-channels)
    - [e) Operators](#e-operators)
    - [f) Configuration](f-configuration)
- [Session 2: Docker](#session-2-docker)
    - [a) Running images](#a-running-images)
    - [b) Dockerfiles](#b-dockerfiles)
    - [c) Building images](#c-building-images)
    - [d) BONUS: Upload the container to Docker Hub](#d-bonus-upload-the-container-to-docker-hub)
- [Session 3: FlowCraft](#session-3-flowcraft)
    - [a) Installation](#a-installation)
    - [b) How to build a FlowCraft Component](#b-how-to-build-a-flowcraft-component)
    - [c) Building a pipeline with FlowCraft](#c-building-a-pipeline-with-flowcraft)
- [Session 4: Running Nextflow Pipelines on The Cloud on Deploit](#session-4-running-nextflow-pipelines-on-the-cloud-on-deploit)
    - [a) Creating an account](#a-creating-an-account)
    - [b) Importing a Nextflow pipeline on Deploit](#b-importing-a-nextflow-pipeline-on-deploit)
    - [c) Running the pipeline](#c-running-the-pipeline)
    - [d) Monitoring an analysis](#d-monitoring-an-analysis)

## Setup
So that you have the `testdata` within the repository it is recommended that you clone this reposiotry. To do this open a terminal & enter the following:
```
git clone https://github.com/lifebit-ai/lbf-hack-tutorial.git
&& cd lbf-hack-tutorial
```

For the tutorial we will be working through a series of steps. If you are behind you can view the branches we have made for that section. The final branch is 2b. 

![branches](https://raw.githubusercontent.com/lifebit-ai/lbf-hack-tutorial/master/images/branches.png)

To swtich to the branch on the command line you can enter:
```bash
git checkout <branch_name>
```

eg:
```bash
git checkout 1b
```

<br />

## Session 1: Nextflow

![nextflow_logo](https://raw.githubusercontent.com/PhilPalmer/lbf-hack-tutorial/master/images/nextflow.png)

What is Nextflow? Why use it? [See about Nextflow slides]()

**Main outcome:** *During the first session you will build a [FastQC](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/) & [MultiQC](https://multiqc.info/) pipeline to learn the basics of Nextflow including:*
- [Parameters](https://www.nextflow.io/docs/latest/getstarted.html?highlight=parameters#pipeline-parameters)
- [Processes](https://www.nextflow.io/docs/latest/process.html) (inputs, outputs & scripts)
- [Channels](https://www.nextflow.io/docs/latest/channel.html)
- [Operators](https://www.nextflow.io/docs/latest/operator.html)
- [Configuration](https://www.nextflow.io/docs/latest/config.html)


### a) Installation
### i. Installing Nextflow
You will need to have Java 8 or later installed for Nextflow to work. You can check your version of Java by entering the following command:
```bash
java -version
```

If you have [Conda](https://docs.conda.io/en/latest/) installed then you can run the following to install Nextflow:
```bash
conda install -c bioconda nextflow
```

To install Nextflow open a terminal & enter the following command:
```bash
curl -fsSL get.nextflow.io | bash
```

This will create a `nextflow` executable file in your current directory. To complete the installation so that you can run Nextflow run anywhere you may want to add it a directory in your $PATH, eg:
```
mv nextflow /usr/local/bin
```

You can then test your installation of Nextflow with:
```
nextflow run hello
```

### ii. Installing Docker

To check if you have Docker installed you can type:
```bash
docker -v
```

If you need to install docker you can do so by following the instructions [here](https://docs.docker.com/v17.12/install/). Be sure to select your correct OS:
[![install_docker](https://raw.githubusercontent.com/PhilPalmer/lbf-hack-tutorial/master/images/install_docker.png)](https://docs.docker.com/v17.12/install/)

  
### b) Parameters

Now that we have Nextflow & Docker installed we're ready to run our first script

1. Create a file `main.nf` & open this in your favourite code/text editor eg VSCode or vim
2. In this file write the following:
```nextflow
// main.nf
params.reads = false

println "My reads: ${params.reads}"
```

The first line initialises a new variable (`params.reads`) & sets it to `false`
The second line prints the value of this variable on execution of the pipeline.

We can now run this script & set the value of `params.reads` to one of our FASTQ files in the testdata folder with the following command:
```
nextflow run main.nf --reads testdata/test.20k_reads_1.fastq.gz
```

This should return the value you passed on the command line

#### Recap
Here we learnt how to define parameters & pass command line arguments to them in Nextflow

### c) Processes (inputs, outputs & scripts)

Nextflow allows the execution of any command or user script by using a `process` definition. 

A process is defined by providing three main declarations: 
the process [inputs](https://www.nextflow.io/docs/latest/process.html#inputs), 
the process [outputs](https://www.nextflow.io/docs/latest/process.html#outputs)
and finally the command [script](https://www.nextflow.io/docs/latest/process.html#script).

In our main script we want to add the following:
```nextflow
//mainf.nf
reads = file(params.reads)

process fastqc {

    publishDir "results", mode: 'copy'

    input:
    file(reads) from reads

    output:
    file "*_fastqc.{zip,html}" into fastqc_results

    script:
    """
    fastqc $reads
    """
}
```

Here we created the variable `reads` which is a `file` from the command line input.

We can then create the process `fastqc` including:
 - the [directive](https://www.nextflow.io/docs/latest/process.html#directives) `publishDir` to specify which folder to copy the output files to 
 - the [inputs](https://www.nextflow.io/docs/latest/process.html#inputs) where we declare a `file` `reads` from our variable `reads`
 - the [output](https://www.nextflow.io/docs/latest/process.html#outputs) which is anything ending in `_fastqc.zip` or `_fastqc.html` which will go into a `fastqc_results` channel
 - the [script](https://www.nextflow.io/docs/latest/process.html#script) where we are running the `fastqc` command on our `reads` variable
 
We can then run our script with the following command:
```bash
nextflow run main.nf --reads testdata/test.20k_reads_1.fastq.gz -with-docker flowcraft/fastqc:0.11.7-1
```

By running Nextflow using the `with-docker` flag we can specify a Docker container to execute this command in. This is beneficial because it means we do not need to have `fastqc` installed locally on our laptop. We just need to specify a Docker container that has `fastqc` installed.


### d) Channels

Channels are the preferred method of transferring data in Nextflow & can connect two processes or operators.

<!--
There are two types of channels:
1. [Queue channels](https://www.nextflow.io/docs/latest/channel.html#queue-channel) can be used to connect two processes or operators. They are usually produced from factory methods such as [`from`](https://www.nextflow.io/docs/latest/channel.html#from)/[`fromPath`](https://www.nextflow.io/docs/latest/channel.html#frompath) or by chaining it with methods such as [`map`](https://www.nextflow.io/docs/latest/operator.html#operator-map). **Queue channels are consumed upon being read.**
2. [Value channels](https://www.nextflow.io/docs/latest/channel.html#value-channel) a.k.a. singleton channel are bound to a single value and can be read unlimited times without consuming there content. Value channels are produced by the value factory method or by operators returning a single value, such us first, last, collect, count, min, max, reduce, sum.
-->

Here we will use the method [`fromFilePairs`](https://www.nextflow.io/docs/latest/channel.html#fromfilepairs) to create a channel to load paired-end FASTQ data, rather than just a single FASTQ file.

To do this we will replace the code from [1c](https://github.com/PhilPalmer/lbf-hack-tutorial/blob/master/README.md#c-processes-inputs-outputs--scripts) with the following 

```nextflow
//main.nf
reads = Channel.fromFilePairs(params.reads, size: 2)

process fastqc {

    tag "$name"
    publishDir "results", mode: 'copy'
    container 'flowcraft/fastqc:0.11.7-1'

    input:
    set val(name), file(reads) from reads

    output:
    file "*_fastqc.{zip,html}" into fastqc_results

    script:
    """
    fastqc $reads
    """
}
```

The `reads` variable is now equal to a channel which contains the reads prefix & paired-end FASTQ data. Therefore, the input declaration has also changed to reflect this by declaring the value `name`. This `name` can be used as a tag for when the pipeline is run. Also, as we are now declaring two inputs the `set` keyword has to be used. Finally, we can specify the container name within the processes as a directive.

To run the pipeline:
```bash
nextflow run main.nf --reads "testdata/test.20k_reads_{1,2}.fastq.gz" -with-docker flowcraft/fastqc:0.11.7-1
```

#### Recap
Here we learnt how use to the [`fromFilePairs`](https://www.nextflow.io/docs/latest/channel.html#fromfilepairs) method to generate a channel for our input data.

### e) Operators

Operators are methods that allow you to manipulate & connect channels.

Here we will add a new process `multiqc` & use the [`.collect()`](https://www.nextflow.io/docs/latest/operator.html#collect) operator

Add the following process after `fastqc`:
```nextflow
//main.nf
process multiqc {

    publishDir "results", mode: 'copy'
    container 'ewels/multiqc:v1.7'

    input:
    file (fastqc:'fastqc/*') from fastqc_results.collect()

    output:
    file "*multiqc_report.html" into multiqc_report
    file "*_data"

    script:
    """
    multiqc . -m fastqc
    """
}
```

Here we have added another process `multiqc`. We have used the `collect` operator here so that if `fastqc` ran for more than two pairs of files `multiqc` would collect all of the files & run only once.

The pipeline can be run with the following:
```bash
nextflow run main.nf --reads "testdata/test.20k_reads_{1,2}.fastq.gz" -with-docker flowcraft/fastqc:0.11.7-1
```

#### Recap
Here we learnt how to use operators such as `collect` & connect processes via channels

### f) Configuration

Configuration, such as parameters, containers & resources eg memory can be set in `config` files such as [`nextflow.config`](https://www.nextflow.io/docs/latest/config.html#configuration-file).

For example our `nextflow.config` file might look like this:
```
docker.enabled = true
params.reads = false

process {
  cpus = 2
  memory = "2.GB"

  withName: fastqc {
    container = "flowcraft/fastqc:0.11.7-1"
  }
  withName: multiqc {
    container = "ewels/multiqc:v1.7"
  }
}
```

Here we have enabled docker by default, initialised parameters, set resources & containers. It is best practice to keep these in the `config` file so that they can more easily be set or removed. Containers & `params.reads` can then be removed from `main.nf`.

The pipeline can now be run with the following:
```bash
nextflow run main.nf --reads "testdata/test.20k_reads_{1,2}.fastq.gz"
```

#### Recap
Here we learnt how to use configuration files to set parameters, resources & containers

<br />

## Session 2: Docker

![docker logo](https://raw.githubusercontent.com/PhilPalmer/lbf-hack-tutorial/master/images/docker.gif)

What is Docker? Why use it? [See about Docker slides]()

**Main outcome:** *During this session, you will learn how to build & run your own Docker container to bundle dependencies for FastQC & MultiQC*

### a) Running images

Running a container is as easy as using the following command: 

```bash
docker run <container-name> 
```

For example: 

```bash
docker run hello-world  
```

#### Run a container in interactive mode 

Launching a BASH shell in the container allows you to operate in an interactive mode 
in the containerised operating system. For example: 

```
docker run -it flowcraft/fastqc:0.11.7-1 bash 
``` 

Once the container is launched you will notice that's running as root (!). 
Use the usual commands to navigate in the file system.

To exit from the container, stop the BASH session with the `exit` command.

### b) Dockerfiles

Docker images are created by using a so called `Dockerfile` i.e. a simple text file 
containing a list of commands to be executed to assemble and configure the image
with the software packages required.    

In this step, you will create a Docker image containing the FastQC & MultiQC tools.


Warning: the Docker build process automatically copies all files that are located in the 
current directory to the Docker daemon in order to create the image. This can take 
a lot of time when big/many files exist. For this reason, it's important to *always* work in 
a directory containing only the files you really need to include in your Docker image. 
Alternatively, you can use the `.dockerignore` file to select the path to exclude from the build. 

Then use your favourite editor eg. `vim` to create a file named `Dockerfile` and copy the 
following content: 

```Dockerfile
FROM nfcore/base

LABEL authors="phil@lifebit.ai" \
      description="Docker image containing fastqc & multiqc for LBF hackathon tutorial"

RUN conda install -c bioconda fastqc=0.11.8 && \
    conda install -c bioconda multiqc=1.7
```

When done save the file. 

### c) Building images

Build the Docker image by using the following command: 

```bash
docker build -t my-image .
```

Note: don't miss the dot in the above command. When it completes, verify that the image 
has been created listing all available images: 

```bash
docker images
```

#### For example:
With the `Dockerfile` from above you might want to run:
```bash
docker build -t lifebitai/lbf-hack .
```

![conda](https://raw.githubusercontent.com/PhilPalmer/lbf-hack-tutorial/master/images/conda.png)

And then you can enter inside the container to check everything is working:
```bash
docker run -it lifebitai/lbf-hack:latest bash
```

The container can be used in our Nextflow pipeline replacing the two different containers we currently have because it has both `fastqc` & `multiqc` installed


### d) BONUS: Upload the container to Docker Hub

Publish your container in Docker Hub to share it with other people. 

Create an account in the https://hub.docker.com web site. Then from your shell terminal run 
the following command, entering the user name and password you specified registering in the Hub: 

```
docker login 
``` 

Tag the image with your Docker user name account: 

```
docker tag my-image <user-name>/my-image 
```

Finally, push it to the Docker Hub:

```
docker push <user-name>/my-image 
```

After that anyone will be able to download it by using the command: 

```
docker pull <user-name>/my-image 
```

Note how after a pull and push operation, Docker prints the container digest number e.g. 

```
Digest: sha256:aeacbd7ea1154f263cda972a96920fb228b2033544c2641476350b9317dab266
Status: Downloaded newer image for nextflow/rnaseq-nf:latest
```

This is a unique and immutable identifier that can be used to reference container image 
in a univocally manner. For example: 

```
docker pull nextflow/rnaseq-nf@sha256:aeacbd7ea1154f263cda972a96920fb228b2033544c2641476350b9317dab266
```

<br />

## Session 3: FlowCraft

![flowcraft logo](https://raw.githubusercontent.com/PhilPalmer/lbf-hack-tutorial/master/images/flowcraft.png)

What is FlowCraft? Why use it? 
[See FlowCraft slides](https://slides.com/diogosilva-1/nextflow-workshop-2018-6#/)

**Main outcome:** *During this session, you will learn how to build your own Fastqc FlowCraft component & GATK pipeline*

### a) Installation

FlowCraft is available to install via both Conda & Pip. However, as we are going to building components we want to install the development version. This can be done with the following commands:
```bash
git clone https://github.com/assemblerflow/flowcraft.git
cd flowcraft
python3 setup.py install
```

### b) How to build a FlowCraft Component
FlowCraft allows you to build pipelines from components. In order to create a new Component, two files are required. These are the template & the class.

### i. Templates
Inside of the `flowcraft` directory, create & open a new [file](https://github.com/assemblerflow/flowcraft/commit/7a4575bc0fab7c54d7f427805dff5b47ef0a666b) `flowcraft/generator/templates/fastqc2.nf` in your favourite code editor:
```nextflow
process fastqc2_{{ pid }} {

    {% include "post.txt" ignore missing %}

    tag { sample_id }
    publishDir "results/fastqc2_{{ pid }}", mode: 'copy'

    input:
    set sample_id, file(fastq_pair) from {{ input_channel }}

    output:
    file "*_fastqc.{zip,html}" into {{ output_channel }}
    {% with task_name="fastqc2" %}
    {%- include "compiler_channels.txt" ignore missing -%}
    {% endwith %}

    script:
    """
    fastqc $fastq_pair
    """
}

{{ forks }}
```

This is standard Nextflow code which is used as a template. Any code in the double curley brackets `{{}}` is FlowCraft code which will be replaced when building pipelines.

### ii. Classes

Inside of the `flowcraft` directory, open & add the following [changes](https://github.com/assemblerflow/flowcraft/commit/43d9ffdb7b1ca5c9e65a4444356cdc7c6bdae404) to the file `flowcraft/generator/components/reads_quality_control.py` in your favourite code editor:

```python
class Fastqc2(Process):
    def __init__(self, **kwargs):
        super().__init__(**kwargs)

        self.input_type = "fastq"
        self.output_type = "fastq"

        self.directives = {"fastqc2": {
            "cpus": 2,
            "memory": "'4GB'",
            "container": "flowcraft/fastqc",
            "version": "0.11.7-1"
        }}

        self.status_channels = [
            "fastqc2"
        ]
```

Here we set the following:
- the **inputs & outputs** which allows processes to be connected
- the **parameters** required by the process (none in this case)
- the **directives** for the process, including the docker container we want to use. Here the `version` is the `tag` of the docker container
- the **status channels** for the process to log its status

### c) Building a pipeline with FlowCraft

Now if we add the directory containing `flowcraft.py` to our path, we can then build a pipeline from any directory, eg:
```bash
export PATH=$PATH:/path/to/flowcraft/flowcraft
```

Now we can test the component we have built with the command:
```bash
flowcraft.py build -t "fastqc2" -o fastqc.nf
```

This will create a Nextflow script `fastqc.nf`    

More complex pipelines such as a GATK pipeline can be built with one command:
```bash
flowcraft.py build -t "bwa mark_duplicates haplotypecaller" -o main.nf --merge-params
```

Here the `merge-params` flag is used to merges all parameters with the same name in a single parameter

<br />

## Session 4: Running Nextflow Pipelines on The Cloud on Deploit

![deploit logo](https://raw.githubusercontent.com/PhilPalmer/lbf-hack-tutorial/master/images/deploit.png)

**Main outcome:** *During this session, you will learn how to scale the GATK pipeline you built in the previous session to run an analysis on the Cloud using the Deploit platform.*

[Deploit](https://lifebit.ai/deploit) is a bioinformatics platform, developed by Lifebit, where you can run your analysis over the Cloud/AWS.

### a) Creating an account
First, create an account/log in [here](https://deploit.lifebit.ai/register). You will get $10 free credits. If you prefer you can connect & use your own AWS account/credentials.

![create_deploit_account](https://raw.githubusercontent.com/PhilPalmer/lbf-hack-tutorial/master/images/create_deploit_account.png)

### b) Importing a Nextflow pipeline on Deploit

We are able to import the GATK pipeline we created with FlowCraft from the previous section (Session 3) on Deploit. This will enable us to scale our analyses. All we need to import a pipeline is the URL from GitHub. For simplicity, we have already created a GitHub repository for the pipeline here: https://github.com/lifebit-ai/gatk-flowcraft

To import the pipeline we must first navigate to the pipelines page. This can be found in the navigation bar on the left-hand-side:

![deploit_pipelines](https://raw.githubusercontent.com/PhilPalmer/lbf-hack-tutorial/master/images/deploit_pipelines.png)

To then import the pipeline you need to:
- Click the green `New` button
- `Select` the GitHub icon to import the Nextflow pipeline from GitHub
- Paste the URL of our pipeline [`https://github.com/lifebit-ai/gatk-flowcraft`](https://github.com/lifebit-ai/gatk-flowcraft)
- Name our pipeline, eg `gatk-flowcraft`
- (Optional:) enter a pipeline description
- Click `Next` & `Create pipeline` :tada:

![import_pipeline.gif](https://raw.githubusercontent.com/PhilPalmer/lbf-hack-tutorial/master/images/import_pipeline.gif)


### c) Running the pipeline

Pipelines can be run in three simple steps:
1. Select the pipeline
2. Select data & parameters
3. Run the analysis

#### i. Selecting the pipeline
Once the pipeline is imported it will automatically be selected.

Alternatively, you can navigate to the pipelines page. Where you can find the imported pipeline under `MY PIPELINES & TOOLS`. To select the pipeline you need to click the card for the pipeline.

![my_pipelines](https://raw.githubusercontent.com/PhilPalmer/lbf-hack-tutorial/master/images/my_pipelines.png)

#### ii. Selecting the data & parameters

The pipeline requires three parameters to be set. These are:
- `fastq` - paired-end reads to be analysed in `fastq.gz` format
- `reference` - name of reference genome `fasta`, `fai` & `dict` files
- `intervals` - `interval_list` file to specify the regions to call variants in

To select the data & parameters you must:
- Click the green plus to add more lines to for two additional parameters
- Specify the parameter names for `fastq`, `reference` & `intervals`
- Import the testdata. This has already been added to the AWS S3 bucket `s3://lifebit-featured-datasets/hackathon/gatk-flowcraft` (although you can also upload files from your local machine via the web interface)
- Once the testdata has been imported you must specify the values for each parameter:
    - `fastq` use the blue plus button to `Choose` the imported folder & click `+Regex` & type `*{1,2}.fastq.gz`
    - `reference` you can also use strings to specify the location. Set the reference to `s3://ngi-igenomes/igenomes/Homo_sapiens/GATK/GRCh37/Sequence/WholeGenomeFasta/human_g1k_v37_decoy`
    - For the intervals click the blue plus again & select the `GRCh37WholeGenome.interval_list` file within the imported folder
- Finally, click `Next`

See below for all of the steps:

![select_data_params](https://raw.githubusercontent.com/PhilPalmer/lbf-hack-tutorial/master/images/select_data_params.gif)

#### iii. Run the job - selecting the project & resources

Select a project & instance:

Before running the job you must:
1. Select the project (which is like a folder used to group multiple analyses/jobs). You can select the already created `Demo` project
2. Choose the instance to set the compute resources such as CPUs & memory. Here you can select `Dedicated Instances` > 16 CPUs > `c4.4xlarge`
3. Finally, click `Run job`

![run_job](https://raw.githubusercontent.com/PhilPalmer/lbf-hack-tutorial/master/images/run_job.gif)

### d) Monitoring an analysis

To monitor jobs you can click on the row for any given job. Immediately after running a job its status will be initialising. This is where AWS in launching the instance. This normally occurs for ~5mins before you are able to view the progress of the job. 

Once on the job monitor page, you can see the progress of the job update in real time. Information such as the resources i.e. memory & CPUs is displayed. Once the job has finished the results can be found in the results tab as well as any reports for select pipelines.

![monitor_job](https://raw.githubusercontent.com/PhilPalmer/lbf-hack-tutorial/master/images/monitor_job.gif)

You can view a successfully completed example job [here](https://staging.lifebit.ai/public/jobs/5d0534f3ee251700be6884ba):

[![shared_job](https://raw.githubusercontent.com/PhilPalmer/lbf-hack-tutorial/master/images/shared_job.png)](https://staging.lifebit.ai/public/jobs/5d0534f3ee251700be6884ba)

### Thanks for taking part

Well done you survived! You’ve made it to the end of the hackathon tutorial. You’ve learned about the magic of Nextflow, Docker, Flowcraft & Deploit. You can now go out & analyse all the things.

![all_the_things](https://raw.githubusercontent.com/PhilPalmer/lbf-hack-tutorial/master/images/all_the_things.jpg)

Hope you enjoyed the conference & let us know if you have any [feedback](https://forms.gle/u78r5byJeZENbKdF8) or questions.

Is there anything we could have improved on? It would be much appreciated if you could fill out this [feedback form](https://forms.gle/u78r5byJeZENbKdF8). For any questions, email me phil@lifebit.ai

## Credits

Credit to [Lifebit](https://lifebit.ai/) & [The Francis Crick Institute](https://www.crick.ac.uk/) for organising & hosting the event

Many thanks to everyone who helped out along the way, including (but not limited to): 
@ODiogoSilva, @cgpu, @clairealix, @cimendes & @pprieto

Thanks to everyone involved in the [nf-hack17-tutorial](https://github.com/nextflow-io/nf-hack17-tutorial) which was heavily used as inspiration for this tutorial