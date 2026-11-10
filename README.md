# mw-mhd-integration-only-README-tested-locally
mw-mhd-integration-only-README-tested-locally

### This README is based on the README in the original repo at https://github.com/MetabolomicsHub/mw-mhd-integration

For any other updates, please refer to the original repo https://github.com/MetabolomicsHub/mw-mhd-integration

This file is for internal use for convenience. These commands have been tested locally on MW dev server at UCSD.

# Metabolomics Workbench - MHD Model Integration Framework

## Development Environment

Development environment for linux or mac

```bash

# install python package manager uv
# Already installed
#curl -LsSf https://astral.sh/uv/install.sh | sh

# add $HOME/.local/bin to your PATH, either restart your shell or run
# This line can be added to the file like .bashrc or .bash_profile (assuming bash shell) in your home folder, e.g., ${HOME}
export PATH=$HOME/.local/bin:$PATH

# install git from https://git-scm.com/downloads
# Linux command
# Already installed
#apt update; apt install git -y

# Mac command
# brew install git

# clone project from github
git clone https://github.com/MetabolomicsHub/mw-mhd-integration.git

cd mw-mhd-integration

# install python if it is not installed
uv python install 3.12

# install python dependencies
uv sync

# install pre-commit to check repository integrity and format checking
uv run pre-commit

# open your IDE (vscode, pycharm, etc.) and set python interpreter as .venv/bin/python

```

## Commandline Usage

Install with Python 3.12 or newer, then inspect the CLI:

```console
# If python 3.12 (or higher is not default python, then use pip3.12
#$ pip install mw-mhd-integration
$ pip3.12 install mw-mhd-integration
$ mw-mhd-cli --help
Usage: mw-mhd-cli [OPTIONS] COMMAND [ARGS]...
Commands:
  client    utilities to communicate with MHD Server.
  convert   Convert Metabolomics Workbench metadata to MHD file formats.
  create    utilities to create and convert files (announcement file, etc.).
  download  Download study inputs and MHD files from Metabolomics Workbench.
  validate  utilities to validate MetabolomicsHub files.
```

### Convert

`convert` has two profiles. Both commands require `--mw-study-id` and
`--input-dir`; `--output-dir` defaults to `outputs`. The MS profile also
requires `--mhd-id`.

| Command | Usage | Additional options |
| --- | --- | --- |
| `convert to-mhd-legacy-file` | `mw-mhd-cli convert to-mhd-legacy-file [OPTIONS]` | `--output-filename`, `--ontology-lookup-db-path`, `--log-file`, `--verbose` |
| `convert to-mhd-ms-file` | `mw-mhd-cli convert to-mhd-ms-file [OPTIONS]` | `--mhd-id` plus the options listed for the legacy profile |

