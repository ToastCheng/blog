---
title: "Terraform: Infra as a code"
date: 2020-11-25T18:43:00+0800
tags: [note, terraform, infrastructure]
categories: technique
---

Terraform is a powerful infrastructure provisioning tool.

It just reminds me how kubernetes impress me when I first learned it. For example, kubernetes automates the work flow that previously you have to manually do, such as deploying application on machines, managing the CPU, memory resource for differnet servers, scaling the services, and so on.

Under this kind of development, everything is so handy and so automatic. However, there are a couple things that I still have to manually take care of. Such as...

- setup the initial kubernetes cluster
- configure the node pool
- configure databases (such as createing index in Firestore)
- network policy, firewall

You can imaging that the beautiful automation kubernetes give us happens only after the cluster is created. Before that, you have to have your infrastructure ready.

And when it comes to multiple environments, such as dev, sta, prod. You have to carefully setup the environment precisely identical. Think about having multiple environments and as server loading increase/decrease, new product kicks in, you have to maually update the infra. Sounds like a nightmare. Its a labor-intensive work!

This is where [terraform](https://www.terraform.io/) comes into play.

Using terraform, you can have your infrastruture as a code, automate it, reproduce it in a jiffy.

## Install

If you are using MacOS, it can be install via `brew`.

```bash
brew tap hashicorp/tap
brew install hashicorp/tap/terraform
# autocomplete!!
terraform -install-autocomplete
```

### Write config

Here we are creating a config file `config.tf` that provisions a compute engine in GCP.

**Note**
terraform will look at the current directory and find all the files with `.tf` extension. So it is ok to split different block into different file. You can even leverage this behavior by adding some part of the config that contains credential into `.gitignore`.

#### 1. `provider`

First you have to specify `google` as a provider in `provider` block. You have to set `project` to your own project, and provides the credential file, or you can specify the file path via environment variable `GOOGLE_APPLICATION_CREDENTIALS` just as other GCP sdk does.

**Note**

1. Multiple `required_providers` are not allowed. You can have only one provider in a project i.e., a directory, or you will get `Error: Duplicate required providers configuration`.
2. Carefully grant your credential. Or you will have terraform to create something it does not have permission to. Such as creating network: `Error: Error creating Network: googleapi: Error 403: Required 'compute.networks.create' permission for 'projects/xxxxx/global/networks/terraform-network', forbidden`.

```
provider "google" {
  version = "3.5.0"
  project = "{{YOUR GCP PROJECT}}"
  region  = "us-central1"
  zone    = "us-central1-c"

  credentials = file("service-account.json")
}
```

#### 2. `resource`

Then you specify what `resource` you want for your infra. For example, a compute engine (GCE).

```
resource "google_compute_instance" "vm_instance" {
  name         = "terraform-instance"
  machine_type = "f1-micro"

  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-9"
    }
  }

  network_interface {
    # A default network is created for all GCP projects
    network = "default"
    access_config {
    }
  }
}
```

So the full `config.ts` looks like:

```
provider "google" {
  version = "3.5.0"
  project = "{{YOUR GCP PROJECT}}"
  region  = "us-central1"
  zone    = "us-central1-c"

  credentials = file("service-account.json")
}

resource "google_compute_instance" "vm_instance" {
  name         = "terraform-instance"
  machine_type = "f1-micro"

  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-9"
    }
  }

  network_interface {
    # A default network is created for all GCP projects
    network = "default"
    access_config {
    }
  }
}

```

Then you are good to go. Run the commands below.

```bash
# initialize the project based on config file, it will download the required provider.
terraform init

# provision!!
terraform apply

# cleanup
terraform destroy
```

`terraform init` setup the project, downloads the required provider from Terraform Registry.
`terraform apply` directly provision you need.
`terraform destroy` cleanup.

There are two command worth knowing,
`terraform refresh` query the current state of your infrastructure.
`terraform plan` create an execution plan from current state to desired state, like dry-run.
These two commands are good for knowing what happens, but `terraform apply` actually does the `refresh` and `plan` stuff before actually apply them, so if you really know what is going on, simply run `terraform apply` will suffice.

Wait for a second or two, your infra is now prepared!
`Apply complete! Resources: 2 added, 0 changed, 0 destroyed.`

![](https://i.imgur.com/FgYnYDg.png)

## Other resource

There are other very useful resources:

- [create firestore index](https://registry.terraform.io/providers/hashicorp/google/latest/docs/resources/firestore_index)
- [Configuring cluster and node pool](https://registry.terraform.io/providers/hashicorp/google/latest/docs/resources/container_cluster) such as:
  - add taints
  - add tags
  - ...and so on

## References

- [getting started](https://registry.terraform.io/providers/hashicorp/google/latest/docs/guides/getting_started)
