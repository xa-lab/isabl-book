# Writing Isabl apps

### Introduction

Isabl Applications allow to deploy systematically bioinformatic tools over badges of samples. The apps are mere **python wrappers** that process the inputs and launch the analysis (local, cluster or cloud). Thus, **the tools can be written and implemented in whatever programming language or manner is convenient**, since ultimately the app will simply make a call to that tool. Additionally, apps can be configured to control results at sample level, individual level or even project level. For the latter, merge logic over individual level can be applied, thus allowing to extract more general information.

{% hint style="warning" %}
**Results from applications are stored as analyses, for which uniqueness is a function of the experiments used**. Analyses can be linked to multiple experiments or references experiments, allowing to use:

* Single target analyses (QC) - Tumor-normal pairs (Somatic variant calling)
* Target vs pool of references (Copy number)
* Multiple targets vs multiple references (Contamination testing)&#x20;
{% endhint %}

### Writing a Hello World app

The Isabl Docs show a really easy [example](https://docs.isabl.io/writing-applications#how-does-an-application-look-like) that we can reproduce next. First, we need to load the **`AbstractApplication`** class from `isabl_cli`. This class is simply a template that misses the essential elements. In Python, if a class takes as argument another class, it will inherit its information. Thus, we will create a class that inherits `AbstractApplication` and then we fill the blank attributes needed to make our tool (Hello World) run.

```python
from isabl_cli import AbstractApplication 
from isabl_cli import options


class HelloWorldApp(AbstractApplication):

    NAME = "HELLO WORLD"
    VERSION = "1.0.0"
    cli_options = [options.TARGETS]

    def get_command(self, analysis):
        experiment = analysis.targets[0]  
        return f"echo {experiment.sample.identifier} {experiment.raw_data} "
```

We create an app class called `HelloWorldApp`, that inherits from `AbstractApplication`. First, we will name our app and give it a version number. Additionally, we have also loaded `options` from `isabl_cli`. This script is used to easy implement some arguments in the app's CLI. They are basically [decorators](https://realpython.com/primer-on-python-decorators/) that rely on the [click](https://click.palletsprojects.com/en/8.1.x/) module to build the Command-Line Interface. In this case, we use `TARGETS`, which appends the `-fi` argument in the CLI and will make the API filter by target experiments.

Then, we just need to build the command that will be run with a function that must be called `get_command`. We store the information of the experiment's target , and then we simply return an `echo` that prints the sample identifier and the raw data.

{% hint style="info" %}
In Python, those API's responses are store as [Munch dictionaries](https://github.com/Infinidat/munch). They are essentially like dictionaries, but they allow value retrieval with dots, increasing readability.&#x20;
{% endhint %}

The app can be run with:

```bash
isabl apps hello-world --filters projects 102
```

### Creating advanced applications

#### Adding more metadata

Just like with `NAME` and `VERSION`, we can provide some more information. In case the app needs reference data, we can link it to a specific **genome assembly**, so it will directly load the correct genome version. We must provide the `ASSEMBLY` and the `SPECIES`:

```python
class bwaApp(AbstractApplication):

    NAME = "BWA-MEM2"
    VERSION = "1.0.0"
    ASSEMBLY = "GRCh38"
    SPECIES = "HUMAN"
```

Additionally, the application is stored as a database object, so we can include some metadata that will appear in the database:

```python
class bwaApp(AbstractApplication):
    [...]
    application_description = "This app runs BWA-mem2"
    application_url = "https://github.com/pbousquets/smk_pipelines/tree/main/bwa-pipeline"
```

#### CLI configuration

We saw before that we can pass `options.TARGETS` to the `cli_options` variable. However, is actually a list that can accept more available options:

* `TARGETS`. Enables `-fi` to provide key value pair of RESTful API filters used to retrieve experiments (e.g. `-fi sample.category TUMOR`). Each experiment will be linked to a new analysis in a one-to-one basis using the analysis.targets field.
* `PAIRS`, `PAIR`, `PAIRS_FROM_FILE`. Enable `--pairs (-p)`, `--pair (-p)`, `--pairs-from-file (-pf)` to provide pairs of target, reference experiments (e.g. `-p TUMOR-ID NORMAL-ID`). Each pair will be linked to a new analysis (targets list is one experiment, references list is one experiment).
* `REFERENCES`, `NULLABLE_REFERENCES`. Enable `--references-filters (-rfi)` to provide filters to retrieve reference experiments. This has to be coupled with **`TARGETS`**, each analysis will then be linked to one target, and to as many references.

Definitions extracted from [Isabl Docs](https://docs.isabl.io/writing-applications#command-line-configuration).

With these filters, we'll automatically be returned the experiments (input files) that will be used. Nonetheless, these options may not be enough. In that case, we can implement new options through `click.option` and create the function `get_experiments_from_cli_options` to run custom filters that will return the experiments:

```python
from isabl_cli import api
import click
class bwaApp(AbstractApplication):
    [...]
    cli_options = [
    click.option("--method", help="Technique method", default="WG")
    ]

    def get_experiments_from_cli_options(self, **cli_options):
        method = cli_options["method"]
        filters = dict(technique__method=method, target)
        targets = api.get_instances("experiments", **filters)
        return [(targets, [])]
```

Additionally, there are **three more CLI options** that will be enabled if the variables are set as true:

```python
cli_allow_force = True # Wipe unfinished analyses and start from scratch.
cli_allow_restart = True # Attempt restarting failed analyses from previous checkpoint.
cli_allow_local = True # Run analyses locally, one after the other
```

{% hint style="warning" %}
When using **`--force`**, the **existent analysis won't be removed**. They are moved to `<BASE_STORAGE_DIRECTORY>/.analyses_trash`. A temporal cleanup of this folder is highly recommended. It can easily be programmed with [cron](https://vitux.com/how-to-setup-a-cron-job-in-debian-10/).&#x20;
{% endhint %}

#### Application settings

Apps can be run differently according to the system where it will run (executables' location), references or system requirements. Those can be provided through the **`application_settings`**. It's a simple dictionary that will be used later to automatically adjust these variables.

```python
class bwaApp(AbstractApplication):
    [...]
    application_settings = {
      available_threads: 80,
      available_RAM: 300000, #Mb
    }
```

{% hint style="warning" %}
When using docker-based tools, we can use the `application_settings` dictionary to pass the beginning of the command by making use of a simple function (`get_docker_command`) that comes within the Isabl code:

```python
class fastqQC(AbstractApplication):
    [...]
    application_settings = {
    "fastqc": get_docker_command("biocontainers/fastqc:v0.11.9_cv8", "fastqc"),
    "multiqc": get_docker_command("ewels/multiqc:v1.12")
    }
```

This function needs the image id and, optionally, the base command that will be run. In the first case, we explicitly ask to run the command `fastqc`, while the _multiqc_ image doesn't need that because it runs the program automatically.&#x20;
{% endhint %}

If a setting is intended to be **optional**, it can be declared as _`None`_. If not **declared yet, but compulsory**, they can be set as _`NotImplemented`_ Also, **all declared settings will be considered the default settings**, but they can be overridden using the database application field `settings`.

```python
bwaApp().application.settings
{
  "available_threads": 20,
  "reference": "reference_data_id:genome_fasta"
}
```

* **Application settings validation**

If application settings are being used, it's a good practice to validate if they were properly introduced. **`validate_settings`** function can be used for this task:

```python
class bwaApp(AbstractApplication):
    [...]
    def validate_settings(self, settings):
        assert isinstance(settings.available_threads, int), f"available_threads is expected to be an integer, {settings.available_threads} was introduced."
        self.validate_reference_genome(settings.reference)
```

Here we check if both variables are integers, else we return an `AssertionError` with `assert`.

#### Experiment validation

Before running an app, we can make use of the fact that they are metadata-driven to check that our input makes sense with the app that will be run. For that, we use the **`validate_experiments`** function, that will work with the same logic than `validate_settings`.

```python
class bwaApp(AbstractApplication):
    [...]
    def validate_experiments(self, targets, references):
        assert len(targets) == 1, "Only one target allowed per analysis"
        self.validate_dna_only(targets) 
```

Some of the **most common validators**, like `validate_dna_only` or `validate_reference_genome`, are **already included** in the CLI. Check `${path_to_isabl}/isabl_cli/isabl_cli/app.py` to see what else is included.

#### Command building

Until now, we have: a) specified the type of input, b) specified the settings and c) validated the inputs. Now, we can build the command, using both the settings and the analysis' experiments.

