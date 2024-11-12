# Load Balancer Solution with Nginx and SSL/TLS

## Introduction
In this project, we’ll enhance the DevOps Tooling Website setup by:
1. Replacing the Apache load balancer with Nginx.
2. Adding an Elastic IP and configuring a custom domain.
3. Implementing SSL/TLS to secure traffic with HTTPS.

## Project Overview
This project builds on the previously established DevOps Tooling Website Solution, Load Balancer Solution, and CI/CD deployment with Jenkins, integrating more advanced configurations. We’ll focus on:
- Setting up an Nginx load balancer.
- Associating an Elastic IP and domain.
- Adding SSL/TLS for secure communication.

## Architecture
![](/images/Screenshot%20(629).png)

## Prerequisites
Before starting, ensure you have:
1. Completed the [DevOps Tooling Website Solution](https://github.com/kenneth-stack/DevOps-Tooling-website-solution) setup, including:
   - NFS Server
   - Database Server
   - Three Web Servers
2. Implemented the load balancer as described in the [Load Balancer Solution with Apache](https://github.com/kenneth-stack/Load-Balancer-With-Apache).
3. Set up CI/CD with Jenkins, following the [Jenkins Deployment Guide](https://github.com/kenneth-stack/tooling/blob/main/Documentation.md).
4. A basic understanding of Linux and command-line interfaces.
5. Familiarity with Jenkins.
6. Access to an AWS account.
7. A GitHub account and repository for the project.

### SSL/TLS and Certificate Authorities (CAs)

Befor we dive into the task, let's take a look at what SSL is all about.

**SSL (Secure Sockets Layer)** and **TLS (Transport Layer Security)** are protocols for encrypting internet traffic and verifying server identity. SSL/TLS prevents data interception by encrypting data between the client and server.

A **Certificate Authority (CA)** is a trusted entity that issues digital certificates, verifying a website’s legitimacy. Popular CAs include Let's Encrypt, DigiCert, and GlobalSign. CAs validate the domain owner’s identity before issuing a certificate, adding credibility and security to a domain.

### Domains, DNS, and Common DNS Records
A **Domain Name** is the website's address that users type into a browser. **DNS (Domain Name System)** maps domain names to IP addresses, making it possible for browsers to load resources from servers.

#### Common DNS Records:
1. **A Record**: Maps a domain to an IPv4 address.
2. **AAAA Record**: Maps a domain to an IPv6 address.
3. **CNAME Record**: Maps a domain or subdomain to another domain (e.g., www.example.com to example.com).
4. **MX Record**: Directs email to a mail server.
5. **TXT Record**: Holds arbitrary text, often for domain verification and security settings.

### Key Concepts for Nginx Load Balancing
Nginx is commonly used for load balancing due to its flexibility and high performance. Key concepts include:
1. **Upstream Servers**: Define backend servers to which Nginx directs traffic.
2. **Load Balancing Methods**: Nginx supports multiple methods, such as:
   - **Round Robin**: Distributes requests sequentially.
   - **Least Connections**: Sends traffic to the server with the fewest connections.
3. **SSL/TLS Termination**: Decrypts SSL traffic at the load balancer, reducing the load on backend servers.
4. **Health Checks**: Ensures backend servers are available, rerouting traffic if a server fails.

## Task

This project consists of two parts:
1. Configure Nginx as a Load Balancer.
2. Register a new domain name and configure secured connection using SSL/TLS certificates.

## Implementation

1. **Set Up Nginx Load Balancer**:
   - Launch a new EC2 instance for the Nginx load balancer (similar to the Apache load balancer setup).
   - Install Nginx:
```bash
     sudo apt update
     sudo apt install nginx -y
 ```
 ![](/images/Screenshot%20(630).png)

    ```bash
    sudo systemctl status nginx
    ```
![](/images/Screenshot%20(631).png)

2. **Edit the Nginx configuration** (usually at `/etc/nginx/nginx.conf` or in `/etc/nginx/conf.d/`):
   - Use hostnames for backend servers to allow easy management through `/etc/hosts`.

    ```bash
    sudo nano /etc/hosts
    ```
   - Map hostnames to IPs for the backend servers:

```bash
      172.31.6.35 webserver1
      172.31.2.9 webserver2
```
![](/images/Screenshot%20(632).png)

   - Define the `upstream` block.

```bash
   sudo nano /etc/nginx/nginx.conf
```
* Insert the following configuration into the http section:

```bash
upstream myproject {
    server webserver1 weight=5;
    server webserver2 weight=5;
}

server {
    listen 80;
    server_name www.domain.com;

    location / {
        proxy_pass http://myproject;
    }
}
```


Comment out this line: ```# include /etc/nginx/sites-enabled/*;```

3. **Test and Restart Nginx**:
   - Test the configuration:
```bash
     sudo nginx -t
```
     > NB: If the configuration is error-free, you’ll see output similar to:
     nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
     nginx: configuration file /etc/nginx/nginx.conf test is successful

   - Restart Nginx:
```bash
     sudo systemctl restart nginx
```
![](/images/Screenshot%20(633).png)

4. Register a domain

Get a domain name from any domain providers but if you in Nigeria, I will recommend getting `.ng` or `.net.ng` from [Qservers](https://process.qservers.net/index.php?m=br_dnsmanager&id=204583)

![](/images/Screenshot%20(634).png)

5. Attach an Elastic IP and Configure Domain
    
    - In AWS, allocate an Elastic IP and attach it to the Nginx instance.
    ![](/images/Screenshot%20(635).png)

    - create an A record in your domain DNS manager and pointing to the Elastic IP.
    - Create a CNAME record and point to the A record.
    ![](/images/Screenshot%20(637).png)

    - update the server_name in your nginx config with `<domainname>.com www.<domainname>.com` e.g `www.delightdev.ng`

    ![](/images/Screenshot%20(638).png)
    - Visit [DNS Check](https://dnschecker.org/) to test your dns record.

6. Install SSL/TLS Certificate
    - Make sure the snapd service is active:
```bash
    sudo systemctl status snapd
```
  - Install Certbot for Nginx:

 ```bash
    sudo snap install --classic certbot
 ```
 ![](/images/Screenshot%20(639).png)

   - Request an SSL certificate:

```bash
sudo ln -s /snap/bin/certbot /usr/bin/certbot
sudo certbot --nginx
```

7. Simulate the renewal process:
    By default, LetEncrypt certificate is valid for 90 days, it is recommended to renew it every 60 days to ensure your site remains secured. to simulate the renewal process, run the command:
```bash
    sudo certbot renew --dry-run
```
    >NB: Remember to open the https port on AWS

    - **Schedule Auto-Renewal**:
```bash
    crontab -e
```
    Add:
```bash
    */12 * * * * root /usr/bin/certbot renew > /dev/null 2>&1
```

    >This command runs twice daily (every 12 hours).
    >The **> /dev/null 2>&1** redirect suppresses all output, both standard and error messages, keeping the system logs clean.

    >NB: This will check the SSL certificate validity twice daily. You can adjust it if twice is too much, see [cron guru](https://crontab.guru/) for reference

We have now successfuly configured a Nginx based Load Balancer for our webservers, ensured it can be accessed by a domain name and has SSL installed for security.

## Benefits of Nginx Load Balancer with SSL/TLS Configuration

**1.Improved Performance and Scalability:**

Nginx efficiently distributes incoming requests across multiple web servers, balancing the load. This setup allows for handling a larger number of simultaneous requests, improving the overall performance and enabling the system to scale as demand increases.

**2. Enhanced Security:**

SSL/TLS encrypts data in transit, protecting it from Man-in-the-Middle (MITM) attacks and ensuring sensitive data, like login credentials and personal information, remains secure.
Using Certbot to automate SSL certificate issuance and renewal with Let's Encrypt ensures the site remains secure without requiring manual intervention for certificate management.

**3. Cost-Efficient Scaling:**

You can scale horizontally by adding more web servers to handle increased demand instead of investing in a single, powerful server. This horizontal scaling is often more cost-effective for growing applications.

**4. Flexible and Customizable Traffic Management:**

Nginx provides options to configure advanced load balancing algorithms and settings. You can prioritize certain servers or adjust server weights to handle high-traffic scenarios and optimize resource use.


# Challenges of Nginx Load Balancer with SSL/TLS Configuration

**1.Complex Setup and Maintenance:**

Configuring and managing an Nginx load balancer along with SSL/TLS certificates requires careful setup and an understanding of network and server management.
Ongoing maintenance, including SSL certificate renewals and ensuring the load balancer functions correctly as the server environment changes, can add complexity.

**2. Potential Downtime During Configuration:**

Improper configuration can lead to downtime if the load balancer isn’t correctly set up to route traffic, or if SSL/TLS settings cause issues with connections. Testing each part thoroughly is essential to avoid service interruptions.

**3. Certificate Management Overhead:**

While Certbot automates SSL certificate management, there may still be occasional manual troubleshooting required. Issues can arise if the domain or DNS configuration changes or if there are issues with Let's Encrypt's validation process.


# Conclusion:

This project demonstrates how to set up a load balancer using Nginx to distribute traffic across multiple web servers. It's designed to improve the performance and reliability of web applications by evenly distributing incoming requests.