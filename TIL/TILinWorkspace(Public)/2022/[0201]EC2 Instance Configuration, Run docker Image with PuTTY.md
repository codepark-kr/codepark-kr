## EC2 Instance Configuration, Run docker Image with PuTTY
(repo. [Zwillie-API-SERVER](https://github.com/codepark-kr/Zwillie-API-SERVER))  
`last update: 2022.02.01.`

---
## Introduction
Document how to configure Timezone install Java, Change hostname for new EC2 Instance with PuTTY.

## Explanation
> ### Allocate Elastic IP to new EC2 Instance
> ![image](https://user-images.githubusercontent.com/68538569/151949425-d5b5110b-d2f1-4031-a87e-3fa24a72b360.png)  
> ![image](https://user-images.githubusercontent.com/68538569/151949633-361e7c42-977c-4bc0-b1f0-17468ca55fd4.png)  
> 1\. Click to Allocate Elastic IP address in EC2 > Elastic IPs.
>
> ![image](https://user-images.githubusercontent.com/68538569/151949704-e364ea90-96b5-4e47-aa51-7f3764273667.png)  
> 2\. Press Allocate for generate new Elastic IP.
>
> ![image](https://user-images.githubusercontent.com/68538569/151949830-5694a842-2623-410c-8a3f-bc802fed90b7.png)  
> ![image](https://user-images.githubusercontent.com/68538569/151950051-c9bf35bd-4f3c-4264-9430-7010927a754d.png)  
> 3\. Check the new Elastic IP and click to Actions > Associate Elastic IP address. Choose Instance, Fill up the blanks, then press `Associate`.
>
> ![image](https://user-images.githubusercontent.com/68538569/151951075-0d980d70-517c-4f73-bf0b-68dfb81f0098.png)  
> 4\. At last, copy the Allocated IPv4 address and change to Host Name in PuTTY.



> ### Install Java 11 to EC2 Instance with PuTTY
>
> 1\. Execute this following commands to install Java 11:  
> `sudo apt-get update`  
> `sudo apt-get install openjdk-11-jre-headless`
>
> 2\. Execute this following commands to Change the Timezone.(omissible) The default timezone is UTC.  
> `sudo rm /etc/localtime`  
> `sudo ln -s /usr/share/zoneinfo/Asia/Seoul /etc/localtime`  
> `date`
>
> 3\. Change hostname  
> `sudo vim /etc/hosts`  
> `127.0.0.1 localhost {HOSTNAME-WHAT-YOU-WANT}`  
> `sudo hostnamectl set-hostname {HOSTNAME-WHAT-YOU-WANT}`  
> `hostname`
>
> ![image](https://user-images.githubusercontent.com/68538569/151944131-acee5a2c-af96-4338-a7e7-c6ca0f46b1ae.png)  
> 4\. That's all!


> ### Create new Docker Container
>
> ![image](https://user-images.githubusercontent.com/68538569/151955907-c99a7e26-0816-4cf2-97ce-0448193b1a9b.png)  
> 1\. Execute this following commands in PuTTY :  
> `sudo chmod 666 /var/run/docker.sock`  
> `docker login` -> then login with username and password of Dockerhub  
> `docker run {DOCKERHUB-USERNAME}/{DOCKERHUB-REPOSITORY(IMAGE)-NAME}`
>
> ![image](https://user-images.githubusercontent.com/68538569/151957864-c6394952-b057-44fd-87c4-27b776b6f97a.png)  
>  2\. Done!

## Reference
[How do I assign a static hostname to an Amazon EC2 instance running Ubuntu Linux?](https://aws.amazon.com/premiumsupport/knowledge-center/linux-static-hostname/?nc1=h_ls)  
[https://github.com/occidere/TIL/issues/116](https://github.com/occidere/TIL/issues/116)