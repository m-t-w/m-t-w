# User data for ECs, in Terraform

Where is the user data fie stored on an E2?
*/var/lib/cloud/instances/[instance-id]/user-data.txt*

Where are the logs from the cloud_config set up step?
`less /var/log/cloud-init.log`
`less /var/log/cloud-init-output.log`

see [cloud init docs](https://cloudinit.readthedocs.io/en/latest/topics/tutorial.html)
[cloud config examples] (https://cloudinit.readthedocs.io/en/latest/topics/examples.html#yaml-examples)
[terraform doc re: cloud config] (https://registry.terraform.io/providers/hashicorp/cloudinit/latest/docs/data-sources/cloudinit_config)
[good blog post re: cloud init and terraform] (https://sammeechward.com/cloud-init-and-terraform-with-aws/)
[another blog post that was really useful in understanding this pattern] (https://www.puppeteers.net/blog/multi-part-cloud-init-provisioning-with-terraform/)

### don't forget:
` #cloud-config`
* The first line #cloud-config is needed to tell the cloud-init program that this is a cloud-config file.
* Don't write files to /tmp from cloud-init use /run/somedir instead. (because of race conditions) 
