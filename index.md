# User data for ECs, in Terraform

### Where is the user data file stored on an E2?
`/var/lib/cloud/instances/[instance-id]/user-data.txt`

### Where are the logs from the cloud_config set up step?

`less /var/log/cloud-init.log`

`less /var/log/cloud-init-output.log`

### See:
* [cloud init docs](https://cloudinit.readthedocs.io/en/latest/topics/tutorial.html)

* [cloud config examples] (https://cloudinit.readthedocs.io/en/latest/topics/examples.html#yaml-examples)

* [terraform doc re: cloud config] (https://registry.terraform.io/providers/hashicorp/cloudinit/latest/docs/data-sources/cloudinit_config)
* [good blog post re: cloud init and terraform] (https://sammeechward.com/cloud-init-and-terraform-with-aws/)
* [another blog post that was really useful in understanding this pattern] (https://www.puppeteers.net/blog/multi-part-cloud-init-provisioning-with-terraform/)

### don't forget:
* The first line #cloud-config is needed to tell the cloud-init program that this is a cloud-config file.
 ` #cloud-config`
* Don't write files to /tmp from cloud-init use /run/somedir instead. (because of race conditions) 

### You must include a merge-type header if you are using multi-part MIME
This is a way to specify how cloud-config YAML (user-data) are merged together when there are multiple YAML file. Previously the merging algorithm was very simple and would only overwrite and not append lists, or strings. There are a few different ways to do this, in the example below we're including merge-type header for every cloud-config YAML file.   
```
#cloud-config
merge_how:
 - name: list
   settings: [append]
 - name: dict
   settings: [no_replace, recurse_list]
runcmd:
  - echo
  - echo "*** Install bamboo-agent jar" # Install jar
  - pushd /usr/local/bin
  - curl -LOs https://s3.amazonaws.com/donorschoose-docker-assets/atlassian-bamboo-agent-installer-${bamboo_version}.jar
  - popd 
  ```
  [merge types docs] (https://cloudinit.readthedocs.io/en/latest/topics/merging.html)
