# zepto-data-runner

Reproducible data analysis pipeline/workflow runner for regular scientists and researchers.


To see **Installation** and **Examples**, scroll down to the bottom of this page, past the all too elaborate,
"Motivation", "Prior art", and "Best practices for reproducible data analysis" sections.



### Motivation:

One of the most important aspects of science is that it is expected to be reproducible.
For a long time, *reproducibily* was a woefully neglected aspect of scientific research and scientific publishing.
Fortunately, after decades of hard to reproduce science has flooded the scientific literature,
"reproducible research" is now becoming an increasingly popular and recognized topic.

Guidelines and standard for conducting experiments and analyzing eperimental data are, in my opinion,
often either too vague, abstract, or entirely missing.

Fact is, most data analysis performed by students and researchers at academic institutions,
is performed on a rather ad-hoc basis, with little focus on *reproducing* the data analysis.
Some people may argue that "most data analysis does not need to be reproduced".
But that argument often leads to a state where the data analyses that actually need to be
reproduced are not reproducible. Researchers are only people, and people are creatures of habbits.
If a researcher is used to analyzing his data in one way, he or she is not going to change
the way they analyze their data, just because it might be published.
In fact, more often than not, it is not known in advance, if an experimental dataset and the
analysis thereof will be published. Sometimes, the experiment may not even be published
until after the researcher conducting the experiment and analysis has left the lab.

The best way to resolve this issue is to make sure that the *default* way that researchers
are doing their data analyses, is *reproducible*. And the best way to get as many researchers
to adopt a reproducible data analysis workflow, is to make it easy for them to get started.


