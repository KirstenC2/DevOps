# Gitlab Project Test

## Create a git project

## SSH Key Configuration
![alt text](images/ssh_key_page.png)

First we have to add the client's public key to this gitlab

Fill in below as required.
![alt text](images/ssh-key-fill-in.png)


When the SSH key is being added to Gitlab successfully.
![alt text](images/ssh-key-added.png)

Now your client will be able to clone the project.
![alt text](images/clone-result.png)

After created a sample bash script. push it to the gitlab.

Success case
![alt text](images/success-pipeline.png)

Failed case
![alt text](images/failed-pipeline.png)

Further on, this can have another feature such as notification, and logging, running npm applications.

based on the runner's executor being chosen.