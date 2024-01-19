## Latest listmonk instance with already backed up data
1. First start the db container with `docker compose up db -d`.
2. Setup this new db for listmonk `docker compose run --rm app ./listmonk --install`.
3. Go inside db container, restore the db with pg_restore 
```
docker exec -it $(docker compose ps -q db) sh
export PGUSER=listmonk
pg_restore -d listmonk -c /db_backups/listmonk.tar
exit
```
### __If the backup and restore is done from the current version then no need upgrade db again.__

4. Since the pg_dump may be done from older version so upgrade the db to latest version now
```
docker compose run --rm app ./listmonk --upgrade
```

After successful upgrade, do backup the db again for future upgrade

## Backup and Restore postgres database running in a container

### 1. Backup the database 
  1. Go inside the container with `docker exec -it $(docker compose ps -q db) sh`
  
  2. Export default user `export PGUSER=listmonk` to avoid hitting ___-U listmonk___ in every postgres command

  2. Now you need to backup the db, since we already have the __db_backups__ folder outside the container and we have mapped it as volume in docker-compose, we can use __pg_dump__ command here.
  Enter the below command to output a .tar file as backup to db_backups folder. 
  ```
  pg_dump -F t listmonk > /db_backups/listmonk.tar
  ```
  Here is the oneliner command for the above steps.
  ```bash
  docker exec -it $(docker compose ps -q db) sh -c "export PGUSER=listmonk && pg_dump -F t listmonk > /db_backups/listmonk.tar"
  ```
  If password is asked provide "9988".

  Your backups can be accessed on ___db_backups___ folder outside the container.

 ### 2. Restore the database
  1. To restore the db enter the db container following above step 1

  2. On the terminal, hit the below command
  ```
  pg_restore -d listmonk -c /db_backups/listmonk.tar
  ```

  Here is the oneliner code for above restore steps
  ```bash
  docker exec -it $(docker compose ps -q db) sh -c "export PGUSER=listmonk && pg_restore -d listmonk -c /db_backups/listmonk.tar"
  ```

  __If you are doing for the first time, you may see a lot of erros in terminal but you can safely ignore them. ___(Your db is restored now.)_____
  If it is second then, just run once and you are fine.

  3. After successful restore, do restart all the container with
  `docker compose down; docker compose up -d`


## Automatically Backup and Commit
Automatically backup and commit is done through cronjob.
In terminal hit `crontab -c` and add the below line in it.
```bash
0 0,8,16 * * * cd /home/ubuntu/listmonk && (docker ps | grep $(docker compose ps -q db))  && docker exec $(docker compose ps -q db) sh -c "export PGUSER=listmonk && pg_dump -F t listmonk > /db_backups/listmonk.tar" && git add /db_backups/listmonk.tar && git commit -m "backup: $(git rev-parse --short HEAD)" && git push origin main
```
This command will run 3 times a day in 8 hour time difference.


 ## Change email template
 If you change any of the email template, do restart the app container service with `docker compose down; docker compose up -d`
