### Installation

```
git clone https://github.com/Factors25/symfony-crm.git
```
copy .env.sample to .env and setup the variables
```
sudo docker-compose build
sudo docker-compose up -d
sudo docker exec -it web_app php /app/symfony-crm/bin/console d:m:m --no-interaction
```