## How to connect an EC2 Instance Server to Putty, Install docker image
(repo. [Zwillie-API-SERVER](https://github.com/codepark-kr/Zwillie-API-SERVER))  
`last update: 2022.02.01.`

---
## Introduction
Document how to connect the server based on Amazon EC2 to Putty and download image from Docker-hub.

## Explanation
> ### Generate PuTTY private key (ppk) for connect
> ![image](https://user-images.githubusercontent.com/68538569/151928165-d7348180-e3f8-4f2b-8090-ba620f522b35.png)  
> 1\. Install PuTTY and PuTTYgen, execute PuTTYgen first.
>
> ![image](https://user-images.githubusercontent.com/68538569/151928393-d5f773c4-4594-4cff-ad50-79fead2881c1.png)  
> 2. Press `Load` in PuTTYgen, change to browsefile option to `All files(*.*)` then select pem key from Amazon AWS.
>
> ![image](https://user-images.githubusercontent.com/68538569/151928612-0fd575b1-a313-4634-9b84-5b38edffc0c9.png)  
> 3. Check the notice, press OK.
>
> ![image](https://user-images.githubusercontent.com/68538569/151928767-61079730-acdd-47d1-83fa-b4fea81a6997.png)  
> 4. Press Save private key, press Yes when PuTTYgen Warning appeared.
>
> ![image](https://user-images.githubusercontent.com/68538569/151928932-6c60d43e-675f-420c-904e-10b3f9cd405f.png)  
> 5. At last, save the new ppk key(PuTTY Private Key Files) with any key name what you want.

> ### Connect Amazon EC2 Instance to PuTTY
>
> ![image](https://user-images.githubusercontent.com/68538569/151932370-767f1444-9ce3-485d-b153-c2113ed244c9.png)    
> ![image](https://user-images.githubusercontent.com/68538569/151932256-137b4b7c-aa04-46b3-bcbe-56b5afe29ddd.png)    
> 1\. Open PuTTY, fill the blank `HostName(or IP address)` with public IP address of EC2 Instance. Then enter the Session name what you want, press `Save`.
>
> ![image](https://user-images.githubusercontent.com/68538569/151932905-e3ff8818-f2c6-49e4-af3c-dcf717cb4c1f.png)  
> 2. In category `SSH > Auth`, press `Browse` and select the ppk file that generated with PuTTYgen.
>
> 3\. Back to `Session`, press save and Open.
>
> ![image](https://user-images.githubusercontent.com/68538569/151933229-8f78d8ec-2c52-4357-b379-cd5b26a60bde.png)  
> 4. If you choose Ubuntu AMI when launch an new Instance in Amazon AWS, login as `ubuntu`.

> ### Install docker, get docker image to EC2 server with putty
>
> ![image](https://user-images.githubusercontent.com/68538569/151935658-c4c340a1-5165-4357-9a70-c5db45416bcf.png)  
> 1. After login execute following commands to PuTTY for install Docker :  
     > `sudo apt-get update`  
     > `sudo apt install apt-transport-https`  
     > `sudo apt install ca-certificates`  
     > `sudo apt install curl`  
     > `sudo apt install software-properties-common`  
     > `curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -`  
     > `sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu bionic stable"`  
     > `sudo apt update`  
     > `sudo apt install docker-ce`  
     > `sudo systemctl start docker`
>
> ![image](https://user-images.githubusercontent.com/68538569/151936178-9fda9d6a-fd47-4643-90f1-2ed9a3ade535.png)  
> 2. Pull the docker image from Dockerhub with this command :  
     > `sudo docker pull {DOCKER-USERNAME}/{DOCKER-REPOSITORY-NAME}:{TAG-NAME}`
>
> 3\. That's all!


## Reference
[https://godbell.tistory.com/37](https://godbell.tistory.com/37)
