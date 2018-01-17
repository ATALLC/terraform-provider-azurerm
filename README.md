AzureRM Terraform Provider
==================

## ATA Notes ##

This project expects to use scripts from https://github.com/Microsoft/openshift-origin. That had some issues so I forked it here https://github.com/ATALLC/openshift-origin. So clone that and then there's an openshift_script_path that's being overrided in ata.auto.tfvars and you can specify the path on your local box. To access the vms, the ssh key as well as other access details is in https://www.dropbox.com/home/ATA-Internal-DevOps/Projects/SOSOA?preview=terraform-example-openshift.tfvars.gpg. Ask Mike about the password. It can be decrypted with the command "gpg -d <file>".

There's a script in the utility-scripts project called add-vms-to-etc-hosts.sh that will put any public ips of vms in our azure subscription into /etc/hosts to make it easier to ssh in. Terraform sets up a bastion vm that has a public ip and can be used to ssh to the other vms, although you may need to copy the private key into it first.

Got the installer working but the nodes besides master were not showing up with "oc get nodes" for some reason. Had to follow[the scaleup instructions](https://docs.openshift.com/enterprise/3.0/install_config/install/advanced_install.html#adding-nodes-advanced) to get them to show up.

- Website: https://www.terraform.io
- [![Gitter chat](https://badges.gitter.im/hashicorp-terraform/Lobby.png)](https://gitter.im/hashicorp-terraform/Lobby)
- Mailing list: [Google Groups](http://groups.google.com/group/terraform-tool)

Requirements
------------

-	[Terraform](https://www.terraform.io/downloads.html) 0.10.x
-	[Go](https://golang.org/doc/install) 1.9 (to build the provider plugin)

Building The Provider
---------------------

Clone repository to: `$GOPATH/src/github.com/terraform-providers/terraform-provider-azurerm`

```sh
$ mkdir -p $GOPATH/src/github.com/terraform-providers; cd $GOPATH/src/github.com/terraform-providers
$ git clone git@github.com:terraform-providers/terraform-provider-azurerm
```

Enter the provider directory and build the provider

```sh
$ cd $GOPATH/src/github.com/terraform-providers/terraform-provider-azurerm
$ make build
```

Using the provider
----------------------

```
# Configure the Microsoft Azure Provider
provider "azurerm" {
  subscription_id = "..."
  client_id       = "..."
  client_secret   = "..."
  tenant_id       = "..."
}

# Create a resource group
resource "azurerm_resource_group" "production" {
  name     = "production"
  location = "West US"
}

# Create a virtual network in the web_servers resource group
resource "azurerm_virtual_network" "network" {
  name                = "productionNetwork"
  address_space       = ["10.0.0.0/16"]
  location            = "West US"
  resource_group_name = "${azurerm_resource_group.production.name}"

  subnet {
    name           = "subnet1"
    address_prefix = "10.0.1.0/24"
  }

  subnet {
    name           = "subnet2"
    address_prefix = "10.0.2.0/24"
  }

  subnet {
    name           = "subnet3"
    address_prefix = "10.0.3.0/24"
  }
}
```

Further [usage documentation is available on the Terraform website](https://www.terraform.io/docs/providers/azurerm/index.html).

Developing the Provider
---------------------------

If you wish to work on the provider, you'll first need [Go](http://www.golang.org) installed on your machine (version 1.9+ is *required*). You'll also need to correctly setup a [GOPATH](http://golang.org/doc/code.html#GOPATH), as well as adding `$GOPATH/bin` to your `$PATH`.

To compile the provider, run `make build`. This will build the provider and put the provider binary in the `$GOPATH/bin` directory.

```sh
$ make build
...
$ $GOPATH/bin/terraform-provider-azurerm
...
```

In order to test the provider, you can simply run `make test`.

```sh
$ make test
```

In order to run the full suite of Acceptance tests, run `make testacc`.

*Note:* Acceptance tests create real resources, and often cost money to run.

```sh
$ make testacc
```