```python
from os.path import join

class bwaApp(AbstractApplication):
    [...]
    def get_command(self, analysis, inputs, settings):
        bwa = settings.bwa
        target = analysis.targets[0]
        ref = settings.reference
        threads = settings.available_threads
        fqs = [i["file_url"] for i in target["raw_data"] if i["file_type"].startswith("FASTQ")]
    fqs = " ".join(fqs)
    out = analysis["storage_url"] + target.sample.identifier
        return f'{bwa} mem -t {threads} -R "@RG\tID:{target.sample.identifier}\tSM:{target.sample.identifier}\tPL:ILLUMINA" {settings.reference} {fqs}] | samtools view -uS -  > {out} && samtools index {out} -@ {threads}'
```

If a custom `cli_option` is needed for the command, it's available using the settings' `run_args` attribute (i.e.: `settings.run_args.MY_OPTION`).

#### Application results

The next step is to specify what results are going to be produced. Although not necessary for the task completion, it will allow databasing the results so they are accessible through the API (and the web).

First, we define what results are expected with `application_results`:

```python
class bwaApp(AbstractApplication):
    [...]
    application_results = [
        "bam": {
            "frontend_type": "text-file",
            "description": "Alignment bam",
            "verbose_name": "Alignment bam",
            "optional": False,
            "external_link": "https://samtools.github.io/hts-specs/SAMv1.pdf",
        },
        "bai": {
            "frontend_type": "text-file",
            "description": "Alignment bam index",
            "verbose_name": "Alignment bam index",
        },
        ...
    ]
```

