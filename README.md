# packer-ansible-gcp
Automated image builds for Google Cloud Platform

# Overview

On occasion, it becomes necessary to prebuild custom golden images for utilization with Google Cloud Platform. This project aims to ease the burden of creating these images via automation. [Packer](https://www.packer.io/) and [Ansible](https://github.com/ansible/ansible) are utilized together, with Packer employing the [the Google Compute builder](https://developer.hashicorp.com/packer/plugins/builders/googlecompute). This builder constructs custom images by first launching a GCP instance from an initial source image, then provisioning and customizing that running instance via user-provided automation (Ansible in the case of this project), and finally shutting the instance down and creating a golden image from the quiesced instance storage. This is all done in the GCP account specified via credentials. The builder will create temporary keypairs, security group rules, etc. that provide temporary access to the instance while the image is being created/customized.

Requirements
------------

* Linux or MacOS system, with `git` and `python` available
* GCP account with service account and related private key. Temporary SSH keypairs will be created by Packer to secure the communication stream.
* Packer from https://www.packer.io/downloads
* Ansible installed via the package manager of choice for the given OS/distribution or via `python\pip` module install (virtualenv/venv recommended)

Configuration
------------
1) Clone this repo to local system
2) *cd* to the `packer-ansible-gcp` directory and then `git checkout` the build branch of interest
3) Either edit `packer-build.json` directly or copy to new json file and edit new file  
<tab>modify the following variables:
* `gcp_project_id`: gcp-project-id
* `gcp_image_name`: "name-of-image-1.0-{{isotime `2006-01-02-150405`}}" (default AMI name will include time stamp of build launch)
* `gcp_image_description`: "Name of Image 1.0 {{isotime `2006-01-02-150405`}}" (description will include time stamp of build launch)
* `red_hat_activation_key`: Red Hat Activation key that contains valid subscriptions for products being installed e.g. Red Hat Enterprise Linux
* `red_hat_org_id`: Red Hat organization ID for account that owns above activation key
* `ah_api_token`: Ansible Hub API token, available here: https://console.redhat.com/ansible/automation-hub/token/

4) Edit `gcp-creds.json` directly  
<tab>modify the following variables:
* `project_id`: "gcp-project-id",
* `private_key_id`: "d5fc711d42d1f8c288557633ffcb66c49fdbac30",
* `private_key`: "-----BEGIN PRIVATE KEY-----\nMIIEvQIBADANBgkqh
kiG9w0BAQEFAASCBKcwggSjAgEAHLgsDTvrzKpbw6O2Pwaj\nsBFnyZaH/+a3CGSeY3fC
rT5b4gYiluE0aU\nDqLr0L3+KaI5S8v/zMWc/AwqoQOQ22pjgZcxEz0ucwhuxU9YB7kG
peFotNMKbrOa7RYhyZSXydkb\ngz34VW/XpW2EM9Am9QPAKmE=\n-----END PRIVATE KEY-----\n",
* `client_email`: "service-account-name@gcp-project-id.iam.gserviceaccount.com",
* `client_id`: "011708813610399799394",
* `client_x509_cert_url`: "https://www.googleapis.com/robot/v1/metadata/x509/service-account-name%40gcp-project-id.iam.gserviceaccount.com"


GCP Image Build
-------------
packer build -machine-readable packer-build.json | tee build_artifact-$(date +%Y-%m-%d.%H%M).txt