1. Step 1: Docker start the containers
```
docker compose up --build -d
```

After the containers are all up,

2. Step 2: 
```
docker exec gitlab cat /etc/gitlab/initial_root_password
```

Gitlab will then preset an initial password for root user.
Then this password will be used to login to the gitlab as root user.

To check the access from client to Gitlab's access:

Verify SSH Access to GitLab:
```
ssh -T git@localhost
```

** Issue 1: **
if it cannot access, normally because the client could not identify the gitlab.


this below command will write connect the localhost to identify the gitlab.
```
sudo sh -c 'echo "127.0.0.1 gitlab.example.com" >> /etc/hosts'
```


after this, the ping to gitlab.example.com should be success.
**Issue 2: publick key (permission denied)**


## check if the public key is generated
ls -l ~/.ssh/id_rsa ~/.ssh/id_rsa.pub

if not listed:
ls: /Users/kirstenchoo/.ssh/id_rsa: No such file or directory
ls: /Users/kirstenchoo/.ssh/id_rsa.pub: No such file or directory

use this command to generate the public key.
```
ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_rsa
```

Remember to keep the private key secure!

then use this to get the public key content.
```
cat ~/.ssh/id_rsa.pub
```

## if this got content return, then the public key is added successfully
ls -l ~/.ssh/id_rsa ~/.ssh/id_rsa.pub
chmod 600 ~/.ssh/id_rsa


then login to the gitlab 

### Homepage:
![/images/homepage.png](/images/homepage.png)

### Setup step 1
![/images/setup_step1.png](/images/setup_step1.png)

### Setup step 2
![/images/setup_step2.png](/images/setup_step2.png)

there are two methods to create runner.
- registration token
- create on gitlab UI

#### Method: create on gitlab UI 
![/images/setup_step2.png](/images/create-runner-on-ui.png)

#### Method: registration token

command:
```
docker exec -it gitlab-runner gitlab-runner register
```

![/images/create-runner-token.png](/images/create-runner-token.png)

##### Get your token here
![alt text](/images/token.png)

This would be the runner created
![alt text](/images/runner-created.png)

then you have done the setup of runners for gitlab ci to run!