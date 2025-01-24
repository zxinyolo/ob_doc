```shell
docker pull minio/minio
docker run -d -p 9000:9000 -p 9001:9001 --name minio -v /home/minio/data:/data -e "MINIO_ROOT_USER=minioadmin" -e "MINIO_ROOT_PASSWORD=minioadmin" minio/minio server /data --console-address ":9001"
```