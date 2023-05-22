# First steps with Isabl

Isabl has been programmed to be easily shared. Indeed, it's a plug-and-play service. It relies on [docker compose](https://docs.docker.com/compose/), which runs different instances of [docker images](https://docs.docker.com/get-started/), allowing the service's installation to be fairly easy process.

{% hint style="info" %}
Docker is an open source [containerization](https://www.ibm.com/in-en/cloud/learn/containerization) platform. It enables developers to package applications into containers-standardized executable components combining application source code with the operating system (OS) libraries and dependencies required to run that code in any environment. Containers simplify delivery of distributed applications, and have become increasingly popular as organizations shift to cloud-native development and hybrid [multicloud](https://www.ibm.com/cloud/learn/multicloud) environments.
{% endhint %}

Please, before installing check the external dependencies to avoid issues in case any is missing.

### Installation

The program can be retrieved from Papaemmanueil's Github, for which **academic access needs to be granted**. Then, we'll clone `isabl_demo` (despite being called _demo_, it's a complete version of the software). As it's a modular software, custom versions of, for instance, the front-end web interface can be programmed easily if needed (check that [here](https://docs.isabl.io/contributing-guide)).

```{.bash
# Download the repository
git clone https://github.com/papaemmelab/isabl_demo.git --recurse-submodules
cd isabl-demo
```

Then, load the `demo-profile` to make the tutorial commands available. If desired, the command can be added in the usr's bashrc so that it's loaded in every connection:

```{.bash
source .demo-profile ## Or echo "source .demo-profile" >> ~/.bashrc to have it done automatically on connection.
```

Now, we can already build the app:

```{.bash
demo-compose build # Create the app
demo-compose up -d # Switch on the app
```

It can be switched off (data won't be lost) with:

```{.bash
demo-compose down
```

### Configuration

**1. Register the first user**

The app will be available through the port 8000, so visit [http://localhost:8000/](http://localhost:8000/) to access Isabl. However, no users have yet been created. Thus, we'll first create a superuser:

```{.bash
demo-django createsuperuser
```

After filling the fields required by this command, we'll have our first user.

**2. Make docker visible within Isabl**

In order to use docker-based apps, we must share the docker binary with the app. In my case, I added `- /usr/bin/docker:/usr/bin/docker` in the `volumes` section of the `docker-compose.yml`, located in `${path_to_isabl}/isabl_demo/docker-compose.yml`

{% hint style="warning" %}
When doing configuration changes, or when including new apps, the app must be reset. In order to do that, use:

{% code overflow="wrap" %}
```bash
demo-compose down
demo-compose up -d --build
```
{% endcode %}
{% endhint %}

**3. Create a the CLI client**

After that, we'll create/update the CLI (command-line interface) client. We just need to run this script:

```bash
demo-cli python3.6 ${path_to_isabl}/assets/metadata/create_cli_client.py
```

{% hint style="info" %}
We'll have to modify and rerun this script when we want to add new apps to the framework.&#x20;
{% endhint %}

**4. Create choices**

As we'll see later, Isabl allows us to import sample data as batches using a simple Excel file. In order to avoid issues with mistyping names or including wrong metadata, some **fields cannot be written freely**.

Instead, we'll run another simple script provided by the Isabl authors that will register whatever we need. It will be as simple as running this command:

```bash
demo-cli python3.6 ${path_to_isabl}/assets/metadata/create_choices.py
```

In order to add custom choices, just edit the list in line 6. This is a toy example with one of each of the available fields we can include choices in:

```python
for i in [
    dict(model="assemblies", name="GRCh37", species="HUMAN"),
    dict(model="centers", name="CNAG", acronym="CNAG"),
    dict(model="diseases", name="Mantle B Cell Lymphoma", acronym="MCL"),
    dict(model="techniques", name="DNA", analyte="DNA", method="WG"),
    dict(model="platforms", manufacturer="ILLUMINA", system="NOVASEQ", version="6000"),
]:
    model...
```

***

### Registering data

#### Importing reference files

Reference files are **dependencies** needed by programs that will be run on the samples. They are intended to be **perpetual**. Samples, to be comparable, **must be processed with the same references**. Usually, we'll work with few references. In our context, we'll have one single reference genome version (GRCh38). Also, some programs have specific indexes derived from this references. We'll include them as before starting our analysis as reference files too, so that **any reference file used by any program will be registered**. This is is essential for reproducibility and traceability.

{% hint style="warning" %}
When files are imported, **Isabl moves them** from their original location to the storage directory. **`--symlink`** can be used to create symbolic links instead.
{% endhint %}

Before starting, we must **add a new genome**. We can register it through the admin site (go to `assemblies > add assembly` and type the name and the species). However, we can use the `create_choices.py` script. Just add a new line with the needed option like the ones shown in the file and run it through `demo-cli`. Then, do:

```bash
mkdir staging && cp -r /path/to/my_data/* staging
isabl import-reference-genome \
    --assembly GRCh38 \
    --genome-path staging/reference.fa
```

As said before, some programs have their custom indexes or databases for a genome version. In that case, we can import those files too, and associate them with the assembly we have just created. Nonetheless, **files in the same directory and name than our fasta will be automatically imported with `imported-reference-genome`**

```bash
isabl import-reference-data \
    --identifier GRCh38 \
    --model assemblies \
    --description "My file" \
    --data-id "my_index" \
    --data-src /path/to/file
```

We can check that the import worked with:

```bash
isabl get-reference --resources GRCh38
```

{% hint style="info" %}
Make sure that the fasta index is registered (as genome\_fasta\_fai) so that IGV can locate it.
{% endhint %}

#### Importing metadata <a href="#importing-metadata" id="importing-metadata"></a>

Before starting, remember that field choices must have been added (see here). As soon as we register the choices, we'll be able to download an Excel form that will allow to register the samples with our custom metadata. First, we will create our first project. Although this can be done with the API, using the web interface is much faster here. Just go to [http://localhost:8000/](http://localhost:8000/) and click on the `+` icon on the upper right side of the page:

Create a project easily with the web interface. GIF taken from [Isabl Docs](https://docs.isabl.io/quick-start#create-project).

After that, it's possible to upload a badge by clicking on the darker `+` icon. A dialog will be open to make the submission. Additionaly, a link stating "**Get form**" will initiate the download of an Excel Macros file. When the file is filled, it can be dragged and dropped on the dialog. After pressing on "**Commit submission**", Isabl will read and check that the input is properly filled.&#x20;

Sample metadata submission. GIF taken from [Isabl Docs](https://docs.isabl.io/quick-start#create-project).

After the submission, a project will be similar to this one:&#x20;

Overview of an Isabl project.

#### Importing experimental data

Similarly to the reference-data, experiments are imported through the CLI. The command may look like this:

```{.bash
isabl import-data \
    -di ${path_to_isabl}/assets/staging \ # provide data location 
    -id sample.identifier  \           # match files using experiment id` 
    -fi projects MCL  # filter samples to be imported 
    --ignore-ownership
```

As seen, the `import-data` function is able to **detect automatically the files and match them with their metadata**. It needs to be told what information it can use to match files and samples. In the example, we use the sample id. Additionally, a filter must be provided. In this case, only the samples from the MCL project will be retrieved. **Files can be provided in a tree-organized directory**, since Isabl will perform recursive searches along the root directory. The command above will try to find and match the files with their metadata. After completion, it will return a summary accounting for skipped, missing and matched samples. **Add `--commit` to perform the actual import**. Also, take in mind that, just like **what** happened with the reference data import, files will be moved from the provided directory. If this behavior is unwanted, `--symlink` can be provided to just generate a symbolic link to the files.

{% hint style="danger" %}
Data import will fail sometimes if there are issues with the file ownership. You may add **`--ignore-ownership`** to prevent it crashing.&#x20;
{% endhint %}

The authors show in their documentation that only the some format files are actually recognized. Nonetheless, **custom extensions can be added**. We'll need to modify the `settings.py` file from both `isabl_api` and `isabl_cli`. For instance, they show how to include MAF files:

```python
# ${path_to_isabl}/isabl_api/isabl_api/settings.py 
EXTRA_RAW_DATA_FORMATS = [("MAF", "MAF")]

# ${path_to_isabl}/isabl_cli/isabl_cli/settings.py 
EXTRA_RAW_DATA_FORMATS = [("\\.maf(\\.gz)?$", "MAF")]
```

{% hint style="info" %}
### Only the next format files are actually used:

```python
RAW_DATA_FORMATS = [
     ("FASTQ_R1", "FASTQ_R1"),
      ("FASTQ_R2", "FASTQ_R2"),
      ("FASTQ_I1", "FASTQ_I1"),
      ("BAM", "BAM"),
      ("PNG", "PNG"),
      ("JPEG", "JPEG"),
      ("TXT", "TXT"),
      ("TSV", "TSV"),
      ("CSV", "CSV"),
      ("PDF", "PDF"),
      ("DICOM", "DICOM"),
      ("MD5", "MD5"),]
```
{% endhint %}

Finally, data import through YAML files is supposedly available. However, at the moment of writing this tutorial I have not been able to do so, although I found at least part of the code for such. Instructions on how to are available [here](https://docs.isabl.io/import-data#import-data-from-yaml). Also, data import from cloud has not been coded, the logic behind the data importer can be extrapolated for such task. It be sufficient with writing an alternative python subclass ([see](https://docs.isabl.io/import-data#customizing-import-logic)).

#### External dependecies {.unnumbered}

* **Docker**. Needed to run the Isabl's images. It's also useful to create images of the software that will be used for reproducibility and portability.
* **Docker-compose**. Needed by Isabl to compile and communicate all the images and services.
* **git**. Needed to clone the repositories (thay can be also manually downloaded)
