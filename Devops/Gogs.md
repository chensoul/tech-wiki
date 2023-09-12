

## Docker安装

    docker run -d --name=gogs -p 3022:22 -p 3000:3000 -e TZ=Asia/Shanghai -v /data/docker/gogs:/data gogs/gogs
