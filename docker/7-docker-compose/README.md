# Exercise 7: Docker Compose
Deploy a fully functional WordPress site using Docker Compose with a MySQL database, demonstrating container orchestration and persistent data handling.

### Prerequisites:
- Basic understanding of Docker
- Docker and Docker Compose installed

### Steps:

1. **Create the Project Directory**  
   First, create a project folder on your local machine:
   ```bash
   mkdir wordpress-compose
   cd wordpress-compose
   ```
   >>This already exist, as sample sub directory, you can rename or copy the content from there. 

2. **Define the `docker-compose.yml` File**  
   Inside the `wordpress-compose` folder, create a `docker-compose.yml` file with the following content:
   ```yaml
   services:
     db:
       image: mysql:5.7
       volumes:
         - db_data:/var/lib/mysql
       restart: always
       environment:
         MYSQL_ROOT_PASSWORD: change_me
         MYSQL_DATABASE: wordpress
         MYSQL_USER: user
         MYSQL_PASSWORD: change_me

     wordpress:
       depends_on:
         - db
       image: wordpress:latest
       volumes:
         - wordpress_data:/var/www/html
       ports:
         - "8000:80"
       restart: always
       environment:
         WORDPRESS_DB_HOST: db:3306
         WORDPRESS_DB_USER: user
         WORDPRESS_DB_PASSWORD: change_me
         WORDPRESS_DB_NAME: wordpress

   volumes:
     db_data: {}
     wordpress_data: {}
   ```

   **Key Points**:
   - Two services: `db` (MySQL) and `wordpress`
   - Persistent volumes for database and WordPress content
   - Environment variables for database credentials

3. **Run the Containers**  
   Run the following command to start the WordPress and MySQL containers:
   ```bash
   docker-compose up -d
   ```

   This command will pull the required images (MySQL and WordPress) and start the containers in detached mode.

4. **Access WordPress**  
   After the containers are up and running, open a web browser and go to `http://localhost:8000`. You should see the WordPress setup page.

5. **WordPress Setup**  
   Complete the WordPress installation by entering the site title, admin username, password, and email. Use the credentials you set in the `docker-compose.yml` file for the database connection.

6. **Stop and Remove Containers**  
   To stop the containers:
   ```bash
   docker-compose down
   ```

For **Part 2**, we'll modify the existing `docker-compose.yml` file to add **phpMyAdmin** as a service for managing the MySQL database through a web interface.

### Steps to Modify the `docker-compose.yml`:

1. **Add the phpMyAdmin Service**  
   We'll add a new service for **phpMyAdmin** in the existing `docker-compose.yml` file to connect to the MySQL database.

   Here's the updated `docker-compose.yml`:

   ```yaml
   services:
     db:
       image: mysql:5.7
       volumes:
         - db_data:/var/lib/mysql
       restart: always
       environment:
         MYSQL_ROOT_PASSWORD: change_me
         MYSQL_DATABASE: wordpress
         MYSQL_USER: user
         MYSQL_PASSWORD: change_me

     wordpress:
       depends_on:
         - db
       image: wordpress:latest
       volumes:
         - wordpress_data:/var/www/html
       ports:
         - "8000:80"
       restart: always
       environment:
         WORDPRESS_DB_HOST: db:3306
         WORDPRESS_DB_USER: user
         WORDPRESS_DB_PASSWORD: change_me
         WORDPRESS_DB_NAME: wordpress

     phpmyadmin:
       depends_on:
         - db
       image: phpmyadmin/phpmyadmin:latest
       restart: always
       ports:
         - "8081:80"
       environment:
         PMA_HOST: db
         MYSQL_ROOT_PASSWORD: change_me

   volumes:
     db_data: {}
     wordpress_data: {}
   ```

   **Key Changes**:
   - Added a new service called `phpmyadmin`.
   - This service pulls the `phpmyadmin/phpmyadmin` image and exposes it on port `8080`.
   - The `PMA_HOST` environment variable is set to the database service (`db`), and we use the same MySQL root password (`MYSQL_ROOT_PASSWORD`) for managing the database.

2. **Run the Updated Docker Compose Setup**  
   Now, run the updated setup:
   ```bash
   docker-compose up -d
   ```

   This will start the `db`, `wordpress`, and `phpmyadmin` services.

3. **Access phpMyAdmin**  
   Once the services are running, you can access phpMyAdmin in your browser at:
   ```
   http://localhost:8081
   ```

   - Login with the MySQL credentials:
     - **Username**: root
     - **Password**: example (as set in the `MYSQL_ROOT_PASSWORD` environment variable)

4. **Verify WordPress and phpMyAdmin**  
   - Your WordPress site should still be accessible at `http://localhost:8000`.
   - Use phpMyAdmin to explore the `wordpress` database, view tables, and manage data.

5. **Cleanup** (if needed):
   Stop and remove containers with:
   ```bash
   docker-compose down
   ```

# END OF EXERCISE