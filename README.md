# Hosting a Java Spring Boot Application on Amazon EC2

## 1. Prepare Your Application

### Clone the Repository

Clone your Spring Boot application repository from GitHub:

```bash
git clone <repository-url>
```

### Install Maven

Follow these detailed steps to install and set up Maven:

1. **Download Maven**

   Download the Maven binary archive from the [Apache Maven website](https://maven.apache.org/download.cgi). Choose the `.zip` or `.tar.gz` file, depending on your preference.

   Example for `.tar.gz`:

   ```bash
   wget https://archive.apache.org/dist/maven/maven-3/3.9.8/binaries/apache-maven-3.9.8-bin.tar.gz
   ```

2. **Extract the Archive**

   Extract the downloaded archive:

   ```bash
   tar xzvf apache-maven-3.9.8-bin.tar.gz
   ```

   This will create a directory named `apache-maven-3.9.8`.

3. **Move Maven to `/opt` (Optional)**

   For system-wide availability, you can move Maven to `/opt`:

   ```bash
   sudo mv apache-maven-3.9.8 /opt/
   ```

4. **Set Up Environment Variables**

   Add Maven to your `PATH` environment variable. Open or create the `~/.bashrc` file:

   ```bash
   nano ~/.bashrc
   ```

   Add the following lines to the end of the file:

   ```bash
   # Maven environment variables
   export M2_HOME=/opt/apache-maven-3.9.8
   export PATH=$M2_HOME/bin:$PATH
   ```

   **Details**:
   - `M2_HOME` points to the Maven installation directory.
   - `PATH` includes the Maven `bin` directory to make the `mvn` command available globally.

5. **Apply the Changes**

   Load the updated `.bashrc` file:

   ```bash
   source ~/.bashrc
   ```

6. **Verify Maven Installation**

   Confirm that Maven is installed and available:

   ```bash
   mvn -v
   ```

   The output should display Maven's version and other details.

### Build the Application

Navigate to your project directory:

```bash
cd <project-directory>/complete
```

Clean and package the application to generate a JAR file:

```bash
mvn clean package
```

The JAR file will be located in the `target` directory, e.g., `target/flightservice-0.0.1-SNAPSHOT.jar`.

## 2. Set Up Amazon EC2

### Launch an EC2 Instance

Go to the Amazon EC2 Management Console and launch an EC2 instance with your desired configuration.

### Install Java on the EC2 Instance

Connect to your EC2 instance via SSH.

Install Java 17 (Amazon Corretto):

```bash
sudo yum install java-17-amazon-corretto-devel
```

### Upload the JAR File to S3

Navigate to the Amazon S3 Management Console and upload your JAR file to an S3 bucket.

### Generate a Presigned URL

In the S3 Management Console:
- Select your uploaded JAR file.
- Click on "Actions" and choose "Create presigned URL".
- Copy the generated URL.

### Download the JAR File to EC2

Use the presigned URL to download the JAR file to your EC2 instance:

```bash
wget -O flightservice-0.0.1-SNAPSHOT.jar <your-presigned-url>
```

## 3. Configure the Application to Run Persistently

### Create a System Service

Create a new service file for your Spring Boot application. Open a file named `flightservice.service` in the `/etc/systemd/system/` directory:

```bash
sudo nano /etc/systemd/system/flightservice.service
```

Add the following content to the file:

```ini
[Unit]
Description=Flight Service Java Application
After=network.target

[Service]
ExecStart=/usr/bin/java -jar /home/ec2-user/flightservice-0.0.1-SNAPSHOT.jar
WorkingDirectory=/home/ec2-user
SuccessExitStatus=143
User=ec2-user
Group=ec2-user

[Install]
WantedBy=multi-user.target
```

**Details**:
- `ExecStart`: The command to start your application. Replace `/home/ec2-user/flightservice-0.0.1-SNAPSHOT.jar` with the actual path to your JAR file.
- `User` and `Group`: Set to `ec2-user` or the appropriate user running the service.

### Reload and Start the Service

Reload the systemd manager configuration:

```bash
sudo systemctl daemon-reload
```

Start the service:

```bash
sudo systemctl start flightservice
```

Enable the service to start on boot:

```bash
sudo systemctl enable flightservice
```

### Verify the Service

Check the status of your service to ensure it's running:

```bash
sudo systemctl status flightservice
```

## 4. Configure Security Group

### Allow TCP Traffic on Port 8080

Go to the Amazon EC2 Management Console:
- Select your EC2 instance.
- Configure the security group to allow inbound TCP traffic on port 8080.

**Steps**:
- Navigate to the "Security Groups" section.
- Select the relevant security group.
- Click on the "Inbound rules" tab and then "Edit inbound rules".
- Add a rule with the following settings:
  - **Type**: Custom TCP Rule
  - **Port Range**: 8080
  - **Source**: Anywhere (or specify a range of IPs)
- Save the changes.

## 5. Access Your Application

Open a web browser and navigate to `http://<ec2-public-ip>:8080` to access your Spring Boot application.

---

This documentation provides a comprehensive guide for setting up a Java Spring Boot application on an Amazon EC2 instance, including building the application, configuring the server, and ensuring the application runs persistently. Adjust paths and configurations as necessary based on your specific setup and requirements.
```
