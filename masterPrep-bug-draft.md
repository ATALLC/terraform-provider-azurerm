The openshift-origin example is calling the openshift masterPrep.sh script with two arguments, the persistent volume storage account name and the admin username, but the current version of that script in https://github.com/Microsoft/openshift-origin takes three arguments, the storage account, the storage account location, and the sudo user. Passing two arguments causes the script to silently fail to create the scgeneric1.yml, which causes failures during the ansible install much later on.

### Terraform Version
Terraform v0.11.2
+ provider.azurerm v1.0.1

### Terraform Configuration Files
```hcl
  provisioner "remote-exec" {
    inline = [
      "chmod +x masterPrep.sh",
      "chmod +x deployOpenShift.sh",
      "sudo bash masterPrep.sh \"${azurerm_storage_account.persistent_volume_storage_account.name}\" \"${var.admin_username}\" && sudo bash deployOpenShift.sh \"${var.admin_username}\" \"${var.openshift_password}\" \"${var.key_vault_secret}\" \"${var.openshift_cluster_prefix}-master\" \"${azurerm_public_ip.openshift_master_pip.fqdn}\" \"${azurerm_public_ip.openshift_master_pip.ip_address}\" \"${var.openshift_cluster_prefix}-infra\" \"${var.openshift_cluster_prefix}-node\" \"${var.node_instance_count}\" \"${var.infra_instance_count}\" \"${var.master_instance_count}\" \"${var.default_sub_domain_type}\" \"${azurerm_storage_account.registry_storage_account.name}\" \"${azurerm_storage_account.registry_storage_account.primary_access_key}\" \"${var.tenant_id}\" \"${var.subscription_id}\" \"${var.aad_client_id}\" \"${var.aad_client_secret}\" \"${azurerm_resource_group.rg.name}\" \"${azurerm_resource_group.rg.location}\" \"${var.key_vault_name}\""
    ]
  }

```

### Expected Behavior
Openshift installed successfully.

### Actual Behavior
What actually happened?
```
azurerm_virtual_machine.master (remote-exec): TASK [Create Storage Class with StorageAccountPV1] *****************************
azurerm_virtual_machine.master (remote-exec): fatal: [sosoa-master-0]: FAILED! => {"changed": true, "cmd": "oc create -f /home/ocpadmin/scgeneric1.yml", "delta": "0:00:00.283921", "end": "2018-01-15 20:24:25.930970", "msg": "non-zero return code", "rc": 1, "start": "2018-01-15 20:24:25.647049", "stderr": "error: the path \"/home/ocpadmin/scgeneric1.yml\" does not exist", "stderr_lines": ["error: the path \"/home/ocpadmin/scgeneric1.yml\" does not exist"], "stdout": "", "stdout_lines": []}
azurerm_virtual_machine.master (remote-exec): 	to retry, use: --limit @/home/ocpadmin/configurestorageclass.retry
```

### Steps to Reproduce
1. `cd terraform-provider-azurerm/examples/openshift-origin/ && terraform apply`
