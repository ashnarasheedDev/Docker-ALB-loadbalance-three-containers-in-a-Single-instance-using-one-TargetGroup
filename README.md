# Docker-ALB-loadbalance-three-containers-in-a-Single-instance-using-one-TargetGroup
In most load balancing scenarios, we have seen each container is typically deployed on a separate instance. What If we have three containers running on a single EC2 instance, Is it still possible to load balance them using a single target group and an Application Load Balancer (ALB)? 

**Let's see how i create this setup:**

I'm creating three containers from httpd:alpine image and deploying three websites on each container. Each container will be mapped to a different port on the host system: 8081 for website1, 8082 for website2, and 8083 for website3.

To distinguish the websites when they are loaded, we have edited the index.html file of each website's templates so as it prints website1, website2 and website3 accordingly.

Create a target group: Configure the target group with the appropriate settings, such as the target type (instance or IP), health checks, and protocol. Add targets to the target group

Create an Application Load Balancer (ALB): Configure it with the desired settings, such as listeners, security groups, and subnets.

Add the target group to the ALB: During the creation of the ALB, specify the target group you created earlier and associate it with the appropriate listener and port.

Add a route53 record for a domain so a sit Alias to ALB'S Ip.

**Here is a simple Block diagram illustrates the setup**

![alt text](https://i.ibb.co/yRDN33M/git-alb.png)

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

