# DockerWebserverZBC
set up a local development environment with Docker that runs an Nginx server, PHP, and a MySQL database using **docker** to define and manage the different services. 

### 1. **Install Docker and Docker Compose**
make sure Docker is running properly
```
sudo systemctl status docker
```

### 2. **Create a Project Directory**
You’ll need a directory to hold the Docker setup and project files. Open up terminal (konsole, PowerShell, or Git Bash) and create a new directory for the project:

```bash
mkdir /home/reb/Documents/Code/DockerWebserverZBC
cd /home/reb/Documents/Code/DockerWebserverZBC
```

### 3. **Create `docker-compose.yml`**
This file will define the Nginx server, PHP service, and MySQL database. Inside  `/home/reb/Documents/Code/DockerWebserverZBC` directory, create a file named `docker-compose.yml`.

Here’s a basic configuration for the setup:

```yaml
version: '3.8'

services:
  nginx:
    image: nginx:latest
    container_name: nginx_container
    ports:
      - "8080:80"  # Exposes Nginx on port 8080 of your host machine
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf  # Nginx config file
      - ./app:/var/www/html  # Your PHP application directory
    depends_on:
      - php
    networks:
      - mynetwork

  php:
    image: php:8.1-fpm
    container_name: php_container
    volumes:
      - ./app:/var/www/html  # The same app folder is shared with PHP
    networks:
      - mynetwork

  mysql:
    image: mysql:8.0
    container_name: mysql_container
    environment:
      MYSQL_ROOT_PASSWORD: root_password
      MYSQL_DATABASE: my_database
      MYSQL_USER: user
      MYSQL_PASSWORD: user_password
    volumes:
      - mysql_data:/var/lib/mysql
    ports:
      - "3306:3306"  # Exposes MySQL on port 3306 of your host machine
    networks:
      - mynetwork

networks:
  mynetwork:
    driver: bridge

volumes:
  mysql_data:
```
note: my system already has docker enabled inside of my configuration.nix file via 
```
virtualisation.docker.enable = true;
```
  Docker also reads and processes standard `docker-compose.yml` files natively.
  
###  Run the Compose file
Navigate to the folder containing the `docker-compose.yml` file in the terminal and start the containers:
```
docker compose up
```

Note: run `docker compose down` to turn off all containers

### Explanation:
- **nginx**: Runs an Nginx container and serves PHP from the `php-fpm` service.
- **php**: Runs PHP-FPM (FastCGI Process Manager), which allows Nginx to serve PHP files.
- **mysql**: Runs a MySQL container with environment variables to set the root password, a database, and user credentials.
- **volumes**: These map directories on your host machine to directories inside the container. This allows you to work with files on your host machine and have them available to Nginx and PHP. Note: save "states" can work here on this volume, otherwise my default, docker containers when restarted, have no saved files.

### 4. **Create Nginx Configuration**
You need to create an Nginx configuration file to tell Nginx how to handle PHP requests. Inside your project directory, create a folder called `nginx` and then a file named `nginx.conf`.

```bash
mkdir nginx
touch nginx/nginx.conf
```

Here’s a basic configuration to add to `nginx.conf`:

```nginx
worker_processes 1;

events { worker_connections 1024; }

http {
    server {
        listen 80;

        root /var/www/html;
        index index.php index.html;

        server_name localhost;

        location / {
            try_files $uri $uri/ =404;
        }

        location ~ \.php$ {
            include fastcgi_params;
            fastcgi_pass php:9000;
            fastcgi_index index.php;
            fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        }

        location ~ /\.ht {
            deny all;
        }
    }
}
```

### 5. **Create the PHP Application Directory**
Now, you’ll set up the PHP application directory. In your project folder, create an `app` directory and a simple PHP file to test:

```bash
mkdir app
touch app/index.php
```

Add the following content to the `index.php` file to ensure PHP is working:
(Please note that you should never expose phpinfo() on a live server, as it will output environment variables.)

```php
<?php
phpinfo();
?>
```

### 6. **Run Docker Compose**
Now that everything is set up, it’s time to start the containers using Docker Compose. From your project directory, run:

```bash
docker-compose up -d
```
Note:
The program `docker-compose` is not in your PATH by default. You can make it available in an
ephemeral shell by typing:
 ```
 nix-shell -p docker-compose
 ```
This will:
- Pull the necessary images (`nginx`, `php-fpm`, and `mysql`).
- Create and start the containers.
- Run Nginx, PHP, and MySQL as separate services.

### 7. **Access the Web Server**
Once the containers are running, open your web browser and go to:

```
http://localhost:8080
```

You should see the **PHP info page**, which indicates that Nginx is correctly serving the PHP content.

### 8. **Connect to MySQL**
To connect to MySQL, you can use any MySQL client (e.g., MySQL Workbench, phpMyAdmin, or the command line) with the following credentials:

- **Host**: `localhost`
- **Port**: `3306`
- **User**: `user`
- **Password**: `user_password`
- **Database**: `my_database`

Note: on terminal, type the following:
```
docker exec -it mysql_container mysql -u user -puser_password my_database
```

### 9. **Stop the Services**
When you’re done and want to stop the containers, simply run:

```bash
docker-compose down
```

This will stop and remove the containers, but the volumes (like the MySQL data) will persist.

### Summary of Steps:
1. Install Docker.
2. Create a project directory.
3. Define services in `docker-compose.yml`.
4. Create Nginx configuration (`nginx.conf`).
5. Create a basic PHP application (`index.php`).
6. Start the services with Docker Compose.
7. Test the web server on `localhost:8080`.
8. Connect to MySQL via any MySQL client.
9. Stop the services with `docker-compose down`.

#  Task 1: Understand & Install
Hvad er et RDBMS, og hvordan etablerer jeg det?

## Launch the database server

Your teacher has handed over the Docker setup you need (I followed his GitHUb tutorial for the Docker setup for the assigment).

Start the MariaDB server.
Then check in Docker Desktop that the container is running. 
Note:  instead of docker desktop, just do:
```
docker ps -a
```

Find:

    Name of the container

    Which port number MariaDB uses

    What image the container uses

Based on the output from my `docker ps -a` command:

* **Name of the container:** `mysql_container`
* **Port number MariaDB/MySQL uses:** Inside the container, it uses standard port **`3306`** (mapped to host port **`3307`** on my machine)
* **Image the container uses:** `mysql:8.0`

MySQL and MariaDB are fully protocol-compatible and use the exact same default port (**`3306`**).

In my setup, the container is running the official **`mysql:8.0`** image, but from the application or database client's perspective, it speaks the exact same database protocol as MariaDB.

## Connect with MySQL Workbench
Note: in my use case scenario, i am using DataGrip by Jetbrains to do this part of the assigment.
Create a new connection in MySQL Workbench.

Use the information your teacher has given you.

When the connection works, drive:

`SELECT VERSION();`

### Examination

Which MariaDB version do you run?
`8.0.46`

### 3. Examine the server

Drive:

`SHOW DATABASES;`

What databases are on the server?
```
1 | information_schema
2 | my_database
3 | performance.schema
```

Then try:

`SELECT CURRENT_USER();`

Which user are you logged in as?
`user@%`

### 4. Think about the architecture

Answer the following:

    1. Is MySQL Workbench a database server?

    2. Is MySQL Workbench a database?

    3. Where does MariaDB run?

    4. What is the workbench connected to?

    5. What does Workbench send to MariaDB?
1. no
2. no
3. MariaDB runs as a background service on a server (can be setup in systemd to auto run at PC startup/boot, same way as you would run, for example, `watchdog` or a firewall in the background, same as daemons that run invisibly in the background),  your local machine, or inside a container, such as when using Docker)
4.  The workbench connects to a running database server instance (such as a MySQL or MariaDB database server running locally, remotely, or inside a Docker container).
5.  MySQL Workbench sends SQL queries, commands, and statements (such as SELECT, INSERT, UPDATE, or SHOW commands) over the network connection to the database server.

Then draw even a small figure that shows:

Workbench → MariaDB → Database
When you're done:
![DataGrip to MySQL architecture](datagrip_mysql_architecture.png)

DataGrip is the client, the MySQL server runs in the Docker container, and my_database is the data that server manages.

Since i am using DataGrip, the connection settings are slightly different from the original assigment by the teacher, which says port 3306. In my `docker-compose.yml` the mapping is `"3307:3306"`, so from my machine i connect to `localhost` on port **3307**. Port 3306 is the port inside the container, which only other containers on `mynetwork` use. Also, the container runs `mysql:8.0`, not MariaDB. The two are protocol-compatible, so the client doesn't care, but the version i am running is MySQL 8.0.46.



You need to be able to explain the difference between:

    Database client, RDBMS/database server and database.

- **Database client**: the program you type queries into (DataGrip, Workbench). It holds no data itself. It only sends SQL and displays the results.
- **RDBMS / database server**: the running software (MySQL here) that accepts connections, parses and executes SQL, and manages access and storage. It's the process listening on port 3307.
- **Database**: the organized collection of data, here `my_database`. It's what the server stores and manages, and it lives in the `mysql_data` volume.

# Task 2 – Examine the database server
Purpose

A RDBMS should not only know your data.  It also needs to know how the database is built. This information is called metadata.

Note: An RDBMS stands for Relational Database Management System. It is a software system used to store, manage, query, and retrieve structured data organized into tables (relations) with rows and columns.
To touch on the quote, an RDBMS doesn't just hold raw data entries; it also stores the database schema (metadata, constraints, data types, and foreign key relationships) which defines how the database is structured and how different tables connect to one another. Examples include MySQL, MariaDB, PostgreSQL, and SQLite.

In this task, you need to examine how MariaDB can tell you about its own structure.

1. What databases are available?

Drive:
`SHOW DATABASES;`

Find the database:
`information_schema`

Don't change anything in this database.
It contains information about the databases.

2. Find tables

Drive:
SELECT
table_schema,
table_name
FROM information_schema.tables;

Examine the result.

You now get a list of tables that MariaDB knows.
Questions

What does the column tell `table_schema`?
What does the column tell `table_name`?
* **`table_schema`**: Tells you the name of the database (or schema) that the table belongs to (such as `information_schema` or `performance_schema`).
* **`table_name`**: Tells you the name of the specific data table itself within that database (such as `CHARACTER_SETS`, `COLUMNS`, or `TABLES`).

3. Examine a specific database
Select a database that already exists on your server.

Drive:

USE <database>;

and then:

SHOW TABLES;

Examine at least one of the tables with:

DESCRIBE <table>;
         
Note: made my own for better testing purposes:
```
SELECT VERSION();
SHOW DATABASES;
SELECT CURRENT_USER();
SHOW DATABASES;

SELECT
    table_schema,
    table_name
FROM information_schema.tables;

USE my_database;
SHOW TABLES;
CREATE TABLE users (
                       id INT AUTO_INCREMENT PRIMARY KEY,
                       name VARCHAR(50)
);

DESCRIBE users;
```
Find out:

    Which columns are available?

    What data types are used?

    Is there a primary key?

    Are some columns NOT NULL?

4. Metadata or data?

Look at the following:

student
-------------------------
id | name | age
1  | Anna | 22
2  | Peter| 25

and:

student
-------------------------
Kolonne: id
Type: INT
Primary Key: YES

Which of the two describes datadata?

Which is the type of metadata?

In your own words, explain what metadata means.
5. Examine MariaDB

Find at least three different things that MariaDB can tell you about a database or its objects.

You may use:

    SHOW ...

    DESCRIBE ...

    information_schema

When you're done

You must be able to explain:

    What metadata is and why a RDBMS needs metadata.

# Task 3 – Build a database
Purpose

Now you have to create a database and some database objects yourself.

You don't have to build a big system. The purpose is to see the connection:

RDBMS → database → tables → data
1. Create the database

Create a database called:

school

Use SQL.

Then check that the database exists.
2. Create the table students

The table shall contain at least:

    id

    name

    email

idmust be the primary key.

Use appropriate data types.
3. Create the table courses

The table shall contain at least:

    id

    name

idmust be the primary key.
4. Create the table enrollments

A student must be able to be enrolled in several courses, and a course must be able to have more students.

Therefore, create a table that connects students and courses.

It shall contain at least:

    student-id-id

    course ID

Use foreign keys so that the database can control the relationships.
5. Insert data

