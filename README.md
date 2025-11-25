# Deploying Jenkins on an AWS Ubuntu EC2 Instance

This guide walks through provisioning an Ubuntu EC2 instance, installing Jenkins, opening access over the network, and troubleshooting common issues.

---

## Prerequisites

- An AWS account with permissions to create EC2 instances and modify security groups
- Ubuntu Server (latest LTS recommended) AMI
- SSH key pair for connecting to the instance
- Ability to open inbound ports in the instance’s Security Group (22 for SSH, 8080 for Jenkins UI)

---

## 1) Provision an EC2 Instance

1. In the **AWS Management Console**, launch an EC2 instance.
2. Choose an **Ubuntu Server** AMI (LTS).
3. Select an instance type (e.g., `t2.micro` for testing).
4. Attach/choose a key pair.
5. Create or select a **Security Group** (see “Open Ports” below).
6. Launch the instance.

<img width="940" height="412" alt="image" src="https://github.com/user-attachments/assets/4be749d7-f659-4e24-a912-bfd2d2e7eafc" />
<img width="940" height="467" alt="image" src="https://github.com/user-attachments/assets/fb9cd686-1303-44ec-9477-6fd7f0e511dd" />


---

## 2) Connect via SSH

From your terminal:


ssh -i /path/to/key.pem ubuntu@<EC2_PUBLIC_IP>

<img width="933" height="192" alt="image" src="https://github.com/user-attachments/assets/977d7e74-87a1-4058-a910-1160712058b8" />


sudo su -
# exit root with:
logout

<img width="533" height="61" alt="image" src="https://github.com/user-attachments/assets/7871a5f5-005a-4579-bd9b-7370bf705565" />

---

## 3) Update the Server:
sudo apt update && sudo apt upgrade -y

<img width="940" height="233" alt="image" src="https://github.com/user-attachments/assets/a0d236ea-290c-4c60-951b-27426fa2aba9" />

---
## 4) Install Java (Required for Jenkins)
sudo apt install -y fontconfig openjdk-17-jdk

<img width="647" height="61" alt="image" src="https://github.com/user-attachments/assets/108e16ed-78f5-4063-b733-7dfa0c77d71f" />
type java -version to verify output shows Java 17+

<img width="940" height="158" alt="image" src="https://github.com/user-attachments/assets/0347cc61-6f1f-4c8f-b498-d9f61a12d9bf" />

---

## 5) Install Jenkins 
Use the official Jenkins repository (LTS recommended):
# Import the Jenkins repo key
sudo mkdir -p /usr/share/keyrings
wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key \
  | sudo tee /usr/share/keyrings/jenkins-keyring.asc > /dev/null

# Add the Jenkins repository
echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/" \
  | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null

# Install Jenkins
sudo apt update
sudo apt install -y jenkins

<img width="940" height="301" alt="image" src="https://github.com/user-attachments/assets/fe27b5da-5471-4208-a82f-60ba52f80d16" />

# Start and enable the service
sudo systemctl enable --now jenkins
sudo systemctl status jenkins

<img width="986" height="352" alt="image" src="https://github.com/user-attachments/assets/e5e1b290-3d4e-42af-8566-df4599c87683" />

You should see active (running) if Jenkins started successfully.

---
## 6) Configure Security Group Rules
Ensure inbound access for:
In the AWS Console → EC2 → Security Groups attached to your instance, add:

Inbound: TCP 22 from your IP (SSH)

Inbound: TCP 8080 from your IP (recommended)

For quick testing you can allow 0.0.0.0/0, but for security you should restrict to specific IPs.

Save the rules.

  <img width="985" height="436" alt="image" src="https://github.com/user-attachments/assets/cb05238f-6340-4f51-9887-bb49c3107065" />


  ---

  ## 7) Access Jenkins in Your Broswer & Retrieve Password
  Access Jenkins on the broswer:
  http://<EC2_PUBLIC_IP>:8080
  
<img width="940" height="554" alt="image" src="https://github.com/user-attachments/assets/2fbb48d8-ea1e-43a2-b7bc-9d2d0c380874" />

  To access the password use the command:
  sudo cat /var/lib/jenkins/secrets/initialAdminPassword
  
<img width="863" height="125" alt="image" src="https://github.com/user-attachments/assets/1e02f509-4651-43a8-a11e-f7c23bbb1b3e" />

Note: The path uses lowercase jenkins. If you see a “No such file” error, double-check capitalization.

Paste this password into the Jenkins setup page, then follow the wizard to install suggested plugins and create your admin user.

<img width="940" height="618" alt="image" src="https://github.com/user-attachments/assets/10a32031-05a2-44a1-8062-b9d92f1b650c" />

---

## 8) Common Issue: Wrong Java Version
Symptom: Jenkins fails to start or the service exits repeatedly.
Cause: Using an unsupported Java (e.g., Java 11).
  
Fix:

Install a supported Java version (17 or 21 LTS):

sudo apt install -y openjdk-17-jdk
sudo update-alternatives --config java  # if multiple Javas are installed
java -version


Restart Jenkins:

sudo systemctl restart jenkins
sudo systemctl status jenkins

---

## 9) Helpful Coomands
# Check Jenkins service
systemctl status jenkins

# View logs
journalctl -u jenkins -f

# Start/stop/restart
sudo systemctl start|stop|restart jenkins

---
Security Tips

- Restrict port 8080 to trusted IP addresses only.

- Consider placing Jenkins behind a reverse proxy (e.g., ALB, Nginx) with HTTPS.

- Regularly update the instance and Jenkins plugins.

- Use IAM roles and credentials storage best practices for any cloud integrations.

---

Cleanup

When finished, stop or terminate the EC2 instance to avoid charges, and remove any open security group rules you no longer need.





