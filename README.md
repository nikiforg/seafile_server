- Create a domain (example.com) and do port forwarding for ports 80 and 443 if needed (these need to be exposed for seafile to work).
- Run a temporary docker container to serve the challenge for TLS certificate creation:
	- `docker run -d --name nginx-temp -p 80:80 -v $(pwd)/nginx.conf.init_tmp:/etc/nginx/conf.d/default.conf:ro -v $(pwd)/certbot/www:/var/www/certbot nginx:stable-alpine`
- Initiate TLS certificate creation (change with correct domain):
	- `docker run --rm -it -v $(pwd)/certbot/conf:/etc/letsencrypt -v $(pwd)/certbot/www:/var/www/certbot certbot/certbot certonly --webroot -w /var/www/certbot -d example.com`
- Clean up residue container:
	- `docker stop nginx-temp`
	- `docker rm nginx-temp`
- Change the volume mount drive/point and also the following lines in `docker-compose.yml`:
	```
	SEAFILE_SERVER_HOSTNAME
	SEAFILE_ADMIN_EMAIL
	```
- Change the domain from `example.com` to personal domain in `nginx.conf`
- Start seafile:
	- `docker compose up -d`
- Fix CSRF and TLS:
	- `docker exec -it seafile /bin/bash`
		- `vi /shared/seafile/conf/seahub_settings.py`
		- Add the following:
			```
			CSRF_TRUSTED_ORIGINS = [
			    'https://example.com',
			    'https://www.example.com',
			]
			```
		- Change `FILE_SERVER_ROOT` and `SERVICE_URL` to point to https instead of http
	- `exit`
- Restart seafile:
	- `docker restart seafile`
- Login to seafile with `SEAFILE_ADMIN_EMAIL` and password "password" and change password immediately.
- Install Seafile app on Android/iOS and set up server. 
