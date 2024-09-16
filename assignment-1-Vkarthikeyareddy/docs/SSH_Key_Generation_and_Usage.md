# SSH Key Generation and Usage

Created by: Zara Ahmad-Post
Created time: March 9, 2024 9:51 PM
Tags: Engineering, How To

# An Overview of SSH Key Pairs

## What is SSH?

SSH stands for Secure Shell. SSH is a cryptographic network protocol that encrypts all data transmitted between two machines, often a client and a server. The data that SSH encrypts includes login credentials, commands, and data exchanged during the session.

SSH is commonly used for remote administration of servers, file transfers, and for tunneling other network protocols securely over and insecure network. Developers and sysadmins often use SSH to securely access remote servers. 

## How does SSH work?

1. Initialize the connection: a client initiates an SSH connection by sending a request to a server. Clients can be laptops, or any user computer, or another device trying to access a server remotely.
2. Negotiating Encryption Algortihms: the server receives the connection request, and then both the client and server negotiate the encryption algorithms and cryptographic keys used for securing the connection. 
3. Authentication: After the encryption parameters are agreed upon, the client authenticates itself to the server. 
4. Establishing the Encrypted Channel: once the client is authenticated, SSH establishes an encrypted communication channel between the client and server. This channel makes all communication and data exchange secure from eavesdropping or tampering.
5. Executing Commands: with an encrypted channel established, commands are executed on the server and the results are sent back to the client over the encrypted channel.
6. Closing the connection: once the client has finished executing commands or performing other tasks, it closes the SSH connection. This will terminate the encrypted communication channel between the client and server.

## Cryptographic Techniques:

How does this secure connection get established? SSH employs various cryptographic techniques, including symmetric encryption for encrypting data transmitted over the connection, asymmetric encryption for securely exchanging encryption keys and authenticating parties, and cryptographich hash functions for ensuring data integrity. These techniques collectively provide a secure means of remote access and management over untrusted networks. 

## Why use SSH instead of Usernames + Passwords?

### Protection against password sniffing

- Without SSH, when you login to a server using a username and password, those credentials may be sent over the network in plain text. This makes them vulnerable to interception by attackers using packet sniffing tools. SSH encrypts this information, making it much harder for attackers to obtain your credentials.

### Protection against man-in-the-middle attacks

- SSH uses cryptographic keys for authentication. This helps prevent man-in-the-middle attacks, where an attacker intercepts and alters the communication between the client and server

### Public-key authentication:

- SSH supports public-key authentication, where the client generates a key pair (public and private key), keeps the private key secure on their local machine, and uploads the public key to the server. This eliminates the need to transmit passwords over the network.

# How to generate SSH keys

## SSH Clients

- An SSH client is required to generate and use SSH key pairs. MacOS’s terminal is an SSH client, as is Window’s Powershell.

### MacOS, Linux:

1. Open a terminal or command prompt on your local machine.
2. Use the ‘ssh-keygen’ command with appropriate options to generate the key pair:

```bash
ssh-keygen -t ed25519 -C "your_emaiil@example.com"
```

1. You will be prompted to choose the location to save the key pair. Press Enter to accept the default location (usually ‘~/.ssh/id_rsa’ for the private key and ‘~/.ssh/id_rsa.pub for the public key) or specify a different location if needed.
2. You may also be prompted to enter a passphrase to encrypt the private key. This passphrase adds an extra layer of security, but it’s optional. If you choose to use a passphrase, you’ll need to enter it every time you use the private key. 

### Windows:

On Windows 10 and later, you can generate an SSH key pair using PowerShell and a client called “OpenSSH”:

1. Open PowerShell by searching for PowerShell in the Start menu and selecting it. Then use the following command to generate an SSH key pair:

```bash
ssh-keygen -t ed25519
```

1. You will be prompted to choose the location to save the key pair. Press Enter to accept the default location (`C:\Users\YourUsername\.ssh\id_rsa` for the private key and `C:\Users\YourUsername\.ssh\id_rsa.pub` for the public key) or specify a different location if needed.
- Once the key pair is generated, you’ll see output indicating that the keys were successfully created.
- Your public key will be stored in a file with a `.pub` extension. This is the key you’ll share with servers or services where you want to authenticate using SSH public-key authentication.
- Your private key will be stored in a file without an extention (e.g., `id_rsa`). This is the key you’ll use when you’re authenticating to the server. Keep this file secure and do not share it with anyone.

## Different encryption algorithms:

