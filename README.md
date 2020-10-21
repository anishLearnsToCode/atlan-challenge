# Real Time Task Management API

This repository contains teh solution for the Atlan Internshop challenge. This project provides the user with a mechanism for creating tasks and pausing, resuming and stopping them as well.

## 📖 Overview
1. [Inroduction](#introduction)
1. [Running It On Your Machine](#running-it-on-your-machine)
1. [Testing The API](#testing-the-api)
1. [Viewing The Changes Made In The Database](#viewing-the-changes-made-in-the-database)


## Running It On Your Machine
Clone this on your machine
```bash
git clone --edit this madan--
cd atlan-challenge (you can change name of repo if you want madan, then change this as well)
```

Build the Docker Image and run the server
```bash
docker-compose up --build
```

You can now open the web app on your machine on port 3000 at [http://localhost:3000](http://localhost:3000). On the wenb page you can start a modification request which will change the dates for all actors in the databases and this transaction will begin in excatly 1 second (--see this madan--), which allows you (the user) to test the pause/resume functionality.

The solution uses worker threads and transactions. Every time a submit request is made, a new worker thread is created to run the query. The query will run in a transaction. 

Once the query completes or fails, the query is cancelled and the worker thread dies.

The query runs inside a transaction to keep the update atomic. 
- If user pauses the update, the connection in that new thread is paused.
- If the user then clicks on resume, that connection can now be resumed.
- If user cancels, the connection will be destroyed, and then worker thread will be killed. 
    (In such a case, the transaction will anyway be rolled back by the ```nodejs-mysql``` module, so partial update isn't an issue here)
- If the user does not pause, cancel or resume the transaction, and the query successfully completes, it will be committed to the database.
 
 ## Testing the API
The server uses a `SLEEP(1000)` query before and after the `UPDATE` query in the transaction in the `getDataForDateRange()` function, to give time to test the pause, resume and cancel features. 

- These features can be checked by observing that if pause/resume/cancel are pressed- even after more than 20 seconds (enough time for the entire transaction to have run), the logs would not show a successfully committed message. 
- For resume, the transaction would continue, and the changes are successfully committed- unless it was a cancel operation, in which case it would have rolled back and not displayed successful commit message.


## Viewing The Changes Made In The Database
After creating the containers, use `docker exec -it mysql-db-container mysql -u root -p` and enter the password __password@root123__.

> The password for test purposes has been selected as __password@root123__

Then run query `SELECT * FROM sakila.customers` in the sakila db to check the `active` field to check if the changes have been reflected in the database.
