# Blueprint Staging

## Logging into Blueprint Staging Server

One ssh using a username and the IP address. We are using the id_ed25519 (private) key. The final command is: `ssh username@ipaddr -i .ssh/id_ed25519`. -i is a flag that means identity file and it chooses the private SSH key located within a given directory.

## Getting Admin Access

Once in, let's do `su -`. Just having a single dash is the same as -l or -login. It "provides a environment similar to if a user logged in directly." `su` allows users to switch to root acount and perform admin tasks. `sudo` is for users who execute specific commands with elevated privileges. su is very limited in scope.

## Getting to Docker User

`su - docker`. As we already logged in to root, we will be able to switch to the docker user withotu much issue.

## Getting Into the Thick of It

Let's use http (can't edit) to clone the admin_backend repo. Comparing to the blueprint_admin and c4p docker-compose.yaml files, we will add things.

### Original docker-compose.yaml

`
version: '3.7'
services:
  postgres:
    image: postgres:10.5
    restart: always
    environment:
      - POSTGRES_USER=blueprint_admin_backend
      - POSTGRES_PASSWORD=postgres
      - POSTGRES_DB=postgres
    logging:
      options:
        max-size: 10m
        max-file: "3"
    ports:
      - '5432:5432'
    volumes:
      - ./postgres-data:/var/lib/postgresql/data
`

### New Code

`
version: '3.7'
services:
  admin-backend-db:
    image: postgres:10.5
    restart: always
    environment:
      - POSTGRES_USER=blueprint_admin_backend
      - POSTGRES_PASSWORD=postgres
      - POSTGRES_DB=postgres
    networks:
      admin-backend:
        ipv4_address: 10.10.4.2
  admin-backend-app:
    networks:
      admin-backend:
        ipv4_address: 10.10.4.3
    image: ghcr.io/stevensblueprint/blueprint_admin_backend:latest
    depends_on:
      - admin-backend-db
    env_file:
      - .env

networks:
  admin-backend:
  driver: bridge
  ipam:
    config:
      - subnet: 10.10.4.0/24
        gateway: 10.10.4.1
`

Some notes: There were issues with using postgres when composing the dockerfile. Just do not mention any services with postgres. Logging and ports are not needed in this time. The networks element is defined at the bottom. We call it admin-backend. Driver specifies which driver is used for the network. [Bridge](https://docs.docker.com/network/drivers/bridge/) allows containers to be connected to the same network but provides isolation from other containers by enforcing rules that prevent comms between containers. The containers must have the same Docker daemon host. ipam stands for IP address management. We can then config the subnet and gateway we make up. blueprint_admin uses 10.10.3... so we can increment that by  1 10.10.4... IP addresses begininng with 10 are private or local. /24 is the CIDR notation or short for the subnet mask. 24 means that it is is equivalent to 255.255.255.0 submask which converted from decimal to binary is 24 1/"on"-bits.

## Running the compose file.

`docker compose up`. There is no .env file. Let's create one (it can be blank). `touch .env`. Let's re-run the command.

## Errors

If the docker compose is going wrong, exit the docker user and head back to root. Let's see what process to kill:

`ps aux | grep docker`. ps displays the active processes. a shows processes for all users, u displays process's user/owner, and x shows processes not attached to a terminal. grep searches for patterns in the ps. Ok, I see the docker process. `kill -9 <pid>`. kill sends a signal to the process. -9 sends an exit signal to the pid.
