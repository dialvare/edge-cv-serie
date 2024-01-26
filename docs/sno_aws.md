# Installing Single Node OpenShift on AWS
Looking for a simple and quick way to get into the world of Kubernetes and the cloud? If this is your case, you're in the right place!

Here begins a series of blogs that will guide you through the process from the cloud to the edge, to run computer vision models. In this blog, we will embark on a step-by-step exploration of the process for installing and configuring a Single Node OpenShift (SNO) environment on Amazon Web Services (AWS). Throughout this journey, we'll guide you through the essential elements you need to master for a successful installation. From spinning up an AWS instance to configuring your OpenShift environment, we'll cover every detail so you can deploy and run containerized applications efficiently.

## Create and configure an AWS account
For this blogpost we have choosen to use Amazon Web Service as the cloud provider, but remeber that OpenShift can be deployed in other Cloud vendors if you wish. For this blogpost, you will need to use an active AWS account or create a new one from the [Amazon Web Service home page](https://aws.amazon.com/).

### Creating and configuring the Route 53 service
As part of the account configuration and, in order to being able to deploy OpenShift, it is needed to set up a public hosted zone in the Route 53 service.  

## Creating the IAM user
When the AWS account was created, it was provisioned with a highly-privileged account. However, the creation of a specific IAM user for OpenShift on AWS is a recommended security practice to add an additional layer of security and facilitate the management and auditing of the accesses and actions performed by OpenShift on the AWS infrastructure. 



1. In the AWS Web Console navigate again to "**Services**".
2. Now click on "**Security, Identity & Compliance**" and select the "**IAM**" option.
3. On the left column in the *IAM Dashboard*, go to the "**Users**" page.
4. Click on "**Create user**", on the upper-right corner.
5. On the new page, type "**User name**": *`dialvare`* (introduce here the name of your new user). Then, click on "**Next**".
6. [X] Verify that the "**Add user to group**" box is selected.
6. Select "**Create group**" and follow this set up:
   * **User group name**: *`admin`*.
   * [X] Check the "**AdministratorAccess**" policy.
7. Click on "**Create user group**" again, and you will be redirected to the *User creation* form.
9. [X] Select the new "**admin**" group name.
8. Click "**Next**", review the summary and complete the user creation clicking on "**Create user**".
9. Back in the *Users* page, select your user. in my case "**dialvare**".
10. There, you will find some information about the user. Navigate to the "**Security credentials**" tab.
11. Scroll down to the *Access keys* section and select "**Create access key**".
12. [X] Check the "**Command Line Interface (CLI)**" option.
15. [X] Check the Confirmation box at the bottom.
12. Click "**Next**". And skip the description tag step. Now click "**Create access key**".
13. Please, copy the "**Access key**" and the "**Secret access key**". You may need it in the future.

## Create AWS instance
1. Navigate to the AWS Web Console and login using your credentials.
2. In the upper-left corner, click on "**Services**".
3. In the drop-down menu, click on "**Compute**" and then "**EC2**" on the right side, to create the virtual server.
4. On the *Resources* dashboard, press "**Launch instance**".
5. Complete the following fields:
   * **Name**: *`host`* (insert any preferred name).
   * **Amazon Machine Image (AMI)**: *`Amazon Linux 2023 AMI`*.
   * **Architecture**: *`64-bit (x86)`* (you can use Arm architecture if preferred).
   * **Instance type**: *`t3.micro`*.
   * **Key pair**: click on "**Create new key pair**" and configure it:
     * **Key pair name**: *`my-keys`* (type any preferred name).
     * **Key pair type**: *`RSA`*.
     * **Private key file format**: *`.pem`* (as we will be using ssh to connect).
   * Once completed, click "Create key pair". This will download the keys automatically.
   * On the *Networking settings* section, complete the following fields:
     * **VPC**: click on "**Create a new default VPC**". Then click on the Refresh arrow to autmatically detect your new VPC.
     * **Subnet**: click on the Refresh arrow and *`No Preference`* will be selected automatically.
     * **Auto-assign public IP**: *`Enable`*.
   * [X] For the Firewall set up , check the "**Create security group**" box.
   * [X] Check the "**Allow SSH trafic from**" -> "**Anywhere 0.0.0.0/0**".
   * **Configure storage**: *`1x8GiB gp3`* Root volume.
6. On the right side, now you can press the "**Launch instance**" button. Wait until the creation finished successfully.
7. The next step will be selecting "**Connect to your instance**". This will open a new tab. 
8. Navigate to the "**SSH client**" where the steps to connect using SSH are described. Copy the command displayed at the bottom.
9. Open a new terminal and give the keys file the right permissions. Then, paste the command. Remember to modify the path to your keys file:
    ```
    chmod 400 ~/Downloads/my-keys.pem
    ```
    ```
    ssh -i ~/Downloads/my-keys.pem ec2-user@ec2-51-20-54-172.eu-north-1.compute.amazonaws.com
    ```
That's it! We have just created and connected to our host machine. 

## Create AWS user


## Installing the OCP client and the installer
1. Navigate to the [Red Hat Hybrid Cloud Console](https://console.redhat.com/openshift) and log in using your Red Hat credentials.
2. On the left pannel, navigate to the "**Downloads**" page.
3. Locate the "**OpenShift command-line interface (oc)**". Select "**Linux**" as the *OS system* and your *Architecture type*.
4. Instead of clicking Download, **right-click** on the *Download* button and select **Copy Link Address**.
5. Back to the terminal, ensure you are connected to the AWS host machine and run this command. Remember to paste the Link Address copied before:
   ```
   wget https://mirror.openshift.com/pub/openshift-v4/x86_64/clients/ocp/stable/openshift-client-linux.tar.gz
   ```
7. Now, scroll down to the "**OpenShift for x86_64 Installer**". Again, select "**Linux**" as the *OS system*.
8. Instead of clicking Download, **right-click** on the Download button and select **Copy Link Address**.
9. In the Terminal window, run the following command:
   ```
   wget https://mirror.openshift.com/pub/openshift-v4/x86_64/clients/ocp/stable/openshift-install-linux.tar.gz
   ```    
11. Once the downloads finish, unzip both files: 
   ```
   tar -xvf openshift-client-linux.tar.gz
   tar -xvf openshift-install-linux.tar.gz
   ``` 
13. To complete the oc installation move the extracted files to the user path:
    ``` 
    sudo mv oc kubectl /usr/local/bin
    ``` 
15. Before starting the OCP installation, move the installation file to the user path too:
    ``` 
    sudo mv openshift-install /usr/local/bin
    ```
16. Check the version you will be installing. Run: 
    ``` 
    openshift-install version
    ```

## SSH key pair creation
1. In your terminal run the following command to create the SSH keys:
   ```
   ssh-keygen -t ed25519 -N '' -f ${HOME}/.ssh/ocp4-aws-key
   ```
3. Now, check and copy your new public key:
   ```
   cat ${HOME}/.ssh/ocp4-aws-key.pub
   ```
5. Add the SSH private key identity to the SSH agent for your local user by running:
   ```
   ssh-add ${HOME}/.ssh/ocp4-aws-key
   ```

## Creating a hosted zone

## OCP installation
1. On the terminal, you will need to create a config file to specify the cluster details. Run:
   ```
   openshift-install create install-config --dir=./
   ```   
2. Use the arrow keys in your keyboard and select the following configuration:
   * **SSH Public Key**: *`/home/ec2-user/.ssh/ocp4-aws-key.pub`*.
   * **Platform**: *`aws`*.
   * **AWS Access Key ID**: Paste the one copied when you created your user.
   * **AWS Secret Access Key ID**: Paste the one copied when you created your user.
   * **Region**: Select the region where the host was created (*`eu-north-1`* in my case).
   * **BaseDomain**: Select your domain (*`sandbox517.opentlc.com`* in my case).
   * **Cluster name**: Type your preferred name for the cluster. I will choose *`sno`*.
   * **Pull Secret**: Copy your pull secret from the [Hybrid cloud Console](https://console.redhat.com/openshift/downloads#tool-pull-secret).
4. Now you can take a look to the newly created config file:
   ```
   vi install-config.yaml 
   ```
6. To deploy a Single Node OpenShift change the *`worker`* nodes to *`0`* and the *`master`* nodes to *`1`*:
   ```
   compute:
   - architecture: amd64
     hyperthreading: Enabled
     name: worker
     platform: {}
     replicas: 0
   controlPlane:
     architecture: amd64
     hyperthreading: Enabled
     name: master
     platform: {}
     replicas: 1
   ```
8. Finally, run the installation command. The installer will use the configuration file we just modified:
   ```
   openshift-install create cluster --dir=./ --log-level=debug
   ```
10. Note down the Kubeadmin and Password.
