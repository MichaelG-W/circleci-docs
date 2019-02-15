---
layout: classic-docs
title: "CircleCI Trial Installation"
category: [administration]
order: 3
description: "How to install CircleCI on a single VM"
---

CircleCI is a scalable CI/CD platform that supports clusters
of tens or hundreds of build machines. This document provides instructions for installing and running the platform on a single virtual machine as a way to start a small trial in any environment:

* TOC
{:toc}

## Prerequisites

To enusre a successful trial, do the following before starting the installation:

- [Sign-up](https://circleci.com/enterprise-trial-install/) to receive a trial license file.
- Use **GitHub.com or GitHub Enterprise** for version control.
- Make certain that the machines running CircleCI and GitHub are able to reach each other on the network.
- Verify that the machine running CircleCI has outbound internet access. If you are using a proxy server, [contact support](https://support.circleci.com/hc/en-us/requests/new) for instructions.
- Make certain that you have access to an Amazon Web Services (AWS) account. If you don't, you can sign up for an account during the procedure to configure an Amazon Machine Image.

## Installing on AWS EC2 

Follow the steps in the following procedure to install CircleCI on a single EC2 VM by using a pre-made Amazon Machine Image (AMI). An AMI is a special type of virtual appliance that is used to create a virtual machine within the Amazon Elastic Compute Cloud ("EC2").

**Note:** Be careful with the privileges that you assign to your instance because all builds that run on the installed machine have access
to the AWS Identity and Access Management (IAM) privileges associated with its instance profile. In a production environment, you can use `iptables` rules to block
this access. [Contact support](https://support.circleci.com/hc/en-us) for specific instructions.

### Configure the Amazon Machine Image:

<script>
  var amiIds = {
  "ap-northeast-1": "ami-32e6d455",
  "ap-northeast-2": "ami-2cef3242",
  "ap-southeast-1": "ami-7f22a71c",
  "ap-southeast-2": "ami-21111b42",
  "eu-central-1": "ami-7a2ef015",
  "eu-west-1": "ami-ac1a14ca",
  "sa-east-1": "ami-70026d1c",
  "us-east-1": "ami-cb6f1add",
  "us-east-2": "ami-57c7e032",
  "us-west-1": "ami-059b818564104e5c6",
  "us-west-2": "ami-c24a2fa2"
  };

  var amiUpdateSelect = function() {
    var s = document.getElementById("ami-select");
    var region = s.options[s.selectedIndex].value;
    document.getElementById("ami-go").href = "https://console.aws.amazon.com/ec2/v2/home?region=" + region + "#LaunchInstanceWizard:ami=" + amiIds[region];
  };
  </script>

  <select id="ami-select" onchange="amiUpdateSelect()">
  <option value="ap-northeast-1">ap-northeast-1</option>
  <option value="ap-northeast-2">ap-northeast-2</option>
  <option value="ap-southeast-1">ap-southeast-1</option>
  <option value="ap-southeast-2">ap-southeast-2</option>
  <option value="eu-central-1">eu-central-1</option>
  <option value="eu-west-1">eu-west-1</option>
  <option value="sa-east-1">sa-east-1</option>
  <option value="us-east-1" selected="selected">us-east-1</option>
  <option value="us-east-2">us-east-2</option>
  <option value="us-west-1">us-west-1</option>
  <option value="us-west-2">us-west-2</option>
  </select>
  <a id="ami-go" href="" class="btn btn-success" data-analytics-action="{{ site.analytics.events.go_button_clicked }}" target="_blank">Go!</a>
<script>amiUpdateSelect();</script>


1. Select your region from the scroll box and click Go to go to AWS. 
2. When prompted, enter your AWS credentials and click Sign In.
2. In AWS, scroll to select an instance type.
- Choose an instance type with at least 32G of RAM, such as `m4.2xlarge`. 
- Click Next: Configure Instance Details to configure the instance. 
3. On the Configuring Instance Details page, do the following: 
- For Auto-assign Public IP, select Enable.
- For all other settings, accept the defaults.
- Click Next: Add Storage.
![AWS Step 3]({{site.baseurl}}/assets/img/docs/single-box-step3.png)
4. On the Add Storage page, accecpt the default of 100GB of storage, which is enough for this trial, by clicking Next: Add Tages.
5. Skip the Add Tags page by clicking Next: Configure Security Group.
6. On the Configure Security Group page, click Add Rule and use the scroll box to select a protocol. Add rules and set the ports as follows:
- SSH port 22
- HTTP port 80
- HTTPS port 443
- Custom TCP 8800
- (Optional) To enable developers to SSH into builds for debugging purposes, open ports 64535-65535 for Custom TCP.
![AWS Step 5]({{site.baseurl}}/assets/img/docs/single-box-step5.png)
6. Click Review and Launch.
7. On the Review Instance Launch page, confirm details that you have specified for the instance.
- Click Previous to go back to any previous configuration page to make changes or correct errors, if any have been flagged.
-  When you are satisfied with the details, click Launch.
8. (Optionally) Assign a key pair to the instance by selecting one of the following from the scroll box (note that a key pair is required for a Windows AMI but is optional for Linux; it allows secure SSH access):
- Choosing an existing key pair.
- Create a new key pair.
- Proceed without a key pair. 
9. Click Launch Instance.

The AWS console shows details about your instance, similar to the following.
![AWS Console]({{site.baseurl}}/assets/img/docs/aws-console.png)

### Configure CircleCI

1. In a browser type the public or private IP address, or the hostname, for the AWS VM and click return.
2. On the CircleCI welcome page, click Get Started to begin the guided installation process for CircleCI.
1. Choose an SSL certificate option. By default, all machines in a CircleCI installation verify SSL certificates for the GitHub Enterprise instance. 
- Note: If you are using a self-signed cert, or using a custom CA root, please see the [certificates]({{site.baseurl}}/2.0/certificates/) document for a script to add the information to the CircleCI truststore.
2. Upload the CircleCI license file and set the admin password.
3. If you do not need 1.0 build functionality, leave the box for it unchecked. Most users should check the box for 2.0 functionality.
4. Select "Single Box" in the "Builders Configuration" section(s).
5. Register CircleCI as a new OAuth application in GitHub.com at [https://github.com/settings/applications/new/](https://github.com/settings/applications/new/) or in the GitHub Enterprise Settings using the IP address of the AWS instance from Step 6 for the Homepage URL and using `http(s)://AWS instance IP address/auth/github` as the Authorization callback URL. Click the Register Application button.
- **Note:** If you get an "Unknown error authenticating via GitHub. Try again, or contact us." message, try using `http:` instead of `https:` for the Homepage URL and callback URL.
6. Copy the Client ID from GitHub and paste it into the entry field for GitHub Application Client ID.
7. Copy the Secret from GitHub and paste it into the entry field for GitHub Application Client Secret and click Test Authentication.
8. Ensure that "None" is selected in the "Storage" section. In production installations, other object stores may be used but will require corresponding IAM permissions.
9. Ensure that the "VM Provider" is set to "None". If you would like to allow CircleCI to dynamically provision VMs (e.g. to support doing Docker builds) you may change this setting, but it will require additional IAM permissions. [Contact us](https://support.circleci.com/hc/en-us) if you have questions.
10. Agree to the license agreement and save. The application start up process begins by downloading the ~160 MB Docker image, so it may take some time to complete. 
11. Open the CircleCI app and click Get Started to authorize your GitHub account. The Add Projects page appears where you can select a project for your first build. 


<!---
## Installation in a Data Center

1. Launch a VM with at least 8GB of RAM, 100GB of disk space on the root volume, and a version of Linux that supports Docker, for example Ubuntu Trusty 14.04. 

2. Open ports 22 and 8800 to administrators, open ports 80 and 443 to all users, and optionally open ports 64535-65535 to developers to SSH into builds.

3. Install Replicated, the tool used to package and distribute CircleCI, by running the  `curl https://get.replicated.com/docker | sudo bash` command. **Note:** Docker must not use the device mapper storage driver. Check this by running `sudo docker info | grep "Storage Driver"`.)

4. Visit port 8800 on the machine in a web browser to complete the guided installation process.

5. Complete the process by choosing an SSL certificate option, uploading the license, setting the admin password and hostnames,  enabling GitHub OAuth registration, and defining protocol settings. The application start up process begins by downloading the ~160 MB docker image, so it may take some time to complete. 

6. Open the CircleCI app and click Get Started to authorize your GitHub account. The Add Projects page appears where you can select a project for your first build. 
-->







