# Lab session #8: Programming your cloud infrastructure


## Task 8.1: Bootstrap the creation of your web server

This hands-on section guides you through the creation of a load balancer attached to several web servers. We are using a [bootstrapping](https://en.wikipedia.org/wiki/Bootstrapping) technique to create this example.


<p align="center"><img src="./images/Lab06-Schema.png" alt="Layout" title="Layout"/></p>

### Configure the EC2 serving as a seed for the rest of the example

Go to the course "AWS Academy Learner Lab", open the modules and open the "Learner Lab". Click the button "Start Lab",
wait until the environment is up and then click "AWS" at the top of the window and open the AWS Console.

Go to [AWS console](https://us-east-1.console.aws.amazon.com/ec2/home?region=us-east-1#Home:) 

1. Create a security group named ``web-sg``. Add HTTP from "Anywhere-IPv4" and SSH from your current IP address as inbound rules.

2. Launch a new EC2 instance. Use Ubuntu 22.04 as base AMI
 
2. Select `t2.micro` instance type (Free tier elegible)

3. Name the instance as `apache-web-server`

4. Select ``web-sg`` as the security group in the Networking settings.

6. Enable auto-assign a public IP. 

6. Add 8GB of storage space.
7. Add some tags for tracking. 
    - Project = ccbda bootstrap
    - Name = apache-web-server
    - Cost-center = laboratory
8. At the bottom of the page unfold "Advanced details" and copy the following code at the **"User data"**  text box. You can check for errors, when the EC2 is running, at `/var/log/cloud-init-output.log`.
 
    ````bash
    #! /bin/bash -ex
    # This script is for Ubuntu
    sudo apt-get update
    sudo apt-get -y install apache2
    sudo systemctl enable apache2
    sudo systemctl start apache2
    sudo apt-get -y install mysql-client
    sudo apt-get -y install php7.2-mysql php7.2-curl php7.2-cgi php7.2 libapache2-mod-php7.2 php-xml php7.2-zip
    sudo usermod -a -G www-data ubuntu
    sudo chown -R root:www-data /var/www
    sudo chmod 2775 /var/www
    sudo find /var/www -type d -exec chmod 2775 {} +
    sudo find /var/www/ -type f -exec chmod 0664 {} +
    ````

Check that you have an Apache2 web server running by typing on your Internet Browser http://YOUR-SERVER-IP 

### Create a load balancer

Once the EC2 is being lauched, create a "Target Group" and an "Application Load Balancer".

1. "Create target group" named ``primary-apache-web-server-target`` and include the EC2 instance created previously.

2. Create a security group named ``load-balancer-sg``. Add HTTP and HTTPS from "Anywhere-IPv4" as inbound rules.

1. Check the target group and select the action "Associate with a load balancer" of type "Aplication Load Balancer". Name it `load-balancer`, with an internet-facing scheme. Add protocols HTTP and HTTPS using standard ports and select ALL availability zones from your current region. Add the following tags for tracking. 
    - Project = ccbda bootstrap
    - Cost-center = laboratory
2. You would normally obtain an SSL certificate from AWS. For that, you need to have control over the DNS of the server's domain. Select `Upload a certificate to ACM` and, **for testing purposes**, use the following unix command to create a  and create a self-signed certificate for CN (Common Name) "myserver.info". The generated information looks like the text below. Leave the certificate chain empty and select ``ELBSecurityPolicy-TLS-1-2-2017-01`` as the security policy. 

```
   openssl req -x509 -newkey rsa:4096 -keyout key.pem -out cert.pem -sha256 -days 365 -nodes
```


    ```
    -----BEGIN CERTIFICATE-----
    MIIDAzCCAeugAwIBAgIJAOcF+7m0Y7yQMA0GCSqGSIb3DQEBBQUAMBgxFjAUBgNV
    BAMMDW15c2VydmVyLmluZm8wHhcNMTkwMzIxMTQzOTE0WhcNMjkwMzE4MTQzOTE0
    ....
    qPNs9Xnq8GturB3J7qTX2pOX1L0fWm91kqd5saD4/n6FQwiKQX9QywROPQH5IXcm
    WaBsBYeg03iKzcq1HJn0oXjOg3ksQD658tK0ydc9oyjfFFkU/RpfjdKbsVaNsdho
    AbVaYusFQw==
    -----END CERTIFICATE-----
    
    
    -----BEGIN RSA PRIVATE KEY-----
    MIIEpAIBAAKCAQEAv26vJIiiVtkmwSv9bBEtN2v4aW9vA+CGpfDk5LY3DnbKwsAQ
    UR/gIkfgi6siVme/jtbRf6BS3Sv/0eRWAWhIqvmiD3x2SJzc449AqKIcWhdjBAZt
    ....
    bNAB2Yr6GGGx8zpdZnJtCaWpKRTfCYfB0KoHuzCCDyXW5XBDnaD1DsO2OCAccDeL
    7Qrhmkr8Pl353hCmoqH06zzkeHsPD+XxQN9ANL4lsBJdo8r3Z+F6SQ==
    -----END RSA PRIVATE KEY-----
    ```
6. Attach the ELB to the ``load-balancer-sg`` security group.

8. Check the load balancer state and wait while it says "provisioning". Once the ELB state is "active", go to the "Description" tab and copy the DNS name assigned http://load-balancer-1334015960.eu-west-1.elb.amazonaws.com/ and paste it in your browser. 

   <p align="center"><img src="./images/Lab06-ApacheWorking.png" alt="Apache working" title="Apache working"/></p>

9. Once the load balancer is working correctly and showing the web server home page, we will restrict the input source to requests from the load balancer. To achieve that, go to `web-sb` and modify the input address for port 80. Remove the contents of the address "0.0.0.0/0" and type "load-balancer-sg". Select the option that appears. Remove the rule por IPv6 (source "::/0") that has access to port 80.

Now you should not be able to access the web server directly through port 80 (using the EC2 IP address) but you should be able to access the web server using port 80 (HTTP) and 443 (HTTPS) (using the load balancer IP address).

### Modify the web server response and create a base AWS AMI 

1. Use ssh to connect to the running EC2 instance. Remove the file `/var/www/html/index.html`  and copy the contents below to `/var/www/html/index.php`. Close the ssh session.

    ````php
    <html>
    <head></head>
    <body>
    <h1>This is instance
    <?php
        $this_instance_id = file_get_contents('http://169.254.169.254/latest/meta-data/instance-id');
    if(empty($this_instance_id))
        echo "Unknown ID";
    else
        echo (string)($this_instance_id); 
    ?>
    
    alive!!</h1>
    </body>
    </html>
    ````
2. Verify that the php file is working correctly and showing the EC2 instance identifier.

14. As shown below, create a machine image (AMI) using the name `test-web-server-version-1.0` with the description `LAMP web server`.

    <p align="center"><img src="./images/Lab06-AMI.png" alt="AMI" title="AMI"/></p>
    
    <p align="center"><img src="./images/Lab06-AMI-config.png" alt="AMI configure" title="AMI configure"/></p>

### Create an auto scalling group
    
1. Create a **Launch Configuration** using the AMI that you created before. Name it `web-server-auto-scaling-configuration` and attach the `web-sg` security group that you created before.
    <p align="center"><img src="./images/Lab06-LoadBalancer.png" alt="Auto scalling group" title="Auto scalling group"/></p>
    <p align="center"><img src="./images/Lab06-AutoScalingGroup.png" alt="Auto scalling group" title="Auto scalling group"/></p>

2. Create an **Auto Scaling Group** named `web-server-auto-scaling-group`. Start with 2 instances in a VPC. Add all the availability zones that you were using before. 

3. Select the *Keep this group at its initial size* option. Optionally you may want to create a scale-in and scale-out policy, but do that once you've verified that this simpler policy (no auto-scaling) works correctly.

4. Add notifications to your e-mail via an SNS topic.

5. Add some tracking tags
    - Project = ccbda bootstrap
    - Cost-center = laboratory
    
6. Once the auto scaling group is running, you see that you have two more EC2 instances running.

### Test your new system

Check how many EC2 instances are running and, once they are ready, use the ELB URL in your browser and see that the output of the webpage changes when reloading the URL. The EC2 instance ID of the first EC2 instance created does not show since it is not part of the auto scaling group. Two new EC2 instances have been created using the AMI provided.

### Questions

**Q811.** What happens when you use https://your-load-balancer-url instead of http://your-load-balancer-url ? Why does that happen? How could you fix it?

**Q812.** Stop all three EC2 instances and wait aprox. 5 minutes. What happens? Why?

**Q813.** Terminate all three EC2 instances and wait aprox. 5 minutes. What happens? Why?

**Q814.** How are you going to end this section regarding the use of AWS resources?

**Q815.** Create a piece of code (Python or bash) to reproduce the above steps required to launch a new set of web servers with a load balancer. Start using the AMI that you have already created.

 
Write your answers in the `README.md` file for this session.


# How to submit this assignment:

Use the **private** repo named *https://github.com/CCBDA-UPC/2022-6-xx*. It needs to have, at least, two files `README.md` with your responses to the above questions and `authors.json` with both members email addresses:

```json5
{
  "authors": [
    "FIRSTNAME1.LASTNAME1@estudiantat.upc.edu",
    "FIRSTNAME2.LASTNAME2@estudiantat.upc.edu"
  ]
}
```

Make sure that you have updated your local GitHub repository (using the `git`commands `add`, `commit` and `push`) with all the files generated during this session. 

**Before the deadline**, all team members shall push their responses to their private *https://github.com/CCBDA-UPC/2022-6-xx* repository.


