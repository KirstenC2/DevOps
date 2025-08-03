# Gitlab Runner

## Register runner for gitlab
command:
```
docker exec -it gitlab-runner gitlab-runner register
```
** 
Before running this, you should get the registration token first.
**
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



![/images/create-runner-token.png](/images/create-runner-token.png)

##### Get your token here
![alt text](/images/token.png)

This would be the runner created
![alt text](/images/runner-created.png)

then you have done the setup of runners for gitlab ci to run!