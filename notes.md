# Samba

```console
$: sudo mount //<IP>/files test -o user=<USER>,defaults,noperm
```

These options **defaults,noperm** ate 1h30' out of my life.
HUGE HUGE thanks to `shrimpwagon` [from the comments](https://unix.stackexchange.com/questions/92168/samba-share-permission-denied-user-writing-file-but-still-shows).

# Docker

## Installation

Install **docker, docker-compose and docker-buildx** on arch linux.
Start and enable **docker.socket** instead of **docker.service** to start docker only on first use, decreasing boot time.
Add yourself to the docker group. Requires logout or `newgrp` command.
