# Maniaplanet Docker Dedicated Server

## Requirements

* A machine with Docker engine.
* A [ManiaPlanet Dedicated Server account](https://maniaplanet.com/account/dedicated-servers)

## Usage

### Docker Compose
1. Create a project directory and cd into it.
2. Create an `.env` file and edit its variables:

```shell
LOGIN=mp_dedicated_user # Maniaplanet server account user
PASSWORD=mp_dedicated_pass # Maniaplanet server account password
TITLE=TMStadium@nadeo # Titlepack
TITLE_PACK_URL=https://v4.live.maniaplanet.com/ingame/public/titles/download/TMStadium@nadeo.Title.Pack.gbx # URL to get the titlepack from
TITLE_PACK_FILE=TMStadium@nadeo.Title.Pack.gbx # Titlepack filename
MATCH_SETTINGS=MatchSettings/default.txt # MatchSettings filename
SERVER_NAME="My Server Name" # Server name
SERVER_PASSWORD="" # Password used by players to join

### OPTIONAL ###

### Force the server to open on this IP address and port ###
# FORCE_IP_ADDRESS="0.0.0.0"
# FORCE_IP_PORT="2350"

### Server CFG file name (recommended to leave as default since the variables above override the config) ###
# DEDICATED_CFG="config.txt"
```

3. Create a `docker-compose.yml` file and change the `dedicated_vars.env` filename to match yours:

```yml
services:
  dedicated:
    image: gerassb/maniaplanet-dedicated
    restart: unless-stopped
    env_file: ./dedicated_vars.env
    volumes:
      - ./UserData:/app/dedicated/UserData
      - ./Logs:/app/dedicated/Logs
    ports:
      - 5000:5000
      - 2350:2350
      - "2350:2350/udp"
      - "3450:3450"
      - "3450:3450/udp"
```

4. Create the `Logs` and `UserData` directories (or else Docker will create them as root and there'll be permission issues):
```bash
mkdir UserData Logs
```
5. Run `docker compose up` and your server should start.
> [!IMPORTANT]
> If you're using a virtual LAN service like Hamachi or ZeroTier, you should set `FORCE_IP_ADDRESS` to your service's or friends won't be able to join through the friend or server list.

### PyPlanet

To use PyPlanet, simply use this as your `docker-compose.yml` instead (be sure to edit the `dedicated_vars.env` setting and set `GAME_SETTINGS` to only the file name you'll be using):

```yml
services:
  dedicated:
    image: gerassb/maniaplanet-dedicated
    restart: unless-stopped
    env_file: ./dedicated_vars.env
    volumes:
      - ./UserData:/app/dedicated/UserData
      - ./Logs:/app/dedicated/Logs
    ports:
      - 5000:5000
      - 2350:2350
      - "2350:2350/udp"
      - "3450:3450"
      - "3450:3450/udp"
  db:
    image: mariadb:latest
    restart: unless-stopped
    environment:
      MARIADB_ROOT_PASSWORD: "secret"
      MARIADB_USER: "pyplanet"
      MARIADB_PASSWORD: "pyplanet"
      MARIADB_DATABASE: "pyplanet"
    volumes:
      - "mariadbData:/var/lib/mysql"
  pyplanet:
    image: pyplanet/pyplanet:latest
    restart: unless-stopped
    depends_on:
        - db
        - dedicated
    environment:
        GAME_SETTINGS: "matchsettings.txt"
        PYPLANET_DEBUG: "False"
    volumes:
      - "pyplanetData:/app/server"
      - "./UserData/Maps:/server/UserData/Maps"

volumes:
  tmserverData: null
  pyplanetData: null
  mariadbData: null
```

Start the server with `docker compose up` and wait for the `Welcome to PyPlanet` message to come up. Copy the command it provides and paste it in the in-game chat to claim admin rights.

#### Dedimania
To use Dedimania records, make sure that your `matchsettings.txt` has no `game_mode` block (if it does, delete it entirely) and `script_name` to just the text file name (like `TimeAttack.script.txt`).

Now you just need to link Dedimania to your Maniaplanet account and generate a new code for your server, as instructed [here](https://micmost-net.gitbook.io/maniaplanet-servers-docs/en/setup-dedimania-plugin).