Create at least:

    3 students

    2 courses

    4 registrations

6. Examine what you have built

Use SQL to examine the database.

You must be able to show:

school
├── students
├── courses
└── enrollments

Also, check the tables' metadata.
7. Think like a database server

Imagine that another developer gives you this SQL:

SELECT *
FROM students;

What does MariaDB need to know in order to perform it?

Make a list of at least five things.

Example:

    MariaDB needs to know which database studentsis in.

When you're done

You need to be able to explain the difference between:

    RDBMS

    database

    table

    row

    database object

    metadata

# Task 4 – Persistence and Docker
Purpose

MariaDB must store his data somewhere.

In this task, you will examine the difference between:

    a Docker Container

    Persistent storage

    the database itself

You should also find that the life of the container is not necessarily the same as the life of the data.
1. Check your data

Make sure that your school- database still exists.

Drive e.g.:

SELECT *
FROM students;

You need to be able to see the students you have previously created.
2. Stop the container

Stop the MariaDB container in Docker Desktop.

Start it again.

Then connect to MariaDB with Workbench.

Examination:

SHOW DATABASES;

and:

SELECT *
FROM school.students;

Questions

Is your data still there?

Why?
3. Delete the container

Before you do this:

Your teacher should check that your MariaDB container uses a persistent Docker volume.

Then delete the container.

Create a new MariaDB container with the same persistent volume.

Connect again with Workbench.
4. Find your data

Examination:

SHOW DATABASES;

and:

SELECT *
FROM school.students;

Questions

Is the database still there?

Are the students still there?

Why?
5. Persistence or backup?

Answer:
A

What does it mean that data is persistent?
B

Is a Docker Volume a Backup?
C

Imagine that by mistake someone is driving:

DROP DATABASE school;

Will a Docker volume protect you from that?
D

What would you need if you were to be able to recreate the database after such a bug?
6. The important model

Explain in your own words this model:

Computer
│
└── Docker
│
└── Container
│
└── MariaDB
│
└── Database
│
└── Tables
│
└── Data

And explain where a Docker volume fits in.
When you're done

You need to be able to explain the difference between:

    The lifetime of the container, the lifetime of the database and persistent storage.

You also need to be able to explain why persistent storage is not the same as backup.

# Day 2 – Build and secure the database
Today's goal

On Day 1, you established a database server and worked with a database.

Today we go one step further.

A RDBMS is not just about storing data. It must also be able to manage:

    database objects

    data integrity

    users

    rights

    access to specific objects

When you're done with the day, you should be able to:

    explain what central database objects are used for

    explain the purpose of constraints

    explain what an index is and what it can be used for

    Explain what a view is

    explain at a basic level what a stored procedure is

    Distinguishing between authentication and authorization

    create database users

    assign and examine permissions

    explain the principle of least privilege

    explain that permissions can be given to certain database objects

    test whether a permissions model actually works

We’ll continue with the database you created on Day 1:

school
├── students
├── courses
└── enrollments

1. From database to database management

On Day 1 we saw on this model:

RDBMS
│
└── Database
├── Tables
├── Indexes
├── Views
└── Constraints

Today we need to work more actively with these objects.

This means that we are no longer just asking:

    How do I save my data?

We also ask:

    How should the database be built?

    What rules should the database enforce?

    Who should have access?

    What should the different users do?

It is an important difference between regular database programming and database management.
2. Database Objects

A database can contain many different kinds of objects.

On this course, we work with:

Database
│
├── Tables
├── Indexes
├── Views
├── Constraints
└── Stored procedures / functions

The different objects have different purposes.
Object 	What is it used for?
Table 	Saved data
Constraint 	Enforce rules for data
Index 	Make certain posts more effective
View 	Present data through a saved query
Great procedure 	Save database code that can be called

These are not five different ways to store the same kind of data.

They solve different problems.
3. Constraints

A constraint is a rule that the database enforces.

Imagine our students-table:

CREATE TABLE students (
id INT PRIMARY KEY AUTO_INCREMENT,
name VARCHAR(100) NOT NULL
);

Here we already have two constraints:

PRIMARY KEY
NOT NULL

PRIMARY KEYsays that ididentify the row unequivocally.

NOT NULLsays that nameNot to be NULL. .

Constraints are thus part of the database's integrity rules.
4. Primary key

A primary key identifies a series uniquely in a table.

Example:

students

id    name
-----------
1     Alice
2     Bob
3     Clara

Here are:

id

primary key.

This means, among other things, that two rows cannot have the same primary key value.

So we can't have:

id    name
-----------
1     Alice
1     Bob

in the same table, if idis the primary key.

Primary keys are not just useful for programming.

The RDBMS uses them as part of the structure and integrity rules of the database.
5. Foreign keys

An alien key is used to describe a relationship between tables and to enforce referentelial integrity.

Imagine:

students
id    name
-----------
1     Alice
2     Bob

and:

courses
id    name
-----------
1     Programming
2     Databases

We may have a table:

enrollments

student_id    course_id
----------------------
1             1
1             2
2             1

Her refererer:

student_id

to a student.

Og:

course_id

refererer til et course.

Vi kan derfor definere foreign keys:

FOREIGN KEY (student_id)
REFERENCES students(id)

and:

FOREIGN KEY (course_id)
REFERENCES courses(id)

Databasen kan dermed kontrollere, at en tilmelding ikke refererer til en student eller et kursus, der ikke findes.
6. Hvorfor skal databasen håndhæve reglerne?

Det kan være fristende at tænke:

    Det sørger mit program for.

Example:

Program
↓
Kontroller student_id
↓
Send INSERT til database

But the database can be used by more than one program.

Example:

Webapplikation ───┐
│
Adminværktøj ─────┼──→ MariaDB
│
Script ───────────┘

If all programs themselves need to implement all integrity rules, they may end up implementing them differently.

Constraints giver derfor databasen mulighed for selv at håndhæve centrale regler.

Det betyder:

    Data skal ikke kun være korrekte, fordi et bestemt program forsøger at gøre dem korrekte.

Databasen kan selv håndhæve bestemte regler.
7. NOT NULL

NOT NULLis a simple constraint.

Example:

name VARCHAR(100) NOT NULL

Det betyder, at en række ikke kan have:

name = NULL

Det er forskelligt fra en tom tekst.

''

er en tekststreng med længde 0.

NULL

means that the value is missing/not specified.

Constraints handler derfor ikke kun om programmeringens "happy path".

De er en del af databasens regler.
8. UNIQUE

One UNIQUEconstraint is used when a value or combination of values must not occur several times.

Example:

email VARCHAR(255) UNIQUE

For example, if we use email as a unique identifier, the database must prevent:

alice@example.com
alice@example.com

i at forekomme flere gange på samme måde.

Det er igen et eksempel på en regel, som databasen selv kan håndhæve.
9. Constraints samlet

Vi kan nu se flere typer constraints:

Constraints
│
├── PRIMARY KEY
├── FOREIGN KEY
├── NOT NULL
└── UNIQUE

They solve different problems.
Constraint 	Purpose
PRIMARY KEY 	Identificerer en række entydigt
FOREIGN KEY 	Håndhæver en relation til en anden tabel
NOT NULL 	Forhindrer NULL i en kolonne
UNIQUE 	Forhindrer dubletter af en værdi/kombination

Du behøver ikke kunne alle muligheder for constraints på Dag 2.

Det vigtige er princippet:

    Constraints er regler, som RDBMS'et selv håndhæver.

10. Index

Et indeks er et databaseobjekt, som kan gøre bestemte opslag mere effektive.

Forestil dig en tabel med 1.000.000 studerende.

Hvis vi ofte søger efter:

SELECT *
FROM students
WHERE name = 'Alice';

kan det være relevant at have et indeks på name.

An index provides the RDBMS with a data structure that can be used to find relevant rows more efficiently.

Det er ikke det samme som at gemme en ekstra kopi af hele tabellen.
11. Et indeks er ikke gratis

Et indeks kan gøre opslag hurtigere.

Men det har også en pris.

Indekset skal:

    bruge storage

    vedligeholdes, når data ændres

    updated by relevant INSERT

    opdateres ved relevante UPDATE

    opdateres ved relevante DELETE

Derfor er:

    Flere indeks = altid bedre performance

ikke en god regel.

Det korrekte princip er:

    Indeks er et værktøj til at optimere bestemte adgangsmønstre, men indeks har også omkostninger.

På Dag 4 arbejder vi mere med performance og med at undersøge, om et indeks faktisk bliver brugt.
12. Opret et indeks

Et eksempel:

CREATE INDEX idx_students_name
ON students(name);

Her opretter vi et indeks på:

students.name

Vi kan undersøge indeksene på tabellen med:

SHOW INDEX FROM students;

Det giver os blandt andet mulighed for at se:

    hvilket indeks der findes

    hvilken tabel det tilhører

    which columns it covers

13. Tænk over

Antag, at vi har:

students

med én million rækker.

We often look for:

WHERE name = 'Alice'

Why can an index help?

Og hvorfor kan vi ikke bare sige:

    Så laver vi et indeks på alle kolonner?

Tænk både på:

    storage

    ændringer af data

    vedligeholdelse af indeks

Den dybere performanceanalyse kommer senere.
14. Views

Et view er et databaseobjekt, der præsenterer resultatet af en forespørgsel som et tabel-lignende objekt.

Forestil dig, at vi vil vise en oversigt over elever og deres kurser.

Dataene ligger måske i:

students
courses
enrollments

Vi kan skrive en forespørgsel med JOIN:

SELECT
students.name AS student_name,
courses.name AS course_name
FROM students
JOIN enrollments
ON students.id = enrollments.student_id
JOIN courses
ON courses.id = enrollments.course_id;

We can save this query as a view.

Example:

CREATE VIEW student_overview AS
SELECT
students.name AS student_name,
courses.name AS course_name
FROM students
JOIN enrollments
ON students.id = enrollments.student_id
JOIN courses
ON courses.id = enrollments.course_id;

Then we can write:

SELECT *
FROM student_overview;

15. Et view er ikke nødvendigvis en kopi

Det er vigtigt at forstå, hvad et almindeligt view er.

Når vi opretter:

CREATE VIEW student_overview AS
...

har vi ikke nødvendigvis lavet en ny kopi af alle data.

Vi har oprettet et databaseobjekt, der repræsenterer en forespørgsel.

En forsimplet model er:

student_overview
│
▼
Query
│
├── students
├── enrollments
└── courses

Når viewet bruges, kan RDBMS'et arbejde ud fra definitionen.

Det betyder, at et view kan give os en anden måde at præsentere eller tilgå eksisterende data på.
16. Hvorfor bruge views?

Views kan blandt andet bruges til at:

    samle data fra flere tabeller

    skjule kompleksiteten i en forespørgsel

    give brugere en bestemt måde at se data på

    skabe et mere begrænset interface til data

Example:

En bruger behøver måske ikke adgang til hele students-tabellen.

I stedet kan brugeren få adgang til:

student_overview

Det bliver interessant, når vi senere kombinerer views med permissions.
17. Undersøg et view

Vi kan se viewets definition med:

SHOW CREATE VIEW student_overview;

Det er igen et eksempel på, at RDBMS'et indeholder metadata om sine egne objekter.

So we now have several different kinds of database objects:

school
│
├── students
├── courses
├── enrollments
├── indexes
└── student_overview

RDBMS'et skal vide, at disse objekter findes, og hvordan de er defineret.
18. Stored procedures

Et RDBMS kan også indeholde databasekode.

En stored procedure er en procedure, som gemmes i databasen og kan kaldes af en klient.

Example:

CALL GetStudent(1);

Proceduren kan eksempelvis udføre en forespørgsel efter en bestemt student.

En meget simpel procedure kan konceptuelt være:

GetStudent
│
├── modtag student_id
│
└── find student

Stored procedures kan være langt mere avancerede end dette.

Det er dog ikke vores mål i dag.
19. Hvorfor har vi stored procedures med?

På dette kursus er stored procedures især interessante af én grund:

    De er et databaseobjekt, som vi kan administrere permissions på.

Vi kan derfor have:

students
student_overview
GetStudent

og give forskellige brugere forskellige former for adgang.

Example:

Student
│
└── EXECUTE → GetStudent

