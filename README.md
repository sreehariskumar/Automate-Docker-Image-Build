# Automate-Docker-Image-Build (Jenkins & Ansible)

![docker-git-ansible drawio](https://user-images.githubusercontent.com/68052722/222080242-7623f7fa-3f5c-4eb8-a6b4-66a4fd04b1ad.png)


In this article, we will use Ansible to automate the Docker image build process on every commit on the GitHub repo. We will use a playbook that will build a Docker image from a Flask app repository, push it to Docker Hub, and run it on a test server.

### Set Up the Environment
- A Git repository containing the code.
- An Ansible control node with Jenkins [installed](https://www.jenkins.io/doc/tutorials/tutorial-for-installing-jenkins-on-AWS/) and [configured](https://plugins.jenkins.io/ansible/).
- A Docker Hub account.

We will create a deployment directory and move all the necessary files.
Remember to change the ownership of the directory to “jenkins” as the jenkins process will be run under the privilege of this user, making it easier for the user to access the files.

```s
sudo mkdir /var/deploy
sudo mv main.yml dockerhub_creds.yml aws.pem hosts /var/deploy/
sudo chown -R jenkins:jenkins /var/deploy/
```

### Create the Ansible Playbook

The playbook is organized into two plays, each of which targets a different set of hosts. The first play targets the “build” hosts and contains tasks to build the Docker image and push it to Docker Hub. The second play targets the “test” hosts and contains tasks to pull the image from Docker Hub and run it in a container.

**Play 1: Building Docker Image and Pushing to Docker Hub**
- **installing packages**: installs required packages such as git, pip, and docker.
- **adding ec2-user to docker group**: adds the ec2-user to the docker group, which grants permission to use Docker without sudo.
- **installing python extension for docker**: installs the docker-py Python library, which allows Ansible to interact with Docker.
- **restarting & enabling docker service**: restarts the Docker service and ensures it is set to start on boot.
- **creating cloning repo**: creates a directory to store the cloned repository.
- **cloning from repo**: clones the GitHub repository specified in repo_url to the clone_dir.
login to docker hub: logs in to Docker Hub using the credentials specified in the dockerhub_creds.yml file.
building docker image and pushing image to dockerhub: builds a Docker image from the cloned repository and pushes it to Docker Hub with the specified name and tags.
- **logout from dockerhub**: logs out of Docker Hub to prevent unauthorized access.

**Play 2: Running Docker Image on Test Server**
- **installing packages**: installs required packages such as docker and pip.
- **adding ec2-user to docker group**: adds the ec2-user to the docker group, which grants permission to use Docker without sudo.
- **installing python extension for docker**: installs the docker-py Python library, which allows Ansible to interact with Docker.
- **restarting & enabling docker service**: restarts the Docker service and ensures it is set to start on boot.
- **pulling docker image**: pulls the Docker image from Docker Hub with the specified name and tag.
- **running container**: runs the Docker image in a container with the specified port mappings


### Integrate with Git
Now that we have created the Ansible playbook, we need to integrate it with Git to automate the Docker image build process on every commit.

To do this, we will use a Git webhook that will trigger the Ansible playbook whenever a commit is made to the Git repository. We will use the ansible-playbook command to run the playbook on the Ansible control node.

To set up the webhook, we need to create a new webhook on the Git repository settings page. We will configure the webhook to trigger on push events, and we will provide the URL of the Ansible control node along with the path to the Ansible playbook.

- Go to the settings page of your GitHub project and select “Webhook” option. Now add a new webhook with the public IP of the Jenkins server.
_eg: http://1.2.3.4:8080/github-webhook/_

![8](https://user-images.githubusercontent.com/68052722/222048340-c7b09a8e-b6a3-42a9-8d44-2217a63bc234.png)

### Test the Automation
Finally, we need to test the automation to ensure that the Docker image is built and pushed to Docker Hub on every commit.

To test the automation, we will make a commit to the GitHub repository containing the Flask app code. We will then check Docker Hub to ensure that the Docker image has been built and pushed successfully.

![1](https://user-images.githubusercontent.com/68052722/222048455-1ca7be87-de42-4d06-9dcd-286c3993da96.png)

![2](https://user-images.githubusercontent.com/68052722/222048469-92c528fd-947d-4fae-bb64-150d4636f550.png)

![3](https://user-images.githubusercontent.com/68052722/222048521-e6398352-f29f-4db2-87b7-424d72d7a529.png)

![4](https://user-images.githubusercontent.com/68052722/222048538-54c9ca86-9d05-4c63-adc3-1f607ce1925b.png)

![6](https://user-images.githubusercontent.com/68052722/222048555-53c8bbb8-f671-4820-b7a5-b8f8151a2cdc.png)

![7](https://user-images.githubusercontent.com/68052722/222048565-85d58eef-399d-4ec1-afc1-c431e958d58d.png)

![WhatsApp Image 2023-02-24 at 7 00 54 PM (1)](https://user-images.githubusercontent.com/68052722/222048964-7820710b-c4ad-4ab5-92e4-a03f9e630c6d.jpeg)

![WhatsApp Image 2023-02-24 at 6 59 44 PM](https://user-images.githubusercontent.com/68052722/222048976-63bafb15-2862-484a-9b6b-184dfb0df8d5.jpeg)




### Conclusion

By automating the Docker image build process, we can ensure that our applications are always up-to-date and consistent across all environments. It also allows us to focus on developing and improving our applications without worrying about the deployment process.
