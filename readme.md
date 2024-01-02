## Backup and Restore postgres database running in a container

### 1. Backup the database 
  1. Go inside the container with `docker exec -it <database container id here> sh`

  2. Now you need to backup the db, since we already have the __db_backups__ folder outside the container and we have mapped it as volume in docker-compose, we can use __pg_dump__ command here.
  Enter the below command to output a .tar file as backup to db_backups folder. 
  `pg_dump -U listmonk -W -F t postgres > /db_backups/listmonk.tar `

  Your backups can be accessed on ___db_backups___ folder outside the container.

 ### 2. Restore the database
  1. To restore the db enter the psql environment following above step 1 and 2

  2. On the terminal, hit the below command
  `pg_restore -U listmonk -C -d postgres /db_backups/listmonk.tar`
  3. If successful, do restart all the container with
  `docker compose down; docker compose up -d`
