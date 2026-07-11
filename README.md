# pterodactyl_docker
this configuration is used for running in `docker` and only uses one machine.\
i have used these `docker-compose` file for references:
- https://github.com/pterodactyl/wings/blob/develop/docker-compose.example.yml
- https://github.com/pterodactyl/panel/blob/1.0-develop/docker-compose.example.yml
> [!note]
> - in this setup, you will need a domain/subdomain, you can use `duckdns` if you don't have one.

## setup
1. clone the repository
```bash
git clone https://github.com/rinkomizu/pterodactyl_docker
```

2. clone the [.env.example](./.env.example) file to `.env`
```bash
cp .env.example .env
```

3. generate the `base64` for pterodactyl panel and then **COPY** it (don't exclude the `base64:` prefix).
```bash
echo "base64:$(openssl rand -base64 32)"
```

4. **READ ALL COMMENTS** in the `.env` file **CAREFULLY**, and follow the instructions inside the file.
5. edit the [Caddyfile](./Caddyfile), change the `panel.example.com` and `wings.example.com` to the domain or subdomains that you own.
6. edit the [docker-compose.yml](./docker-compose.yml) file.
> [!note]
> - READ ALL COMMENTS inside the file carefully.
> - on line `53`, you can edit the `panel`'s environment variables.

7. after finish all of the configuration above, start all containers inside this directory.
```bash
sudo docker compose up -d
```

8. finalize the database setup for pterodactyl panel.
```bash
sudo docker compose exec panel php artisan migrate --seed --force
```

9. create an ADMINISTRATOR user on the pterodactyl (create an ADMINISTRATOR account is required at first).
```bash
sudo docker compose exec panel php artisan p:user:make
```

## setup the wings
> [!note]
> - you can only follow this section if you have finished the [#setup](#setup) section above.\
> - for the uploading limit to the panel, you should use `1024`M by default.

1. open a browser and go to `https://panel.example.com` (where `panel.example.com` is the subdomain/domain that you own)
2. log into the ADMINISTRATOR account from step 9 from [#setup](#setup) section.
3. On the top, there's a bar, find the button when you hover on, it will popup the word `Admin`, and then click on it.
4. `Management` > `Locations`, and click on `Create new` and create a new location.
5. `Management` > `Nodes`, click on `Create new`, and follow the instructions:
- `name`: the name of server/node you want.
- `description`: you can leave it blank or add description about the server/node.
- `location`: the location created in step 4.
- `Node Visibility`: this one is up to you.
- `FQDN`: use `https://wings.example.com` (where the `wings.example.com` is the subdomain/domain you used for the wings from first section)
- `Communication over SSL`: just keep SSL by default, unless you make the setup use `http://` then you will need to use to check `Use HTTP connection` instead.
- `Behind Proxy`: use `Not behind proxy` by default unless you use Cloudflare with orange cloud in the DNS settings
> [!note]
> the orange cloud will proxy request to cloudflare server before the traffic reaches your server.
- `daemon server file directory`: ignore it, just leave it to the pterodactyl decide.
- `daemon port`: ignore it, just leave it to the pterodactyl decide.
- `daemon sftp`: ignore it, just leave it to the pterodactyl decide.

7. `Management` > `Nodes` > `Configuration`, copy all the configuration file content, it should look like this:
```yaml
debug: false
uuid: the_long_uuid_that_is_automatically_generated_by_pterodactyl
token_id: the_long_string_that_is_automatically_generated_by_pterodactyl
token: the_long_token_that_is_automatically_generated_by_pterodactyl
api:
  host: 0.0.0.0
  port: 8080
  ssl:
    enabled: false
    cert: /etc/letsencrypt/live/wings/fullchain.pem
    key:/etc/letsencrypt/live/wings/privkey.pem
  upload_limit: 100
system:
  data: /var/lib/pterodactyl/volumes
  sftp:
    bind_port: 2022
allowed_mounts: []
remote: 'https://panel.example.com'
```
- and then use a text editor and change the configuration a bit, make sure to double check and know what you are doing, and then paste it into [./data/wings/config.yml](./data/wings/config.yml) file.
- you should change the `upload_limit` from `100` to `1024`M.
8. and then go back to your terminal, restart all containers in this directory:
```bash
sudo docker compose down
sudo docker compose up -d
```
9. go back to your browser, `Management` > `Servers`, click on `Create new` and few in the blanks.
10. you should be able to do the rest if you have gone this far, i need to stop here because the rest is up to you, some names and RAM configuration is already on the web, you can read it and setup it.

# errors
if the pterodactyl panel setup has error, you should debug by checking docker logs:
```bash
sudo docker compose logs -f
```

- most errors are in the [./data/wings/config.yml](./data/wings/config.yml).
