# jenkins使用

### Jenkins安装

```shell
docker run \
  --rm \
  -u root \
  -p 8011:8011 \
  -v jenkins-data:/var/jenkins_home \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v "$HOME":/home \
  jenkinsci/blueocean
```
