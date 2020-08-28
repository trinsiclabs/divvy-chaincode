# Divvy chaincode

Chaincode is used to update and query ledger state.

## Getting started

Make sure you have set up the host VM as described in the
[platform docs](https://github.com/trinsiclabs/divvy).

Normally chaincodes are started and maintained by peers. However in "dev mode",
chaincode is built and started by the user. This mode is useful during
chaincode development phase for rapid code/build/run/debug cycle turnaround.

We start "dev mode" by using pre-generated orderer and channel artifacts to
bring up a dev network.

Log into the host VM:

```
$ vagrant ssh
```

If the "main" Divvy network is running, you need to stop it.

```
$ cd /home/vagrant/api
$ sudo docker-compose down

$ cd /home/vagrant/application
$ sudo docker-compose down

$ cd /home/vagrant/network
$ ./network.sh down
```

Start the dev network:

```
$ cd /home/vagrant/chaincode
$ sudo docker-compose -f ./dev/docker-compose.yaml up -d
```

### Start the chaincode

Log into the chaincode container:

```
$ sudo docker exec -it chaincode sh
```

Rebuild the native node binaries so they work in the container:

```
$ npm rebuild
```

Start the chaincode:

```
$ npm run dev
```

### Use the chaincode

Even though you are in `--peer-chaincodedev` mode, you still have to install
the chaincode so the life-cycle system chaincode can go through
its checks normally.

Log into the cli container from the host VM:

```
$ sudo docker exec -it cli bash
```

Install and instantiate the chaincode:

```
$ peer chaincode install -p ./chaincode -n mycc -v 0 -l node
$ peer chaincode instantiate -n mycc -v 0 -c '{"Args":[]}' -C myc
```

Invoke the initial transaction:

```
$ peer chaincode invoke -n mycc -c '{"Args":["com.divvy.share:instantiate","myorg"]}' -C myc
```

Query the ledger:

```
$ peer chaincode query -n mycc -c '{"Args":["queryShare","myorg","1"]}' -C myc
```

### Making changes

The `npm run dev` script does not watch for changes, so after making updates
you need to exit the script (Control C), and run it again.

### Tidy up

When you're finished development don't forget to stop the dev network and
rebuild the native node binaries for the host:

```
$ cd /home/vagrant/chaincode/dev
$ sudo docker-compose down
$ cd ../src
$ sudo rm -rf ./node_modules
$ nvm use
$ npm install
```