Det betyder ikke nødvendigvis, at brugeren samtidig skal have samme adgang til alle tabeller, som proceduren arbejder med.

Det giver os en interessant sikkerhedsmodel.
20. Databaseobjekter samlet

Vi har nu set:

school
│
├── Tables
│   ├── students
│   ├── courses
│   └── enrollments
│
├── Constraints
│
├── Indexes
│
├── Views
│
└── Stored procedures

De har forskellige formål:

Table
→ gemmer data

Constraint
→ håndhæver regler

Index
→ understøtter effektive opslag

View
→ præsenterer data gennem en gemt forespørgsel

Stored procedure
→ gemmer databasekode, som kan kaldes

Det er databaseadministrationens første del.

Den næste del handler om:

    Hvem må bruge disse objekter?

21. Brugere på databaseserveren

Indtil nu har du sandsynligvis arbejdet som en bruger med forholdsvis mange rettigheder.

Det er praktisk under udvikling.

Men en rigtig databaseserver kan have mange brugere.

Example:

MariaDB
│
├── school_admin
├── school_teacher
└── school_student

De skal ikke nødvendigvis have samme adgang.

En administrator skal eksempelvis kunne administrere databasen.

En lærer skal måske kunne arbejde med elever.

En elev skal måske kun kunne læse bestemte oplysninger.

Det betyder, at RDBMS'et skal kunne kontrollere:

    Hvem er brugeren?

and:

    Hvad må brugeren gøre?

22. Authentication

Authentication handler om identitet.

Spørgsmålet er:

    Hvem er du?

Et simpelt eksempel er:

username + password
│
▼
MariaDB
│
▼
school_student

Hvis loginoplysningerne accepteres, ved MariaDB, hvilken databasebruger forbindelsen repræsenterer.

Authentication handler altså om at etablere brugerens identitet.
23. Authorization

Når MariaDB ved, hvem du er, er det næste spørgsmål:

    Hvad må du gøre?

Det er authorization.

Example:

school_student

SELECT students       ✓
INSERT students       ✗
UPDATE students       ✗
DELETE students       ✗
CREATE TABLE          ✗
CREATE USER           ✗

Brugeren kan altså godt være korrekt identificeret og stadig få adgang nægtet.

Det er en vigtig forskel.
24. Authentication vs. authorization

Husk:

Authentication
↓
Hvem er du?

Authorization
↓
Hvad må du gøre?

Et eksempel:

    Alice logger ind med sit brugernavn og password.

Det er authentication.

MariaDB accepterer Alice som:

school_student

Derefter forsøger Alice:

DELETE FROM students
WHERE id = 1;

MariaDB kontrollerer hendes permissions.

Hvis hun ikke har DELETE, bliver handlingen afvist.

Det er authorization.
25. Permissions og privileges

En permission eller privilege beskriver, hvad en bruger må gøre.

Examples:

SELECT
INSERT
UPDATE
DELETE
CREATE
DROP
EXECUTE

For example, a user may have:

SELECT

men ikke:

DELETE

Det betyder, at brugeren kan læse data, men ikke slette data.

Rettigheder kan dermed bruges til at begrænse brugerens adgang.
26. Ikke alle brugere skal have samme rettigheder

Forestil dig tre roller:

school_admin
school_teacher
school_student

Vi kan eksempelvis tænke:
Administrator

Kan administrere databasen.
Teacher

Kan arbejde med relevante data.
Student

May make limited posts.

A possible model could be:
Action 	Admin 	Teacher 	Student
SELECT 	✓ 	✓ 	✓
INSERT 	✓ 	✓ 	✗
UPDATE 	✓ 	✓ 	✗
DELETE 	✓ 	✓ 	✗
CREATE TABLE 	✓ 	✗ 	✗
DROP TABLE 	✓ 	✗ 	✗
CREATE USER 	✓ 	✗ 	✗

Det er ikke en universel rigtig model.

Det er en permissions-model, som vi designer ud fra kravene til systemet.
27. Permissions er en del af systemets design

Det er vigtigt ikke kun at tænke:

    Hvilke SQL-kommandoer kan denne bruger køre?

Start i stedet med:

    Hvad skal denne bruger kunne gøre?

Example:

    En student skal kunne se information om kurser, men ikke ændre databasen.

Det krav kan derefter omsættes til permissions.

Det er samme princip, som når vi som programmører omsætter krav til funktionalitet.
28. CREATE USER

MariaDB kan oprette databasebrugere.

Et eksempel er:

CREATE USER 'school_student'@'%'
IDENTIFIED BY 'et-password';

The user is now on the database server.

But that doesn’t automatically mean that the user has to do everything.

Det er vigtigt:

    At have en konto er ikke det samme som at have adgang til alle databaseobjekter.

Authentication og authorization er stadig to forskellige ting.
29. GRANT

Rettigheder kan tildeles med GRANT.

Et simpelt eksempel kunne være:

GRANT SELECT
ON school.students
TO 'school_student'@'%';

Det betyder grundlæggende:

    Brugeren school_student får SELECT-adgang til school.students.

Vi har dermed kombineret:

Bruger
+
Permission
+
Databaseobjekt

Det er kernen i permissions-modellen.
30. Permissions kan være specifikke

Vi behøver ikke nødvendigvis give en bruger adgang til hele serveren.

Vi kan give adgang til bestemte områder.

Example:

school_student
│
└── SELECT
│
└── school.students

Eller:

school_student
│
└── SELECT
│
└── school.student_overview

Det giver mulighed for meget mere præcis adgangskontrol.
31. SHOW GRANTS

Når vi administrerer permissions, er det vigtigt at kunne undersøge dem.

MariaDB giver os blandt andet:

SHOW GRANTS FOR 'school_student'@'%';

Det giver os mulighed for at se, hvilke rettigheder brugeren har fået.

Det er et vigtigt administrationsværktøj.

Når noget ikke virker, kan vi derfor spørge:

    What permissions does the user actually have?

Instead of just guessing.
32. Least privilege

Et centralt sikkerhedsprincip er least privilege.

Det betyder:

    En bruger skal kun have de rettigheder, som brugeren har brug for.

Forestil dig en applikation, der kun skal kunne læse elevdata.

Hvis applikationen får:

ALL PRIVILEGES

har den langt flere rettigheder end nødvendigt.

Hvis applikationen bliver kompromitteret, kan angriberen potentielt bruge de samme rettigheder.

En mere begrænset model kunne være:

Application
│
└── SELECT
│
└── nødvendige data

i stedet for:

Application
│
└── ALL PRIVILEGES

Least privilege begrænser altså konsekvenserne af fejl og kompromittering.
33. Hvorfor er GRANT ALL en dårlig standard?

Imagine:

GRANT ALL
ON school.*
TO 'school_student'@'%';

Så har school_student fået meget omfattende adgang.

Det kan være praktisk.

Men det fortæller ikke noget om, hvad brugeren faktisk har brug for.

Hvis eleven kun skal kunne læse data, er:

SELECT

mere passende end:

ALL

Det er derfor en god praksis at starte med kravene og derefter tildele de nødvendige rettigheder.
34. Permissions på views

Views bliver særligt interessante, når vi kombinerer dem med permissions.

Forestil dig, at vi har:

students

med mange kolonner.

Måske indeholder tabellen oplysninger, som en bestemt bruger ikke skal kunne se.

Vi kan i stedet lave:

student_overview

som kun viser de oplysninger, brugeren skal have adgang til.

Example:

students

id
name
email
internal_note

View:

student_overview

name
email

Vi kan derefter give en bruger adgang til viewet frem for hele tabellen.

Det kan give et mere begrænset interface til data.
35. Views og sikkerhed

Det betyder ikke, at views automatisk er en perfekt sikkerhedsmekanisme.

Sikkerheden afhænger af blandt andet:

    hvilke permissions brugeren har

    hvilke objekter viewet refererer til

    hvordan viewet er defineret

    hvordan RDBMS'et håndterer sikkerhedskonteksten

Men den overordnede idé er vigtig:

    Databaseobjekter kan bruges som en del af adgangsmodellen.

Det er derfor, målpinden ikke kun handler om permissions på tabeller.
36. Permissions på stored procedures

Stored procedures kan også have permissions.

En bruger kan eksempelvis få:

EXECUTE

on a procedure.

Example:

school_student
│
└── EXECUTE
│
▼
GetStudent

The interesting thing is that the user can thus access a specific database function.

Det er en anden model end:

school_student
│
└── SELECT
│
▼
students

This makes stored procedures relevant to target stick 5.

We do not need to learn advanced procedure programming to understand this model.
37. Commands, objects og permissions

Når vi taler om permissions, kan vi derfor have flere niveauer.

Eksempelvis:

Database
│
├── Table
│
├── View
│
└── Stored procedure

Og brugeren kan få forskellige former for adgang:

SELECT  → læse data
INSERT  → indsætte data
UPDATE  → ændre data
DELETE  → slette data
EXECUTE → køre procedure

Den konkrete permissionsmodel afhænger af RDBMS'et.

Men den generelle idé er:

    Adgang kan kontrolleres både ud fra hvem brugeren er, hvad brugeren forsøger at gøre, og hvilket objekt brugeren forsøger at arbejde med.

38. Test permissions – stol ikke på konfigurationen

Det er ikke nok at skrive:

GRANT ...

og derefter antage, at sikkerheden virker.

We should test it.

Det svarer til almindelig softwareudvikling.

If we write:

if user.isAdmin
allow
else
deny

vil vi også teste begge tilfælde.

Det samme gælder databasepermissions.
39. Test som school_student

Log ind med:

school_student

og forsøg at læse data:

SELECT *
FROM school.students;

Hvis brugeren har SELECT, bør det virke.

Forsøg derefter:

INSERT INTO school.students (...);

Hvis brugeren ikke har INSERT, bør MariaDB afvise operationen.

Prøv også:

DELETE FROM school.students
WHERE id = 1;

og eksempelvis en administrativ handling.

Det interessante er ikke kun, om det virker.

Det interessante er:

    Virker det på den måde, vi havde planlagt?

40. Permission denied er information

Forestil dig, at du får en fejl:

Access denied

Det betyder ikke nødvendigvis:

    Min database er ødelagt.

Det kan være præcis det resultat, permissions-modellen skal give.

Hvis:

school_student

ikke må slette data, er:

DELETE ...

med fejl et forventet resultat.

Vi skal derfor lære at skelne mellem:

Uventet fejl

and:

Forventet afvisning

41. Fejlfinding af permissions

Hvis en handling bliver afvist, kan vi undersøge brugerens permissions:

SHOW GRANTS FOR 'school_student'@'%';

Vi kan derefter sammenligne:

Hvad forventede vi?
↓
Hvad har brugeren faktisk?
↓
Hvor er forskellen?

Det er en meget vigtig administrativ arbejdsmetode.

I stedet for at gætte:

    Prøv at give brugeren mere adgang.

kan vi undersøge:

    Hvilken permission mangler faktisk?

42. Et komplet eksempel

Forestil dig følgende database:

school
│
├── students
├── courses
├── enrollments
│
├── student_overview
│
└── GetStudent

Og tre brugere:

school_admin
school_teacher
school_student

We can imagine the following model:

                         school_admin
                              │
                    ┌─────────┼─────────┐
                    ▼         ▼         ▼
                  tables    views    procedures


                         school_teacher
                              │
                    ┌─────────┼─────────┐
                    ▼         ▼
                 tables      views


                         school_student
                              │
                    ┌─────────┴─────────┐
                    ▼                   ▼
                  views              procedure

Det er ikke en bestemt MariaDB-konfiguration.

Det er en mental model for, hvordan adgang kan designes.
43. Databaseadministration handler også om design

Når vi administrerer permissions, designer vi faktisk en del af systemet.

Example:

    En elev skal kunne se kursusoplysninger.

Det kan implementeres på mange måder.

Vi kunne give:

SELECT på courses

eller:

SELECT på et view

eller måske:

EXECUTE på en procedure

Hvilken løsning der er bedst, afhænger af systemets krav.

Derfor er databaseadministration ikke kun:

    Kør denne SQL-kommando.

It's also about understanding why a particular access model is appropriate.
44. Fra krav til permissions

A useful way to work is:

Krav
↓
Hvad skal brugeren kunne?
↓
Hvilke objekter skal brugeren kunne bruge?
↓
Hvilke permissions kræves?
↓
GRANT
↓
Test

