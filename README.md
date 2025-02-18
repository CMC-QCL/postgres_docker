# Instructions for PostgreSQL Docker Container

NOTE: If you have already set up docker container, you can skip to Section 2.

## 1. Rootless Docker and `docker-compose.yml` Settings

Docker containers have been utilized extensively on numerous applications as they allow efficient resource management and rapid testing and deployments. As Data Science Capstone project data are not meant to be stored on long-term basis, we are providing this manual to take advantage of dockerized postgreSQL environment and effectively manage your clients' data.

Originally, Docker daemon had to be started by root user, exposing possible attacks and unintended root access even to unprivileged users. Recently, Docker added rootless mode, which creates a separate instance of Docker daemon for unprivileged users and safeguards from the above risks. As CMC DS Capstone projects do not require root access to QCL GPU machine, we have set up rootless Docker daemon to your account. For more detailed information on rootless Docker, please check [this link](https://docs.docker.com/engine/security/rootless/) or reach out to [Samuel (QCL Graduate Fellow)](mailto:slee19@students.cmc.edu).

To set up PostgreSQL docker container, we have prepared `docker-compose.yml` file, which allows us to configure the docker container and start services with a single command. To learn more about Docker Compose process, you can check [this link](https://docs.docker.com/compose/). Hence, you can find `docker-compose.yml` setup right below:

```yaml
version: '3.3'
services:
  postgres:
    image: postgres
    hostname: postgres
    ports:
      - "$PORT:5432"
    environment:
      - POSTGRES_USER=$POSTGRES_USER
      - POSTGRES_PASSWORD=$POSTGRES_PASSWORD
      - POSTGRES_DB=$POSTGRES_DB
    volumes:
      - ./postgres/postgres.conf:/usr/local/etc/postgres/postgres.conf
      - ./postgres/data:/var/lib/postgresql/data
    restart: unless-stopped
volumes:
  postgres-data:
```

When sharing `docker-compose.yml` and other relevant files to compose Docker container, it is essential to store sensitive information (e.g., account credentials) locally to ensure that the container and data within do not get compromised. Therefore, we have stored porting and relevant user environment information on a `.env` file (and specified on .gitignore file so that it does not get pushed onto this repository). Hence, porting and database environment information will be provided separately.

After you have set up `.env` on current working directory, you need to create a `postgres` folder  and a `postgres.conf` file within the folder. To do so, please follow the steps below:

1. `mkdir postgres`: create a folder called `postgres`
2. `cd postgres`: change current directory to `postgres` folder
3. `vim postgres.conf`: open vim editor to create/edit file `postgres.conf`

After you have conducted the above commands, you will be on vim editor. Please insert `listen_addresses = '*'`. To insert, you first need to press `i` on your keyboard. Hence, to save and exit, type out `:wq` and press Enter.

To create and run docker container, you need to go back to the original directory and run `docker-compose` command:

1. `cd ..`: change directory back to `postgres_docker`
2. `docker-compose up -d`: run `docker-compose` to set up PostgreSQL docker container and execute in background

You can check container status by executing `docker container ps` command.

## 2. SSH Tunneling and Ports

To establish connection between the QCL GPU machine and your workstation, you need to establish SSH connection. To do so, you need to open terminal application (e.g., `Git Bash` under Git folder in startup menu):

```sh
ssh -L LOCAL_PORT:DESTINATION:DESTINATION:PORT USER_ID@SERVER_IP -p PORT_NUMBER
```
where:
- `LOCAL_PORT` specifies which port the ssh client has to use to bind on the localhost (or IP if specified)
- `DESTINATION:DESTINATION_PORT` specifies the IP or hostname and relevant port of the QCL GPU machine
- `USER_ID@SERVER_IP` is your team's User ID and IP address of the QCL GPU machine
- `PORT_NUMBER` is a specific port that the QCL GPU machine is listening to

After running the above command, you will be prompted to provide your password. If you do not see any error and have access to the QCL GPU machine terminal, SSH connection has been made between the two devices. Also, please note that you have to establish SSH connection every session.

NOTE: We will be providing the above variables in person, as these credentials are sensitive information.

## 3. PgAdmin Setup

Since you have established connection between workstation and the QCL GPU machine, we need to enter server information on pgAdmin. On pgAdmin, you need to click `Object` dropdown on menu bar and select `Create` and `Server...` to configure new server settings.

Please fill out following information on `Create-Server` window:

* General
  * `Name`: Name of the server that you would like to call
* Connection
  * `Host name/address`: localhost
  * `Port`: `LOCAL_PORT` value from Section 2 (same as `$PORT` from Section 1)
  * `Maintenance database`: `$POSTGRES_DB` value from Section 1
  * `Username`: `$POSTGRES_USER` value from Section 1
  * `Password`: `$POSGRES_PASSWORD` value from Section 1

After you have saved the above properties, you should be able to access postgreSQL server from pgAdmin client.

# Using NVIDIA GPUs with Rootless Docker

First, verify that GPUs are available by running `nvidia-smi`.

Second, pull and run NVIDIA's GPU image by running `docker run -it --gpus all nvidia/cuda:11.4.0-base-ubuntu20.04 nvidia-smi`. If this command fails, ensure that the system is not trying to use `cgroups`. To do so, add 
```
[nvidia-container-cli]
no-cgroups = true
```
to `/etc/nvidia-container-runtime/config.toml`.


Once this command runs successfully, you can use a container interactively by running `docker run -it --gpus all nvidia/cuda:11.4.0-base-ubuntu20.04 sh` or build your own image from this base to execute.
