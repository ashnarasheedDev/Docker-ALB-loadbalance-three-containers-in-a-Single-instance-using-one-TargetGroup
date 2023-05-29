# Docker-ALB-loadbalance-three-containers-in-a-Single-instance-using-one-TargetGroup
In most load balancing scenarios, we have seen each container is typically deployed on a separate instance. What If we have three containers running on a single EC2 instance, Is it still possible to load balance them using a single target group and an Application Load Balancer (ALB)? 

**Let's see how i create this setup:**

I'm creating three containers from httpd:alpine image and deploying three websites on each container. Each container will be mapped to a different port on the host system: 8081 for website1, 8082 for website2, and 8083 for website3.

To distinguish the websites when they are loaded, we have edited the index.html file of each website's templates so as it prints website1, website2 and website3 accordingly.

Create a target group: Configure the target group with the appropriate settings, such as the target type (instance or IP), hAealth checks, and protocol. Add targets to the target group

Create an Application Load Balancer (ALB): Configure it with the desired settings, such as listeners, security groups, and subnets.

Add the target group to the ALB: During the creation of the ALB, specify the target group you created earlier and associate it with the appropriate listener and port.

Add a route53 record for a domain so a sit Alias to ALB'S Ip.

**Here is a simple Block diagram illustrates the setup**

![alt text](https://i.ibb.co/yRDN33M/git-alb.png)
<!--  -->
### Step 1- Launch an instance and Install Docker 

- Launch an EC2 instance: Start by launching an EC2 instance that will serve as the host for containers. 

- Install Docker: Once the EC2 instance is up and running, connect to it via SSH and install Docker on the instance. You can follow the Docker installation instructions given below:

```
sudo yum install docker -y
sudo systemctl enable docker
sudo systemctl restart docker
```
###  Step 2 - Create 3 containers

Create Docker containers: Create three Docker containers, each serving one website. 

><b>Create container1</b>
- docker container run: This command is used to run a new Docker container.
- --name website1: Assigns the name "website1" to the container.
- -d: Runs the container in detached mode, meaning it runs in the background.
- -p 8081:80: Maps port 8081 on the host machine to port 80 in the container. This allows accessing the website through port 8081 on the host.
- --restart always: Specifies that the container should always restart if it stops for any reason.
- -v $(pwd)/website1/:/usr/local/apache2/htdocs/: Bind Mounts the local directory "website1" (assuming it's in the current working directory) to the container's /usr/local/apache2/htdocs/ directory. This allows the container to serve the website's content from the mounted directory.
- httpd:alpine: Specifies the Docker image to use for the container, in this case, httpd:alpine, which is a lightweight Apache HTTP Server image based on Alpine Linux.

You can use a similar command to run the other containers for website2 and website3, changing the container name, port mapping, and mounted directory accordingly.

```
docker container run --name website1 -d -p 8081:80 --restart always -v $(pwd)/website1/:/usr/local/apache2/htdocs/ httpd:alpine
```
><b>Create container2</b>

```
docker container run --name website2 -d -p 8082:80 --restart always -v $(pwd)/website2/:/usr/local/apache2/htdocs/ httpd:alpine
```
><b>Create container3</b>

```
docker container run --name website3 -d -p 8083:80 --restart always -v $(pwd)/website3/:/usr/local/apache2/htdocs/ httpd:alpine
```
### Step 3 - Create Target Group

  - Open the AWS Management Console and navigate to the EC2 service.
  - In the left navigation pane, click on "Target Groups" under the "Load Balancing" section.
  - Click on the "Create target group" button.
  - Configure the target group settings:
        Give the target group a name and optionally provide a description.
        Select the desired target type, which is usually "Instance" in this case.
        Choose the protocol and port that the target group will use to communicate with the targets.
        Select the VPC where your instance resides.
        Optionally, you can configure health checks for the targets.
        Click on the "Next" button.
   - Configure the target group targets:
        Select the instance where your containers are running as a target.
        Specify the target's port for each container in the filed mentioned "Ports for the selected instances"(e.g., 8081, 8082, 8083).
        Click on the "Include as pending below" button after entering port
        Repeat these steps for each container and port combination.
    Review the target group settings and click on the "Create target group" button.
    
    
> <b>Please refer the screenshot below where I Created my Target Group</b>
    
![alt text](https://i.ibb.co/26fSKnV/git-tg.png)

### Step 4 - Create the Application Load Balancer

1. Create ALB 

  - Go to the EC2 service in the AWS Management Console.
  - Select "Load Balancers" from the left navigation pane.
  - Click "Create Load Balancer".
  - Select "Application Load Balancer".
  - Configure the load balancer settings such as name, scheme (internet-facing or internal), and IP address type.
  - Select the appropriate VPC and subnets for the load balancer.
  - Configure the security settings by selecting existing security groups or creating new ones.
  - Enable or disable deletion protection as desired.
  - Add tags for better organization and management.
    
2. Configure the listener and ACM certificate:

  - Configure the listener protocol and port(HTTPS 443).
  -  Select the target group you created in step 1 as the default action.
  - Click "Add listener" to add the listener.
  - In the "SSL certificate" section, select "Choose a certificate from ACM.
  - Create LoadBalancer
  - Once create, add a listener HTTP:80 where default action is to redirect all HTTP requests to HTTPS.

> <b>Please refer the screenshot below where I Created my ALB</b>
    
![alt text](https://i.ibb.co/gmpnBxJ/git-alb.png)
   
### Step 5 - Configure Route53 Record

- Open the AWS Management Console and navigate to the Route 53 service.
- In the navigation pane, click on "Hosted zones".
- Select the hosted zone for the domain "ashna.online" or create a new one if it doesn't exist.
- Click on "Create Record Set".
- In the "Name" field, enter "docker.ashna.online".
- Set the Alias to desired ALB

### Result

While we load the domain name, ALB manages the incoming traffic and distributes it among the containers and as a result we get three websites in preferred algorithm.
You can see the screenshots below where i got three different websites just by calling **docker.ashna.online**

![alt text](https://i.ibb.co/GVykj4H/git-alb-website3.png)

![alt text](https://i.ibb.co/80dJ0FT/git-alb-website1.png)