Example:

Krav:
Student skal kunne se kursusinformation

↓

Student skal kunne læse data

↓

SELECT

↓

Måske:

SELECT → student_overview

↓

Test som student

Det er en mere robust tilgang end bare at give alle brugere meget brede rettigheder.
45. Den samlede sikkerhedsmodel

Vi kan nu udvide vores mentale model fra Dag 1.

På Dag 1 havde vi:

Client
↓
RDBMS
↓
Database
↓
Storage

Nu har vi:

Client
↓
Authentication
↓
User
↓
Authorization
↓
Permissions
↓
Database objects
↓
Data

RDBMS'et binder disse ting sammen.
46. Authentication og authorization i praksis

Imagine:

MySQL Workbench
│
│ username + password
▼
MariaDB
│
│ Authentication
▼
school_student
│
│ Authorization
▼
"Må denne bruger
udføre denne handling
på dette objekt?"
│
├── YES → udfør
│
└── NO  → afvis

Det er en af de vigtigste mentale modeller fra Dag 2.
47. Hvorfor bruger vi forskellige brugere?

Det kan være fristende at have én databasebruger:

application
↓
admin

Så slipper applikationen for permissions-problemer.

Men det giver meget brede rettigheder.

Hvis applikationen bliver kompromitteret, kan angriberen potentielt gøre alt det, databasebrugeren må gøre.

Det er derfor bedre at tænke:

Applikation
↓
Kun nødvendige permissions

It's least privilege in practice.
48. Databasebrugere er ikke det samme som applikationsbrugere

Der er også en vigtig skelnen her.

Din applikation kan have brugere som:

Alice
Bob
Charlie

Det betyder ikke nødvendigvis, at MariaDB skal have:

Alice
Bob
Charlie

som databasebrugere.

En applikation kan eksempelvis bruge én databasekonto til at forbinde til databasen:

Alice ──┐
Bob ────┼──→ Application ──→ MariaDB
Charlie ┘

Applikationen håndterer så sine egne brugere.

MariaDB handles the identity and permissions of the database connection.

This distinction becomes more important when we later work with applications and database security.
49. Databasebrugeren repræsenterer en tillidsgrænse

Når en applikation forbinder til MariaDB, skal den bruge en databasekonto.

Den konto bør have de rettigheder, som applikationen faktisk har brug for.

Example:

Web application
│
▼
school_app
│
├── SELECT students
├── SELECT courses
├── INSERT enrollments
└── ...

Hvis applikationen ikke skal kunne:

DROP TABLE
CREATE USER
DROP DATABASE

bør kontoen ikke have disse permissions.

Det er igen least privilege.
50. Databaseobjekter og sikkerhed hænger sammen

Vi kan nu se, hvorfor vi begyndte dagen med databaseobjekter.

Vi har:

Tables
Indexes
Views
Constraints
Stored procedures

og nu:

Users
Permissions

De to områder hænger sammen.

For eksempel:

User
│
└── SELECT
│
▼
View
│
▼
Tables

eller:

User
│
└── EXECUTE
│
▼
Stored procedure
│
▼
Data

Det er en del af den måde, et RDBMS kan bruges til at kontrollere adgang til data.
51. En permissions-matrix

En praktisk måde at dokumentere en permissions-model på er en matrix.

Example:
Action 	Admin 	Teacher 	Student
SELECT students 	✓ 	✓ 	✗
INSERT students 	✓ 	✓ 	✗
UPDATE students 	✓ 	✓ 	✗
DELETE students 	✓ 	✓ 	✗
SELECT courses 	✓ 	✓ 	✓
CREATE TABLE 	✓ 	✗ 	✗
DROP TABLE 	✓ 	✗ 	✗
CREATE USER 	✓ 	✗ 	✗
EXECUTE GetStudent 	✓ 	✓ 	✓

En sådan matrix er ikke selve MariaDB-konfigurationen.

Den er et designværktøj.

Den gør det lettere at svare på:

    Hvad skal denne bruger egentlig kunne?

52. Permissions skal kunne forklares

En god permissions-model er ikke bare en liste af GRANT-kommandoer.

Du skal også kunne forklare:

    Hvorfor har school_teacher denne permission?

and:

    Hvorfor har school_student ikke denne permission?

Example:

school_student
INSERT students = ✗

Forklaring:

    Studenten skal kunne læse information, men skal ikke kunne ændre registrerede studerende.

Det er en bedre begrundelse end:

    Fordi det var den kommando, vi fik.

53. Tænk over

En lærer skal kunne:

    se elever

    oprette elever

    ændre elever

but may not:

    create database users

    slette hele databasen

    ændre serverens sikkerhedsopsætning

Hvilke permissions har læreren brug for?

Og endnu vigtigere:

    What permissions don't the teacher need?

54. Hvad med roller?

Mange RDBMS'er understøtter også roles.

En rolle kan bruges til at samle permissions.

Konceptuelt:

Teacher role
│
├── SELECT students
├── INSERT students
└── UPDATE students

Derefter kan en bruger få rollen:

Alice
│
└── Teacher role

Vi bruger ikke roles som et selvstændigt emne på Dag 2.

Grunden er, at den centrale model er vigtigere:

User
↓
Permissions
↓
Objects

Roles er en måde at organisere permissions på og kan være relevante senere.
55. Databaseobjekter er ikke alle det samme

Det er også vigtigt ikke at blande objekterne sammen.

An index:

Index

er ikke en alternativ tabel.

Et view:

View

is not necessarily a copy of data.

En constraint:

Constraint

er ikke noget, brugeren "læser".

En stored procedure:

Stored procedure

er databasekode.

De er forskellige objekttyper med forskellige formål.
56. Dagens samlede model

Vi kan nu samle Dag 1 og Dag 2:

                              CLIENT
                         MySQL Workbench
                                │
                                ▼
                         Authentication
                                │
                                ▼
                              USER
                                │
                                ▼
                         Authorization
                                │
                                ▼
                           PERMISSIONS
                                │
                ┌───────────────┼────────────────┐
                ▼               ▼                ▼
             TABLES           VIEWS         PROCEDURES
                │               │                │
                ├───────────────┼────────────────┤
                │
                ▼
            CONSTRAINTS
                │
                ▼
               DATA

Og bagved hele systemet:

RDBMS
│
├── Users
├── Permissions
├── Metadata
├── Database objects
├── Query processing
└── Storage

Det er den centrale model for Dag 2.
57. Fra Dag 1 til Dag 2

On Day 1, you learned:

Client
↓
RDBMS
↓
Database
↓
Storage

Du lærte også:

Metadata

and:

Persistence

På Dag 2 har vi tilføjet:

Database objects

and:

Users
Permissions
Authentication
Authorization

Dermed kan vi nu beskrive databaseserveren mere præcist.

Et RDBMS er ikke bare et program, der opbevarer data.

Det er et system, der administrerer:

    data

    struktur

    metadata

    integritetsregler

    users

    rights

    databasekode

    storage

58. En vigtig sikkerhedsregel

Hvis du kun skal huske ét sikkerhedsprincip fra Dag 2, så er det:

    Giv ikke en bruger flere rettigheder, end brugeren har brug for.

Det er least privilege.

Det gælder både:

    mennesker

    applikationer

    scripts

    services

    andre systemkomponenter

En bruger, der kun skal læse data, behøver ikke DELETE.

En applikation, der kun skal læse og oprette data, behøver ikke DROP DATABASE.

En lærer, der skal arbejde med elever, behøver ikke nødvendigvis CREATE USER.

Permissions skal derfor tage udgangspunkt i behovet.
59. Når sikkerheden ikke virker

Hvis en bruger kan gøre noget, som brugeren ikke burde kunne gøre, er permissions-modellen for bred.

Hvis en bruger ikke kan gøre noget, som brugeren skal kunne gøre, er permissions-modellen for snæver.

Begge situationer kan undersøges systematisk:

1. Hvem er brugeren?
   ↓
2. Hvilke permissions har brugeren?
   ↓
3. Hvilket objekt forsøger brugeren at bruge?
   ↓
4. Hvilken handling forsøger brugeren at udføre?
   ↓
5. Er permissionen tildelt?

Dette er en simpel, men meget nyttig fejlsøgningsmodel.
60. Centrale begreber
    Databaseobjekt

Et objekt, som RDBMS'et administrerer som en del af databasen.

Examples:

    table

    index

    view

    constraint

    stored procedure

Constraint

En regel, som RDBMS'et håndhæver for data eller relationer.

Examples:

PRIMARY KEY
FOREIGN KEY
NOT NULL
UNIQUE

Index

En datastruktur, som kan hjælpe RDBMS'et med at finde data mere effektivt.

Indeks bruger storage og skal vedligeholdes.
View

A table-like database object based on a saved query.

A regular view is not necessarily a separate copy of data.
Great procedure

Databasekode, som er gemt i RDBMS'et og kan kaldes.
Authentication

At kontrollere eller etablere, hvem en bruger er.

    Hvem er du?

Authorization

At afgøre, hvad en identificeret bruger må gøre.

    Hvad må du gøre?

Permission / privilege

En rettighed til at udføre en bestemt handling på et bestemt niveau eller objekt.
Least privilege

Princippet om, at en bruger kun skal have de rettigheder, der er nødvendige for brugerens opgave.
61. Tænk over – den samlede model

Prøv at forklare følgende situation:

En elev forbinder til MariaDB med:

school_student

MariaDB accepterer login.

Eleven forsøger:

SELECT *
FROM school.students;

Det virker.

Eleven forsøger derefter:

DELETE FROM school.students
WHERE id = 1;

Det bliver afvist.

Forklar trin for trin:

    Hvad er authentication-delen?

    Hvad er authorization-delen?

    Hvilken permission kræves til SELECT?

    Hvilken permission kræves til DELETE?

    Hvorfor kan den samme bruger godt have den ene permission uden den anden?

62. One last example

Imagine a school application.

The application must be able to:

    show students

    Show courses

    register an enrolment

But it should not be able to:

    create database users

    delete tables

    change database configuration

    delete the entire database

One possible mental model is:

School application
│
▼
database user
│
├── SELECT students
├── SELECT courses
├── INSERT enrollments
└── ...

Everything else should not be allowed unless there is a specific need.

It is least privilege applied to an application.
63. What can you do after Day 2?

You need to be able to view a database and not just think:

    Here are some tables.

You need to be able to see:

Database
│
├── Tables
├── Constraints
├── Indexes
├── Views
└── Stored procedures

and at the same time:

Users
│
└── Permissions
│
└── Database objects

You must be able to explain:

    What is this object?

    What is it used for?

    Who can use it?

    What action can the user perform?

And when something doesn't work:

    What permission is missing?

64. The most important mental model of the day

If you need to bring one model from Day 2, it is this:

                    RDBMS
                      │
          ┌───────────┴───────────┐
          │                       │
      DATABASE                  USERS
          │                       │
          │                  AUTHENTICATION
          │                       │
          │                  AUTHORIZATION
          │                       │
          ▼                       ▼
DATABASE OBJECTS          PERMISSIONS
│                       │
├── Tables              │
├── Indexes             │
├── Views        ───────┤
├── Constraints         │
└── Procedures ─────────┘
│
▼
DATA

Database management is therefore both about:

    How the database is built

and:

    Who can use it and what they can do.

That is the basis for the rest of the work with the database server.

# Hardening of a database server

A database server often contains data that should not be accessible to everyone. It may be personal data, login details, financial data or data on which an application is dependent.

This means that a database server should not just fungerefunction. It also needs to be configured so that it only provides access to the people, systems, and networks that actually need it.

It's called hardening.

Hardening means basic:

    To reduce a server's unnecessary attack interfaces and ensure that the features, users and accesses that are not needed are not open.

Hardening is not a single command. It's a series of safety considerations and configurations.
The problem: A database server may be more open than necessary

When you install a database server, you get a system with a number of functions:

    user accounts

    network access

    administrative accounts

    database objects

    rights

    services and functions

    configuration options

If these things are not reviewed, the server may end up providing more access than necessary.

A simple example:

Internet
│
│ 3306
▼
MariaDB
│
├── root
├── user1
├── user2
└── application

If MariaDB simultaneously accepts connections from all over the Internet and more users have extensive rights, an unnecessarily large attack surface has been created.

