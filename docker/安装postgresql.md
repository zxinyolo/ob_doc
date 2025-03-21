```shell
docker run -d --name=postgresql -v f:\docker\postgres-data:/var/lib/postgresql/data -p 5432:5432 -e POSTGRES_PASSWORD=123456 postgres
```

