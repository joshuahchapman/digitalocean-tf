# Lesson 1

### Housekeeping

#### Create new SSH key for Digital Ocean droplets

1. In terminal run: `ssh-keygen -t rsa -b 4096 -C "your_email@example.com"`, replacing the exampleemail address a
    - Save the keys to your ssh directory, eg `/Users/johndoe/.ssh/do_rsa`


#### Fix environment variables

1. Open your bash profile: `~/.bash_profile`
2. At the bottom you should see your entries from the Digital Ocean tutorial:

```
export DO_PAT=1230jfoweaj02014812048120218240rfhiohf203r82483240328
export DO_SSH_FINGERPRINT=12:11:22:33:44:55:66:77:88:99:00:11:22:33:44:55
```

Change the environment variable names to match the variable names in the `.tf` files and prefix them with `TF_VAR_`
```
export TF_VAR_do_token=1230jfoweaj02014812048120218240rfhiohf203r82483240328
export TF_VAR_ssh_fingerprint=12:11:22:33:44:55:66:77:88:99:00:11:22:33:44:55
```

Below those lines add new environment variables for your public and private key locations:

```
export TF_VAR_pvt_key=/Users/johndoe/.ssh/do_rsa
export TF_VAR_pub_key=/Users/johndoe/.ssh/do_rsa.pub
```
Save and exit

3. In your terminal run `source ~/.bash_profile`, then make sure the variables have loaded by printing them, eg `echo $TF_VAR_pub_key`

4. Now in the `loadbalance` folder run `terraform init && terraform plan` to run them in succession
    - Now you should be able to run terraform commands without the extra `-var` flags


#### Organize variables

1. Create a new `variables.tf` file in the `loadbalance` folder
2. Move the variables defined in `provider.tf` to `variables.tf`

```
variable "do_token" {}
variable "pub_key" {}
variable "pvt_key" {}
variable "ssh_fingerprint" {}
```

3. Add descriptions to the variables to make it easier to remember what they do when you come back to this project at a later time

```
variable "pub_key" {
  description = "Public SSH key used for Digital Ocean droplet provisioning"
}
variable "pvt_key" {
  description = "Private SSH key used for Digital Ocean droplet provisioning"
}
variable "ssh_fingerprint" {
  description = "MD5 fingerprint for Public SSH key"
}
```

4. Create a file called `www_variables.tf` in the `loadbalance` folder
5. Create a variable for each parameter in the `digitalocean_droplet` resource definition, excluding the provisioner provisioner block
    - For the `name` variable, be sure to add interpolation create unique naming for the droplets
```
resource "digitalocean_droplet" "www-1" {
  name = "${var.droplet_name}-1"
  ...
  ...
  ...
}  
```  

6. Create a file called `loadbalancer_variables.tf` in the `loadbalance` folder
7. Create a variable for each parameter in the `digitalocean_loadbalancer` resource definition, reusing previously created `region` variable 
- For the `name` variable, be sure to add interpolation to map the droplet name to the load balancer to help organize future services 

```
resource "digitalocean_loadbalancer" "www-lb" {
  name = "${var.droplet_name}-lb"
  ...
  ...
  ...
}

```

#### Creating an output

1. Create a file called `outputs.tf` in the `loadbalance folder`
    - Create an output for the load balancer IP, this will make it easier to grab the IP address for testing and the IP might change between rebuilds 

```
output "www-lb-ip" {
    value = "${digitalocean_loadbalancer.www-lb.ip}"
}
```


* After you complete these steps, run a plan and apply to make sure everything is still working as expected. You should now see the load balancer's Public IP address:

```
Outputs:

www-lb-ip = 121.122.123.124
```


#### Wrap up 


* In this tutorial, you:
    - Updated the environment variables to make it easier to work with terraform commands
    - Organized the variables to make it easier to keep variables in sync across multiple resources (eg. `region`)
    - Created an output to retrieve the load balancer IP so that you can check the nginx functionality without having to lookup the IP in the DigitalOcean console

* Remember to run `terraform destroy` at the end to deprovision the resources 
5. Create a variable for each parameter in the `digitalocean_droplet` resource definition