A more sensible setup could be:

Kun nødvendigt netværk
│
▼
MariaDB
│
┌────┴────┐
▼         ▼
application   administrator
│             │
begrænsede      administrative
rettigheder      rettigheder

The goal is not necessarily to make the server inaccessible.

The goal is to make it as open as necessary – and no more.
1. Restrict network access

The first question should be:

    Who should be able to connect to the database server?

If a MariaDB server is only to be used by programs on the same computer, there is no need to make it available from the entire network.

In Docker, for example, you can choose to only expose the gate on localhost:

127.0.0.1:3306

Instead of:

0.0.0.0:3306

The difference is important.

127.0.0.1

in this context, means:

    Accept connections from this computer.

Whereas a binding to all interfaces can make the database server available from other machines, depending on the firewall and other network configuration.
Think about:

    If no one outside of this computer needs to use the database, why should port 3306 be open to others?

2. Administrative users need to be treated differently

A database administrator needs powerful privileges.

A regular application usually does not have.

It is therefore a bad idea to let an application connect as root. .

Example of a bad model:

Webapplikation
│
▼
root
│
├── SELECT
├── INSERT
├── UPDATE
├── DELETE
├── DROP
└── ...

If the application is compromised, the attacker has potentially gained access to everything that rootcan.

A better model is:

Webapplikation
│
▼
application_user
│
├── SELECT
├── INSERT
└── UPDATE

The user only gets the rights that the application actually needs.

It's least privilege.
3. Use only the users that are needed

En databaseserver bør ikke have en masse gamle eller ukendte konti.

For hver bruger bør man kunne svare på:

    Hvem er brugeren?

    Hvad bruges kontoen til?

    Hvilke rettigheder har den?

    Hvor må den logge ind fra?

    Is the account still necessary?

Hvis en konto ikke længere bruges, bør den normalt fjernes eller deaktiveres.

Det er også vigtigt at være opmærksom på konti, som kan være oprettet automatisk eller eksistere efter en standardinstallation.
4. Brug stærk authentication

Authentication handler om:

    Hvem er du?

The database server must be able to verify that the person or system that tries to log in is actually the user to whom the login credent belongs.

Passwords should therefore be:

    tilstrækkeligt lange

    vanskelige at gætte

    forskellige mellem konti

    ikke genbrugt på tværs af systemer

Administrative konti bør især beskyttes godt.
5. Authorization er mindst lige så vigtigt

Authentication fortæller os:

    "Dette er application_user."

Authorization fortæller:

    "Hvad må application_user gøre?"

Det er her, GRANT og REVOKE kommer ind.

Example:

GRANT SELECT
ON school.students
TO 'application_user'@'%';

Det betyder ikke:

    Brugeren må gøre alt med databasen.

This means:

    Brugeren må læse students.

Det er netop denne forskel, I har arbejdet med i forbindelse med databasebrugere og rettigheder.
Bruger og host – to dele af en databasekonto

Når I arbejder med MariaDB-rettigheder, vil I ofte se en konto skrevet sådan:

'username'@'host'

De to dele beskriver forskellige ting.

Example:

'incident_user'@'%'

betyder:

    'incident_user'= who is trying to log in?
    '%' = hvorfra må brugeren logge ind?

Brugeren

Den første del identificerer databasekontoen:

'incident_user'

Det er altså den bruger, som skal autentificeres.
Host

Den anden del angiver, hvorfra kontoen må anvendes:

'%'

% er et wildcard og betyder i praksis:

    Brugeren må logge ind fra enhver host, der ellers kan nå databaseserveren.

Det kan være praktisk i et undervisningsmiljø, men det er ikke nødvendigvis en god sikkerhedsindstilling.

Hvis brugeren kun skal kunne logge ind fra den lokale computer, kan man i stedet have en konto med en mere begrænset host, eksempelvis:

'incident_user'@'localhost'

Det giver dermed to forskellige sikkerhedsspørgsmål:

    Hvem må logge ind?

and:

    Hvorfra må de logge ind?

Begge dele skal vurderes.
Hvorfor er % interessant?

Forestil jer:

'app_user'@'%'

Hvis databaseserveren samtidig er tilgængelig fra et helt netværk, kan app_user potentielt forsøge at logge ind fra enhver maskine på dette netværk.

Det betyder ikke automatisk, at brugeren er usikker. Brugeren kan stadig have meget begrænsede database-rettigheder.

Men der er to separate angrebsflader:

'app_user'@'%'
│       │
│       └── Hvorfra må kontoen logge ind?
│
└────────── Hvilken konto er det?

Og derefter kommer et tredje spørgsmål:

Hvad må kontoen gøre,
når den først er logget ind?

Example:

'app_user'@'%'

kombineret med:

GRANT SELECT ON school.* ...

betyder derfor grundlæggende:

    Denne bruger kan forsøge at logge ind fra enhver tilladt host og har derefter SELECT-rettigheder på school.

Det er vigtigt at skelne mellem netværksadgang, authentication og database-rettigheder. En begrænset GRANT gør ikke en konto mindre eksponeret på netværket, og en begrænset netværksadgang gør ikke automatisk en bruger mindre privilegeret, når den først er logget ind.

Sikkerhed handler derfor om begge dele:

    Who can come in – from where – and what can they do when they are in?

6. User and host – two parts of a database account

Når I arbejder med MariaDB-rettigheder, vil I ofte se en konto skrevet sådan:

'username'@'host'

De to dele beskriver forskellige ting.

Example:

'incident_user'@'%'

betyder:

    'incident_user'= who is trying to log in?
    '%' = hvorfra må brugeren logge ind?

Brugeren

Den første del identificerer databasekontoen:

'incident_user'

Det er altså den bruger, som skal autentificeres.
Host

Den anden del angiver, hvorfra kontoen må anvendes:

'%'

% er et wildcard og betyder i praksis:

    Brugeren må logge ind fra enhver host, der ellers kan nå databaseserveren.

Det kan være praktisk i et undervisningsmiljø, men det er ikke nødvendigvis en god sikkerhedsindstilling.

Hvis brugeren kun skal kunne logge ind fra den lokale computer, kan man i stedet have en konto med en mere begrænset host, eksempelvis:

'incident_user'@'localhost'

Det giver dermed to forskellige sikkerhedsspørgsmål:

    Hvem må logge ind?

and:

    Where can they log in from?

Both must be assessed.
Hvorfor er % interessant?

Forestil jer:

'app_user'@'%'

Hvis databaseserveren samtidig er tilgængelig fra et helt netværk, kan app_user potentielt forsøge at logge ind fra enhver maskine på dette netværk.

Det betyder ikke automatisk, at brugeren er usikker. Brugeren kan stadig have meget begrænsede database-rettigheder.

Men der er to separate angrebsflader:

'app_user'@'%'
│       │
│       └── Hvorfra må kontoen logge ind?
│
└────────── Hvilken konto er det?

Og derefter kommer et tredje spørgsmål:

Hvad må kontoen gøre,
når den først er logget ind?

Example:

'app_user'@'%'

kombineret med:

GRANT SELECT ON school.* ...

betyder derfor grundlæggende:

    Denne bruger kan forsøge at logge ind fra enhver tilladt host og har derefter SELECT-rettigheder på school.

Det er vigtigt at skelne mellem netværksadgang, authentication og database-rettigheder. En begrænset GRANT gør ikke en konto mindre eksponeret på netværket, og en begrænset netværksadgang gør ikke automatisk en bruger mindre privilegeret, når den først er logget ind.

Sikkerhed handler derfor om begge dele:

    Who can come in – from where – and what can they do when they are in?

7. Fjern ikke sikkerheden for nemheds skyld

En klassisk fejl er at give en bruger:

GRANT ALL

fordi det er nemmere.

Det kan være praktisk under udvikling, men det betyder samtidig, at man giver brugeren langt flere muligheder end nødvendigt.

Hvis en bruger kun skal kunne læse data, er:

GRANT SELECT

bedre end:

GRANT ALL

Princippet er:

    Giv den mindst mulige adgang, som stadig løser opgaven.

8. Et installationsværktøj kan hjælpe – men løser ikke alt

MariaDB leverer blandt andet værktøjet:

mariadb-secure-installation

Det kan hjælpe med en grundlæggende sikkerhedsopsætning af en MariaDB-installation.

Det kan blandt andet hjælpe med forhold omkring administrative konti og unødvendige brugere.

Men det er vigtigt at forstå:

    mariadb-secure-installation er ikke det samme som hardening.

Hardening er en samlet vurdering af serverens sikkerhed.

Man skal stadig overveje:

    network access

    users

    authentication

    authorization

    passwords

    administrative accounts

    database- og objektrettigheder

    firewall

    opdateringer

    logging og overvågning

    backup og recovery

Der findes altså ikke én kommando, som gør en databaseserver "sikker".
9. Docker ændrer ikke de grundlæggende principper

I dette kursus kører MariaDB i Docker.

Det ændrer ikke de grundlæggende sikkerhedsprincipper.

Vi skal stadig spørge:

Hvem kan forbinde?
↓
Hvem kan logge ind?
↓
Hvad må brugeren?
↓
Hvilke data kan brugeren tilgå?
↓
Hvad kan brugeren ændre?

Docker kan hjælpe med isolering og netværkskontrol, men Docker gør ikke automatisk en databaseserver sikker.

A MariaDB container with a port exposed to a network must still be secured like any other database server.
The main concepts

You do not need to be able to do all the details of database security after this course.

But you need to keep the following concepts in mind:
Authentication

Who are you?

Example:

incident_user

Authorization

What can you do?

Example:

SELECT
INSERT
UPDATE
DELETE

Least privilege

Get only the rights that are necessary.
Network exposure

Who can even reach the database server?
Administrative accounts

Powerful accounts must be protected and used for administration – not common database operations.
Hardening

Remove or limit unnecessary functionality and access.
Checklist – basic hardening

When you are responsible for a database server, you can use the following as a simple first check.
Network

    Who should be able to connect to the database server?

    Is the database server's port exposed wider than necessary?

    Are firewall/network rules appropriate?

    Is the database only available from the necessary networks?

Users

    Do we know all the users on the server?

    Are all users necessary?

    Are administrative accounts protected?

    Are there old or unnecessary accounts?

    Uses applications a separate account rather than root?

Authentication

    Do users have appropriately strong passwords?

    Are passwords not reused unnecessarily?

    Is administrative login information particularly protected?

Rights

    Do users have the necessary rights only?

    Avoiding Unnecessarily GRANT ALL?

    Are database, table and object permissions appropriate?

    Can the user do more than it really needs?

Configuration

    Are unnecessary features and accounts removed or restricted?

    Is the database server updated?

    Is there a conscious configuration of the server's network access?

    Is the safety setup documented?

And most importantly

    Can we explain why each user has the rights it has?

    Can we explain who can reach the server – and why?

    Can we explain what would happen if an account were compromised?

The last question is often the best safety test:

    If this account gets compromised, how much damage can the attacker do?

The less unnecessary access the account has, the less the consequence.

# Task 1 – Manage the database server

A database server should not only be able to store data. As an administrator, you also need to be able to examine what the server is doing and what databases and objects exist.
1. Examine the server

Use both MySQL Workbench and SQL to examine the database server.

Examine, among other things:

SHOW DATABASES;
SHOW PROCESSLIST;
SHOW VARIABLES;
SHOW STATUS;

Find at least three pieces of information that you think may be relevant to a database administrator.

For each information, briefly explain:

    What does it show?
    Why would an administrator need it?

2. GUI or SQL?

Find at least one administrative action that you can perform through MySQL Workbench.

Then find the SQL that performs the same action, or perform the action directly with SQL.

Explain briefly:

    What is the benefit of the GUI?

and:

    What is the advantage of being able to perform the action with SQL?

3. Automation

Imagine that every day an administrator has to perform the same database task.

Describe one task that could be automated.

Explain:

    What should be done?
    When should it be carried out?
    What should the result be saved as?
    What will happen if the task fails?

Then make a small script that can perform the task automatically.

# Task 2 – Backup

Now you need to secure your school- database.
1. Make a backup

Make a logical backup of your database with mariadb-dump. .

The backup file must be stored outside the MariaDB container itself.

Make sure that the backup file has actually been created.

