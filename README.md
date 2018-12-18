## Prerequirements
Requires vagrant 2.1.2+ due to bug: https://github.com/hashicorp/vagrant/issues/10158 , I've used 2.1.5 from https://www.vagrantup.com/downloads.html (installed it on my ubuntu 18.04 from debian 64-bit package).
```
wget https://releases.hashicorp.com/vagrant/2.1.5/vagrant_2.1.5_x86_64.deb
dpkg -i vagrant_2.1.5_x86_64.deb
```

## Access
* in virtualbox, connect to port 3622 with ssh


## Running with LXC
1. Install dependencies (on ubuntu 18.04)
`sudo apt-get -y install build-essential git ruby lxc lxc-templates cgroup-lite redir`
2. install vagrant plugin
`vagrant plugin install vagrant-lxc`
3. up with lxc provider
`vagrant up --provider=lxc`


## Running with Azure cloud
Based on https://blog.scottlowe.org/2017/12/11/using-vagrant-with-azure/
1. Install azure cli https://docs.microsoft.com/en-us/cli/azure/install-azure-cli-apt?view=azure-cli-latest
2. Install vagrant plugin
`vagrant plugin install vagrant-azure`
3. auth with az tools: https://docs.microsoft.com/en-us/cli/azure/authenticate-azure-cli?view=azure-cli-latest
4. Create a dummy box
`vagrant box add azure-dummy https://github.com/azure/vagrant-azure/raw/v2.0/dummy.box --provider azure`
5. Create your service principal and save the output
`az ad sp create-for-rbac`
6. get your account information
`az account show`
7. Create your env file and fill it in a safe place, no spaces between `=` and other symbols, `source envfile` at the end.
```
AZURE_TENANT_ID="put value of tenant from az ad sp create-for-rbac output"
AZURE_CLIENT_ID="put value of appId  from az ad sp create-for-rbac output"
AZURE_CLIENT_SECRET="put value of password from az ad sp create-for-rbac output"
AZURE_SUBSCRIPTION_ID="put value of id from az account show output"

export AZURE_TENANT_ID AZURE_CLIENT_ID AZURE_CLIENT_SECRET AZURE_SUBSCRIPTION_ID
```
8. In any bash shell intended for running vagrant up or ssh or destroy commands source access variables with `source envfile`
9. Modify playbook.yml to have strong passwords for vagrant and dbaN accounts.
10. Modify `Vagrantfile:az.vm_name` if needed (and VM size, currently it's 1 core & 1GB)
11. *Uncomment* `require azure-plugin` at the beginning of Vagrantfile to avoid error:
`The resource group 'vagrant' is in deprovisioning state and cannot perform this operation.`
12. Start vagrant box
`vagrant up --provider=azure`
13. Wait until creation and ansible run will be finished
14. Connect with attender account: `ssh dbaN@tutorialengine.westus.cloudapp.azure.com` where N from 1 to 50
15. Do not forget to remove VM after work to avoid wasting money: `vagrant destroy -f`
16. Check at https://portal.azure.com, VM should be destroyed completely after 10-15 minutes (Delete procedure is really slow)

## Running with Amazon AWS cloud
https://github.com/mitchellh/vagrant-aws
1. Install provider
```
vagrant plugin install vagrant-aws
vagrant box add dummy https://github.com/mitchellh/vagrant-aws/raw/master/dummy.box
```
2. Update Vagrant file to provide your account details. If aws configure was executed on your machine it's enough to change just:
```
override.ssh.private_key_path = '~/.ssh/id_rsa-hadar'
aws.keypair_name = "nickolay.ihalainen"
```
3. Start the box `vagrant up --provider=aws`
4. ssh dba1@ip
5. Do not forget to shutdown with `vagrant destroy -f` to save money


## Troubleshooting
* More to follow
`vagrant ssh`


## Typical usage scenario
1. Start vagrant box: `vagrant up --provider=virtualbox`
2. Change something - then:
3. Update push changes to existing vagrant box: `vagrant provision`
4. When you finished for today stop box: `vagrant halt`
5. If you have created, but stopped box you can start it: `vagrant up`
6. Renaming task files:
  * login to box: `vagrant ssh`
  * remove existing task files in box: `rm -rf tutorial/*`
  * exit from vagrant ssh
  * push all tutorial files to the box: `vagrant provision`
7. During demonstration you should have pre-configured halted box, just start it: `vagrant up`. The box is already provisioned, thus it will not require internet connection to start.

## Extending
### Installing packages from apt
`playbook.yml` file contains declarations what packages and files should be in the box. In order to add new packages add following lines before `# keep allow password login at the end` comment:
```
    - name: install sysbench
      package:
        name: sysbench
        update_cache: no
        state: present
```
`update_cache: yes` instructs ansible to run `apt update` before installing package.

Be careful with spaces and tabs, yml format is like a python programming language.

