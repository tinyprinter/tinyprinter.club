---
title: "Sirius Server Troubleshooting"
date: 2020-05-11
draft: false
description: "Make your own Little Printer!"
ogimage: "littleprinter_table.jpg"
---

Making a (fake) printer doesn't always work as intended. Here are some errors folks have run into, and how to fix them. 

If you run into an error that isn't listed here, please reach out!


## FATAL:  database "sirius" does not exist

Sometimes the server doesn't set up its database correctly, so it will need to be manually created. 

If you're not already connected to the container, run `docker-compose exec sirius bash`, first. 

From here we need to connect to the Postgres server. When prompted, enter `plop` for the password. 

```
psql -h sirius-database -U sirius-dev   # Connect to the server
```

Create the database:
```
CREATE DATABASE sirius;                 # Create the missing database
quit;                                   # Disconect from the server
```

Next we need to run the database migrations and create the printer

```
./manage.py db upgrade                  # Run the migration
./manage.py fake printer                # Create the printer
exit                                    # Exit from the Docker container
```