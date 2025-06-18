# 使用docker oh-my-env 镜像配置开发环境，并运行next.js全栈项目，该项目需要用到Redis服务和postgres数据库

#### 准备工作

`oh-my-env`的网络环境是network1，确保有创建过network1，若没有则打开一个终端，执行`docker network create network1`。

------



#### Redis 数据库

使用docker Redis镜像，为该next.js全栈项目独立启动一个redis容器，使其能连接成功

##### 创建数据库

```powershell
docker run -d --name redis-container -p 6379:6379 --network=network1 redis:latest
```

##### 启动数据库

```powershell
docker restart redis-container
```

##### 连接数据库

```
# .env 文件
# ...
# 如果你的 Next.js 应用也在 Docker 容器中运行，需要使用容器名称而不是 localhost
# 下面redis 配置是根据Next.js 应用是在容器中运行的
REDIS_URL=redis://redis-container:6379
REDIS_HOST=redis-container
REDIS_PORT=6379
```

确认端口映射：

```
docker port redis-container
```

如果没有看到端口映射 (如 0.0.0.0:6379->6379/tcp)，则重新创建容器

然后测试是否可正常连接：

```powershell
docker exec -it redis-container redis-cli ping
# 应返回 PONG
```

##  postgres 数据库

使用docker postgres镜像，为该next.js全栈项目独立启动一个postgres容器，使其能连接成功

##### 创建数据库

```
docker run -d --name postgres-container -p 5432:5432 -e POSTGRES_DB=nhblog -e POSTGRES_USER=postgres -e POSTGRES_PASSWORD=12345678 -v postgres_data:/var/lib/postgresql/data --network=network1 postgres:latest 
```

##### 启动数据库

```
docker restart postgres-container
```

确认端口映射：

```
docker port redis-container
```

如果没有看到端口映射 (如 0.0.0.0:5432->5432/tcp)，则重新创建容器

##### 连接数据库

```
# .env 文件
AUTH_JWT_SECRET=8mHv-QpF5-TEFk-gr2e-5iXU-A9fo-5jRY-CnuM
# 如果你的 Next.js 应用也在 Docker 容器中运行，需要使用容器名称而不是 localhost
# 下面 postgresql 配置是根据Next.js 应用是在容器中运行的
DATABASE_URL=postgresql://postgres:12345678@postgres-container:5432/nhblog
DIRECT_URL=postgresql://postgres:12345678@postgres-container:5432/nhblog
# 下面 redis 配置是根据Next.js 应用是在容器中运行的
REDIS_URL=redis://redis-container:6379
REDIS_HOST=redis-container
REDIS_PORT=6379
```

```powershell
# 测试 PostgreSQL 连接
docker exec -it postgres-container psql -U postgres -d nhblog -c "SELECT version();"
```



**连接失败可检查如下几项**：

1. 项目是否运行在`oh-my-env`的环境，它与redis容器、postgres容器是否在同一网络即network1；
2. 容器端口映射；

 ```
// 检查所有的network，找到network1的network id
docker network ls
// 查看redis-container容器的Networks.network1.networkId是否与上面的一致
docker inspect redis-container
// 同上
docker inspect postgres-container
 ```

运行项目启动的命令，这里以启动开发环境为例`pnpm dev`





