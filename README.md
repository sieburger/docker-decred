This is the the [Docker image](https://hub.docker.com/r/jpbriquet/decred/) for [Decred cryptocurrency releases](https://github.com/decred/decred-release).

# What is Decred?

<img src="https://raw.githubusercontent.com/decred/decredweb/master/content/images/logo.png" alt="Logo" width="200px;"/>

Decred is an open and progressive cryptocurrency with a system of community-based governance integrated into its blockchain. The fusion of technology, community, and governance the Decred way means development is self-funding and remains sustainable.

> Website: [www.decred.org](https://www.decred.org)

> Documentation : [docs.decred.org](https://wiki.decred.org)

> Wiki: [wiki.decred.org](https://wiki.decred.org)


# Image Information

This Decred docker image is based on the Ubuntu image 16.04 LTS (Xenial Xerus) and
contains all binaries in decred releases:

- [Decred Daemon (chainserver) / dcrd](https://github.com/decred/dcrd)
- [Decred Wallet / dcrwallet](https://github.com/decred/dcrwallet)
- [Decred Controller (rpc client) / dcrctl](https://github.com/decred/dcrd/tree/master/cmd/dcrctl)
- [Decred Ticket Buyer / dcrticketbuyer](https://github.com/decred/dcrticketbuyer)


# Supported tags and respective `Dockerfile` links

#### Releases:

-	[`0.7`, `0.7.0`, `latest` (*0.7.0/Dockerfile*)](https://raw.githubusercontent.com/jpbriquet/docker-decred/v0.7.0/Dockerfile)

[![](https://imagelayers.io/badge/jpbriquet/decred:latest.svg)](https://imagelayers.io/?images=jpbriquet/decred:latest)

Only official [Decred release binaries](https://github.com/decred/decred-release/releases) are used to build this image. These binaries are verified based on the recommended [binaries verification process](https://wiki.decred.org/Verifying_Binaries). (refer to Dockerfile content for details)


# How to use this image

This Decred Docker image can be used to create containers that can start each of the included executables (dcrd, dcrwallet, ...).

The image can be used as follow:
```console
$ docker run -it jpbriquet/decred:latest DECRED_COMMAND
```

e.g. :
```console
$ docker run -it jpbriquet/decred:latest dcrd
```


# Decred infrastructure based on Docker containers

This guide describes how to configure a Decred daemon (dcrd) container. The next step will be to connect a Decred wallet (dcrwallet) container on the daemon.
Both containers will be manageable with the Decred controller tool (dcrctl), which will give a way to unlock the Decred wallet, or get Decred network information from the Decred daemon.
The last step explain how to configure the automatic Decred ticket buyer (dcrticketbuyer) container.

As it is an advanced configuration, it is strongly recommended to be familiar with [Docker](https://www.docker.com) and with [Decred documentation](https://wiki.decred.org)

## Prerequisites

### Create Decred private docker network
Our Decred containers will communicate on their own Docker bridge network called decrednet, they will be isolated from other containers eventually running on the Docker engine.

```console
$ docker network create --driver bridge decrednet
```

### Create Docker data-volume containers
Docker volume will be used to store decred containers data that has to be persistent:
 * blockchain data
 * wallet data
 * ...

For more information about this Docker storage mode refer to  [Docker data-volume](https://docs.docker.com/engine/tutorials/dockervolumes).


#### Create a volume for the Decred daemon container:
```console
$ docker volume create --name dcrd-vol
```

#### Create a volume for the Decred wallet container:
```console
$ docker volume create --name dcrwallet-vol
```

#### Create a volume for the Decred ticker buyer container:
```console
$ docker volume create --name dcrticketbuyer-vol
```

### Configuration files

In this guide, configuration files are located outside of the containers and are in a "conf" directory. These configuration files will be exposed to our containers via a host file mount.

```console
`-- conf
    |-- dcrctl.conf
    |-- dcrd.conf
    |-- dcrwallet.conf
    `-- ticketbuyer.conf
```

## Decred daemon container configuration (dcrd)

This chapter describe how to configure and start a Decred daemon container.
The Decred daemon container automatically connects to the Decred P2P network and then synchronise to the latest blockchain head block.
Later in this guide we will connect a wallet container to the daemon container.


### Prepare configuration file
Decred daemon can be started with command line arguments, however, this is not recommended for sensitive information like the RPC user/password.
It is possible to set this configuration in a dcrd.conf configuration file:

```console
; -------------------------------------------------------
; RPC server options - The following options control the built-in RPC server
; which is used to control and query information from a running dcrd process.
;
; NOTE: The RPC server is disabled by default if no rpcuser or rpcpass is
; specified.
; -------------------------------------------------------

rpclisten=0.0.0.0:9109
rpcuser=whatever_username_you_want
rpcpass=whatever_password_you_want
```

Adjust this configuration file to your needs, to view all options available, refer to the sample configuration of the [Decred daemon](https://github.com/decred/dcrd/blob/master/sample-dcrd.conf)

Save this configuration in the file dcrd.conf (in conf directory).


### Create container

The daemon container will use the data-volume  and the dcrd.conf configuration file created previously.

```console
$ docker run -d --name dcrd --net=decrednet -h dcrd -v dcrd-vol:/home/decred/.dcrd -v $PWD/conf/dcrd.conf:/home/decred/.dcrd/dcrd.conf jpbriquet/decred:latest dcrd
```

Once launched Decred daemon will read local blockchain data and synchronise with the p2p network, it may take a while depending of the connection speed.
Docker logs can be use to check what the container is doing.

```console
docker logs dcrd
15:39:18 2016-12-31 [INF] DCRD: Version 0.7.0-beta
15:39:18 2016-12-31 [INF] DCRD: Home dir: /home/decred/.dcrd
15:39:18 2016-12-31 [INF] DCRD: Loading block database from '/home/decred/.dcrd/data/mainnet/blocks_ffldb'
15:39:18 2016-12-31 [INF] DCRD: Block database loaded
15:39:18 2016-12-31 [INF] INDX: Exists address index is enabled
15:39:18 2016-12-31 [INF] CHAN: Blockchain database version 2 loaded
...
```

The daemon container is now up and running.

### Stop/Start container

When the container has been created, standard Docker commands can be used.

#### Stop container
```console
docker stop dcrd
```

#### Start container
```console
docker start dcrd
```

### Dcrctl console for daemon container

The console can be connected on dcrd to get information about the blockchain.

```console
docker exec -it dcrd dcrctl --terminal

Starting terminal mode.
Enter h for [h]elp.
Enter l for [l]ist of commands.
Enter q for [q]uit.
> getinfo
{
  "version": 70000,
  "protocolversion": 2,
  "blocks": 94999,
  "timeoffset": 0,
  "connections": 7,
  "proxy": "",
  "difficulty": 802194.38990989,
  "testnet": false,
  "relayfee": 0.01,
  "errors": ""
}

```

## Decred wallet container (dcrwallet)

The Decred wallet container needs to connect on a Decred daemon container to have access to the blockchain and interact with the p2p network.

### Prepare configuration files
Decred wallet can be started with command line arguments, however, this is not recommended for sensitive information like the RPC user/password.
Prepare the dcrwallet.conf configuration file for the Decred wallet as follow:

```console
; -------------------------------------------------------
; RPC settings (both client and server)
; -------------------------------------------------------
username=whatever_wallet_username_you_want
password=whatever_wallet_password_you_want
dcrdusername=whatever_username_you_want
dcrdpassword=whatever_password_you_want


; ------------------------------------------------------------------------------
; RPC server settings
; ------------------------------------------------------------------------------

rpclisten=0.0.0.0:9110

; -------------------------------------------------------
; RPC client settings
; -------------------------------------------------------

; The server and port used for dcrd websocket connections.
rpcconnect=dcrd:9109

; File containing root certificates to authenticate a TLS connections with dcrd
cafile=~/.dcrd/rpc.cert
```

* dcrdusername and dcrdpassword are user and password to connect the wallet on the rpc service of the daemon, this variable must have the same value than in dcrd.conf.
* username and password are used to authenticate incoming connections on the rpc service of the wallet.

To view all options available, refer to the sample  [Decred wallet configuration](https://github.com/decred/dcrwallet/blob/master/sample-dcrwallet.conf)
Adjust it to your needs.

Save this configuration file as dcrwallet.conf in conf directory.


The Decred controller console (dcrctl) needs to know the RPC credentials to connect on the wallet.
Create following configuration file and save it as dcrctl.conf in conf directory.

```console
; ------------------------------------------------------------------------------
; RPC client settings
; ------------------------------------------------------------------------------

rpcuser=whatever_wallet_username_you_want
rpcpass=whatever_wallet_password_you_want
```



### Create or import wallet

The following command will launch the wallet creation assistant that can create a brand new wallet or import an existing wallet seed.

```console
$ docker run -it --rm -v dcrwallet-vol:/home/decred/.dcrwallet jpbriquet/decred:latest dcrwallet --create

Enter the private passphrase for your new wallet:
Confirm passphrase:
Do you want to add an additional layer of encryption for public data? (n/no/y/yes) [no]: no
Do you have an existing wallet seed you want to use? (n/no/y/yes) [no]:
Your wallet generation seed is:
...
Creating the wallet...
The wallet has been created successfully.
```

### Create container

The wallet container named 'dcrwallet' uses the wallet volume, the daemon volume (in read-only) and the dcrwallet.conf configuration file previously created.

This container is launched in interactive mode because the Decred wallet ask to input the wallet passphrase in order to unlock it at the first run.
The automatic voting on tickets is also enabled, remove it if needed.

```console
$ docker run -it --name dcrwallet --net=decrednet -h dcrwallet -v $PWD/conf/dcrwallet.conf:/home/decred/.dcrwallet/dcrwallet.conf -v $PWD/conf/dcrctl.conf:/home/decred/.dcrctl/dcrctl.conf -v dcrwallet-vol:/home/decred/.dcrwallet -v dcrd-vol:/home/decred/.dcrd:ro jpbriquet/decred:latest dcrwallet --enablevoting

16:23:52 2017-01-01 [INF] DCRW: Version 0.7.0-beta
16:23:52 2017-01-01 [INF] DCRW: Generating TLS certificates...
16:23:53 2017-01-01 [INF] DCRW: Done generating TLS certificates
16:23:53 2017-01-01 [INF] DCRW: Attempting RPC client connection to dcrd:9109
16:23:53 2017-01-01 [INF] RPCS: Listening on 127.0.0.1:9110
16:23:53 2017-01-01 [INF] RPCS: Listening on [::1]:9110
16:23:53 2017-01-01 [INF] CHNS: Established connection to RPC server dcrd:9109
16:23:53 2017-01-01 [INF] WLLT: Opened wallet
*** ATTENTION ***
Since this is your first time running we need to sync accounts. Please enter
the private wallet passphrase. This will complete syncing of the wallet
accounts and then leave your wallet unlocked. You may relock wallet after by
calling 'walletlock' through the RPC.
*****************
Enter private passphrase:
16:24:00 2017-01-01 [INF] WLLT: The wallet has been unlocked without a time limit
16:24:00 2017-01-01 [INF] WLLT: Beginning a rescan of active addresses using the daemon. This may take a while.
16:24:03 2017-01-01 [INF] WLLT: The last used account was 0. Beginning a rescan for all active addresses in known accounts.
...
16:25:19 2017-01-01 [INF] WLLT: Blockchain sync completed, wallet ready for general usage.
...
```

Wait for the end of the synchronisation, and then hit CTRL+C.


### Stop/Start container

When the container has been created, standard Docker commands can be used.

#### Stop container
```console
docker stop dcrwallet
```

#### Start container
```console
docker start dcrwallet
```

### Dcrctl console for wallet container

The console can be connected on the dcrwallet to do wallet related operations.
For instance unlocking the wallet or creating new transactions.

```console
docker exec -it dcrwallet dcrctl --terminal --wallet

Starting terminal mode.
Enter h for [h]elp.
Enter l for [l]ist of commands.
Enter q for [q]uit.
> getstakeinfo
{
  "blockheight": 95000,
  "poolsize": 41672,
  "difficulty": 56.49212635,
  "allmempooltix": 0,
  "ownmempooltix": 0,
  "immature": 0,
  "live": 0,
  "proportionlive": 0,
  "voted": 0,
  "totalsubsidy": 0,
  "missed": 0,
  "proportionmissed": 0,
  "revoked": 0,
  "expired": 0
}

```

## Decred ticket buyer container configuration (dcrticketbuyer)

The Decred ticket buyer bot purchases automatically tickets based on a user defined set of rules and configuration.
It connects on the daemon and the wallet to get blockchain and mempool information and use that to make decisions about buying tickets.

### Prepare configuration file


```console
#########################################
### Basic Connectivity and Monitoring ###
#########################################
## Login information for the daemon and wallet RPCs.
dcrduser=user
dcrdpass=pass
dcrdserv=dcrd:9109
dcrdcert=/home/decred/.dcrd/rpc.cert
dcrwuser=user
dcrwpass=pass
dcrwserv=dcrwallet:9110
dcrwcert=/home/decred/.dcrwallet/rpc.cert

...

```

To view all options available, refer to the sample configuration of the [Decred ticket buyer](https://github.com/decred/dcrticketbuyer/blob/master/ticketbuyer-example.conf)
Adjust the configuration to your needs.

Save this configuration file as ticketbuyer.conf in the conf directory.

### Create container

The ticket buyer container needs both the daemon and the wallet volume in read-only mode to be able to communicate with them. The ticketbuyer.conf configuration file created previously is also needed.

```console
$ docker run -d --name dcrticketbuyer --net=decrednet -h dcrticketbuyer -v dcrd-vol:/home/decred/.dcrd
-v dcrwallet-vol:/home/decred/.dcrwallet -v dcrticketbuyer-vol:/home/decred/.dcrticketbuyer -v $PWD/conf/ticketbuyer.conf:/home/decred/.dcrticketbuyer/ticketbuyer.conf jpbriquet/decred:latest dcrticketbuyer
```

Once launched the ticket buyer bot will connect on dcrd and dcrwallet, and then may try to purchase tickets depending of the configuration.

```console
docker logs dcrticketbuyer
18:46:14 2017-01-01 [INF] RPCC: Established connection to RPC server dcrd:9109
18:46:14 2017-01-01 [INF] RPCC: Established connection to RPC server dcrwallet:9110
18:46:14 2017-01-01 [INF] TKBY: Daemon and wallet successfully connected, beginning to purchase tickets
...
```

### Stop/Start container

When the container has been created, standard Docker commands can be used.

#### Stop container
```console
docker stop dcrticketbuyer
```

#### Start container
```console
docker start dcrticketbuyer
```


# License

View [license information](https://github.com/decred/dcrwallet/blob/master/LICENSE) for the software contained in this image.

# Supported Docker versions

This image is officially supported on Docker version 1.11.x and above.

Support for older versions (down to 1.6) is provided on a best-effort basis.

Please see [the Docker installation documentation](https://docs.docker.com/installation/) for details on how to upgrade your Docker daemon.

# User Feedback

## Issues

If you have any problems with or questions about this image, please contact us through a [GitHub issue](https://github.com/jpbriquet/docker-decred/issues).

If you have any problems with or questions about Decred tools and utilies provided in this image, please contact Decred Developer team on [GitHub](https://github.com/decred).

## Contributing

You are invited to contribute new features, fixes, or updates, large or small; we are always thrilled to receive pull requests, and do our best to process them as fast as we can.

Before you start to code, we recommend discussing your plans through a [GitHub issue](https://github.com/jpbriquet/docker-decred/issues), especially for more ambitious contributions. This gives other contributors a chance to point you in the right direction, give you feedback on your design, and help you find out if someone else is working on the same thing.