1. RSA:
    1. The RSA (Rivest-Shamir-Adleman) algorithm is one of the oldest and most widely used asymmetric encryption algorithms.
    2. It relies on the mathematical properties of large prime numbers for its security.
    3. RSA keys are typically generated in various lengths, commonly 2048 or 4096 bits.
    4. While RSA is widely supported and considered secure when used with sufficiently long key lengths, it is slower compared to some newer algorithms
    
    generate an RSA key pair using key-gen:
    
    ```bash
    ssh-keygen -t rsa -b 2048
    ```
    
2. Ed25519
    1. Ed25519 is a newer elliptic curve cryptography (ECC) algorithm.
    2. It is based on the Twisted Edwards curve `edwards25519`
    3. Ed25519 keys are smaller in size compared to RSA keys, typically around 256 bits.
    4. It offers high security with faster key generation and authentication compared to RSA
    5. Ed25519 is becoming increasingly popular due to its security and efficiency. 
    
    generate an Ed25519 key pair using:
    
    ```bash
    ssh-keygen -t ed25519
    ```
    

# Adding an SSH Key to a Remote Machine (or Virtual Machine)

## When generating a VM:

When generating a VM, you will often be presented with an option for authentication- if you choose SSH, a keypair will be generated for you; you will be prompted to download the private key. You should save the private key to the .ssh folder in your home directory. The public key will be place on the remote virtual machine for you. 

## When the VM is already created:

## DigitalOcean:

### **Using the DigitalOcean Control Panel:**

1. **Log in to your DigitalOcean account**: Go to the DigitalOcean website and log in to your account.
2. **Access the Droplets page**: Navigate to the "Droplets" section from the sidebar menu.
3. **Select your droplet**: Click on the droplet to which you want to add the SSH key.
4. **Access the "Access" tab**: Once you're on the droplet's detail page, find the "Access" tab, and click on it.
5. **Add SSH key**: Scroll down to the "SSH keys" section, and click the "Add SSH Key" button.
6. **Paste your public SSH key**: Copy the contents of your SSH public key file (usually **`id_rsa.pub`**), and paste it into the text field provided.
7. **Save the SSH key**: Click the "Add SSH Key" button to save the SSH key to your droplet.

### **Using the `doctl` Command-Line Tool:**

If you prefer using the command line, you can also add your SSH key to your droplet using the **`doctl`** command-line tool provided by DigitalOcean. Here's how:

1. **Install `doctl`**: If you haven't already, install the **`doctl`** command-line tool on your local machine. You can find instructions for installing **`doctl`** in the DigitalOcean documentation.
2. **Authenticate with DigitalOcean**: Run **`doctl auth init`** and follow the prompts to authenticate with your DigitalOcean account.
3. **Add SSH key**: Use the **`doctl compute ssh-key import`** command to add your SSH key to your DigitalOcean account. Replace **`SSH_KEY_NAME`** with a name for your SSH key, and **`PUBLIC_KEY_PATH`** with the path to your public SSH key file.
    
    ```vbnet
    vbnetCopy code
    doctl compute ssh-key import SSH_KEY_NAME --public-key-file PUBLIC_KEY_PATH
    
    ```
    
4. **Associate SSH key with droplet**: After adding the SSH key, you can associate it with your droplet using the DigitalOcean control panel or the **`doctl`** command-line tool.

These steps should allow you to add your SSH public key to your virtual machine (droplet) running on DigitalOcean, enabling you to authenticate with SSH using your private key.

## AWS:

To add your SSH public key to a virtual machine (EC2 instance) running on Amazon Web Services (AWS), you typically use the AWS Management Console or the AWS CLI (Command Line Interface). Here's how you can do it:

### **Using the AWS Management Console:**

1. **Log in to the AWS Management Console**: Go to the AWS website and log in to your AWS account.
2. **Navigate to EC2 Dashboard**: From the services menu, select EC2 to access the EC2 Dashboard.
3. **Select your instance**: Click on "Instances" from the sidebar menu and select the EC2 instance to which you want to add the SSH key.
4. **Access Instance Settings**: With your instance selected, click on the "Actions" button, then select "Instance Settings", and finally click on "Edit User Data".
5. **Add SSH key**: In the "User Data" section, paste your SSH public key into the text area under "SSH key pairs". Make sure your public key is in the correct format and doesn't contain any additional spaces or characters.
6. **Save changes**: Click on "Save" to apply the changes.

### **Using the AWS CLI:**

If you prefer using the command line, you can also add your SSH key to your EC2 instance using the AWS CLI. Here's how:

1. **Install and configure AWS CLI**: If you haven't already, install the AWS CLI on your local machine and configure it with your AWS credentials.
2. **Get Instance ID**: Run the following command to get the Instance ID of your EC2 instance:
    
    ```css
    cssCopy code
    aws ec2 describe-instances --query "Reservations[*].Instances[*].{Instance:InstanceId,Name:Tags[?Key=='Name']|[0].Value}"
    
    ```
    
