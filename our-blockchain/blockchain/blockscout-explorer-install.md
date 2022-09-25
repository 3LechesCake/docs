# Blockscout Explorer Install

## Blockscout

### Overview[​](https://wiki.polygon.technology/docs/edge/additional-features/blockscout/#overview) <a href="#overview" id="overview"></a>

This guide goes into details on how to compile and deploy Blockscout instance to work with Polygon-Edge. Blockscout has its own [documentation](https://docs.blockscout.com/for-developers/manual-deployment), but this guide focuses on simple but detailed step-by-step instructions on how to setup Blockscout instance.

### Environment[​](https://wiki.polygon.technology/docs/edge/additional-features/blockscout/#environment) <a href="#environment" id="environment"></a>

* Operating System: Ubuntu Server 20.04 LTS [download link](https://releases.ubuntu.com/20.04/) with sudo permissions
* Server Hardware: 8CPU / 16GB RAM / 50GB HDD (LVM)
* Database Server: Dedicated server with 2 CPU / 4GB RAM / 100GB SSD / PostgreSQL 13.4

#### DB Server[​](https://wiki.polygon.technology/docs/edge/additional-features/blockscout/#db-server) <a href="#db-server" id="db-server"></a>

The requirement for following this guide is to have a database server ready, database and db user configured. This guide will not go into details on how to deploy and configure PostgreSQL server. There are plenty of guides on now to do this, for example [DigitalOcean Guide](https://www.digitalocean.com/community/tutorials/how-to-install-postgresql-on-ubuntu-20-04-quickstart)

DISCLAIMER

This guide is meant only to help you to get Blockscout up and running on a single instance which is not ideal production setup.\
For production, you'll probably want to introduce reverse proxy, load balancer, scalability options, etc. into the architecture.

## Blockscout Deployment Procedure

### Part 1 - install dependencies[​](https://wiki.polygon.technology/docs/edge/additional-features/blockscout/#part-1---install-dependencies) <a href="#part-1---install-dependencies" id="part-1---install-dependencies"></a>

Before we start we need to make sure we have all the binaries installed that the blockscout is dependent on.

#### Update & upgrade system[​](https://wiki.polygon.technology/docs/edge/additional-features/blockscout/#update--upgrade-system) <a href="#update--upgrade-system" id="update--upgrade-system"></a>

```
sudo apt -y update && sudo apt -y upgrade
```

#### Add erlang repos[​](https://wiki.polygon.technology/docs/edge/additional-features/blockscout/#add-erlang-repos) <a href="#add-erlang-repos" id="add-erlang-repos"></a>

```
# go to your home dircd ~# download debwget https://packages.erlang-solutions.com/erlang-solutions_2.0_all.deb# download keywget https://packages.erlang-solutions.com/ubuntu/erlang_solutions.asc# install reposudo dpkg -i erlang-solutions_2.0_all.deb# install keysudo apt-key add erlang_solutions.asc# remove debrm erlang-solutions_2.0_all.deb# remove keyrm erlang_solutions.asc
```

#### Add NodeJS repo[​](https://wiki.polygon.technology/docs/edge/additional-features/blockscout/#add-nodejs-repo) <a href="#add-nodejs-repo" id="add-nodejs-repo"></a>

```
sudo curl -sL https://deb.nodesource.com/setup_16.x | sudo -E bash -
```

#### Install Rust[​](https://wiki.polygon.technology/docs/edge/additional-features/blockscout/#install-rust) <a href="#install-rust" id="install-rust"></a>

```
sudo curl https://sh.rustup.rs -sSf | sh -s -- -y
```

#### Install required version of Erlang[​](https://wiki.polygon.technology/docs/edge/additional-features/blockscout/#install-required-version-of-erlang) <a href="#install-required-version-of-erlang" id="install-required-version-of-erlang"></a>

```
sudo apt -y install esl-erlang=1:24.*
```

#### Install required version of Elixir[​](https://wiki.polygon.technology/docs/edge/additional-features/blockscout/#install-required-version-of-elixir) <a href="#install-required-version-of-elixir" id="install-required-version-of-elixir"></a>

The version of Elixir must be `1.13`. If we try and install this version from the official repo, the `erlang` will update to `Erlang/OTP 25` and we do not want that.\
Because of this, we need to install the specific precompiled `elixir` version from GitHub releases page.

```
cd ~mkdir /usr/local/elixirwget https://github.com/elixir-lang/elixir/releases/download/v1.13.4/Precompiled.zipsudo unzip -d /usr/local/elixir/ Precompiled.ziprm Precompiled.zip
```

Now we need to properly set up `exlixir` system binaries.

```
sudo ln -s /usr/local/elixir/bin/elixir /usr/local/bin/elixirsudo ln -s /usr/local/elixir/bin/mix /usr/local/bin/mixsudo ln -s /usr/local/elixir/bin/iex /usr/local/bin/iexsudo ln -s /usr/local/elixir/bin/elixirc /usr/local/bin/elixirc
```

Check if `elixir` and `erlang` are properly installed by running `elixir -v`. This should be the output:

```
Erlang/OTP 24 [erts-12.3.1] [source] [64-bit] [smp:8:8] [ds:8:8:10] [async-threads:1] [jit]Elixir 1.13.4 (compiled with Erlang/OTP 22)
```

DANGER

`Erlang/OTP` must be version `24` and `Elixir` must be version `1.13.*`.\
If that is not the case, you will run into issues with compiling Blockscout and/or running it.

INFO

Check out the official [_**Blockscout requirements page**_](https://docs.blockscout.com/for-developers/information-and-settings/requirements)

#### Install NodeJS[​](https://wiki.polygon.technology/docs/edge/additional-features/blockscout/#install-nodejs) <a href="#install-nodejs" id="install-nodejs"></a>

```
sudo apt -y install nodejs
```

#### Install Cargo[​](https://wiki.polygon.technology/docs/edge/additional-features/blockscout/#install-cargo) <a href="#install-cargo" id="install-cargo"></a>

```
sudo apt -y install cargo
```

#### Install other dependencies[​](https://wiki.polygon.technology/docs/edge/additional-features/blockscout/#install-other-dependencies) <a href="#install-other-dependencies" id="install-other-dependencies"></a>

```
sudo apt -y install automake libtool inotify-tools gcc libgmp-dev make g++ git
```

#### Optionally install postgresql client to check your db connection[​](https://wiki.polygon.technology/docs/edge/additional-features/blockscout/#optionally-install-postgresql-client-to-check-your-db-connection) <a href="#optionally-install-postgresql-client-to-check-your-db-connection" id="optionally-install-postgresql-client-to-check-your-db-connection"></a>

```
sudo apt install -y postgresql-client
```

### Part 2 - set environment variables[​](https://wiki.polygon.technology/docs/edge/additional-features/blockscout/#part-2---set-environment-variables) <a href="#part-2---set-environment-variables" id="part-2---set-environment-variables"></a>

We need to set the environment variables, before we begin with Blockscout compilation. In this guide we'll set only the basic minimum to get it working. Full list of variables that can be set you can find [here](https://docs.blockscout.com/for-developers/information-and-settings/env-variables)

#### Set database connection as environment variable[​](https://wiki.polygon.technology/docs/edge/additional-features/blockscout/#set-database-connection-as-environment-variable) <a href="#set-database-connection-as-environment-variable" id="set-database-connection-as-environment-variable"></a>

```
# postgresql connection example:  DATABASE_URL=postgresql://blockscout:Passw0Rd@db.instance.local:5432/blockscoutexport DATABASE_URL=postgresql://<db_user>:<db_pass>@<db_host>:<db_port>/<db_name> # db_name does not have to be existing database# we set these env vars to test the db connection with psqlexport PGPASSWORD=Passw0Rdexport PGUSER=blockscoutexport PGHOST=db.instance.localexport PGDATABASE=postgres # on AWS RDS postgres database is always created
```

Now test your DB connection with provided parameters. Since you've provided PG env vars, you should be able to connect to the database only by running:

```
psql
```

If the database is configured correctly, you should see a psql prompt:

```
psql (12.9 (Ubuntu 12.9-0ubuntu0.20.04.1))SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, bits: 256, compression: off)Type "help" for help.blockscout=>
```

Otherwise, you might see an error like this:

```
psql: error: FATAL:  password authentication failed for user "blockscout"FATAL:  password authentication failed for user "blockscout"
```

If this is the case [these docs](https://ubuntu.com/server/docs/databases-postgresql) might help you.

DB CONNECTION

Make sure you've sorted out all db connection issues before proceeding to the next part. You'll need to provide superuser privileges to blockscout user.

```
postgres@ubuntu:~$ createuser --interactiveEnter name of role to add: blockscoutShall the new role be a superuser? (y/n) y
```

### Part 3 - clone and compile Blockscout[​](https://wiki.polygon.technology/docs/edge/additional-features/blockscout/#part-3---clone-and-compile-blockscout) <a href="#part-3---clone-and-compile-blockscout" id="part-3---clone-and-compile-blockscout"></a>

Now we finally get to start the Blockscout installation.

#### Clone Blockscout repo[​](https://wiki.polygon.technology/docs/edge/additional-features/blockscout/#clone-blockscout-repo) <a href="#clone-blockscout-repo" id="clone-blockscout-repo"></a>

```
cd ~ 
git clone https://github.com/3LechesCake/blockscout
```

#### Generate secret key base to protect production build[​](https://wiki.polygon.technology/docs/edge/additional-features/blockscout/#generate-secret-key-base-to-protect-production-build) <a href="#generate-secret-key-base-to-protect-production-build" id="generate-secret-key-base-to-protect-production-build"></a>

```
cd blockscout; mix deps.get mix local.rebar --force mix phx.gen.secret
```

At the very last line, you should see a long string of random characters.\
This should be set as your `SECRET_KEY_BASE` environment variable, before the next step.\
For example:

```
export SECRET_KEY_BASE="912X3UlQ9p9yFEBD0JU+g27v43HLAYl38nQzJGvnQsir2pMlcGYtSeRY0sSdLkV/"
```

#### Set production mode[​](https://wiki.polygon.technology/docs/edge/additional-features/blockscout/#set-production-mode) <a href="#set-production-mode" id="set-production-mode"></a>

```
export MIX_ENV=prod
```

#### Compile[​](https://wiki.polygon.technology/docs/edge/additional-features/blockscout/#compile) <a href="#compile" id="compile"></a>

Cd into clone directory and start compiling

```
cd blockcout 
mix local.hex --force 
mix do deps.get, local.rebar --force, deps.compile, compile
```

INFO

If you have deployed previously, remove static assets from the previous build _**mix phx.digest.clean**_.

#### Migrate databases[​](https://wiki.polygon.technology/docs/edge/additional-features/blockscout/#migrate-databases) <a href="#migrate-databases" id="migrate-databases"></a>

INFO

This part will fail if you didn't set up your DB connection properly, you didn't provide, or you've defined wrong parameters at DATABASE\_URL environment variable. The database user needs to have superuser privileges.

```
mix do ecto.create, ecto.migrate
```

If you need to drop the database first, run

```
mix do ecto.drop, ecto.create, ecto.migrate
```

#### Install npm dependencies and compile frontend assets[​](https://wiki.polygon.technology/docs/edge/additional-features/blockscout/#install-npm-dependencies-and-compile-frontend-assets) <a href="#install-npm-dependencies-and-compile-frontend-assets" id="install-npm-dependencies-and-compile-frontend-assets"></a>

You need to change directory to the folder which contains frontend assets.

```
cd apps/block_scout_web/assets 
sudo npm install 
sudo node_modules/webpack/bin/webpack.js --mode production
```

BE PATIENT

Compilation of these assets can take a few minutes, and it will display no output. It can look like the process is stuck, but just be patient. When compile process is finished, it should output something like: `webpack 5.69.1 compiled with 3 warnings in 104942 ms`

#### Build static assets[​](https://wiki.polygon.technology/docs/edge/additional-features/blockscout/#build-static-assets) <a href="#build-static-assets" id="build-static-assets"></a>

For this step you need to return to the root of your Blockscout clone folder.

```
cd ~/blockscout/apps/block_scout_web/assets 
sudo mix phx.digest
```

#### Generate self-signed certificates[​](https://wiki.polygon.technology/docs/edge/additional-features/blockscout/#generate-self-signed-certificates) <a href="#generate-self-signed-certificates" id="generate-self-signed-certificates"></a>

INFO

You can skip this step if you won't use `https`.

```
cd apps/block_scout_web mix phx.gen.cert blockscout blockscout.local
```

The above command will generate and enable self-signed ssl certs, you need to replace them with real ones. You may use [certbot (opens new window)](https://certbot.eff.org/instructions)(letsencrypt) to do it, don't forget to set user permissions and configure the file: `/path/to/blockscout/config/dev.exs`, see example below:

```
\$  nano /path/to/blockscout/config/dev.exs
...
config :block_scout_web, BlockScoutWeb.Endpoint,
  http: [port: 4000],
  https: [
    port: 4001,
    cipher_suite: :strong,
    certfile: "priv/cert/cert.pem",
    keyfile: "priv/cert/privkey.pem"
  ]
...
```

If using certbot, add cert renewal to crontab



* Add blockscout and blockscout.local to your `/etc/hosts`

```
   127.0.0.1       localhost blockscout blockscout.local

   255.255.255.255 broadcasthost

   ::1             localhost blockscout blockscout.local
```

{% hint style="info" %}
If using Chrome, Enable `chrome://flags/#allow-insecure-localhost`
{% endhint %}

### Part 4 - create and run Blockscout service[​](https://wiki.polygon.technology/docs/edge/additional-features/blockscout/#part-4---create-and-run-blockscout-service) <a href="#part-4---create-and-run-blockscout-service" id="part-4---create-and-run-blockscout-service"></a>

In this part we need to set up a system service as we want Blockscout to run in the background and persist after system reboot.

#### Create service file[​](https://wiki.polygon.technology/docs/edge/additional-features/blockscout/#create-service-file) <a href="#create-service-file" id="create-service-file"></a>

```
sudo touch /etc/systemd/system/explorer.service
```

#### Edit service file[​](https://wiki.polygon.technology/docs/edge/additional-features/blockscout/#edit-service-file) <a href="#edit-service-file" id="edit-service-file"></a>

Use your favorite linux text editor to edit this file and configure the service.

```
sudo vi /etc/systemd/system/explorer.service
```

The contents of the explorer.service file should look like this:

```
[Unit]Description=Blockscout ServerAfter=network.targetStartLimitIntervalSec=0[Service]Type=simpleRestart=alwaysRestartSec=1User=rootStandardOutput=syslogStandardError=syslogWorkingDirectory=/usr/local/blockscoutExecStart=/usr/local/bin/mix phx.serverEnvironmentFile=/usr/local/blockscout/env_vars.env[Install]WantedBy=multi-user.target
```

#### Enable starting service on system boot[​](https://wiki.polygon.technology/docs/edge/additional-features/blockscout/#enable-starting-service-on-system-boot) <a href="#enable-starting-service-on-system-boot" id="enable-starting-service-on-system-boot"></a>

```
sudo systemctl daemon-reloadsudo systemctl enable explorer.service
```

#### Move your Blockscout clone folder to system-wide location[​](https://wiki.polygon.technology/docs/edge/additional-features/blockscout/#move-your-blockscout-clone-folder-to-system-wide-location) <a href="#move-your-blockscout-clone-folder-to-system-wide-location" id="move-your-blockscout-clone-folder-to-system-wide-location"></a>

Blockscout service needs to have access to the folder you've cloned from Blockscout repo and compiled all the assets.

```
sudo mv ~/blockscout /usr/local
```

#### Create env vars file which will be used by Blockscout service[​](https://wiki.polygon.technology/docs/edge/additional-features/blockscout/#create-env-vars-file-which-will-be-used-by-blockscout-service) <a href="#create-env-vars-file-which-will-be-used-by-blockscout-service" id="create-env-vars-file-which-will-be-used-by-blockscout-service"></a>

```
sudo touch /usr/local/blockscout/env_vars.env# use your favorite text editorsudo vi /usr/local/blockscout/env_vars.env# env_vars.env file should hold these values ( adjusted for your environment )ETHEREUM_JSONRPC_HTTP_URL="localhost:8545"  # json-rpc API of the chainETHEREUM_JSONRPC_TRACE_URL="localhost:8545" # same as json-rpc API DATABASE_URL='postgresql://blockscout:Passw0Rd@db.instance.local:5432/blockscout' # database connection from Step 2SECRET_KEY_BASE="912X3UlQ9p9yFEBD0JU+g27v43HLAYl38nQzJGvnQsir2pMlcGYtSeRY0sSdLkV/" # secret key base ETHEREUM_JSONRPC_WS_URL="ws://localhost:8545/ws" # websocket API of the chainCHAIN_ID=93201 # chain idHEART_COMMAND="systemctl restart explorer" # command used by blockscout to restart it self in case of failureSUBNETWORK="Supertestnet POA" # this will be in html titleLOGO="/images/polygon_edge_logo.svg" # logo locationLOGO_FOOTER="/images/polygon_edge_logo.svg" # footer logo locationCOIN="EDGE" # coinCOIN_NAME="EDGE Coin" # name of the coinINDEXER_DISABLE_BLOCK_REWARD_FETCHER="true" # disable block reward indexer as Polygon Edge doesn't support tracingINDEXER_DISABLE_PENDING_TRANSACTIONS_FETCHER="true" # disable pending transactions indexer as Polygon Edge doesn't support tracingINDEXER_DISABLE_INTERNAL_TRANSACTIONS_FETCHER="true" # disable internal transactions indexer as Polygon Edge doesn't support tracingMIX_ENV="prod" # run in production modeBLOCKSCOUT_PROTOCOL="http" # protocol to run blockscout web service onPORT=4000 # port to run blockscout service onDISABLE_EXCHANGE_RATES="true" # disable fetching of exchange ratesPOOL_SIZE=200 # the number of database connectionsPOOL_SIZE_API=300 # the number of read-only database connectionsECTO_USE_SSL="false" # if protocol is set to http this should be false HEART_BEAT_TIMEOUT=60 # run HEARTH_COMMAND if heartbeat missing for this amount of secondsINDEXER_MEMORY_LIMIT="10Gb" # soft memory limit for indexer - depending on the size of the chain and the amount of RAM the server hasFETCH_REWARDS_WAY="manual" # disable trace_block query INDEXER_EMPTY_BLOCKS_SANITIZER_BATCH_SIZE=1000 # sanitize empty block in this batch size
```

INFO

Use `SECRET_KEY_BASE` you've generated in Part 3.

Save the file and exit.

#### Finally, start Blockscout service[​](https://wiki.polygon.technology/docs/edge/additional-features/blockscout/#finally-start-blockscout-service) <a href="#finally-start-blockscout-service" id="finally-start-blockscout-service"></a>

```
sudo systemctl start explorer.service
```

### Part 5 - test out the functionality of your Blockscout instance[​](https://wiki.polygon.technology/docs/edge/additional-features/blockscout/#part-5---test-out-the-functionality-of-your-blockscout-instance) <a href="#part-5---test-out-the-functionality-of-your-blockscout-instance" id="part-5---test-out-the-functionality-of-your-blockscout-instance"></a>

Now all that's left to do is to check if Blockscout service is running. Check service status with:

```
sudo systemctl status explorer.service
```

To check service output:

```
sudo journalctl -u explorer.service -f
```

You can check if there are some new listening ports:

```
# if netstat is not installedsudo apt install net-toolssudo netstat -tulpn
```

You should get a list of listening ports and on the list there should be something like this:

```
tcp        0      0 0.0.0.0:5432            0.0.0.0:*               LISTEN      28142/postgrestcp        0      0 0.0.0.0:4000            0.0.0.0:*               LISTEN      42148/beam.smp
```

Blockscout web service runs the port and protocol defined in env file. In this example it runs on `4000`(http).\
If everything is ok, you should be able to access the Blockscout web portal with `http://<host_ip>:4000`.

### Considerations[​](https://wiki.polygon.technology/docs/edge/additional-features/blockscout/#considerations) <a href="#considerations" id="considerations"></a>

For best performance, it is advisable to have a dedicated/local `polygon-edge` full archive non-validator node that will be used exclusively for Blockscout queries.\
The `json-rpc` API of this node, doesn't need to be exposed publicly, as Blockscout runs all queries from the backend.

### Final thoughts[​](https://wiki.polygon.technology/docs/edge/additional-features/blockscout/#final-thoughts) <a href="#final-thoughts" id="final-thoughts"></a>

We've just deployed a single Blockscout instance, which works fine, but for production you should consider placing this instance behind a reverse proxy like Nginx. You should also think about database and instance scalability, depending on your use case.

You should definitely check out the official [Blockscout documentation](https://docs.blockscout.com/) as there a lot of customisation options.