During a recent review and [comparison of build-systems, workflow-management systems, and workflow-engines for reproducible data analysis](https://docs.google.com/spreadsheets/d/1eYu1VCWYoseYV8W4xjDhro5JPgIWoLn1fK9BObwGd5Q/edit?usp=sharing),
I grew frustrated with how difficult the systems were to use for most regular scientists and researchers.


* **Most researchers use Windows**. (sadly)
  Yet, most of the systems availble for running reproducible data analysis workflows requires Linux or MacOS to run.
  Even the vast majority of "guides" teaching researchers how to run reproducible data analysis workflows,
  can only be run on Linux or MacOS.
  And no, asking researchers to install and use Vagrant, Docker, or other virtualization or containerization
  systems, just to analyze a simple piece of data is not sustainable.
* **Most researchers are not programmers.** (sadly)
  Yet, many of the good data-analysis pipeline/workflow systems requires the user to be able to read and write
  at least basic code in either bash or python.
  In fact, most researchers don't even know how to use a terminal and command line tools, or even the difference
  between a plain-text editor and a word-processor (e.g. Word).

These are the people we need to convince to start using reproducible data analysis workflows.
But it is hyprocritical to ask researchers to follow best-practices for data-analysis, if the tools
required to follow said best practices are not readily available to the researchers on the computers they use,
and if following the best-practices requires skills that most researchers do not have.



### Criteria & requirements:


The following criteria are important in order to create a powerful but still easy to use
data analysis workflow system.


1. True cross-platform support for all three operating systems (Windows, Mac, Linux).
   While the workflow may call programs that only runs on a specific operating system,
   there shouldn't be anything in the data-analysis workflow system itself that
   makes it exclusive to only a specific operating system.

2. A simple to use syntax, that is still sufficiently versatile to accommodate all use-cases,
   even for advanced users.

3. Support for parameterization. Often, a researcher may want to run given analysis process
   with multiple different parameters, and then run furher parameterized analysis of each of the
   multiple outputs. This should be easy to set up, and obvious to understand.

4. Support for multiple environments (preferred). Either using local python venv/conda virtual
   environments or containers such as Docker.

5. Support for parallelized and clusterized computation.





## Prior art:


### Human language:

This is basically how a lot of data analysis "reproduction" is done right now.

Yes, there are groups and labs working with large data and complex, well-described
workflows. But for the vast majority of small research groups, that is simply
not how the day-to-day data processing is done. It is done with a simple GUI
application, where the user performs some more or less well-defined mouse-clicks
and operations in order to analyze the data, often not even recording exactly
what has been done, but only saving the output to a file.
Fingers crossed that the software is still available at a later point,
and that whatever version we happen to download and install will work the same
way and produce the same output!

A slightly better variant of this is to create a text file, e.g. just next to
where you expect to save the output file, which describes how the output was
generated, and perhaps even what versions was used.

If command line interfaces are used, rather than a GUI program, the operations
can even be described accurately by including the actual commands invoked
in the text file, e.g.

```
# picasso master branch commit b1bb1d75
> python -m picasso localize -b 9 -g 10000 --fit-method mle --baseline 0.0 --qe 0.85
```

The user may even run a `pip freeze > pip-freeze.txt` command to describe the
python environment in which the commands were invoked.


This may be sufficient for very simple data analysis, e.g. if the analysis
only requires literally a single step, and if the workflow does not change
significantly over time.

If, however, the data analysis requires multiple commands (or the same command
with multiple different sets of parameters),
or if the workflow, or the environment in which the workflow is executed,
changes over time,
then it is probably better to use a dedicated file and to track the



### Simple shell scripts:


For simple data analyses on small datasets, it may be sufficient to
simply place all the commands needed to run the data analysis
in a single shell script (bash or batch).

This has the advantage of keeping all commands togeher.
It also ensures that you don't forget to run one of the commands.
For instance, you could include `pip freeze > pip-freeze.txt`
as the last command in the script, to ensure that you always have an
up-to-date description of your environment.


The disadvantage is that if e.g. the first step of the data analysis takes
a long time, but subsequent steps are much faster (e.g. plotting graphs),
then re-running the script will go through all the time-consuming steps
all over again.

Of course, it is possible to write the scripts so they check if e.g.
intermediate data files exists, and not re-create those. But if the
parameters used to create those intermediate files have changed, then
you *want* the intermediate data files to be recomputed.


Finally, shell scripts are generally *not portable between different operating systems*.
A bash script written for Linux or MacOS by one researcher is not
easy to run by another researcher running Windows. Of course, this *can* be
done using WSL, Cygwin, MSYS2, Vagrant, Docker, or similar.
But even using git-bash, which comes with git, is often beyond what
regular researchers can be asked to do.


<!--
For a long time, shell scripts were also the primary method for building software.
-->





### Makefiles and GNU-Make:

A very large number of "reproducible research" guides tell researchers to use
`make` and *Makefiles* to run their data analysis workflows.

Make and makefiles has been a staple in POSIX operating systems (including Linux and MacOS).
It has been the *defacto* method for compiling and installing software from source code
for a very long time.

When compiling software (written in e.g. C), it is important to compile it in the correct order.
Make solves this problem by describing all input files (dependencies) and output files (targets)
as a network graph (technically a directed acyclic graph, or "DAG"), and then figuring out which
files needs to be compiled first (and whether they needs to be re-compiled in subsequent runs).

A data analysis workflow can often be described in a similar fashion:
First, the raw data is processed, and the results perhaps saved to some intermediate files,
then some statistics are calculated and saved, and finally rendering and saving plots or figures,
again as files. The latter files (plots and figures) depends on the former, creating a very similar
network of dependencies (a "DAG"). Because the processes are so similar, is thus very natural to
consider using the same, well-established tool used to build software for to manage data analysis
pipelines.

Windows, however, does not ship with Make.
The GnuWin32 package, provides one native Windows port of GNU-Make, but installing it
(via e.g. the recommended GetGnuWin32 installer) is not super straight-forward.

Make also supports a bash-like scripting inside the Makefiles.
While bash/shell scripting is the *franco lingo* of data scientists, most regular researchers do
not understand bash syntax, and may be put off by its very technical appearance.
Worse still, researchers running Windows will not be able to execute the bash-scripts within
a Makefile, so Makefile scripts created on a Linux machine may not run at all on a Windows computer.

Finally, Make does not support many of the slightly more advanced features we would like to have
in a data analysis workflow management system, such as support for multiple environments and
cluster-based computations.



### Other build-systems:

Building software is obviously important to software developers.
And since software developers like to build software development tools,
there is a huge number of alternative *build-systems* in addition to Make.

Many of these focus on automating the compilation process of C and C++ code.
(E.g. CMake, Premake, SCons, etc.)

During my research, I did not find any *build-system* that was really well-suited
as a data-analysis workflow manager. And why would they? They are, after all,
designed to *build software*, not as a general-purpose data workflow tool.



### Workflow-engines and workflow management systems:





### Common Workflow Language





### Snakemake:

Snakemake is the workflow management system that comes closest to checking the boxes for all
the criteria listed above. It features a simple, yet powerful syntax defining the workflows,
and it supports multiple environments, and can easily do parallelized computations and
even submit the job to various clusters.

The only issue is that Snakemake does not run on Windows.
The Snakemake tutorial suggests installing and using Vagrant, but in my opinion that is
still a little too esoteric and advanced for most researchers to adopt it in their
everyday data analysis work.

Still, Snakemake is a great, mature application, which I can only highly recommend.
In fact, before starting this project, I tried to simply modify snakemake to run
on Windows. That, however, was a bit more than I could manage.

> NOTICE: There is an ongoing effort to add Windows support to Snakemake.
> See the "win\_tests" branch on the Snakemake GitHub repo,
> https://github.com/snakemake/snakemake/tree/win_tests,
> and the [corresponding pull request #222](https://github.com/snakemake/snakemake/pull/222).

> In fact, even though the Snakemake [tutorial](https://snakemake.readthedocs.io/en/stable/tutorial/setup.html#requirements)
> ([archived](http://web.archive.org/web/20180628093703/http://snakemake.readthedocs.io/en/stable/tutorial/setup.html#requirements))
> still says to use Vagrant in order to install on Windows,
> the actual [Installation page](https://snakemake.readthedocs.io/en/stable/getting_started/installation.html)
> ([archived](https://web.archive.org/web/20190709195640/https://snakemake.readthedocs.io/en/stable/getting_started/installation.html))
> does not say anything about Snakemake not running on Windows.
> (Right now you just *can't* install on Windows using the `snakemake` conda package,
> but that appears to be an unintentional bug.)





### Other workflow management systems and workflow engines:

A huge number of other workflow management systems and workflow engines are availalbe.

Some of these workflow engines are mostly concerned with cloud calculations (often not even
storing intermediate data to files), which is generally not what the regular researcher want.

Other workflow management systems require extensive programming or boilerplate code,
which may be beyond the capabilities of regular researchers.





## Best practices for reproducible data analysis


1. First and foremost, in the context of reproducible research,
   having reproducible data analysis workflow is a moot point,
   if the underlying data is not itself clearly described and reproducible.
   All data should have (1) a file documenting the data in a human-readalbe
   format, typically in the form of a README file, and (2) a file containing
   metadata in a structured format, typically YAML, JSON, or XML.

2. Raw data should be *immutable*. Once experimental data has been acquired,
   the data files should not ever be changed. There are many reasons for this,
   for example scientific research integrity to just plain question of
   "the data has changed - but what exactly has changed?", to practical issues
   related to changing file checksums/hashes.
   For this reason, I also recommend keeping structured, textual meta-data in a
   separate file next to the raw data file. Then, if the meta-data needs to be
   updated, it is easy to determine what has changed, and easy to determine that
   the actual data has not been altered.
   To ensure that the raw data is not accidentally altered, I recommend setting
   marking the file as read-only under file attributes, and also creating a
   checksum file containing the hash of the file (e.g. MD5 or SHA256), which is
   marked as read-only. The file checksum can easily be copied to other locations,
   email to a central server, or similar, which can all eventually be used at a
   later time to verify the integrity of the raw data.

3. Data and data-analysis should be kept separate.
   The reasons for this are three-fold:
   	1. It should be easy to apply a different data analysis to the same dataset.
   	2. It should be easy to apply the same data analysis to a different dataset.
   	3. In terms of filesize, datasets are often many orders of magnitude larger
       than the workflow descriptions and scripts used for the data-analysis.
       For that reason, it is a good practice to use different systems for tracking
       and version controlling *data* versus a *data-analysis workflow*.
       Specifically, `git` is often used for tracking source code and other text
       files, i.e the files required for a data analysis. Git, however, does not
       work well with large binary files. For that reason, a different tool,
       `dvc` "Data Version Control", is used used to track data. I said above that
       raw data should not be changed, so technically there is no need for version
       control of that. But *derived data*, produced e.g. as a result of one or
       more data analysis processes, may change over time, as different analysis
       parameters or tools are used. It can be valuable to a researcher to track
       changes to these derived data files. But doing so should be done separately
       from the data analysis files.



## Installation:




## Examples:


### The simplest example:

```yaml

processes:
  picasso-localize:
    input: data/0-raw-data/{dataset}.tif
    output: data/1-picasso-localize/{dataset}.hdf5
    command: python -m picasso localize -o {output_file} {input_file}

```

This is equivalent to:

```yaml

processes:
  picasso-localize:
    input: data/0-raw-data/{dataset}.tif
    output: data/1-picasso-localize/{dataset}.hdf5
    command: python -m picasso localize -o data/1-picasso-localize/{dataset}.hdf5 data/0-raw-data/{dataset}.tif

```


### A simple example using parameters:

```yaml

parameters:
  box_size: [7, 9]
  gradient: [5000, 10000]


processes:
  picasso-localize:
    input: data/0-raw-data/{dataset}.tif
    output: data/1-{process-name}/{box_size}px_{gradient}_{fit_method}/{dataset}.hdf5
    command: python
    arguments: >
      -m picasso localize --baseline 0.0 --qe 0.85  --fit-method {fit_method}
      -b {box_size} -g {gradient}
      --output-dir {output_file.parent}
      data/0-raw-data/{dataset}.tif

```


### A more elaborate example:

```yaml
project:
    template: https://github.com/scholer/cookiecutter-PAINT-analysis

parameters:
  fit_method: mle
  box_size: [7, 9]
  gradient: [5000, 10000, 20000]


processes:

  - name: get-input-data
    input: $CENTRAL_DATA_STORE}/$EXPERIMENT_ID/$ACQUISITION_ID/*.tif
    output: data/0-raw-data/
    command: cp
    arguments: >
      -u "{input}" "{output}"

  - name: picasso-localize
    input: data/0-raw-data/{dataset}.tif
    output: data/1-{process-name}/{box_size}px_{gradient}_{fit_method}/{dataset}.hdf5
    command: python
    arguments: >
      -m picasso localize --baseline 0.0 --qe 0.85  --fit-method {fit_method}
      -b {box_size} -g {gradient}
      --output-dir data/1-{process-name}/{box_size}px_{gradient}_{fit_method}/
      data/0-raw-data/{dataset}.tif

  - name: picasso-render
    input: data/1-{process.name}/{box_size}px_{gradient}_{fit_method}/{dataset}.hdf5
    output: >
      data/2-{process.name}/{box_size}px_{gradient}_{fit_method}/
      {dataset}_{oversampling}x_0-{vmax}.png
    command: python
    local_parameters:
      - {oversampling: 8, vmax: 10}
        {oversampling: 32, vmax: 2}
    arguments: >
      -m picasso render --silent --scaling no
      -o {oversampling} --vmax {vmax}
      data/1-{process-name}/{box_size}px_{gradient}_{fit_method}

config:
  always_export_env_info: true

```


### Processes:

The `processes:` directive can be either a dict of dicts, or list of dicts.

These are functionally identical:

```yaml
processes:
  picasso-localize:
    input: data/0-raw-data/{dataset}.tif
    output: data/1-{process-name}/{dataset}.hdf5
    command: python -m picasso localize --output-dir "data/1-{process-name}" "{input_file}"

processes:
  - name: picasso-localize
    input: data/0-raw-data/{dataset}.tif
    output: data/1-{process-name}/{dataset}.hdf5
    command: python -m picasso localize --output-dir "data/1-{process-name}" "{input_file}"
```

The only difference is that
(a) when using a dict with process-name as key, you cannot have the same process name multiple times,
and (b) using a list indicates order, whereas YAML and Python historically did not guarantee order.
This has now changed, so you should be able to maintain the order from YAML to Python,
but the order is not persisted the other way around (rather, keys are saved in alphabetic order):

```python
>>> print(yaml.dump(yaml.safe_load("""
c: hej
d: word
a: up
b: der
""")))
```

yields:

```
a: up
b: der
c: hej
d: word
```



### Parameters:

If `parameters` (or `local_parameters`) is a list-of-dicts, it is assumed to be an explicit
list of individual parameters sets.
If `parameters` (or `local_parameters`) is just a dict, then it is assumed to be an
implicit combination of parameters. In that case, `zepto-data-runner` will simply
generate all different combinations of the parameters and make separate run for
each combination.

That is to say, the following would be equivalent:

```yaml

parameters:
  fit_method: mle
  box_size: [7, 9]
  gradient: [5000, 10000, 20000]

parameters:
- box_size: 7
  fit_method: mle
  gradient: 5000
- box_size: 7
  fit_method: mle
  gradient: 10000
- box_size: 7
  fit_method: mle
  gradient: 20000
- box_size: 9
  fit_method: mle
  gradient: 5000
- box_size: 9
  fit_method: mle
  gradient: 10000
- box_size: 9
  fit_method: mle
  gradient: 20000

```

In effect, the dictionary form is converted to a parameters list using:

```python
[dict(tups) for tups in itertools.product(*[[(k, v) for v in (vlist if isinstance(vlist, list) else [vlist])] for k, vlist in p.items()])]
```


### Command and arguments:

```yaml
    command: cp
    arguments: >
      -u "{input}" "{output}"
```

If you want, you can also just put arguments in the command and omit the `arguments:` entry:

```yaml
    command: cp -u "{input}" "{output}"
```

If you are worried about escaping your arguments, you can also provide arguments as a list,
and let the program deal with escaping.
For instance, in the above, if the value of `output` contains a quotation marks, then the
program could fail, when trying to invoke:

```
> cp -u /path/to/network/store/experiment/*.tif "the destination folder contains a quote between A " and B"
```


What commands are actually executed for each process/step?

* The runner determines what variables are used in the command arguments,
  and produces a



### Variables and namespaces:

The examples above used variables when creating commands (using string format placeholders),
e.g.:

* In input files: `data/0-raw-data/{dataset}.tif`
* In output files: `data/1-{process.name}/{box_size}px_{gradient}_{fit_method}/{dataset}.hdf5`
* In commands: `-b {box_size} -g {gradient} `.

The variable names comes from three sources:

* Global workflow parameters, i.e. `parameters:`.
* Local process parameters, i.e. `local_parameters:`.
* Input filenames.
  For instance, in the case of `input: data/0-raw-data/{dataset}.tif` the runner will find
  all files in `data/0-raw-data/` matching the glob pattern `*.tif`.
  For each file, it will then extract the part before `.tif` and place it in a variable
  named `dataset`.
  Each input filename can also be referred to as `{c:input_file}` or simply `{input_file}`.
  * PS: The idea of extracting variables from input filenames this was is shamelessly
    stolen from the excellent *Snakemake* workflow-management-system.
* Process information metadata, i.e. `{process.name}`.
  This also includes `{process.all_input_files}` (a string with containing the whole list of
  input files, concatenated/joined by spaces.


  * Discussion: Many build-systems such as `make` includes special short-hand variables
    for referring to inputs and outputs, i.e. `$?` and `$@` (make automatic variables).
    For now, such short-hands are not available, and you should instead use the
    long, explicit variables:
    `{process.all_input_files}`, or `{process.updated_input_files}`,
    and `{process.all_output_files}`.
    We might consider shortening this to just `{process.inputs}` and `{process.outputs}`.
    Notice the plural `s`, indicating this is referring to *all* input files, not each individual file.
    But since `input` is technically a *list*, and what you expect is a `string`,
    then adding those might cause confusion?
    And as always, you can also use them without the namespace qualifier:
    `{all_input_files}` and `{all_output_files}`, or even
    `{inputs}` and `{outputs}`.


The resolution order is:

1. Variables extracted from input filename.
2. Local process parameters (`local_parameters`).
3. Global workflow parameters (`parameters`).

If you want to be more explicit about where a variable comes from, you can use
`i:`, `l:` and `g:` as variable prefixes (for input, local-parameters, and global-parameters, respectively).
This might look something like:

```yaml
  output: >
    data/2-{process:name}/{g:box_size}px_{g:gradient}_{g:fit_method}/
    {i:dataset}_{l:oversampling}x_0-{l:vmax}.png
```


### Nomenclature:


* The whole file is called a `workflow`.
  * The execution of a workflow is called a `workflow run`.
* The workflow consists of individual `processes` (otherwise called "steps" or "rules").
  * Each command call/invocation within a process is called a `process command call`.





