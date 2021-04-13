## Docker 最佳实践

在容器内直接使用宿主机docker

```bash
docker run -d -p 8080:8080 --name jenkins -e TZ="Asia/Shanghai" -v jenkins_home:/var/jenkins_home -v /var/run/docker.sock:/var/run/docker.sock  -v $(which docker):$(which docker)  jenkins/jenkins:lts
 docker exec -it --user root jenkins bash
 chmod 666 /var/run/docker.sock
```

