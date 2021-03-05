# Docker Starter template for Magento 2.4 Project

Docker starter Template for Magento 2.4 project

# Example Setup

- Create a www folder
- Edit your vhost in /apache directory
- Edit or adjust docker-compose.yml
- Adjust permisions

# Usage (TODO)
- do
- don't

# Basics Commands

Connect to PHP container
`  docker exec -it docker_httpd bash`

Connect to mysql
`  docker exec -it docker_mysql bash`

# Add a new database or import database

You can setup a new database in the docker-compose.yml

To Import a database, copy your dump in data/mysql

Connect to mysql
`  docker exec -it docker_mysql bash`

Log into mysql (mysql -uUSERNAME -p)
Create and import your database as usual
For example use source /var/lib/mysql/my_database_dump.sql to import your dump
You may have to fix some permissions

# Start your project

Connect into your php container
`  docker exec -it docker_php bash`

Install Magento using composer etc...

# Fix permissions example

In your container run  
`chmod -R www-data /my-folder`





