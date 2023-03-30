## To build the `.jar` file

```shell
mvn clean package
```

- `clean` will delete the `target` directory if it exists.
-  `package` will build the `.jar` file (in `target` directory) and run any tests
- If no profile is defined it will build with default application properties: `src/manin/resources/application.properties`

<details><summary>Building for different environments: <code>dev</code> and <code>prod</code></summary>

---
The application settings may vary dependent on what database you are using. You can apply different properties to different profiles by adding them to the `pom.xml` file:

```xml
    <profiles>
        <profile>
            <id>dev</id>
            <properties>
                <app.properties.path>src/main/resources/application.properties</app.properties.path>
            </properties>
        </profile>
        <profile>
            <id>prod</id>
            <properties>
                <app.properties.path>src/main/resources/application.prod.properties</app.properties.path>
            </properties>
        </profile>
    </profiles>
```
The two files can then define different database connection settings. To use a specific profile you can then specify it to maven like this:

```shell
mvn clean package -P dev
mvn clean package -P prod

```
    
---
</details>

## To run the application

```
java -jar target/*.jar
````

This will start the application and host it on port 8080

<details><summary>How to access your running app</summary>

---
### Local host
If you started it up on your local machine simply browse to http://localhost:8080/

### Codespace
If you are running from a Codespace it will automatically forward port 8080 - if you wait a bit you'll see a popup in the lower right corner:

![image](https://user-images.githubusercontent.com/155492/228451815-311c22e7-b3e4-4e57-8fe5-143079ed641e.png)

If you miss it you can can hit the _**Ports**_ tab in the lower panel. The port 8080 will be listed here - hover over the _**Local Adress**_ three small icons will appera in an overlay

![image](https://user-images.githubusercontent.com/155492/228452215-eda81091-6cdf-4c3c-b31c-083708d2df84.png)

Hit the globe icon.

<img width="444" alt="image" src="https://user-images.githubusercontent.com/155492/228737414-e72bb20e-6ba2-4408-917c-4beceb384edb.png">

### Deployed
If you - or your GitHub Actions - have successfully deployed it to a cloud host - you simply go there - wherever that is.

---
</details>

## How to connect a database

The app uses a database - In the LoginSample we use a MySql database. 

You probably need two databases

- One for development
- Another for production

To simplify the development process we'll skip installing dependencies and instead run them from docker containers. That gives us the benefit that the process is the same wether you're on a Mac, a Windows PC or an online environment and regardless of what IDE you're on.

The following process requires that you have access to the `docker` CLI. If you are in a GitHub Codespace it's already available - if you are on your own PC (Mac or Windows) simply [install and start Docker Desktop](https://www.docker.com/products/docker-desktop/).

<details><summary>Use a MySql database for development hosted on your localhost:3306 - running in a Docker container</summary>

---  
To create the database you simply give the parameters you want to work with when you start it.  The command is one line, but using the back slash character `\` I've broken it up for readability:

```shell
docker run \
  -e MYSQL_USER=loginsample \
  -e MYSQL_PASSWORD=loginsample \
  -e MYSQL_ROOT_PASSWORD=root \
  -e MYSQL_DATABASE=useradmin \
  -v $(git rev-parse --show-toplevel):/app:rw \
  --workdir /app \
  --name mysql1 \
  -p 3306:3306 \
  -d mysql:8.0
```
You should change the values of the parameters to suit your context and application. 

**Notes - on potential issues**

- You should consider better passwords and you should consider _not_ passing in passwords in plain text on the command line, but store them as environment variables or repository secrets.
- If the container is stopped - and you want to reuse it simply start it again:
  ```shell
  docker container restart mysql1
  ```
- If you want to wipe and recreate the container - and the database in it. It you must first kill and remove it. For that you need to obtain the `CONTAINER ID`
  ```shell
  docker container ls
  ```

  Then kill the process (if it's running) and remove the container:
  ```
  docker kill <CONTAINER-ID>
  docker container rm <CONTAINER-ID>
  ```
  ...and you are good to go again
- You can run multiple containers - it's practical if you want to try out different approaches of development. But simply running the same command again will clash both on conflicting names and published ports. You can have the same host run multiple instances by publishing the docker container's port `3306` on a different port on the host say `3307`and then come up with a new name say `mysql2`; Starting a `mysql2` container on port `3307` would look like this:
  ```shell
  docker run \
    -e MYSQL_USER=loginsample \
    -e MYSQL_PASSWORD=loginsample \
    -e MYSQL_ROOT_PASSWORD=root \
    -e MYSQL_DATABASE=useradmin \
    -v $(git rev-parse --show-toplevel):/app:rw \
    --workdir /app \
    --name mysql2 \
    -p 3307:3306 \
    -d mysql:8.0
  ```

---
</details>

## Working with your database

You can manipulate your database in different ways:

- From a sql console
- Executing `.sql` files against it
- Through a database management tool 

<details><summary>Using a console</summary>
    
---
The Docker container you created has the `mysql` CLI installed, you can utilize that feature from a inside a running container through the `docker exec` command:

```shell
docker exec  -it mysql1 mysql -uloginsample -ploginsample
```

It will open a `mysql` prompt in which you can execute any valid SQL statement:

```sql
mysql> SHOW tables IN useradmin;
+---------------------+
| Tables_in_useradmin |
+---------------------+
| Users               |
+---------------------+
1 row in set (0.00 sec)
mysql> SELECT * from useradmin.Users
    -> ;
+----+---------------------+----------+----------+
| id | email               | password | role     |
+----+---------------------+----------+----------+
|  1 | sometwo@nowhere.com | two      | customer |
|  2 | someone@nowhere.com | one      | customer |
+----+---------------------+----------+----------+
2 rows in set (0.00 sec)
```
    
---
</details>

<details><summary>Executing scripts against your database</summary>
    
---
One approach is to use the `source` command from the `mysql` prompt.

The container has mapped the root of the git repository in as it's `--workdir``

So you should specify your script path relative to that:

```sql
mysql> source src/main/resources/users.sql;
```

Actually you don't even have to enter the `mysql` prompt to do it - you can run it directly from the bash terminal like this:

```shell
docker exec  -i mysql1 mysql -uloginsample -ploginsample  < src/main/resources/users.sql

```

---
</details>

<details><summary>Using a database management tool</summary>
    
---
In a tool like MySql Workbench all you need to set up a connection is:

- **username**
- **password**
- **database**
- **host**
- **port**


---
</details>



