# Example scripts for the Python SDK for Prisma Cloud APIs

Example scripts that utilize the Python SDK for Prisma Cloud APIs.


## Table of Contents

* [Support](#Support)
* [Setup](#Setup)
* [Configuration](#Configuration)
* [Usage](#Usage)
    * [CSPM Scripts](#CSPM-Scripts)
    * [CWP Scripts](#CWP-Scripts)


## Support

This project has been developed by members of the Prisma Cloud CS and SE teams, it is not Supported by Palo Alto Networks.
Nevertheless, the maintainers will make a best-effort to address issues, and (of course) contributors are encouraged to submit issues and pull requests.


## Setup

These scripts are written and tested in Python 3.x.
If you need to install Python 3, you can get more information at [Python's Home Page](https://www.python.org/).
You will also need [PIP](https://pypi.python.org/pypi/pip). 

These scripts require the `prismacloud-api` Python package.

Install the SDK via `pip3`:

```
pip3 install prismacloud-api
```

Please refer to [PyPI](https://pypi.org/project/prismacloud-api) for details.

These scripts may also require other packages. To install those packages, execute:

```
pip3 install -r requirements.txt 
```


## Configuration

Configuration for these scripts can be specified each time on the command-line, via environment variables, or via a configuration file.

Configuration options include:

- `--name NICKNAME`     (Optional) Nickname for the Prisma Cloud Tenant (or On-Premise Compute Console)
- `--url URL`           (Required) URL for the Prisma Cloud Tenant (or On-Premise Compute Console). 
- `--identity IDENTITY` (Required) Prisma Cloud Access Key (or On-Premise Compute Username)
- `--secret SECRET`     (Required) Prisma Cloud Secret Key (or On-Premise Compute Password)
- `--verify VERIFY`     (Optional) SSL Verification. Options: `true`, `false`, or the path to a certificate bundle (Default: `true`)
- `--config FILE`       (Optional) Configuration file. (Default: `~/.prismacloud/credentials.json`)
- `--save`              (Optional) Save configuration to a configuration file

Configuration is stored as cleartext JSON, by default in the `~/.prismacloud/` directory, unless you specify an alternative via `--config`.

The following environment variables can be used instead of the equivalent command-line options or a configuration file:

```
settings = {
    "name":     os.environ.get('PC_NAME'),
    "url":      os.environ.get('PC_URL'),
    "identity": os.environ.get('PC_IDENTITY'),
    "secret":   os.environ.get('PC_SECRET'),
    "verify":   os.environ.get('PC_VERIFY')
}
```


## Usage

For detailed documentation of each script's parameters, specify `-h` or `--help` when executing the script.


### CSPM Scripts


#### pcs_compliance_export

Use this script to export an existing Compliance Standard (and its Requirements and Sections) to a file, for backup ... or to import into another tenant.

Example:

```
python3 pcs_compliance_export.py "GDPR" "example-compliance-standard.json"
```


#### pcs_compliance_import

Use this to script import an exported Compliance Standard (and its Requirements and Sections) into a new Compliance Standard.
To import the associated Policy mappings, specify the `--policy` parameter. 
To associate custom Policies requires first running `pcs_policy_custom_export` to generate a mapping file (and `pcs_policy_custom_import` when importing into another tenant).
It will check for duplicates before importing.

Example:

```
python3 pcs_compliance_import.py "example-compliance-standard.json" "GDPR Imported" --policy
```


#### pcs_policy_custom_export

Use this script to export custom Policies to a file, for backup ... or to import into another tenant.

Example:

```
python3 pcs_policy_custom_export.py "example-custom-policies.json"
```


#### pcs_policy_custom_import

Use this to script import custom Policies.
By default, imported Policies will be disabled, to maintain the status of imported Policies, specify `--maintain_status`
It will check for duplicates before importing.

Example:

```
python3 pcs_policy_custom_import.py "example-custom-policies.json"
```


#### pcs_policy_set_status

Use this script to enable or disable Policies globally for a tenant (all Policies or filtered by Cloud Type, Policy Type, Policy Severity, or Compliance Standard).
This is primarily used to set up a new environment with every Policy enabled, or to update an environment after a large number of new Policies have been released.

Example:

```
python3 pcs_policy_set_status.py --policy_type config enable
```

Use this script to enable Policies that are associated with a specific Compliance Standard (or Compliance Standards).

Example:

```
python3 pcs_policy_status.py --policy_type all disable

python3 pcs_policy_status.py --compliance_standard "GDPR" enable
```


#### pcs_rotate_service_account_access_key

Service accounts are limited to two access keys.
Use this script to rotate service account access keys, deleting the oldest key (if it exists) and creating a new key.
It appends an incremented version number (` vN`) to the name of the new key.

Example:

```
python3 pcs_rotate_service_account_access_key.py "Example Service Account Key" --days 30

Next Access Key: {'id': 'aaaaaaaa-bbbb-cccc-dddd-eeeeeeeeeeee', 'secretKey': 'fffffffffffffffffffffffffff=', 'name': 'Example Service Account Key v2'}
```


#### pcs_user_import

Use this script to import a list of Users from a CSV file, assigning the imported Users to the specified Role.
It will check for duplicates before importing.

Example:

```
python3 pcs_user_import.py "example-import-users.csv" "Example Prisma Cloud Role to assign to the imported Users"
```


### CWP Scripts


#### pcs_compute_forward_to_siem

Use this script to forward Audits, and Console History and Logs from Prisma Cloud Compute to a SIEM.

It is expected to be called once an hour, by default, to read from the Prisma Cloud API and write to your SIEM API.

It depends upon the SIEM to deduplicate data, and requires you to modify the `outbound_api_call()` function for your SIEM API.

Example:

```
python3 pcs_compute_forward_to_siem.py --console_history --console_logs
```

You can specifically disable forwarding of Audits with `--no_audit_events`.

Note that `--host_forensic_activities` results in high-volume/time-intensive API calls.

#### pcs_incident_archiver

Use this script to archive runtime incidents in bulk based on the contents of a CSV file.

Workflow:

1. Navigate to Compute > Monitor > Runtime > Incident Explorer > Active.
1. Click the "CSV" button to export active incidents.
1. Remove (e.g. in Excel, Google Sheets, etc) rows that should NOT be archived.
1. Save the CSV.
1. Invoke this script on the saved CSV to archive each incident it contains.

Assumptions:

- The input CSV file MUST have a header row.
- The header row for incident IDs MUST be named `ID`.

Example:

```
$ cat input.csv
ID
62fff91fc3be7f962ec7ea47
62fff91fc3be7f962ec7ea4c
62fff91fc3be7f962ec7ea4c
62fff91fc3be7f962ec7ea4f
62fff91fc3be7f962ec7ea51
62fff9b6c3be7f962ec7ea56
```

```
python3 scripts/pcs_incident_archiver.py input.csv
Preparing to archive 5 incidents ...

Ready to execute commands against your Prisma Cloud tenant ...
Would you like to continue (y or yes)? y

Archived incident 62fff9b6c3be7f962ec7ea56
Archived incident 62fff91fc3be7f962ec7ea4c
Archived incident 62fff91fc3be7f962ec7ea47
Archived incident 62fff91fc3be7f962ec7ea51
Archived incident 62fff91fc3be7f962ec7ea4f
```

#### pcs_images_packages_read

Use this script to inspect the packages in all of the container images (or one image, specified by `--image_id`) that have been scanned by Prisma Cloud Compute.

Example:

```
python3 pcs_images_packages_read.py

python3 pcs_images_packages_read.py --image_id "sha256:c004737361182d3cd7f38e6d9ce4a44f2a349b8dc996834e2cba0defcd0cb522"
```

An alternate usage is to specify a package via `--package_id` to search for containers with a specific package, and optionally a version.

Example:

```
python3 pcs_images_packages_read.py --package_type jar --package_id log4j

python3 pcs_images_packages_read.py --package_type jar --package_id log4j:2.14.1 --version_comparison lt --output_to_csv True
```


### Other CSPM and CWP Scripts


#### pcs_cloud_account_import_azure (in progress)**

This is the framework for importing a CSV (template in the `templates` directory) with a list of Azure accounts into Prisma Cloud.
Note: This is still a work in progress: the basic import framework is running, but validation of CSV and duplicate name checking has not been implemented yet.

Example:

```
python3 pcs_cloud_account_import_azure.py prisma_cloud_account_import_azure_template.csv
```


#### pcs_posture_endpoint_client

This is a generic tool for prototyping with the Cloud Security Posture API.
It sends output to stdout (and optionally to file) and errors/info sent to STDERR, so that it works in a pipeline which makes it `jq` friendly.
Please note this tool is not intended as a replacement for well-formed scripts and functions.

Example 1: GET

```
python3 pcs_posture_endpoint_client.py GET v2/policy
```

Example 2: POST

```
cat > body.json <<EOF
{ "name": "test standard", "description":"test description" }
EOF
python3 pcs_posture_endpoint_client.py POST /compliance --request_body body.json
```


#### pcs_compute_endpoint_client

This is identical to `pcs_posture_endpoint_client`, except it uses the CWP API rather than the CSPM API.
 
Example 1: GET

```
python3 pcs_compute_endpoint_client.py GET api/v1/statuses/intelligence
```
