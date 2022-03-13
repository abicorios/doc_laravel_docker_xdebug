# doc_laravel_docker_xdebug
How to start the Laravel project by the Docker, the Xdebug and the VS Code at the Ubuntu 20.04

1. Install the Docker https://docs.docker.com/engine/install/ubuntu/
```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io
```
Do the post install steps https://docs.docker.com/engine/install/linux-postinstall/

```bash
sudo usermod -aG docker $USER
```
Restart the computer and check is it working.
```bash
docker run hello-world
```
2. Install the docker compose 2 https://docs.docker.com/compose/cli-command/
```bash
DOCKER_CONFIG=${DOCKER_CONFIG:-$HOME/.docker}
mkdir -p $DOCKER_CONFIG/cli-plugins
curl -SL https://github.com/docker/compose/releases/latest/download/docker-compose-linux-x86_64 -o $DOCKER_CONFIG/cli-plugins/docker-compose
chmod +x $DOCKER_CONFIG/cli-plugins/docker-compose
docker compose version
```
3. Go to the projects directory and create the template of the project https://laravel.com/docs/9.x/installation#getting-started-on-linux
```bash
curl -s https://laravel.build/laravel-example?with=mysql | bash
```
In the project directory in the `.env` file append the Xdebug settings lines:
```
SAIL_XDEBUG_MODE=debug
SAIL_XDEBUG_CONFIG="client_host=localhost discover_client_host=On"
```
Open project by the `VS Code`. Check `PHP Debug` extension is installed. Create the config `.vscode/launch.json` with lines:
```
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Listen for Xdebug",
            "type": "php",
            "request": "launch",
            "port": 9003,
            "pathMappings": {
                "/var/www/html/": "${workspaceFolder}"
            }
        },
    ]
}
```

Open the terminal and start the web application in the container by the sail tool:
```bash
./vendor/bin/sail up -d
```
More info about the sail: https://laravel.com/docs/9.x/sail

You can use `./vendor/bin/sail` with subcommands: `stop`, `restart`, `build`, `shell`, `root-shell`, `down`, and more. These are `docker compose` subcommands and several additional. You can see the full list of additional subcommands by check `[ "$1" ==` in the source https://github.com/laravel/sail/blob/1.x/bin/sail
The dockerfile is `vendor/laravel/sail/runtimes/8.1/Dockerfile`. The entry point is `public/index.php`. You can add the `public/i.php` with `<?php phpinfo();` and open http://localhost/i.php to check the settings of the Xdebug. And finally you can start the debugging by the graphical interface like on it is shown in the doc https://marketplace.visualstudio.com/items?itemName=xdebug.php-debug and with the browser extension as it is described here https://xdebug.org/docs/step_debug
Also you can short the `./vendor/bin/sail` by some alias in the `~/.bashrc`. For example `alias sail=./vendor/bin/sail`.

4. Remove all (also important data can be deleted, so think about it)
```bash
./vendor/bin/sail down
docker container prune -f #optional
docker image prune -a -f
docker network prune -f #optional
docker volume prune -f #can have important data from a database
```
And remove the project directory.

Check the deletion:
```bash
docker container ls -a
docker images
docker network ls #also several default networks cannot be deleted, it is ok
docker volume ls
```
