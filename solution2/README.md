# Jenkins + Gitlab + Docker for git-flow based deployment (3)

Inherits from solution 1.

Inspired by [this answer](https://stackoverflow.com/a/59537295/3326749), again, I altered the container after its creation from outside:

```sh
docker exec -it --user root jenkins-blueocean bash -c 'groupadd -g $(getent group docker | cut -d: -f3) docker && usermod -aG docker jenkins'
```

and restart the container.

The command adds a group `docker` with the same gid from host environment (where the original docker installation lives). Then restart the container.

Now the jenkins service started by User `jenkins` inside the container, has permission to operate on resources of group `docker`. Since `/var/run/docker.sock` mounted is a resource  ruled by group docker outside. Both groups `docker` share the same gid, operations inside is feasible now.

> refs
> - https://askubuntu.com/questions/639990/what-is-the-group-id-of-this-group-name
> - https://stackoverflow.com/questions/42009172/execute-two-commands-with-docker-exec
> - https://www.cnblogs.com/sparkdev/p/9614164.html docker uses the same kernal with the host machine, which mean uid and gid are identical on both sides, even with different names.

## Conclusion

It's better but not perfect enough
