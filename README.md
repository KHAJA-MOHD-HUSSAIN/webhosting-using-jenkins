# Web Hosting Using Jenkins on AWS

This project demonstrates how to automate the deployment of an HTML webpage (`hussain.html`) to an AWS EC2 instance using Jenkins. We are using Git for version control, and Jenkins for Continuous Integration/Continuous Deployment (CI/CD) to manage the deployment process.

## Project Structure

- **hussain.html**: A simple HTML webpage that will be deployed to the web server.
- **Jenkins Pipeline**: Automates the process of pulling the code from Git, and deploying the `hussain.html` to an AWS EC2 instance.

## Prerequisites

### Software Requirements
- **AWS EC2 instance** (Linux-based, e.g., Ubuntu) with a web server (Apache or Nginx) installed.
- **Jenkins** installed on your local machine or a server. Follow the [Jenkins Installation Guide](https://www.jenkins.io/doc/book/installing/).
- **GitHub repository** to store your `hussain.html` file.
- **AWS CLI** and **SSH Key** for Jenkins to deploy to EC2.
- **Jenkins plugins**: Git, SSH, AWS EC2 Plugin, etc.

### AWS Setup
1. **EC2 Instance**: Create an EC2 instance on AWS and install a web server (Apache or Nginx).
2. **Security Group**: Ensure the security group of your EC2 instance allows inbound HTTP traffic on port `80`.
3. **SSH Key**: Create an SSH key pair to access your EC2 instance securely. Upload the public key to the EC2 instance and save the private key for Jenkins.
4. **Install Apache/Nginx on EC2**: Install and configure Apache/Nginx on the EC2 instance to serve the webpage.
   - For Apache: `sudo apt install apache2`
   - For Nginx: `sudo apt install nginx`

## Setup Instructions

### 1. **Install Jenkins**

Follow the official Jenkins installation guide for your operating system:

- [Jenkins Installation Guide](https://www.jenkins.io/doc/book/installing/)

### 2. **Install Jenkins Plugins**

1. Navigate to Jenkins > Manage Jenkins > Manage Plugins.
2. Install the following plugins:
   - **Git Plugin**: For SCM integration.
   - **SSH Agent Plugin**: For managing SSH credentials to connect to your EC2 instance.
   - **Pipeline Plugin**: To create Jenkins pipelines.

### 3. **Create a New Jenkins Job**

1. Open Jenkins in your browser (e.g., `http://localhost:8080`).
2. Click **New Item** and choose **Pipeline**.
3. Give your project a name (e.g., `AWS_Web_Hosting`), and click **OK**.

### 4. **Configure Jenkins SCM (GitHub)**

1. In the **Pipeline** configuration page, scroll down to the **Pipeline** section.
2. Under **Definition**, select **Pipeline script from SCM**.
3. Select **Git** as the SCM type.
4. Add your GitHub repository URL where `hussain.html` is stored.
5. Provide the credentials if needed (your GitHub token or SSH keys).

### 5. **Configure AWS EC2 Deployment**

1. **Configure SSH Keys**: 
   - Go to Jenkins > Manage Jenkins > Manage Credentials.
   - Add a new SSH credential using your private key (the key you use to access your EC2 instance).
   
2. **Add EC2 Information to Jenkins**:
   - Navigate to **Manage Jenkins > Configure System**.
   - Under **SSH Servers**, add your EC2 instance details:
     - **Host**: Your EC2 public IP address.
     - **Username**: `ubuntu` (for Ubuntu instances) or appropriate username for your EC2 instance.
     - **Private Key**: Choose the SSH private key created earlier.

### 6. **Write Jenkins Pipeline Script**

In the **Pipeline** script section, add the following Groovy code to automate the deployment process:

```groovy
pipeline {
    agent any

    environment {
        EC2_USER = 'ubuntu'  // Adjust if necessary
        EC2_HOST = 'your-ec2-public-ip'  // Replace with your EC2 public IP address
        REMOTE_DIR = '/var/www/html'  // Directory on the EC2 instance where HTML files are stored
    }

    stages {
        stage('Clone Repository') {
            steps {
                script {
                    // Clone the GitHub repository containing the hussain.html file
                    git url: 'https://github.com/yourusername/your-repository.git', branch: 'main'
                }
            }
        }

        stage('Deploy to AWS EC2') {
            steps {
                script {
                    // Copy the hussain.html to the remote server using SCP or Rsync
                    sh """
                        scp -i /path/to/your/private-key.pem hussain.html ${EC2_USER}@${EC2_HOST}:${REMOTE_DIR}/hussain.html
                    """
                }
            }
        }

        stage('Restart Web Server') {
            steps {
                script {
                    // Restart the web server on EC2 to reflect changes (Apache or Nginx)
                    sh """
                        ssh -i /path/to/your/private-key.pem ${EC2_USER}@${EC2_HOST} 'sudo systemctl restart apache2'
                    """
                }
            }
        }
    }

    post {
        success {
            echo 'Deployment successful!'
        }
        failure {
            echo 'Deployment failed!'
        }
    }
}
```

# Jenkins Web Deployment Pipeline

## 7. Run the Jenkins Job

After saving the pipeline, click on **Build Now** to trigger the job. Jenkins will:

- Clone the repository.
- Deploy the `hussain.html` file to the EC2 instance.
- Restart the web server (Apache/Nginx) to reflect the changes.

## 8. Verify the Deployment

- Open your browser and navigate to the public IP address of your EC2 instance:
```http://<EC2-PUBLIC-IP>:8080```

- You should see the `hussain.html` webpage hosted.

## Troubleshooting

If you encounter issues, check the following:

- **EC2 Security Group**: Ensure that the EC2 security group allows inbound HTTP traffic on port 80.
- **Jenkins Permissions**: Make sure the Jenkins machine has the correct permissions and can access the EC2 instance via SSH.
- **Private Key Path**: Double-check the paths to the private key in the pipeline script.

## Conclusion

By using Jenkins, Git, and AWS, you can automate the deployment of your web pages and applications, ensuring that changes are reflected on the web server as soon as they are committed. This setup is scalable and can be extended to support more complex web applications and services.

