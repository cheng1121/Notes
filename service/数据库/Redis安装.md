# Redis安装

## 一、CentOS安装Redis

执行安装命令:

```shell
# 获取并添加 redis仓库到apt
curl -fsSL https://packages.redis.io/gpg | sudo gpg --dearmor - /usr/share/keyrings/redis-archive-keyring.gpg

echo "deb [signed-by=/usr/share/keyrings/redis-archive-keyring.gpg] https://packages.redis.io/deb $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/redis.list
# 升级apt
sudo apt-get update
#使用apt安装 redis
sudo apt-get instal redis
```

