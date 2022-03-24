## Generate Dockerfile and Image, create an EC2 Instance for API SERVER
(repo. [Zwillie-API-SERVER](https://github.com/codepark-kr/Zwillie-API-SERVER))  
`last update: 2022.02.01.`

---
## Introduction
Document how to generate Docker Image, Dockerfile, AWS EC2 Instance for API Server.

## Explanation
> ### Generate Dockerfile, Docker Image for project
> ![image](https://user-images.githubusercontent.com/68538569/151923038-d9f59f57-fdad-4545-b91f-bd7075c4c662.png)    
> 1\. Generate Dockerfile in root path of the project. The image above is for only basic of basic settings. `EXPOSE {PORT-NUMBER}` is omissible. **The files name must be `Dockerfile`. The first letter `D` must be Uppercase, `f` must be lowercase.** If the naming rule is not followed, it cause an error : `failed to solve with frontend dockerfile.v0:`.
>
> ![image](https://user-images.githubusercontent.com/68538569/151924570-994be893-bbee-4263-a4b4-6100065976b3.png)  
> 2. Execute `gradle build` and `gradle bootjar`.
>
> ![image](https://user-images.githubusercontent.com/68538569/151924966-abc5fd7f-f517-4d37-b055-c2bf46345313.png)  
> 3. Check the jar file successfully generated.
>
> ![image](https://user-images.githubusercontent.com/68538569/151925252-d5b867cc-7715-495e-89eb-ce72a881afd1.png)  
> 4. Create new Repository in Dockerhub.
>
> ![image](https://user-images.githubusercontent.com/68538569/151925694-a153b49e-5ca5-43d3-8fdb-9d03193e4ed6.png)  
> 5. Execute command `docker build --build-arg DEPENDENCY=build/dependency -t {DOCKER-HUB-USERNAME}/{DOCKERHUB-REPOSITORY-NAME} .` in terminal. **Don't forget to type `.` as last character!**
>
> ![image](https://user-images.githubusercontent.com/68538569/151926094-85a64b77-dad1-4891-9887-afcaafb4bda0.png)    
> ![image](https://user-images.githubusercontent.com/68538569/151926224-8a164708-08e3-4e5d-81d5-022e28b666ee.png)  
> 6. Done!

> ### Create an EC2 Instance for API Server
> ![image](https://user-images.githubusercontent.com/68538569/151926355-f7d2e0b8-7f6d-419a-a6e9-aae0b8e0fc0b.png)  
> 1. Search `EC2` in Amazon AWS, press `Launch Instances`.
>
> ![image](https://user-images.githubusercontent.com/68538569/151926511-54e0286a-9e0c-40eb-a535-2e73437d21ce.png)  
> 2. Select AMI depend on your OS. In my case, I selected to Ubuntu 20.04 LTS.
>
> ![image](https://user-images.githubusercontent.com/68538569/151926956-9dd999c9-7feb-425f-b368-19070b18620a.png)  
> 3. Select `t2.micro` as Instance type and configure instance and security group, add storage, tags. (omissible)
>
> ![image](https://user-images.githubusercontent.com/68538569/151927149-3e1a559e-2c63-4194-979a-93cbf3a63a41.png)  
> 4. Select an existing key pair or create new key, then press Launch Instances.
>
> ![image](https://user-images.githubusercontent.com/68538569/151927505-f97a998d-337c-4250-b091-ebf1688697c4.png)  
> 5. Super easy.


## Reference
[https://sas-study.tistory.com/399](https://sas-study.tistory.com/399)