If necessary, check the file:

    What does it contain?
    Do you recognize SQL commands?
    Does it contain both structure and data?

2. Create a backup strategy

Imagine that schoolis the database of a real school.

Make a simple backup strategy.

Answer:
Questions 	Your solution
How often is backup taken? 	 
How many backups are stored? 	 
Where are they stored? 	 
What if the database server breaks down? 	 
How to detect a failed backup? 	 
How to test for backup to be restored?

Explain also:

    Why is a Docker volume not the same as a backup?

# Task 3 – Disaster

Now you have to prove that your backup works.
1. Before the disaster

Make sure that your school-database contains data.

Then make a backup.
2. The Disaster

Imagine that an administrator accidentally deletes the database.

Delete school-database:

DROP DATABASE school;

Make sure the database is gone.
3. Restoration

Restore the database from your backup.

Then check:

    Is the database back?
    Are the tables?
    Do you have constraints?
    Are there views?
    Are there a stored procedures?
    Is the original data?

4. Reflection

Answer:

    What would have happened if you hadn't had backup?
    What would happen if the backup file was on the same disk that was corrupted along with the database server?
    Why is it not enough just to make backups – why should you also test restore?
    How long did it take to restore the database?

# 
Day 4 – Diagnosis and optimize
Completion requirements
Database Server / RDBMS – Day 4
Diagnosize and Optimise

On the first three days, we have been working on establishing, building, securing and operating a database server.

Now we're taking the final step:

    How do we find out what is wrong — and how do we know if our solution is actually better?

It's about performance, monitoring, client connections, data relocation and troubleshooting.

We work in particular with:

    indexes

    query performance

    EXPLAIN

    monitoring

    client connection

    data exports and imports

    Defaction

That's the equivalent of target needles 8–11.
1. From operation to diagnosis

A database server may well be properly installed and still have problems.

For example:

    a query can be slow

    a client cannot connect

    a user cannot log in

    a server can be loaded

    Data may be missing

    a database may have been corrupted

The interesting thing is not just:

    "Does the database work?"

But also:

    "How do I find out why it doesn't work as expected?"

Here we can use a mindset that is very similar to debugging in programming.

Problem
↓
Observation
↓
Hypotese
↓
Undersøgelse
↓
Ændring
↓
Måling
↓
Konklusion

It is one of the most important working methods on this day.
2. What does performance mean?
   Concept: Performance

Performance describes, among other things, how effectively a system performs its work.

For example, for example, for a database server, we may be interested in:

    How Long a Quement Takes

    How many rows to examine

    how much work the server performs

    how many queries the server can handle

    whether the server's resources are burdened

Performance is not just:

    "It feels slow."

It's an observation, but not a particularly accurate measurement.

If we want to optimize something, we should be able to compare the situation before and after.
3. Why can a simple query slow?

See this query:

SELECT *
FROM students
WHERE name = 'Peter';

If studentscontains 20 rows, it almost doesn't matter how the database finds Peter.

But imagine:

20 rækker
100 rækker
1.000 rækker
100.000 rækker
10.000.000 rækker

The more data there is, the more important it becomes how the database finds the relevant rows.

For example, the database could have to examine many rows to find those that meet the condition.

This is where indexes get interesting.
4. Indexes

An index is a data structure that the database server can use to find specific data more efficiently.

We worked with indexes on Day 2.

Now we use them in a different way:

On Day 2, we asked:

    "Hvad er et index?"

På Dag 4 spørger vi:

    "Bruger databasen faktisk indexet, og hjælper det?"

That's an important difference.
5. A concrete example

Assume that studentscontains:

id    name
----------------
1     Anna
2     Peter
3     Maria
4     Jonas
...

We perform:

SELECT *
FROM students
WHERE name = 'Peter';

Without a relevant index, the database may need to examine many of the rows to find Peter.

We can create an index:

CREATE INDEX idx_students_name
ON students(name);

Now the database server has a data structure that can be used for postings on name. .

But:

    The fact that an index exists does not necessarily mean that the database uses it for a particular query.

This is exactly what we need to investigate.
6. EXPLAIN
   Concept: EXPLAIN

EXPLAINis a tool to investigate how MariaDB plans to perform a query.

For example, we can write:

EXPLAIN
SELECT *
FROM students
WHERE name = 'Peter';

We can ask the database:

    "How do you intend to perform this query?"

It is a very useful mental model.

EXPLAINDon’t just tell us if the SQL is syntactically correct.

It provides information about the query-planquery plan that the database expects to use.
7. Query-plan
   Concept: Query-plan

A query plan describes how the database server expects to find and process the data that a query requires.

There may be several possible ways to perform the same query.

The database must therefore choose a strategy.

For example:

Query
↓
Find students
↓
Find relevante rows
↓
Returnér resultat

But how does it find the relevant rows?

Among other things, it may depend on:

    What indexes exist

    hvor meget data der findes

    hvilke kolonner der bruges i betingelser

    hvordan tabeller kombineres

    databaseoptimererens vurdering af den forventede arbejdsbyrde

Vi behøver ikke lære hele query-optimeringssystemet.

Vi skal lære at undersøge resultatet.
8. De vigtigste felter i EXPLAIN

Når vi bruger EXPLAIN, kan outputtet indeholde mange forskellige oplysninger.

At this level, we focus on in particular:

    possible_keys

    key

    rows

and possibly:

    type

Example:

possible_keys    key                 rows
---------------  ------------------  ----
idx_students_name idx_students_name  1

The specific output, of course, depends on the database and the query.
9. possible_keys
   Concept: possible_keys

possible_keys viser hvilke indexes, som databaseserveren vurderer kan være relevante for den pågældende del af query'en.

Hvis vi eksempelvis har:

possible_keys
--------------------
idx_students_name

Does this mean that this index is a possible candidate.

But that doesn't necessarily mean:

    "This index is being used."

We'll look at that next.
10. key
    Begreb: key

key viser hvilket index databaseserveren forventer at bruge.

For example, if we get:

possible_keys
--------------------
idx_students_name

key
--------------------
idx_students_name

har databasen altså valgt dette index som en del af planen.

Det er derfor interessant at sammenligne possible_keys og key.

En mulig kandidat er ikke nødvendigvis den valgte løsning.
11. rows
    Begreb: rows

rows giver et estimat for, hvor mange rækker databaseserveren forventer at skulle undersøge i den pågældende del af planen.

Det er ikke nødvendigvis det præcise antal rækker, som faktisk bliver behandlet.

Det er et estimat.

Men estimatet kan stadig være meget nyttigt.

Hvis vi eksempelvis sammenligner:

rows = 100000

med:

rows = 5

kan det være en stærk indikation af, at databasen forventer meget forskelligt arbejde.

Det er netop den slags information, vi kan bruge til at undersøge performance.
12. Sammenlign før og efter et index

The most interesting thing about EXPLAINIn this process, is not to look at one query.

It's to compare.

First:

EXPLAIN
SELECT *
FROM students
WHERE email = 'student1000@example.com';

Then we create a relevant index:

CREATE INDEX idx_students_email
ON students(email);

And driving:

EXPLAIN
SELECT *
FROM students
WHERE email = 'student1000@example.com';

Now we can investigate:

    Did the query plan change?

    Is the index a possible candidate?

    Will the index be selected?

    changed rowssay?

    Why?

This is here EXPLAINBecomes a diagnostic tool.
13. An index is not free

It may be tempting to conclude:

    "If indexes make listings faster, we just need to put indexes on all columns."

It's not a good strategy.

Indexes also have costs.

When data is changed, relevant indexes must also be maintained.

Example:

INSERT
↓
Data ændres
↓
Index skal også opdateres

The same applies to relevant:

    UPDATE

    DELETE

Indexes also use storage space.

Therefore, the question is not:

    “Can we make an index?”

But:

    "Is the gains of this index worth paying for?"

14. Indexes is about workload

Et index giver især mening, når en bestemt type opslag udføres ofte eller ellers er dyrt.

Forestil dig en tabel, hvor vi ofte søger:

SELECT *
FROM students
WHERE email = 'student1000@example.com';

Et index på email kan være relevant.

Men hvis en kolonne næsten aldrig bruges til opslag, kan et index være mindre interessant.

Databaseoptimering handler derfor ikke kun om tabellens struktur.

Det handler også om hvordan databasen faktisk bruges.
15. Store datasæt gør performance synlig

Med få rækker kan forskellen mellem forskellige query-planer være svær at se.

Example:

10 rows

En query er sandsynligvis hurtig uanset strategi.

Men ved:

100.000 rows

or:

1.000.000 rows

The differences become more interesting.

Det betyder ikke, at man altid skal have millioner af rækker for at arbejde med performance.

This means:

    Performanceproblemer bliver ofte tydeligere, når systemet arbejder med realistiske datamængder.

16. Performanceoptimering er en måleproces

It is important not to optimize based on the guess.

A better process is:

1. Observer
   ↓
2. Mål
   ↓
3. Find mulig årsag
   ↓
4. Lav ændring
   ↓
5. Mål igen
   ↓
6. Sammenlign

Example:

Query er langsom
↓
EXPLAIN
↓
Index mangler
↓
Opret index
↓
EXPLAIN igen
↓
Sammenlign

That way, one can argue that a change has had an effect.
17. Monitoring

Performance optimization is about improving something.

But what do we do if we don't know what's wrong?

Her kommer monitorering ind.
Begreb: Monitoring

Monitoring betyder at indsamle eller undersøge information om systemets aktuelle eller historiske tilstand.

Vi har allerede set nogle af værktøjerne på Dag 3.

For example:

SHOW PROCESSLIST;

and:

SHOW STATUS;

as well as:

SHOW VARIABLES;

Disse kommandoer kan bruges som en del af en undersøgelse.
18. SHOW PROCESSLIST

SHOW PROCESSLIST;

kan bruges til at se aktuelle forbindelser og processer.

Det kan være interessant, hvis en bruger siger:

    "Databasen hænger."

Vi kan eksempelvis undersøge:

    hvilke forbindelser der findes

    hvilke forespørgsler der kører

    If anything seems to be active

    if there is anything that looks unusual

Det giver os observationer.

Det giver os ikke automatisk årsagen.
19. SHOW STATUS

SHOW STATUS;

giver statusinformation fra MariaDB-serveren.

Det kan bruges til at undersøge forskellige former for aktivitet og tællere.

I praksis er der mange oplysninger, og det er ikke meningen, at I skal kunne huske dem alle.

Den vigtige færdighed er:

    To know that the server itself contains information that can be used for diagnosis.

20. SHOW VARIABLES

SHOW VARIABLES;

displays the server's configuration variable.

This is relevant if, for example, you want to check how the server is configured.

It can be valuable to distinguish between:

STATUS

and:

VARIABLES

As a simplistic mental model:

VARIABLES
→ Hvordan er serveren konfigureret?

STATUS
→ Hvilken information rapporterer serveren om sin aktuelle drift?

The two types of information can be used together.
21. Database-fejlfinding som debugging

Programming students already know the idea of debugging.

We can use exactly the same thinking on the database server.

Suppose a user says:

    "The database doesn't work."

This is not a very useful diagnosis.

Vi skal først finde ud af, hvad der konkret ikke virker.

Then:

Problem
↓
Observation
↓
Hypotese
↓
Test
↓
Resultat
↓
Ny hypotese eller løsning

It is the same basic working method as in debugging a program.
22. Hypoteser

En vigtig del af fejlfinding er at formulere en hypotese.

Example:

    "Perhaps the database server hasn't started."

Then we'll examine it.

Or:

    "Maybe the client is trying to connect to the wrong gate."

Then we'll examine it.

Or:

    "Perhaps the user lacks rights."

Then we'll examine it.

It's better than random to change things until the problem disappears.
23. En systematisk troubleshooting-model

En god generel model er:

Problem
↓
Observer
↓
Formulér hypotese
↓
Find relevant information
↓
Test hypotesen
↓
Foretag ændring hvis nødvendigt
↓
Test igen

Notice the word observerobserving.

Before we change anything, we need to understand the situation as much as possible.

It reduces the risk of solving the wrong problem.
24. Client connection

So far, you have primarily worked with the connection to MariaDB through MySQL Workbench.

Now we have to make the connection explicit.

A client needs to know where the database server is and how it should connect to it.

Conceptual:

