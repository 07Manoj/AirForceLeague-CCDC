**Getting Started with deploying Docker Containers using Ansible**

We are going to create a playbook that would perform the following tasks
- Deploy a postgres database container 
- Deploy a web server from a pre configured Django Image

**Step 1:**  Create a new playbook where we are going to write tasks which deploys containers

```
nano deploy.yml
```
**Step 2:** Deploy a database container by pulling the image and specifying the environment variables. Copy the code below into the deploy.yml file.

```

- hosts: all
  tasks:

#Deploying the database container with environment variables and a named volume.

    - name: Deploying the database container
      become_user: ansible-admin
      docker_container:
        name: db
        hostname: db
        image: postgres:latest
        pull: true
        state: started
        networks:
          - name: website-network
        volumes:
          - db_data:/var/lib/postgresql/data
        env:
          POSTGRES_DB: "postgres"
          POSTGRES_USER: "postgres"
          POSTGRES_PASSWORD: "postgres"
```

- **"Hosts all"** specifies that you want to run this playbook on all hosts and the tasks are as follows.
- **become_user:** will deploy the container as ansible-admin. You do not need this module since we are deploying everything as ansible admin
- **docker_container:** is a built-in ansible module that helps you manage docker containers. We are using this to deploy the docker container 
- **name:** is the name of the docker container 
- **hostname:**  is the hostname of the docker container
- **image:** is basically referring to an image and the tag.
- **pull:true:**  This pulls the image from docker hub. If you do not have the image on the remote host, you will have to set this to true.
- **state:started:** This will be the state of the container once it is deployed. Ansible will ensure the container is started.
- **networks:** This will  attach the docker container to the specified network
- **volumes:**  We are using a named volume in this case. By default, docker stores all the data in the container itself. The data stored in the container would be gone if the container does not exist anymore and hence we use volumes which bind a location on the docker host to the location in the container. By default, docker stores the volume data inn *" /var/lib/docker/volumes"* when you use a named volume. Any changes made in the host machine would reflect in the container and vice-versa.
- **env:** We are defining the environment variables that the image would need. Postgres needs the mentioned parameters and hence we are defining them


**Step 3:** Deploying the Django Containers  

```
# Deploying two Django Web containers to solve for load balancing using haproxy. All of them are on t>
# ports are mapped to the host to access the webpapge.


    - name: Deploying the first django web container onto the Ubuntu Machine
      docker_container:
        image: sh0rtcyb3r/ccdc23_af_django
        name: django-site
        pull: true
        state: started
        published_ports:
          - "8000:8000"
        networks:
          - name: website-network
        env:
          POSTGRES_DB: "postgres"
          POSTGRES_USER: "postgres"
          POSTGRES_PASSWORD: "postgres"
        links:
          - db

  - name: Deploying the second django web container onto the Ubuntu Machine
      docker_container:
        image: sh0rtcyb3r/ccdc23_af_django
        name: django-site1
        pull: true
        state: started
        published_ports:
          - "8001:8000"
        networks:
          - name: website-network
        env:
          POSTGRES_DB: "postgres"
          POSTGRES_USER: "postgres"
          POSTGRES_PASSWORD: "postgres"
        links:
          - db

```




- **published_ports:** This esentially maps the host ports to the ports on the container. The django container by default listens on port 8000 and hence we are mapping port 8000 and 8001 on the host machine to port 8000 on the django containers.  To access the web page hosted on the django containers, we can do the following -

```
localhost:8000 or ipaddress:8000

```


- **links:** This is used to share environment variables between two containers. The django container needs to connect with the database and the database needs the environment variables to connect and hence we use link to share the variables.

We now have a web-server running on ports 8000 and 8001.

HTTP is a protocol which is un-encrypted and is vulnerable. Navigate to Navigate to [Vulnerability](Vulnerability.md) to see how it can be exploited.