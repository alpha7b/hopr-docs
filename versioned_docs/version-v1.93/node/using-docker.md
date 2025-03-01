---
id: using-docker
title: Using Docker
---

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

:::info INFO
The instructions below are for Linux and macOS, it is possible to use Windows, but this may result in some issues. Using a VPS if you are on Windows is recommended.
:::

You can use Docker to install a `hoprd` node on your device quickly without worrying too much about the operating system or any additional software. There are, however, some hardware requirements needed to complete the installation.

To use Docker, you will need a device that supports hardware-level virtualisation: VT-x for Intel-based PCs and AMD-V for AMD processors. Most Mac and Linux machines support it out of the box, so just ensure your device meets the following minimum requirements to run `hoprd`:

- Dual Core CPU ~ 2 GHz
- 4 GB RAM
- at least 3 GB Disk Space

At least 8 GB RAM and 10 GB Disk Space is ideal but not required.

## Installing Docker

Before doing anything else, you need to install **Docker Desktop** on your machine.

<Tabs>
<TabItem value="Linux" label="Linux">

Depending on your distribution, please follow the official guidelines to install and run Docker on your workstation.

- [Installing Docker in Ubuntu](https://docs.docker.com/engine/install/ubuntu/)
- [Installing Docker in Fedora](https://docs.docker.com/engine/install/fedora/)
- [Installing Docker in Debian](https://docs.docker.com/engine/install/debian/)
- [Installing Docker in CentOS](https://docs.docker.com/engine/install/centos/)

</TabItem>
<TabItem value="mac" label="macOS">

1. Visit [Docker](https://www.docker.com/get-started) and download Docker Desktop to your computer.
2. Follow the wizard steps to ensure Docker is installed.
3. Ensure the installation was successful by running `docker ps` in your terminal.

</TabItem>
</Tabs>

All our docker images can be found in [our Google Cloud Container Registry](https://console.cloud.google.com/gcr/images/hoprassociation/global/hoprd). Each image is prefixed with `gcr.io/hoprassociation/hoprd`.

## Using Tmux

If you are using a VPS, it is highly recommended that you use Tmux to run your node in the background. Otherwise, your node will terminate as soon as you exit the terminal.

You can use these basic commands to set up a separate session:

(**1**) First, install Tmux.

```bash
sudo apt install tmux
```

(**2**) Enter `tmux` to open a new session.

```bash
tmux
```

That's it! You now have a new session running in the background even when you close your terminal. To navigate between sessions, you should familiarise yourself with other [Tmux commands](https://linuxize.com/post/getting-started-with-tmux/). The three main ones you will need are:

```bash
tmux ls
```

To output a list of all your open sessions.

```bash
tmux attach-session -t <session ID or name>
```

To navigate to a particular session, the first session you have created will have an id of `0`. Use the list command to view all your current sessions.

```bash
ctrl+b d
```

To exit your current session without closing it. To be clear, you press ctrl and b simultaneously, then press d after letting them go.

Please make sure you are in a newly opened session and haven't exited it before continuing.

## Installing HOPRd: 1.93.5 (Monte Rosa)

:::info NOTE

Before downloading the HOPRd image, make sure **Docker** is installed.

:::

All our docker images can be found in [our Google Cloud Container Registry](https://console.cloud.google.com/gcr/images/hoprassociation/global/hoprd).
Each image is prefixed with `gcr.io/hoprassociation/hoprd`.
The `1.93.5` tag represents the latest community release version.

### Install HOPRd with Grafana

:::info INFO
Using this setup will generate a new node, it will not migrate your old node. To keep using your old node, follow the setup without Grafana.
:::

To install HOPRd with Grafana, you can follow the instructions [here.](./grafana-dashboards.md#docker)

**Note:** Following these instructions will generate a node with no authentication, so no security/api token will be needed to access your node.

### Install HOPRd without Grafana

(**1**) Open your terminal.

(**2**) Create a **Security Token** (password) which satisfies the following requirements:

:::danger Requirements

Security token should contain:

- at least 8 symbols
- a lowercase letter
- uppercase letter
- a number
- a special symbol (don't use `%` or whitespace)

This ensures the node cannot be accessed by a malicious user residing in the same network.

:::

(**3**) Copy the following command and replace **YOUR_SECURITY_TOKEN** with your own.

```bash
docker run --pull always --restart on-failure -m 2g --log-driver json-file --log-opt max-size=100M --log-opt max-file=5 -ti -v $HOME/.hoprd-db-monte-rosa:/app/hoprd-db -p 9091:9091/tcp -p 9091:9091/udp -p 8080:8080 -p 3001:3001 -e DEBUG="hopr*" gcr.io/hoprassociation/hoprd:1.93.5 --environment monte_rosa --init --api --identity /app/hoprd-db/.hopr-id-monte-rosa --data /app/hoprd-db --password 'open-sesame-iTwnsPNg0hpagP+o6T0KOwiH9RQ0' --apiHost "0.0.0.0" --apiToken 'YOUR_SECURITY_TOKEN' --healthCheck --healthCheckHost "0.0.0.0"
```

If you are not logged in as the root user, add sudo to the start of the above command. E.g. `sudo docker run ...`. If you are using a VPS, you are likely a root user and can use the default command.

(**4**) Paste the command into your terminal with your unique security token, and hit enter.

(**5**) Wait until the node is installed. This can take up to 10 minutes.

Please note the `--apiToken` (Security token), as this will be used to access hopr-admin or the hoprd UI. It may also be a good idea to note the `--password`, in case you want to decrypt your identity file and retrieve your private key or funds later.

**Note:** Withdrawing funds is possible through hopr-admin/hoprd. This is just a precaution for safekeeping.

### Launching HOPR admin UI

From 1.92 version HOPR admin UI was separated from the HOPRd node. This means you will need additionally to launch a new docker command. You will need to execute docker command in a new **tmux** session, more details you will find [here](using-docker#using-tmux).

Docker command to start HOPR admin UI:

```bash
docker run -d --name hopr_admin -p 3000:3000 gcr.io/hoprassociation/hopr-admin
```

All ports are mapped to your local host, assuming you stick to the default port numbers. You should be able to view the `hoprd` UI interface at [http://localhost:3000](http://localhost:3000) (replace `localhost` with your `server IP address` if you are using a VPS, for example `http://142.93.5.175:3000`).

If you are in the process of registering your node on the network registry, please complete the process [here](./network-registry-tutorial.md) before continuing.

Otherwise, the installation process is complete! You can proceed to our [hoprd tutorial](using-hopr-admin).

### Breakdown of arguments

```
hoprd
  --restart on-failure -m 2g                  # restart node if it crashes or consumes 2GB of memory 
                                                (-m 2g)
  --log-opt max-size                          # maximum size of a single log file, will use a new one
                                                if reached
  --log-opt max-file                          # maximum number of log files per docker container
  --identity /app/hoprd-db/.hopr-identity     # store your node identity information in the
                                                persisted database folder
  --password switzerland   	                  # set the encryption password for your identity
  --init 				                      # initialize the database and identity if not present
  --announce 				                  # announce the node to other nodes in the network 
                                                and act as a relay
  --host "0.0.0.0:9091"   	                  # set IP and port of the P2P API to the container's
                                                external IP
  --apiToken <MY_TOKEN>                       # specify password for accessing REST API
```

## Updating to a new release

When migrating between releases, Docker users need to either move their identity file to the newly specified location or edit the command to point to their old location. Otherwise, you will be running a new node instead of accessing your old node's information.

For example, when we update from version `1.91.24` (Bogota) to `1.92.12` (Monte Rosa), the location the command points to will change as such:

1.) `$HOME/.hoprd-db-bogota` to `$HOME/.hoprd-db-monte-rosa`

2.) `$HOME/.hoprd-db-bogota/.hopr-id-bogota` to `$HOME/.hoprd-db-monte-rosa/.hopr-id-monte-rosa`

### Moving your identity file over

Preferably you would move your identity file to the new location, E.g. with the following commands for a migration from Bogota to Monte Rosa.

```bash
cp -r $HOME/.hoprd-db-bogota $HOME/.hoprd-db-monte-rosa
cp -r $HOME/.hoprd-db-bogota/.hopr-id-bogota $HOME/.hoprd-db-monte-rosa/.hopr-id-monte-rosa
```

## Default ports

- 3000 on TCP : Admin UI port (speaks HTTP protocol)
- 3001 on TCP: REST API port (speaks HTTP)
- 8080 on TCP: Healthcheck service - is used to see that the node is up & running (speaks HTTP)
- 9091 on TCP: main P2P port used for HOPR protocol
- 9091 on UDP: used for STUN requests by other non-public nodes reaching out to you to see what their IP address is

In general, you will only want to change these port numbers if you intend to run multiple nodes simultaneously. Otherwise, use the Docker command with the default mapping.

**Note:** For the initial Monte Rosa release, you will only be allowed to register a single node at a time.

## Collecting Logs

If your node crashes, you will want to collect the logs and pass them on to our ambassadors on telegram or create an issue on GitHub.

To collect the logs:

(**1**) In your terminal, enter the command:

```bash
docker container ls --all
```

This will create a list of all your docker containers similar to the one below:

![Container ID logs](/img/node/container-ID-logs.png)

(**2**) look through the list and find the `container ID` of the most recently exited container. The example above would be `5955dbd23bb2` as the most recent container is still running.

(**3**) Replace the container ID in the command below with yours from step 2:

```bash
docker logs -t <CONTAINER_ID>
```

This should output your logs, copy them and either:

- Save them in a .txt file and send them to an ambassador on our [telegram](https://t.me/hoprnet) for assistance.
- Or, create an issue using our bug template on [GitHub.](https://github.com/hoprnet/hoprnet/issues)
