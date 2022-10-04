# 排查资源被占用

## 文件被占用

看某个文件在被哪些进程读写:

```bash
lsof <文件名>
```

看某个进程打开了哪些文件:

```bash
lsof -p <pid>
```

## 端口占用

查看 22 端口被谁占用:

```bash
lsof -i :22
```

```bash
netstat -tunlp | grep 22
```
