# Jenkins + Gitlab + Docker for git-flow based deployment (2)

For this solution, it inherits most of the configuration from solution 0. Then it drop the use of *dind*.

Instead it uses `/var/run/docker.sock` volume binding to provide docker service accessing in jenkins container.

---

However, although all the resources required are available. The job using docker won't work. It hits the permission issue.

Searching through internet, many similar question and answers exist. Most of them doesn't work for this workflow or too risky.

like:

- https://stackoverflow.com/questions/44444099/how-to-solve-docker-permission-error-when-trigger-by-jenkins
- https://stackoverflow.com/questions/44978017/permission-denied-while-trying-to-connect-to-docker-daemon-while-running-jenkins
- https://stackoverflow.com/questions/53126950/permission-denied-to-docker-daemon-socket-at-unix-var-run-docker-sock


Of course, some others provide inspirations

- https://stackoverflow.com/a/51921594/3326749
- https://stackoverflow.com/questions/47854463/docker-got-permission-denied-while-trying-to-connect-to-the-docker-daemon-socke

To verify the permission problem quickly, The following command was adopted

```sh
docker exec -u root jenkins-blueocean /bin/chmod -v a+s $(which docker)
```

> `curl --unix-socket /var/run/docker.sock http://localhost/images/json` to test connection

## Conclusion

It's too risky to expose docker to any visitor.
