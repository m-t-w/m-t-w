---
layout: default
overview: true
permalink: /cloudinit-config/
---


# Provisioning EC2s with cloudinit_config user-data and Terraform

### What is cloud init? [See: the Cloud init docs](https://cloudinit.readthedocs.io/en/latest/topics/tutorial.html)


---

### For each file to be used for cloud_init, create a terraform templatefile
This is a terraform data source, called a [template_file](https://registry.terraform.io/providers/hashicorp/template/latest/docs/data-sources/file). This will allow us to pass in variables from terraform.
Note: the data-source declaration here does not have access to variables already defined in variables.tf (or elsewhere) - you have to pass them in. 

```
data "template_file" "install_programs" {
  template = file("${path.module}/scripts/install-programs.yaml")
  vars     = {
               cluster_stack_name   = var.cluster_stack_name,
               region               = data.aws_region.current.name,
               instance_name        = var.instance_name,
             }
}
```

### Each yaml file should conform to the cloud-config yaml syntax
User data can come is several formats - the format I'm using here is cloud-config syntax.  Here is the [module reference.](https://cloudinit.readthedocs.io/en/latest/topics/modules.html) 

#### Gotchas:
* The first line #cloud-config is needed to tell the cloud-init program that this is a cloud-config file.
 ` #cloud-config`
* Don't write files to /tmp from cloud-init use /run/somedir instead. (because of race conditions) 

#### You must include a merge-type header if you are using multi-part MIME
This is a way to specify how cloud-config YAML (user-data) are merged together when there are multiple YAML files. Previously the merging algorithm was very simple and would only overwrite and not append lists, or strings. There are a few different ways to do this, in the example below we're including merge-type header for every cloud-config YAML file.   [(see: merge-types docs)](https://cloudinit.readthedocs.io/en/latest/topics/merging.html)
```
#cloud-config
merge_how:
 - name: list
   settings: [append]
 - name: dict
   settings: [no_replace, recurse_list]
runcmd:
  - echo
  - echo "*** Schedule Docker Cleanup"
  - echo "--- Set up 4 am docker system prune job"
  - touch /var/log/docker-prune.out
  - chown ec2-user:ec2-user /var/log/docker-prune.out
  - echo
  ```
  
### Bring it all together in one cloudinit_config data source, for use with cloudinit
 [See: Terraform cloud_config data source](https://registry.terraform.io/providers/hashicorp/cloudinit/latest/docs/data-sources/cloudinit_config)
```
# Bringing it all together into a single, multi-part cloud-init configuration
data "cloudinit_config" "user_data" {

  part {
    content_type = "text/cloud-config"
    content      = data.template_file.install_programs.rendered
  }

  part {
    content_type = "text/cloud-config"
    content      = data.template_file.configure_docker.rendered
  }
}
```
  
### Troubleshooting
#### Where is the user data file stored on an E2?
`/var/lib/cloud/instances/[instance-id]/user-data.txt`

#### Where are the logs from the cloud_config cloud init?

`less /var/log/cloud-init.log`

`less /var/log/cloud-init-output.log`

#### Boot stages
There are five stages to boot:
1. Generator 
    - determines whether cloud-init should run during boot
3. Local
4. Network 
    - bootcmd 
    - [write-files](https://cloudinit.readthedocs.io/en/latest/topics/modules.html#write-files)
    - users-groups
    - ssh
5. Config 
    - [runcmd](https://cloudinit.readthedocs.io/en/latest/topics/modules.html#runcmd) (*NOTE:* the runcmd module only writes the script to be run later. The module that actually runs the script is *scripts-user* in the Final boot stage)
6. Final
    - Any scripts that a user is accustomed to running after logging into a system should run correctly here. 
    - [scripts-user](https://cloudinit.readthedocs.io/en/latest/topics/modules.html#scripts-user)
    - package installations
    - configuration management plugins (puppet, chef, salt-minion)
    - user-defined scripts (i.e. shell scripts passed as user-data) 

Info from [here](https://git.launchpad.net/cloud-init/tree/config/cloud.cfg.tmpl) and [here](https://stackoverflow.com/questions/34095839/cloud-init-what-is-the-execution-order-of-cloud-config-directives)


#### Helpful docs:
* [The Cloud init docs](https://cloudinit.readthedocs.io/en/latest/topics/tutorial.html)
* [Cloud init: cloud config examples](https://cloudinit.readthedocs.io/en/latest/topics/examples.html#yaml-examples)
* [Terraform docs re: the cloud_config data source](https://registry.terraform.io/providers/hashicorp/cloudinit/latest/docs/data-sources/cloudinit_config)
* [A good blog post re: cloud init and terraform](https://sammeechward.com/cloud-init-and-terraform-with-aws/)
* [Another blog post that was really useful in understanding this pattern](https://www.puppeteers.net/blog/multi-part-cloud-init-provisioning-with-terraform/)
