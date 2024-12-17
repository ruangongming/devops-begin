Sure, here is the `docker-compose.yml` file you need to create for installing Nexus on an Ubuntu EC2 instance:

1. Open a terminal and create the `docker-compose.yml` file:
   ```sh
   sudo vi docker-compose.yml
   ```

2. Paste the following configuration into the `docker-compose.yml` file:
   ```yaml
   version: "3"
   services:
     nexus:
       image: sonatype/nexus3
       restart: always
       volumes:
         - "nexus-data:/sonatype-work"
       ports:
         - "8081:8081"
         - "8085:8085"
   volumes:
     nexus-data: {}
   ```

3. Save the file by entering `:wq!`.

4. Execute the compose file using the Docker compose command to start the Nexus container:
   ```sh
   sudo docker-compose up -d
   ```

5. Get the admin password by executing the following command:
   ```sh
   sudo docker exec -it ubuntu_nexus_1 cat /nexus-data/admin.password
   ```

6. Access the Nexus UI by going to your browser and entering the public DNS name with port `8081`. Replace `change_to_nexus_publicdns_name` with your actual public DNS name:
   ```
   http://change_to_nexus_publicdns_name:8081
   ```

7. Your username will be `admin`. Use the password obtained from step 5 to log in.

This setup will allow you to run Nexus on your Ubuntu EC2 instance using Docker Compose.