3. **Associate SSH key**: Use the **`aws ec2-instance-connect send-ssh-public-key`** command to associate your SSH public key with the EC2 instance. Replace **`INSTANCE_ID`** with your EC2 instance ID and **`SSH_PUBLIC_KEY`** with the contents of your SSH public key file.
    
    ```vbnet
    vbnetCopy code
    aws ec2-instance-connect send-ssh-public-key --instance-id INSTANCE_ID --availability-zone AVAILABILITY_ZONE --instance-os-user ec2-user --ssh-public-key file://PATH/TO/YOUR/SSH/KEY.pub
    
    ```
    
    Replace **`AVAILABILITY_ZONE`** with the availability zone of your instance.
    

These steps should allow you to add your SSH public key to your EC2 instance in AWS, enabling you to authenticate with SSH using your private key.

## GCP:

### **Using the Google Cloud Console:**

1. **Log in to the Google Cloud Console**: Go to the Google Cloud website and log in to your GCP account.
2. **Navigate to Compute Engine**: From the console menu, navigate to "Compute Engine" under the "Compute" section.
3. **Select your VM instance**: Click on the VM instance to which you want to add the SSH key.
4. **Edit instance**: With your VM instance selected, click on the "Edit" button at the top of the page.
5. **Add SSH key**: Scroll down to the "SSH Keys" section, and click on "Show and edit". Here, you can add your SSH public key by pasting it into the text area provided.
6. **Save changes**: After pasting your SSH public key, click on the "Save" button to apply the changes.

### **Using the `gcloud` Command-Line Tool:**

If you prefer using the command line, you can also add your SSH key to your VM instance using the **`gcloud`** command-line tool. Here's how:

1. **Install and configure `gcloud`**: If you haven't already, install the Google Cloud SDK and configure it with your GCP credentials.
2. **Get VM instance name**: Run the following command to get the name of your VM instance:
    
    ```
    Copy code
    gcloud compute instances list
    
    ```
    
3. **Add SSH key**: Use the **`gcloud compute instances add-metadata`** command to add your SSH public key to the VM instance. Replace **`INSTANCE_NAME`** with the name of your VM instance and **`SSH_PUBLIC_KEY`** with the contents of your SSH public key file.
    
    ```sql
    sqlCopy code
    gcloud compute instances add-metadata INSTANCE_NAME --metadata ssh-keys=USERNAME:SSH_PUBLIC_KEY
    
    ```
    
    Replace **`USERNAME`** with the username you want to use for SSH authentication on the VM instance.
    

These steps should allow you to add your SSH public key to your VM instance running on Google Cloud Platform, enabling you to authenticate with SSH using your private key.

## Azure:

### **Using the Azure Portal:**

1. **Log in to the Azure Portal**: Go to the Azure website and log in to your Azure account.
2. **Navigate to Virtual Machines**: From the Azure portal menu, select "Virtual machines".
3. **Select your VM**: Click on the VM to which you want to add the SSH key.
4. **Access the Settings**: With your VM selected, find the "Settings" section and click on "Reset password" or "Reset SSH public key".
5. **Add SSH key**: In the "Reset password" or "Reset SSH public key" pane, you can add your SSH public key by pasting it into the text area provided. If you're adding an SSH key, make sure to specify the username associated with the key.
6. **Save changes**: After adding the SSH public key, click on "Update" or "Save" to apply the changes.

### **Using the Azure CLI:**

If you prefer using the command line, you can also add your SSH key to your VM instance using the Azure CLI. Here's how:

1. **Install and configure Azure CLI**: If you haven't already, install the Azure CLI and configure it with your Azure credentials.
2. **Get VM resource ID**: Run the following command to get the resource ID of your VM:
    
    ```css
    cssCopy code
    az vm show --resource-group RESOURCE_GROUP_NAME --name VM_NAME --query id -o tsv
    
    ```
    
    Replace **`RESOURCE_GROUP_NAME`** with the name of the resource group containing your VM, and **`VM_NAME`** with the name of your VM.
    
3. **Add SSH key**: Use the **`az vm user update`** command to add your SSH public key to the VM. Replace **`VM_ID`** with the VM resource ID obtained in the previous step, and **`SSH_PUBLIC_KEY`** with the contents of your SSH public key file.
    
    ```css
    cssCopy code
    az vm user update --resource-group RESOURCE_GROUP_NAME --name VM_NAME --username USERNAME --ssh-key-value SSH_PUBLIC_KEY
    
    ```
    
    Replace **`USERNAME`** with the username you want to use for SSH authentication on the VM instance.
    

These steps should allow you to add your SSH public key to your VM instance running in Microsoft Azure, enabling you to authenticate with SSH using your private key.