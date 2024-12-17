ssh root@jira.aart.vn -p 2231
aart@2024

```bash
cd /root/devops/main/devtest
```

cập nhật version docker image trong docker-compose.yml
```bash
docker-compose down
docker-compose up -d
```