# Installing WordPress via docker container

## Docker Install

1. Start with the following commands to install Docker directly from Docker repos:
    >```sudo apt update``` (Update machine for latest)

    >```sudo apt install ca-certificates curl gnupg lsb-release```

    >```curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg``` (Pulls from repo)

    >```echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/ >linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null```

    >```sudo apt update```

    >```sudo apt install docker-ce docker-ce-cli containerd.io```

    >```sudo usermod -aG docker admin``` (Admin can be replaced with the name of your user)

2. After Installing Docker we will install Docker Compose through its repo:
    > ```sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose``` (Pulling from the repo)

    >```sudo chmod +x /usr/local/bin/docker-compose``` (Changes docker-compose to an executable)


    >```sudo docker-compose --version``` (This will test and see if compose was installed while also showing version)

3. After entering these commands docker should be working.

## Next is to get WordPress working in the docker

1. First thing that we will need to do is create a folder where we can store the container's contents. For this we will make a directory named "wpdocker"
    >```mkdir wpdocker```

2. Then we will cd into the directory and create another file named "docker-compose.yml"
    >```cd wpdocker```

    >```touch docker-compose.yml```

3. Once those have been created we will paste the default docker-compose.yml file from the docs.docker.com website to our yml file

4. Then we will head back to our terminal and then make sure we are in our "wpdocker" directory and we will run the following command which starts the WordPress docker in detached mode:
    >```docker-compose up -d```

5. After this it will take a minute or two and the WordPress docker will be running.

## Final Steps 

1. Last things that you need to do is to open your browser and type in "localhost:8000". This should take you to the setup page for WordPress.

2. From there you just follow along with the instructions and then login after creating your account/site and then you will be in the admin dashboard.

3. If you ever need to shutdown the docker all you need to do is run the following command:
    >```docker-compose down```

    >```docker-compose down --volumes``` (This command will remove everything including the WordPress database)

## Docker Compose file
```
version: "3.9"
    
services:
  db:
    image: mysql:5.7
    volumes:
      - db_data:/var/lib/mysql
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: somewordpress
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wordpress
      MYSQL_PASSWORD: wordpress
    
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
      WORDPRESS_DB_USER: wordpress
      WORDPRESS_DB_PASSWORD: wordpress
      WORDPRESS_DB_NAME: wordpress
volumes:
  db_data: {}
  wordpress_data: {}
  ```
## Images of the application running
![RunningInDocker](https://user-images.githubusercontent.com/73131611/142069413-2da66317-e44f-4371-989c-7f3e5fc1d071.png)
![FirstLogin](https://user-images.githubusercontent.com/73131611/142069495-212f6251-25a8-46df-aaf0-ac9f8931c866.png)
![AdminPanel](https://user-images.githubusercontent.com/73131611/142069598-2217cbec-08cf-4dc6-b40b-cdf0f579bbf7.png)



## Resources

https://docs.docker.com/samples/wordpress/

