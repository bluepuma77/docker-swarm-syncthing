# docker-swarm-syncthing

Automatically setting up a replicated Syncthing folder on Docker Swarm nodes.

## Status

This is a **proof-of-concept**, currently the script is inline in a `docker-compose.yml` file to ease deployment in a Docker Swarm, no building of dedicated images required.

## Purpose

Syncthing enables the missing piece in Docker Swarm: writable synced storage. Docker Swarm provides `configs` and `secrets`, but those are not updateable and not writable from within the container. So you can not easily use LetsEncrypt to share a created certificate among a few reverse proxys and you can not easily share files like uploaded documents among a few wordpress servers.

This could be done with a shared folder, but if there is no need for real-time sync, why introduce a potential breaking point for the live operation?

## Installation

Run `docker stack deploy -c docker-compose.yml syncthing` to spin up `syncthing` on every node and have a single `syncthing-controller` connect them as devices and activate sharing the default folder among each other. The folder `/var/syncthing/Sync` is the syncronized among the peers.

## Requirements

The `syncthing` instances need to run on nodes that have an already existing folder `/var/syncthing`. Docker Swarm, opposed to regular Docker, will not create the mounted folders on the host when they do not exist, instead container creation on the node will fail.

The single `syncthing-controller` instance must run on a Docker Swarm manager node to be able to use `docker.sock` with all Swarm information

## How does it work

Simple work flow:

1.  Get all Syncthing container IPs from Docker Swarm
2.  Get Syncthing ID from all containers via IP
3.  Get Syncthing “devices” from each container and add IDs which are missing
4.  Get Syncthing default folder from each container and update "devices" with IDs which are missing
5.  Sleep and repeat