* **`description`** Information about the result (required)
* **`verbose_name`** Name displayed for the result in the results list (required)
* **`optional`** If `False` and result is missing, an alert will be shown online (optional)
* **`external_link`** URL to a resource that may explain about the result (optional)
* **`frontend_type`** Defines how the result should be displayed in the frontend. Options are: `text-file` (shown as raw file), `tsv-file` (tabular format: VCF, TSV), `string`, `number`, `image` (a preview can be displayed), `pdf`, `html` (rendered in a iframe), `igv_bam` (set it as bam:bai).

{% hint style="info" %}
Results are stored as read only by default. Use `application_protect_results = False` to make apps re-runnable.
{% endhint %}

Finally, we link the actual files with the specified expected output, with `get_analysis_results`:

```python
class bwaApp(AbstractApplication):
    [...]
    def get_analysis_results(self, analysis):
        bam = f"{analysis.storage_url}/{target.sample.identifier}.bam")
        index = f"{bam}.bai"
        return {
            "bam":bam,
            "bai": index
        }
```

### Auto-merge

Isabl apps can produce both individual and project level summaries. In some kind of analyses, we may be interested in producing global results. Thus, it's possible to include in the app individual and project level routines to merge the results.

```python
class VariantCallerApp(AbstractApplication):
    [...]
    application_project_level_results = { #Project-level results
        "merged": {
            "frontend_type": "text-file",
            "description": "Merged VCF",
            "verbose_name": "Merged VCF",
        },
        "count": {
            "frontend_type": "number",
            "description": "Number of samples",
            "verbose_name": "Number of samples",
        },
    }
    def merge_project_analyses(self, analysis, analyses):
        with open(join(analysis.storage_url, "merged.vcf"), "w") as f:
            for i in analyses:
                with open(i.results.vcf) as output:
                    f.write(output.read())
        analysis.count_files = len(analysis)

    def get_project_analysis_results(self, analysis):
        merged = join(analysis.storage_url, "merged.vcf")
        count = analysis.count
        return {
            "merged": merged,
            "count": count
        }
```

