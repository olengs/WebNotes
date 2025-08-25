postgres (with docker)

docker version
# You should see a version output. This may be large.
docker compose version
# Again, you should see a version. This will likely be a shorter output.

code docker-compose.yml
#creates docker config file

version: "3.9"
services: 
  # Our Postgres database
  db: # The service will be named db.
    image: postgres # The postgres image will be used
    restart: always # Always try to restart if this stops running
    environment: # Provide environment variables
      POSTGRES_USER: baloo # POSTGRES_USER env var w/ value baloo, username
      POSTGRES_PASSWORD: junglebook #password
      POSTGRES_DB: lenslocked #db name
    ports: # Expose ports so that apps not running via docker compose can connect to them.
      - 5432:5432 # format here is "port on our machine":"port on container"
      #port on machine can change to 5433 if postgres already exists on machine
  # Adminer provides a nice little web UI to connect to databases
  adminer:
    image: adminer
    restart: always
    environment:
      ADMINER_DESIGN: dracula # Pick a theme - https://github.com/vrana/adminer/tree/master/desports:
    ports: 
    - 3333:8080 #our port 3333 to adminer port 8080
    # you can run http://localhost:3333/ to view adminer website

# note that all indents are important!!!

docker compose up
# if works u should see that both postgres and adminer is running on docker.

docker compose up -d
# This command runs docker in detached mode, so you can do other things while running docker

docker compose stop
# To stop running the docker container

docker compose down
# To clear everything in docker

docker ps
# To check docker status

docker compose exec -it db psql -U baloo -d lenslocked
# To run sql (note: db => service name, can also be replace by NAME of container, -d is database, where args is lenslocked => accessing lenslocked db (under POSTGRES_DB))
docker compose exec -it lenslocked-db-1 psql -U baloo -d lenslocked
# same as above, just using image name, in case you have multiple same name