MySQL Workbench
│
│ TCP/IP
│
├── Hostname / IP
├── Port
├── Username
└── Password
│
▼
MariaDB-server

Depending on the situation, the client may also have information about the database it has to work with.
25. Host and port
    Host

Host identifies the machine or address that the client should contact.

For example, it may be:

localhost

eller en IP-adresse eller et DNS-navn.
Port

A database server listens on a particular network port.

The client must therefore know which port it should contact.

In a Docker environment, it is important to distinguish between:

    the gate inside the container

    the port exposed from Docker to the host machine;

For example, Docker can configure:

Windows host
│
│ port 3306
▼
Docker
│
│ port 3306
▼
MariaDB

The specific port number depends on the configuration you have chosen.
26. Loginoplysninger

Klienten skal også identificere den databasebruger, der skal anvendes.

Det betyder typisk:

Username
Password

Men loginoplysninger alene er ikke nok.

Brugeren skal også have de nødvendige rettigheder.

It's directly related to Day 2.

Client
↓
Connection
↓
Authentication
↓
Authorization
↓
Database operation

This means that a user may well be able to connect to the server, but still could not perform the desired SQL.
27. A connection can fail in several places

If a client cannot connect to the database, the problem may lie in different places.

Example:

Klient
↓
Netværk
↓
Host
↓
Port
↓
MariaDB-server
↓
Bruger
↓
Password
↓
Permissions

Det er derfor ikke nok bare at sige:

    "Login virker ikke."

Vi skal finde ud af hvor i kæden fejlen opstår.
28. Test forbindelsen fra bunden

En god troubleshooting-teknik er at etablere forbindelsen igen i stedet for blindt at stole på en gammel konfiguration.

Find blandt andet:

    hostname

    port

    database

    the username

    password

    hvilken server der faktisk kontaktes

Når man kan forklare alle disse oplysninger, har man også en langt bedre forståelse af forbindelsen.
29. Data movement
    Begreb: Data movement

Data movement betyder at flytte data:

    til databaseserveren

    fra databaseserveren

    mellem databaser

    mellem databasen og andre dataformater

Det kan eksempelvis være relevant, når man skal:

    migrere data

    importere eksisterende data

    eksportere data

    oprette testdata

    flytte data til et andet system

30. CSV as an example of data relocation

Et simpelt eksempel er CSV.

For example, we may have:

students.csv

med data:

id,name,email
1,Anna,anna@example.com
2,Peter,peter@example.com
3,Maria,maria@example.com

Data kan eksporteres fra en database til et sådant format og senere importeres igen.

Conceptual:

Database
↓
Export
↓
students.csv
↓
Import
↓
Database

Det er en simpel måde at demonstrere dataflytning på.
31. Eksport er ikke det samme som backup

Det er vigtigt ikke at blande disse begreber sammen.

En CSV-eksport kan indeholde data fra én tabel.

En databasebackup kan derimod være beregnet til at kunne rekonstruere en hel database.

Example:

CSV-eksport
└── students-data

mens:

Databasebackup
├── databaseobjekter
├── tabelstruktur
├── constraints
├── data
├── views
└── eventuelt andre relevante objekter

Den præcise indhold afhænger selvfølgelig af den konkrete backupmetode.

Derfor er en CSV-fil ikke automatisk en erstatning for en databasebackup.
32. Hvorfor flytte data?

Dataflytning kan være relevant i mange situationer.

For example:
Test

production
↓
export
↓
test database

Migration

gammelt system
↓
export
↓
nyt system

Import

CSV
↓
database

Det er derfor en praktisk databaseadministrativ færdighed at kunne flytte data mellem systemer.
33. Når data mangler

Imagine:

    "Nogle data er blevet slettet."

There are several possible answers.

Hvis dataene stadig findes et andet sted, kan de måske importeres igen.

Hvis de findes i en backup, kan de måske gendannes derfra.

That is why we must first examine the situation.

Data mangler
↓
Hvor findes dataene?
│
├── anden kopi?
│
├── eksport?
│
└── backup?
↓
Restore

Here, Day 3 and Day 4 are directly linked.
34. Error situation: The user cannot log in

Suppose a user says:

    "I can't log in to the database."

Vi bør ikke straks ændre brugerens password.

Først undersøger vi:

Brugernavn korrekt?
↓
Password korrekt?
↓
Host korrekt?
↓
Serveren kører?
↓
Forbindelsen når serveren?
↓
Brugeren må logge ind fra denne host?

When login is successful, the next question is:

    Does the user have the necessary permissions?

Dette viser igen forskellen mellem authentication og authorization fra Dag 2.
35. Fejlsituation: En query er langsom

Antag:

    "Denne query plejede at være hurtig, men nu tager den meget lang tid."

Vi kan arbejde systematisk.

Først ser vi på query'en.

Derefter kan vi bruge:

EXPLAIN
SELECT ...

Vi undersøger blandt andet:

    query-plan

    relevante indexes

    possible_keys

    key

    rows

Vi kan også undersøge serverens aktuelle aktivitet med:

SHOW PROCESSLIST;

Det er ikke sikkert, at indexet er problemet.

Det er netop derfor, vi undersøger i stedet for at gætte.
36. Fejlsituation: Databasen er væk

Antag:

    "School-databasen findes ikke længere."

First, we will see the problem:

SHOW DATABASES;

Hvis databasen faktisk mangler, er næste spørgsmål:

    Har vi en backup?

Hvis ja, kan vi bruge restore-processen fra Dag 3.

Problem
↓
Backup findes
↓
Restore
↓
Kontrollér database
↓
Kontrollér data

Her bruger vi altså både troubleshooting og recovery.
37. Fejlsituation: Klienten kan ikke forbinde

Hvis MySQL Workbench ikke kan forbinde, skal vi ikke automatisk konkludere:

    "MariaDB is broken."

We can investigate:

Kører serveren?
↓
Er hostname korrekt?
↓
Er porten korrekt?
↓
Kan netværksforbindelsen etableres?
↓
Er brugernavn korrekt?
↓
Er password korrekt?
↓
Har brugeren adgang?

Dette er et godt eksempel på, hvorfor systematisk fejlfinding er vigtig.
38. Monitorering, performance og troubleshooting

De tre begreber hænger sammen, men er ikke det samme.
Monitoring

Vi undersøger systemets tilstand.

Hvad sker der?

Troubleshooting

Vi undersøger et konkret problem.

Hvorfor sker det?

Performanceoptimering

Vi forsøger at forbedre systemets effektivitet.

Kan vi få det til at fungere bedre?

De samme værktøjer kan være relevante i alle tre situationer.

Example:

SHOW PROCESSLIST
EXPLAIN
SHOW STATUS
SHOW VARIABLES

Men spørgsmålet, vi stiller til informationen, er forskelligt.
39. Ændringer skal kunne begrundes

Forestil dig, at du opretter et index og bagefter siger:

    "Nu er databasen hurtigere."

Det er ikke nødvendigvis forkert.

Men hvordan ved du det?

En bedre tilgang er:

Før ændringen
↓
EXPLAIN / måling
↓
Opret index
↓
EXPLAIN / måling
↓
Sammenlign

Nu kan du forklare:

    "Før forventede databasen at undersøge X rækker, og efter ændringen forventede den Y."

Det er langt stærkere end:

    "Jeg tror, det blev hurtigere."

40. Databaseadministration som debugging

Efter fire dage kan vi samle mange af teknikkerne i én model:

             PROBLEM
                │
                ▼
             OBSERVÉR
                │
       ┌────────┴────────┐
       ▼                 ▼
    Server            Database
    status            struktur
       │                 │
       └────────┬────────┘
                ▼
             HYPOTESE
                │
                ▼
             UNDERSØG
                │
       ┌────────┼────────┐
       ▼        ▼        ▼
    EXPLAIN   STATUS   PROCESSLIST
       │        │        │
       └────────┼────────┘
                ▼
             ÆNDRING
                │
                ▼
             MÅL IGEN
                │
                ▼
            KONKLUSION

That's not necessarily the only way to troubleshoot.

Men det er en nyttig arbejdsmodel.
41. Hvad kan EXPLAIN ikke fortælle dig?

Det er også vigtigt at kende værktøjernes begrænsninger.

EXPLAIN fortæller om den plan, databaseserveren forventer at bruge.

Det betyder ikke nødvendigvis, at:

    "Hvis EXPLAIN ser godt ud, er query'en altid hurtig."

Den faktiske performance kan også påvirkes af blandt andet:

    mængden af data

    hardware

    contemporary enquiries

    cache

    server load

    network

    other parts of the system

Derfor er EXPLAIN ét diagnostisk værktøj blandt flere.
42. Optimér ikke uden et problem

Det kan være fristende at ændre ting, bare fordi man kan.

Example:

    "Vi laver indexes på alle kolonner."

Or:

    "Vi ændrer serverens konfiguration."

Det er ikke nødvendigvis god administration.

En bedre tilgang er:

Problem
↓
Måling
↓
Analyse
↓
Målrettet ændring
↓
Ny måling

Optimization should be a response to an identified problem.
43. The overall model for the four days

We can now see the whole process as one context.
Day 1 – Understand and install

Hvad er en databaseserver?
Hvordan etablerer vi den?

Day 2 – Build and secure

Hvad ligger i databasen?
Hvem må gøre hvad?

Day 3 – Manage and restore

Hvordan driver vi systemet?
Hvordan beskytter vi data?
Hvordan gendanner vi systemet?

Day 4 – Diagnosis and optimize

Hvordan finder vi problemer?
Hvordan måler vi dem?
Hvordan forbedrer vi systemet?

Dermed bevæger forløbet sig gennem hele livscyklussen:

        ETABLÉR
           ↓
          BYG
           ↓
         SIKR
           ↓
         DRIFT
           ↓
        BACKUP
           ↓
       GENDAN
           ↓
       OBSERVÉR
           ↓
       DIAGNOSTICÉR
           ↓
        OPTIMÉR

44. Measuring Needle 8 – Performance

At optimere en databaseserver handler blandt andet om at kunne undersøge:

    indexes

    query plans

    stored procedures

    statistik

    serverens aktivitet

På dette niveau arbejder vi især med indexes og EXPLAIN.

Det vigtigste er ikke at kunne alle detaljer om databaseoptimering.

Det er at kunne:

    Identify a performance problem

    undersøge query'en

    undersøge query-planen

    identificere relevante indexes

    ændre databasen

    undersøge resultatet igen

45. Målpind 9 – Klienttilslutning

En klientforbindelse består ikke bare af:

    "Tryk Connect."

You must be able to identify the information that makes the connection possible:

Host
Port
Database
Username
Password

og forstå, at forbindelsen går fra klienten gennem netværket til databaseserveren.

Hvis forbindelsen fejler, skal du kunne undersøge de enkelte dele i stedet for blot at konstatere, at "databasen ikke virker".
46. Målpind 10 – Data til og fra databaseserveren

Data can be moved between systems through different formats and tools.

A simple example is:

Database
↓
CSV
↓
Database

Men forskellige dataflytningsmetoder har forskellige formål.

En eksport af tabeldata er ikke automatisk en komplet databasebackup.

Det er vigtigt at vælge metode efter formålet.
47. Målpind 11 – Fejlfinding og gendannelse

Fejlfinding handler om systematisk at undersøge problemer.

Gendannelse handler om at få systemet eller data tilbage efter en fejl.

De hænger sammen:

Troubleshooting
↓
Hvad er problemet?
↓
Hvad kan løse det?
↓
Recovery hvis data er gået tabt

De værktøjer og metoder, vi har arbejdet med gennem forløbet, kan derfor kombineres.
48. What should you explain after Day 4?

You should be able to explain:
Performance

    What Database Performance Means

    Why Data Volume Affects Performance

    What an Inexiance Is Used for

    why indexes also have costs

    Why not just need to index everything

EXPLAIN

    what EXPLAINused for

    What a query plan is

    what possible_keysMeans

    what keyMeans

    what rowsMeans

    how EXPLAINcan be used to examine an index

Monitoring

    Why you monitor a database server

    what SHOW PROCESSLISTcan be used for

    what SHOW STATUScan be used for

    what SHOW VARIABLEScan be used for

Troubleshooting

    How to Work Systemically with a Database Problem

    Why to formulate hypotheses

    Why you should Investigate Before Changing

    Why One Should Measure After a Change