The logic can be reused for individual logic just by using:

```python
class VariantCallerApp(AbstractApplication):
    [...]
    # reuse the project merge logic at the individual level
    application_individual_level_results = application_project_level_results
    merge_individual_analyses = merge_project_analyses
    get_individual_analysis_results = get_project_analysis_results
```

Merge can be manually done any time with:

```bash
# for project level auto merge
isabl merge-project-analyses --project <project id> --application <application id>

# for individual level automerge
isabl merge-individual-analyses --individual <individual id> --application <application id>
```

{% hint style="info" %}
Validations at project and individual level are also available through `validate_project_analyses` and `validate_individual_analyses`.&#x20;
{% endhint %}

### Registering a new app

An app's directory, usually looks like this:

```bash
isabl_apps/
├─ apps/
│  ├─ my_new_app/
│  │  ├─ apps.py
│  │  ├─ constants.py
│  │  ├─ __init__.py
```

The app is mostly written in `apps.py`. The `constants.py` script is used to code the static variables, usually the expected outputs (`application_results`), which makes easier to find out when dealing for the first time with the app. If using this file, **you'll need to import the script in `apps.py`**. Finally, **`__init__.py` is an empty file needed to make python consider it is a package**.

When the app is done, just add it it in the client's settings so that it is available in the CLI:

```python
## In ${path_to_isabl}/assets/metadata/create_cli_client.py
"INSTALLED_APPLICATIONS": [
    "example_apps.apps.MyNewApp",
] 
```

{% hint style="info" %}
### Remember to rebuild the app and run the script!

```bash
demo-compose up -d --build
demo-cli python3.6 ./assets/metadata/create_cli_client.py
```
{% endhint %}

### Extra

The text below is a direct reproduction from [Isabl Docs': Optional Functionality](https://docs.isabl.io/writing-applications#optional-functionality)

* **Analyses Inputs and Dependencies on Other Applications**

**Application inputs** are analysis-specific settings (settings are the same for all analyses, yet inputs are potentially different for each analysis). Inputs can be formally defined using `application_inputs`. Inputs set to `NotImplemented` are considered required and must be resolved using `get_dependencies`:

```python
from isabl_cli import utils

class HelloWorldApp(AbstractApplication):
  
    application_inputs = {"previously_generated_result": NotImplemented}

    # get dictionary of inputs this function is run before get_command.
    # must return a tuple: (list of analyses dependencies primary keys, inputs dict)
    def get_dependencies(self, targets, references, settings):
        result, analysis_key = utils.get_result(  # return result value and key of analysis that generated it
            experiment=targets[0],                # get result for the first target experiment
            application_key=123,                  # primary key of previous app in database
            result_key="a_result_name",           # name of result in the app definition
        )

        return [analysis_key], {"previously_generated_result": result}
```

{% hint style="info" %}
The main objective of `application_inputs` and `get_dependencies` is to retrieve results and analyses that should be linked to the newly created analysis. Linked analyses are accessible from the analysis detail frontend view.
{% endhint %}

* **Get after completion status**

In certain cases you don't want your analyses to be marked as `SUCCEEDED` after completion, as you may want to flag them for manual review or leave them to know that you need to run an extra step on them. For these cases, you may want to set the _after-completion_ status to `IN_PROGRESS`:

```python
class HelloWorldApp(AbstractApplication):
    def get_after_completion_status(self, analysis):
        return "IN_PROGRESS"
```

* **Get notified when analyses fail**

You can configure Isabl API to periodically check if any analysis has failed and send you email notifications. To do so, head to the admin site at `/admin/django_celery_beat/periodictask/add/` and in _Task (registered)_ select `isabl_api.tasks.report_status_change_task`, then create a 1 hour interval, and provide the following Keyword arguments `{"status": "FAILED", "seconds": 3600}` (i.e. _every hour check how many analyses failed in the past hour_):&#x20;
