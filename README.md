## 说明

pull [docker 官方 Python2.7 image](https://hub.docker.com/_/python/)失败，于是使用 官方 Python2.7 的[Dockerfile](https://github.com/docker-library/python/blob/93dd2037ba9de1e80e8c17b65649485d7b7112f5/2.7/Dockerfile)，在本地创建

## 使用方法

```
docker pull chaonet/python
```

## 折腾的过程

尝试为一个 Python web 应用 创建 image 时，在第一步 pull python 就卡住了，正常下载 与 vpn 都不行

`Dockerfile`

````
FROM python:2.7
ADD . /code
WORKDIR /code
CMD python run.py runserver
```

于是上官方的python页面，发现有专用于建 image 的 onbuild 版，同样下载卡住

[docker 官方 Python](https://hub.docker.com/_/python/)

````
FROM python:2.7-onbuild
ADD . /code
WORKDIR /code
CMD python run.py runserver
```

```
➜  web git:(master) ✗ docker build -t flask_blog .
Sending build context to Docker daemon 688.6 kB
Step 1 : FROM python:2.7-onbuild
2.7-onbuild: Pulling from library/python
7268d8f794c4: Already exists
a3ed95caeb02: Download complete
d9a49bc2b1b0: Downloading [===========================================>       ]  16.3 MB/18.53 MB
b965864d2d45: Download complete
47bed597ecf4: Downloading [=====>                                             ] 15.14 MB/128.6 MB
061df887fe0c: Download complete
27c92d49cd5e: Download complete
6ffb450d14ab: Download complete
0cf60b4c8101: Download complete

```

想找其他人下好的，没找到

发现 docker hub 中 image 的 Dockerfile 保存在 github ，找到官方 image 的 github，原来是对一个名为 python 的 github 资源的引用，即 official-images 是索引

[official-images python](https://github.com/docker-library/official-images/blob/master/library/python)

```
2.7.11-onbuild: git://github.com/docker-library/python@7663560df7547e69d13b1b548675502f4e0917d1 2.7/onbuild

2.7-onbuild: git://github.com/docker-library/python@7663560df7547e69d13b1b548675502f4e0917d1 2.7/onbuild

2-onbuild: git://github.com/docker-library/python@7663560df7547e69d13b1b548675502f4e0917d1 2.7/onbuild
```

实际的 Python image 的 Dockerfile 文件：

[python Dockerfile](https://github.com/docker-library/python)

普通 Python 2.7 image 的 [Dockerfile](https://github.com/docker-library/python/blob/master/2.7/Dockerfile)

可以看出，资源在 Python 仓库中定义，指定从官网下载源代码包，然后在 official-images 中编写 shell 脚本，执行脚本从而完成 image

官方维护人员的维护[操作记录](https://github.com/docker-library/official-images/pull/1484)

[bashbrew 使用说明](https://github.com/docker-library/official-images/tree/master/bashbrew)

觉得可以尝试 clone [official-images python](https://github.com/docker-library/official-images/blob/master/library/python)到本地，然后自己执行 bashbrew 在本地建立 image

但是，[bashbrew](https://github.com/docker-library/official-images/tree/master/bashbrew)脚本无法运行，很多 Linux 的命令 mac 不支持

于是

直接用[Python 仓库](https://github.com/docker-library/python)的 dockerfile 文件 build image

先用[2.7/Dockerfile](https://github.com/docker-library/python/blob/master/2.7/Dockerfile)创建 Python2.7 image

然后在此基础上，用[2.7/onbuild/Dockerfile](https://github.com/docker-library/python/blob/master/2.7/onbuild/Dockerfile)创建 Python2.7-onbuild image

实际上，最后用的是`python:2.7`，而不是`python:2.7-onbuild`

成功