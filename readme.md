# Outermost dokcer-compose.yml
* It includes services of web, php and db.

# dockerfiles/docker-compose.yml
* Will build images if not exist.
* It build local image and need push if you need.

# Modify files in etc
* To revise default-ssl.conf & default.conf for project.
* To revise etc/php/php.ini & conf.d for project
* etc/ssl/cert.crt & cert.key are **just sample** 
    * You need to produce your owns via  
    `openssl req -x509 -nodes -days 3650 -newkey rsa:2048 -keyout demo.key-out demo.crt`
    * But it still will not produce an effective self-signed cert. Please wait for new updating.

























































































