## Backup and Restore postgres database running in a container

### 1. Backup the database 
  1. Go inside the container with `docker exec -it <database container id here> sh`
  
  2. Export default user `export PGUSER=listmonk` to avoid hitting ___-U listmonk___ in every postgres command

  2. Now you need to backup the db, since we already have the __db_backups__ folder outside the container and we have mapped it as volume in docker-compose, we can use __pg_dump__ command here.
  Enter the below command to output a .tar file as backup to db_backups folder. 
  `pg_dump -W -F t listmonk > /db_backups/listmonk.tar`

  Your backups can be accessed on ___db_backups___ folder outside the container.

 ### 2. Restore the database
  1. To restore the db enter the db container following above step 1

  2. On the terminal, hit the below command
  `pg_restore -d listmonk -c /db_backups/listmonk.tar`
  __If you are doing for the first time, you may see a lot of erros in terminal but you can safely ignore them. ___(Your db is restored now.)_____
  If it is second then, just run once and you are fine.

  3. After successful restore, do restart all the container with
  `docker compose down; docker compose up -d`


 ## Change email template
 If you change any of the email template, do restart the app container service with `docker-compose up --force-recreate -d app`