Client connection

    what host, port, database, username and password means

    how a client connects to a database server

    how to systematically troubleshoot a connection error

Data movement

    what data export and data imports mean

    Why Data Can Be Moved Between Systems

    why a CSV export is not necessarily a database backup

Recovery

    How Troubleshooting Can Lead to Recovery

    how a backup from Day 3 can be used if data has been lost

49. The central model

If you only need to bring one model from Day 4, then it is this:

                  PROBLEM
                     │
                     ▼
                 OBSERVÉR
                     │
                     ▼
                FORMULÉR
                 HYPOTESE
                     │
                     ▼
                  UNDERSØG
                     │
          ┌──────────┼──────────┐
          ▼          ▼          ▼
       EXPLAIN     STATUS    PROCESSLIST
          │          │          │
          └──────────┼──────────┘
                     ▼
                  ÆNDRING
                     │
                     ▼
                  MÅL IGEN
                     │
                     ▼
                 KONKLUSION

In reality, it is the same working method that is used in good debugging of programs.

The difference is that the system we debugger, this time is a database server.

A database server should not only be able to store data.

It must be operated, observed, debugged, optimized and restored.

# 
Task 1 – Find and optimize a slow query
Completion requirements
Task 1 – Find and optimize a slow query

A database may contain correct data and still be slow.

You need to investigate how an index affects a query.
1. Create data

Make sure one of your tables contains at least a few thousand rows.

You can generate test data with SQL or otherwise.
2. Make a query

Find a query where the database needs to find specific rows from a column.

Example:

SELECT *
FROM students
WHERE email = 'student1000@example.com';

3. Examine the query plan

Drive:

EXPLAIN
SELECT *
FROM students
WHERE email = 'student1000@example.com';

Examine, among other things:

    possible_keys
    key
    rows

Explain in your own words:

    How does the database expect to find the data?

4. Create an index

Create an appropriate index on the column you're searching for.

Then Drive EXPLAINagain.

Compare the result before and after.

Answer:

    Did the query plan change?
    Is the index being used?
    What does it mean rows?
    Why aren't indexes just something you should put on all columns?

# Task 2 – Find the problem

An administrator has received the following message:

    "The database doesn't work properly."

It is your job to find out what the problem is. You cannot get in touch with the employee in question, as you have gone home sour because of the problem. Your boss demands it loose today.

You know that the employee in question works broadly with the data in the database. They make both statistics and are responsible for the correctness of the data.
Before you start

Run the delivered Incident 2 script against your school- database.

The script creates a controlled error state in the database. You will not be told in advance what the script has done, as it is precisely part of the task to find the problem. Don't read the script!

The script only changes conditions that belong to this exercise.

Once the script is run, follow the instructions below.
Investigate the problem

Examine the database server with relevant tools and SQL commands.

Log in to the database server as incident_user. .

Bruger: incident_user
Password: Incident123!
Database: school

Check the database and test what actions the user can perform.

The user is created to be able to work with the school data, but something is wrong with the configuration.
Test relevant actions on the database until you find the problem.

Once you have found the problem

Find the specific cause of the problem and correct it.

You must be able to demonstrate:

    What the User Tried to Do
    what worked and what didn't work
    What the cause of the problem was
    How you found the cause
    What you changed to solve the problem
    that the action now works

Finally, test the solution by logging in as incident_userand implement the action that previously failed.
Documentation

Document your investigations along the way.

Your documentation must at least show:

    Your hypotheses (why doesn't work)
    How you researched them
    the results of the studies
    the specific cause of the problem
    the SQL you used to solve the problem
    evidence that the solution works

# Task 2 - Incident 2 Setup
refer to: incident_2_setup_student.sql
OBS: UNDLAD AT LÆSE DENNE FIL IGENNEM!

# Task 3 – Client connection and data
Part A – Client Connection

Disconnect your existing connection to MariaDB.

Create the connection again.

Find and document:

    hostname
    port
    database
    the username
    password
    which database server you connect to

Then explain:

    What happens from the moment you press "Connect" until you can send SQL to the database?

Part B – Move data

Export data from at least one table from your school- database.

Then create a new database, for example:

school_test

Import the data into the new database.

Make sure the data has come up correctly.

Answer:

    What is the difference between exporting data and taking a database backup?

# 
Understand & Install - Teacher notes for "talk inspiration"
Completion requirements
Understand & Install - Teacher notes for "talk inspiration"
1. What is a RDBMS?

RDBMS = Relational Database Management System.

It's not just "a database."

A database is roughly the data:

students
courses
enrollments

A RDBMS is the software/system that manages this data.

It handles, among other things:

    storage of data
    SQL
    transactions
    contemporaries
    access control
    metadata
    indexes
    backup/recovery
    communication with clients
    query optimization

Examples of RDBMSs:

    MariaDB
    MySQL
    PostgreSQL
    Microsoft SQL Server
    Oracle Database

This is important because students easily get this mental model:

    "MySQL is a database."

More precisely:

    MySQL is a RDBMS that can manage many databases.

2. Client/server model

A RDBMS is usually a server process that clients connect to.

┌──────────────────┐
│      Client      │
│                  │
│ MySQL Workbench  │
│                  │
└────────┬─────────┘
│
│ SQL / network protocol
│
▼
┌──────────────────┐
│   Database       │
│     Server       │
│                  │
│      RDBMS       │
│                  │
└──────────────────┘

It is important to separate:

Client

    "I want something done."

from:

RDBMS

    "I manage the database and carry out your query."

MySQL Workbench is not the database itself.

It is an administration tool/client.
3. The server is a process

It's a small but important mental model.

When we say:

    "Database server"

It may sound like it's necessarily a physical machine.

It's not. It's not.

It can be:

Physical machine
│
└── Operating system
│
└── Docker
│
└── Database container
│
└── RDBMS process

Or:

Physical server
│
└── VM
│
└── RDBMS

Or just:

Laptop
│
└── RDBMS process

It is the role/process that makes it the database server.

It fits really well with the Docker teaching.
4. Server → database → objects

Now comes the next level.

An RDBMS can manage multiple databases.

RDBMS Server
│
├── Database A
│    ├── Tables
│    ├── Views
│    ├── Indexes
│    └── ...
│
├── Database B
│    ├── Tables
│    └── ...
│
└── Database C

Students must therefore be able to distinguish between:

    RDBMS

and

    Database

and

    Table

It sounds banal, but it's one of those places where beginners often mix the levels together.
5. What is a Schema?

Here we have to be a little careful because schema means a little different things across RDBMSs.

In the general relational world, schema can be used about the logical structure of the database:

    What tables, columns, relationships, etc. are available?

But concrete RDBMSs use the term differently.

In some systems, for example, schema is a kind of namespace inside a database.

That's why I didn't want to turn schema into a central day-1 concept.

Eleverne skal bare vide:

    "RDBMS'er har forskellige måder at organisere databaseobjekter logisk på."

Vi kan senere forklare præcis, hvordan vores valgte RDBMS gør det.
6. Databaseobjekter

A database doesn't just consist of tables.

The key objects we want to work with are:

Database
│
├── Tables
├── Indexes
├── Views
├── Constraints
├── Stored Procedures
├── Functions
└── Users / permissions

They have different roles.
Table

Gemmer data.
Index

En datastruktur, der kan gøre opslag hurtigere.
View

En gemt databaseforespørgsel, som kan præsenteres som en tabel-lignende struktur.
Constraint

Regler, der beskytter dataintegriteten.

Fx:

PRIMARY KEY
FOREIGN KEY
UNIQUE
NOT NULL
CHECK

Stored Procedure / Function

Kode, der ligger i databasen.
User / permissions

Styrer hvem der må gøre hvad.

Vi kommer til at bruge disse på dag 2.
7. Logical storage vs. physical storage

Her kommer den første egentlige infrastrukturtankegang.

Når en elev siger:

    "Tabellen ligger i databasen."

is it logically correct.

Men fysisk ligger informationen et andet sted.

We can think:

LOGISK
────────────────────

RDBMS
│
└── Database
│
├── Table
├── Index
└── ...

FYSISK
────────────────────

Operating system
│
└── filesystem
│
├── database files
├── log files
└── ...

It's one of the most important points on Day 1:

    En database er både en logisk struktur og noget, der skal repræsenteres fysisk på storage.

8. Database files

Det konkrete RDBMS bestemmer præcis, hvordan data gemmes.

Men generelt har vi:

Database
│
├── Data
│
└── Transaction/log information

Der kan være forskellige fysiske filer, afhængigt af RDBMS.

Det er derfor, vi ikke bør lære eleverne:

    "En database er én fil."

Det er ofte forkert.

Og det bliver særligt vigtigt, når vi taler om:

    storage
    backup
    restore
    performance
    corruption

9. Transaction log

Dette er et centralt begreb.

Når en klient eksempelvis siger:

UPDATE students
SET name = 'Peter'
WHERE id = 17;

skal databasen ikke bare tænke:

    "Skriv den nye værdi."

Den skal også kunne håndtere:

    transactions
    crash
    rollback
    recovery
    samtidighed

Derfor findes der en form for transaction logging.

Forenklet:

Client
│
│ UPDATE
▼
RDBMS
│
├── Transaction log
│
└── Data storage

Hvis systemet crasher midt i en operation, kan RDBMS'et bruge log-informationen til at bringe databasen tilbage til en konsistent tilstand.

Vi behøver ikke undervise eleverne i alle detaljer om write-ahead logging osv. på H1.

De skal forstå:

    Databasen bruger log-information til at kunne håndtere transaktioner og recovery.

Det bliver meget vigtigt på dag 3.
10. Metadata

Dette er et af de vigtigste dag-1-koncepter.

Eleverne kan spørge databasen:

    "Hvilke tabeller findes?"

    "Hvilke kolonner har denne tabel?"

    "Hvilke indexes findes?"

    "Hvilke brugere findes?"

Det kræver, at RDBMS'et selv gemmer information om sin egen struktur.

Det kaldes:

    metadata

Altså:

    Data om data.

Example:

Actual data:

students
17 | Peter | Jensen
18 | Anna  | Hansen

Metadata:

students has columns:
id
first_name
last_name

11. System catalog / system tables

RDBMS'et har derfor interne strukturer, hvor metadata opbevares og gøres tilgængelige.

Conceptual:

RDBMS
│
├── Your data
│    ├── students
│    ├── courses
│    └── ...
│
└── System metadata
├── databases
├── tables
├── columns
├── indexes
├── users
└── permissions

The concrete implementation varies.

MariaDB, for example, provides access to a lot of metadata through information_schema. .

This means that we can make something that almost feels a little magical the first time:

SELECT *
FROM information_schema.tables;

The database tells us about itself.
12. Docker: where does it fit in?

Docker is not part of the RDBMS.

This is the infrastructure around it:

Docker
│
└── Container
│
└── MariaDB
│
├── Databases
├── Tables
├── Metadata
└── Storage

It is important that students do not learn:

    "Docker is the database server."

But:

    "Docker gives us an isolated environment where we can run our database server."

13. Persistence

And here Docker gets a very nice teaching function.

If we have:

Container
│
└── Database files

and deletes the container, data may disappear.

If we have instead:

Container
│
└── mounted volume
│
└── Database files

data can survive the life of the container.

It gives us three concepts:

Container lifecycle
≠
Database data lifecycle

And it will later be directly relevant for:

    backup is not the same as persistence.

A Docker volume is not automatically a backup.

That's an important point that I'd make sure students actually experience.
The mental model I'd like the students to end up with

If a student after day 1 can draw something in this direction, we are far:

                    CLIENT
                       │
                       │ SQL
                       ▼
              ┌────────────────┐
              │     RDBMS      │
              │     SERVER     │
              │                │
              │  Query engine  │
              │  Transactions  │
              │  Security      │
              │  Metadata      │
              │                │
              └───────┬────────┘
                      │
              ┌───────┴────────┐
              │    DATABASE    │
              │                │
              │ Tables         │
              │ Indexes        │
              │ Views          │
              │ Constraints    │
              └───────┬────────┘
                      │
                physical storage
                      │
                      ▼
              ┌───────────────┐
              │ Files / disk  │
              └───────────────┘

They don't have to be able to draw it perfectly. They need to be able to explain the relationships.

