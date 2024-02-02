# Installing Single Node OpenShift on AWS
Looking for a simple and quick way to get into the world of Kubernetes and the cloud? If this is your case, you're in the right place!

In this blog, we will embark on a step-by-step exploration of the process for installing and configuring a Single Node OpenShift (SNO) environment on Amazon Web Services (AWS). Throughout this journey, we'll guide you through the essential elements you need to master for a successful installation. From spinning up an AWS instance to configuring your OpenShift environment, we'll cover every detail so you can deploy and run containerized applications efficiently. The steps outlined in the blog are derived from the official [OpenShift 4.14 documentation](https://access.redhat.com/documentation/en-us/openshift_container_platform/4.14/html/installing/installing-on-aws), which can be referenced for further details and additional configurations.

## Perks of Single Node Openshift on the cloud
In the past decade, we have witnessed exponential growth in the interest and adoption of emerging technologies such as artificial intelligence (AI), edge computing, and the cloud. The convergence of these trends is shaping a dynamic and transformative technological landscape. AI propels innovation in fields like machine learning and intelligent automation, while edge computing brings processing capabilities closer to where data is generated. Simultaneously, the cloud remains the epicenter of the digital revolution, offering unparalleled scalability and flexibility. 

Single Node OpenShift emerges as an ideal solution for developers and testing teams seeking agility in development environments. As the name suggests, this configuration allows running a single instance of OpenShift on a lone node offering control and worker capabilities at the same time. With a smaller footprint, Single Node OpenShift provides a consistent and comprehensive environment for creating, testing, and deploying container-based applications without compromising the power of OpenShift. These are the [minimum system requirements](https://access.redhat.com/documentation/en-us/openshift_container_platform/4.14/html/installing/installing-on-a-single-node#additional-requirements-for-installing-sno-on-a-cloud-provider_install-sno-installing-sno-with-the-assisted-installer) for deploying SNO on the cloud: `8 vCPU cores`, `16GB of RAM` and `120GB of storage`.

## Choosing and creating your cloud account
Having seen some basic concepts about the technologies involved in this demo, it is time to start our hands-on deployment process! In this blogpost we have decided to use Amazon Web Service as the cloud provider, but remember that OpenShift can be deployed in other Cloud vendors if you wish. Here is a table listing the SNO [supported cloud providers](https://access.redhat.com/documentation/en-us/openshift_container_platform/4.14/html/installing/installing-on-a-single-node#supported-cloud-providers-for-single-node-openshift_install-sno-installing-sno-with-the-assisted-installer) and their CPU architectures: 

| Cloud provider              | CPU architecture |
|-----------------------------|------------------|
| Amazon Web Service (AWS)    | x86_64 & aarch64 |
| Google Cloud Platform (GCP) | x86_64 & aarch64 |
| Microsoft Azure             | x86_64           |

Depending on the cloud provider you choose, the steps described may vary slightly, so if you decide not to use AWS, we strongly encourage you to use this guide as a reference and supplement it with the official documentation. 

That being said, let's move on and create a new AWS account from the [Amazon Web Service home page](https://aws.amazon.com/) in case you don't have an active one yet.

## Creating and configuring the Route 53 service
Amazon Route 53 is a Domain Name System (DNS) service provided by Amazon Web Services (AWS). Its primary function is to enable the resolution of domain names to IP addresses and vice versa, thereby facilitating the management of network infrastructure. The need to utilize Route 53 when installing OpenShift on AWS stems from the critical importance of efficiently managing domain names and DNS resolution to ensure accessibility and connectivity for OpenShift components.

In the AWS context, Route 53 uses "hosted zones" as containers for DNS records of a specific domain. Each hosted zone in Route 53 stores information about how a particular domain should be resolved. To create a hosted zone you must first have a **domain name**, that can be purchased directly through AWS, as shown below. If you need more details, refer to [this](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/domain-register.html#domain-register-procedure-section) page:

1. Sign in to the [AWS Management Console](https://console.aws.amazon.com/).
2. Navigate to "**Services**", in the upper-left corner.
3. In the drop-down menu, select "**Networking & Content Delivery**" and click on "**Route 53**".
4. Now, go to "**Registered domains**" on the left side of the Dashboard. There you should see a `No domains to display` message.
5. Press the "**Register domains**" button, in the upper-right corner and type your preferred domain name (g.e. `snodemo.com`) and click "**Search**".
6. At this point, you should see your domain name ready to purchase, if it is available; or a list with suggested domains, in case it was not. "**Select**" the one you want to purchase and "**Proceed to checkout**".
7. On the following page you will need to complete with your information some parameters like the ones listed below. Navigate throughout the different sections by clicking "**Next**".
   * "**Domain pricing options**": *domain name*, *duration*, *auto-renew*.
   * "**Contact information**": *organization*, *name*, *email*, etc.
8. Before clicking "**Submit**" you can see this message: `Route 53 automatically creates a hosted zone for each new domain you register`. This means that once purchased the new domain, we don't need to worry about creating the hosted zone, as AWS will set it up automatically.
9. You will receive an email when your domain gets approved.
10. Finally, navigate again to "**Services**" > "**Networking & Content Delivery**" > "**Route 53**" to verify that the hosted zone is present.

And that's all you need to configure your Route 53 service correctly! Now you can continue with the next step. 

## Creating the IAM user
When the AWS account was created, it was provisioned with a highly-privileged account. However, the creation of a specific IAM user for OpenShift on AWS is a recommended practice to add an additional layer of security and facilitate the management and auditing of the accesses and actions performed by OpenShift on the AWS infrastructure. The official AWS documentation for this purpose can be found [here](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_users_create.html#id_users_create_console). Follow the next procedure to create your user:

1. In the [AWS Web Console](https://console.aws.amazon.com/), navigate again to "**Services**", in the upper-left corner.
2. Now click on "**Security, Identity & Compliance**" and select the "**IAM**" option.
3. On the left column in the *IAM Dashboard*, go to the "**Users**" page.
4. Click on "**Create user**", on the upper-right corner.
5. On the new page, type "**User name**": *`dialvare`* (introduce here the name of your new user). Then, click on "**Next**".
6. Verify that the "**Add user to group**" box is selected. We need to give it some privileges.
7. Select "**Create group**" and follow this set up:
   * "**User group name**": *`admin`*.
   * Check the "**AdministratorAccess**" policy.
8. Click on "**Create user group**" again, and you will be redirected to the *User creation* form.
9. Select the new "**admin**" group name.
10. Click "**Next**", review your choices and complete the user creation clicking on "**Create user**".
11. Back in the *Users* page, select your new user. In my case "**dialvare**".
12. There, you will find some information about the user. Navigate to the "**Security credentials**" tab.
13. Scroll down to the *Access keys* section and select "**Create access key**".
14. Check the "**Command Line Interface (CLI)**" option.
15. Check the *Confirmation* box at the bottom and click "**Next**".
16. Now you can skip the description tag step and click "**Create access key**".
17. Please, copy the "**Access key**" and the "**Secret access key**". You will use them in the future to fire up the SNO installation.
18. To close this window, click on "**Done**".

With that, we have completed most of the prerequisites before starting with the SNO installation. We have ensured that our network connections are configured thanks to the hosted zone and the Route 53 service and we have created our OCP user with admin privileges. Now it's time to create the machine from which we will launch the installation.

## Create AWS instance
It is possible to use our personal computer as bastion node, but to make sure that everyone following this blogpost can do it from start to finish, we are going to create an AWS instance to avoid possible hardware limitations in some users. Here are a few simple steps:

1. Navigate to the [AWS Console](https://console.aws.amazon.com/).
2. In the upper-left corner, click again on "**Services**".
3. In the drop-down menu, click on "**Compute**" and then "**EC2**" on the right side, to create the virtual server.
4. On the *Resources* dashboard, press "**Launch instance**".
5. Complete the following fields:
   * "**Name**": *`host`* (insert any preferred name).
   * "**Amazon Machine Image (AMI)**": *`Amazon Linux 2023 AMI`*.
   * "**Architecture**": *`64-bit (x86)`* (you can use Arm architecture if preferred).
   * "**Instance type**": *`c5.2xlarge`*. This instance matches the minimum requirements: `8vCPU` and `16GB RAM`.
   * "**Key pair**": will be used to connect to the machine. Click on "**Create new key pair**" and configure it:
     * "**Key pair name**": *`my-keys`* (type any preferred name).
     * "**Key pair type**": *`RSA`*.
     * "**Private key file format**": *`.pem`* (as we will be using ssh to connect).
   * Once completed, click "**Create key pair**". The download process will start automatically.
   * On the *Networking settings* section, click on "**Edit**" and complete the following fields:
     * "**VPC**": Go to the [VPC dashboard](https://console.aws.amazon.com/vpcconsole/home?#CreateDefaultVpc:) and click on "**Create a new default VPC**". Back to the *Networking* page, click the *Refresh arrow* to automatically detect your new VPC.
     * "**Subnet**": click on the *Refresh arrow* and *`No Preference`* will be selected automatically.
     * "**Auto-assign public IP**": *`Enable`*.
     * For the Firewall set up, check the "**Create security group**" box.
   * "**Configure storage**": *`1x8GiB gp3`* Root volume.
6. On the right side, now you can press the "**Launch instance**" button. Wait until the creation finishes successfully.
7. The next step will be selecting "**Connect to your instance**". This will open a new tab. 
8. Navigate to the "**SSH client**" tab where the steps to connect using SSH are described. Copy the command displayed at the bottom.
9. Open a new terminal and give the keys file the right permissions. Then, paste the command. Remember to modify the path to your keys file:
    ```
    chmod 400 ~/Downloads/my-keys.pem
    ```
    ```
    ssh -i ~/Downloads/my-keys.pem ec2-user@ec2-16-171-254-104.eu-north-1.compute.amazonaws.com
    ```
That's it! We have just created and connected to our host machine. From here on, the following steps will be done from this AWS machine as we will use it to launch the installation.

## SSH key pair creation
During the Single Node installation, we will need to supply an SSH public key to the installation program. This key is going to be transmitted to the node and will servie as a means of authenticating SSH access to it. Subsequently, the key is appended to the `~/.ssh/authorized_keys` list for the core user in the node, enabling password-less authentication. Here you can find a more detailed [documentation](https://access.redhat.com/documentation/en-us/openshift_container_platform/4.14/html/installing/installing-on-aws#ssh-agent-using_installing-aws-default) if you need it. Follow the next steps to complete the set up:

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

With this, the ssh keys have been generated and we can use them during the SNO installation!

## Installing the OCP client and getting the installation program
We are almost ready to go! It's time to install the oc client and download the installation program in our AWS instance. The steps are desbribed below: 

1. Navigate to the [Red Hat Hybrid Cloud Console](https://console.redhat.com/openshift) and log in using your Red Hat credentials.
2. On the left pannel, navigate to the "**Downloads**" page.
3. Locate the "**OpenShift command-line interface (oc)**". Select "**Linux**" as the *OS system* and your *Architecture type*.
4. Instead of clicking Download, **right-click** on the *Download* button and select **Copy Link Address**.
5. Back to the terminal, ensure you are connected to the AWS host machine and run the following command. Remember to paste the Link Address copied before:
   ```
   wget https://mirror.openshift.com/pub/openshift-v4/x86_64/clients/ocp/stable/openshift-client-linux.tar.gz
   ```
6. Back to the Hysbrid Cloud Console, scroll down until you spot "**OpenShift for x86_64 Installer**". Again, select "**Linux**" as the *OS system*.
7. Instead of clicking Download, **right-click** on the *Download* button and select **Copy Link Address**.
8. In the Terminal window, run the following command:
   ```
   wget https://mirror.openshift.com/pub/openshift-v4/x86_64/clients/ocp/stable/openshift-install-linux.tar.gz
   ```    
9. Once both downloads finish, unzip the files: 
   ```
   tar -xvf openshift-client-linux.tar.gz
   tar -xvf openshift-install-linux.tar.gz
   ``` 
10. To complete the oc installation move the extracted files to the user path:
    ``` 
    sudo mv oc kubectl /usr/local/bin
    ``` 
11. Before starting the OCP installation, move the installation file to the user path too:
    ``` 
    sudo mv openshift-install /usr/local/bin
    ```
12. Check the version you will be installing. Run: 
    ``` 
    openshift-install version
    ```
    
All good? Well, now we can affirm that we are ready to deploy the Single Node OpenShift!

## Single Node deployment
The moment has arrived, in a matter of minutes we will have our SNO deployed and ready to work. Without further delay, let's go through these steps:

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
   * **BaseDomain**: Select your domain (*`snodemo.com`* in my case).
   * **Cluster name**: Type your preferred name for the cluster. I will choose *`sno`*.
   * **Pull Secret**: Copy and paste your pull secret from the [Hybrid cloud Console](https://console.redhat.com/openshift/downloads#tool-pull-secret).
3. Now you can take a look to the newly created config file:
   ```
   vi install-config.yaml 
   ```
4. To deploy a Single Node OpenShift the `controlPlane.replicas` setting in the `install-config.yaml` file should be set to `1` and the `compute.replicas` setting should be `0`. This makes the control plane node schedulable.
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
5. Finally, run the installation command. The installer will use the configuration file we just modified:
   ```
   openshift-install create cluster --dir=./ --log-level=debug
   ```
6. When the installation finishes, the installer will provide you the `kubeadmin` user and the `password` along with your OpenShift Web Console URL. Note them down.
7. In order to be able to acesss your SNO, run the following command to expose the kubeconfig file:
   ```
   export KUBECONFIG=/home/ec2-user/auth/kubeconfig
   ```
8. It's time to test our SNO. Run this command:
   ```
   oc get nodes
   ```
If everything has been configured in a proper way, you should see just one node with master and worker capabilities!

## Next stage
In this blog, we have explored the detailed process of successfully installing Red Hat Single Node OpenShift on Amazon Web Services (AWS). This initial step lays the foundation for creating and managing containers in a fully functional environment. Now, the next exciting step is configuring OpenShift for use in the realm of artificial intelligence. The convergence of OpenShift and artificial intelligence opens up a world of possibilities for the development, deployment, and management of machine learning-driven applications. So, get ready to explore the exciting terrain where container technology meets artificial intelligence in our next blog!