For offline ontology validation, download the MetaboLights ontology lookup
database from [ontology_lookup.db](https://ftp.ebi.ac.uk/pub/databases/metabolights/ontology-lookup/ontology_lookup.db)
and pass its local path with `--ontology-lookup-db-path`:

Note: The file ontology_lookup.db may be several GB in size, so, it can be put at a common location, or, just do online validation, in which case, there is no need to download it locally, i.e., do not run the curl command below. Also, for online validation, Do not include the
--ontology-lookup-db-path ontology_lookup.db
part in the mw-mhd-cli command below.

```bash
curl -L \
  https://ftp.ebi.ac.uk/pub/databases/metabolights/ontology-lookup/ontology_lookup.db \
  -o ontology_lookup.db

mw-mhd-cli convert to-mhd-legacy-file \
  --mw-study-id ST004083 \
  --input-dir outputs/ST004083 \
  --output-dir outputs \
  --ontology-lookup-db-path ontology_lookup.db
```

Use the same option with `convert to-mhd-ms-file` to provide the local database
for that profile as well.

```bash
mw-mhd-cli download mhd-converter-input --mw-study-id ST004083 --output-dir outputs/ST004083
mw-mhd-cli convert to-mhd-legacy-file \
  --mw-study-id ST004083 \
  --input-dir outputs/ST004083 \
  --output-dir outputs
```

#### MHD MS profile examples

Download the MW inputs for the study ST005363:

```bash
mw-mhd-cli download mhd-converter-input \
  --mw-study-id ST005363 \
  --output-dir "outputs/ST005363"
done
```

Set each `MHD_ID_ST005363` variable to the MHD accession assigned to its study, then
convert the studies with the MS profile:

```bash
mw-mhd-cli convert to-mhd-ms-file \
  --mw-study-id ST005363 \
  --mhd-id "$MHD_ID_ST005363" \
  --input-dir outputs/ST005363 \
  --output-dir outputs

```

### Download

The `download` group accepts `--mw-server-url`, `--username`, and `--password`
before the subcommand. If omitted, the credentials are read from
`MW_REST_API_USERNAME` and `MW_REST_API_PASSWORD`. The server URL is read from
`MW_REST_API_SERVER_URL`; if that variable is also unset, downloads use
`https://www.metabolomicsworkbench.org`.
Username and password are only required to download private studies; public
studies can be downloaded without credentials.

```bash
mw-mhd-cli download \
  --mw-server-url https://www.metabolomicsworkbench.org \
  --username "$MW_REST_API_USERNAME" \
  --password "$MW_REST_API_PASSWORD" \
  mhd-converter-input \
  --mw-study-id ST005293 \
  --output-dir outputs/ST005293
```

The group options are shared with each download subcommand.

| Command | Required arguments | Description |
| --- | --- | --- |
| `download mhd-converter-input` | `--mw-study-id`; optional `--output-dir` (default `outputs`) | Download analysis, metabolite, study-file, and study-summary inputs. |
| `download mhd-file` | `--mhd-id`, `--output-file-path` | Download an MHD model JSON file. |
| `download mhd-announcement-file` | `--mhd-id`, `--output-file-path` | Download an MHD announcement JSON file. |

### Create and validate

The `create` group provides:

| Command | Arguments and options |
| --- | --- |
| `create announcement` | `--mhd-common-data-file-path`, `--target-mhd-common-data-file-url`; required if it is mhd ms profile study: `--mhd-id`, optional `--output-dir`, `--output-filename` |
| `create neo4j-input` | `MHD_STUDY_ID MHD_MODEL_FILE_PATH`; optional `--output-dir`, `--output-filename` |
| `create sdrf` | `MHD_STUDY_ID MHD_MODEL_FILE_PATH`; optional `--output-dir`, `--output-filename`, `--assay-name` (omit to generate files for all assays) |

The `validate` group provides:

| Command | Argument and options |
| --- | --- |
| `validate mhd` | `MHD_COMMON_DATA_FILE_PATH`; required if it is mhd ms profile study: `--mhd-id`, optional `--output-file-path` |
| `validate announcement` | `ANNOUNCEMENT_FILE_PATH`; required if it is mhd ms profile study: `--mhd-id`, optional `--output-file-path` |

```bash
mw-mhd-cli create announcement \
  --mhd-common-data-file-path outputs/ST004083.mhd.json \
  --target-mhd-common-data-file-url 'https://example.org/ST004083.mhd.json'
mw-mhd-cli validate mhd outputs/ST004083.mhd.json
```

### MHD server client

The `client` group contains these commands:

| Command | Main inputs or purpose |
| --- | --- |
| `client announce` | Submit an announcement; requires reason, MHD ID, announcement path, and repository dataset ID. |
| `client fetch-mhd-accession` | Fetch/register an accession for `--dataset-repository-identifier`; supports `--accession-type`. |
| `client api-token create` | Create a token using `--token-name` and a signed JWT. |
| `client api-token delete` | Invalidate a token using `--api-token` and signed JWT authorization. |
| `client api-token list` | List token summaries; optionally filter by name or include invalid tokens. |
| `client api-token validate` | Validate `--api-token` using signed JWT authorization. |
| `client key-pair create` | Generate RSA keys; optional `--output-folder` and `--key-size`. |
| `client key-pair validate` | Validate required private and public key paths. |
| `client signed-jwt create` | Create a JWT using repository name and private/public key paths. |
| `client signed-jwt show` | Display a JWT from a value or file; optionally validate with a public key. |

Client commands also accept server URL, API version, and credential options as
shown in their help. For example, to request a legacy accession assignment:

```bash
mw-mhd-cli client fetch-mhd-accession \
  --dataset-repository-identifier ST004083 \
  --accession-type legacy
```

Inspect full options and usage for any command with
`mw-mhd-cli <command> <subcommand> --help`. Credential creation, deletion,
accession registration, and announcement submission perform server actions.

