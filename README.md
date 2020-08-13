## 通过 docker 自动化解密手机微信数据库

### 原理

安卓系统自带的备份功能可以免 root 将第三方应用打包成一个 bak 文件，其中包含了 apk 安装包和应用的所有数据，当然，也包括了微信聊天记录。

所以可以通过将微信的 bak 备份文件解包并且自动化解密的方式来导出未加密的数据库文件，从而方便数据分析或其他用途。

其中 MIUI 系统由于添加了自定义的文件头，所以需要先进行去除，再利用解包[工具](https://github.com/nelenkov/android-backup-extractor)进行提取。

由于 MIUI 系统的隐私照明弹会在微信获取 IMEI 的时候返回空数据，所以在获取不到 ITEM 时微信会自动使用 1234567890ABCDEF 来作为 IMEI，这时在 xml 文件中找不到对应的 IMEI 的值。

只测试了 MIUI 系统和最新版微信，暂未测试其他系统和其他版本的微信，如有问题，欢迎 PR。

### 构建方式

Dockerfile 采用多阶段构建模式。

第一阶段构建 abe-all.jar，第二阶段构建 decrypt 可执行文件，由 python2 打包而来，第三阶段构建 process 可执行文件
，由 python3 打包而来，由于使用到了两个 python 版本，故分为两个阶段来构建，防止冲突。

### 使用方法

首先需要安装 docker，然后需要一个可读写的目录，将手机上备份好的文件放到文件夹中。

使用`docker run --rm -it -v /home/xxx/xxx/:/root/android/data -e password=xxxxxx libra146/decrypt_wechat_database`命令，或者也可以不传入 password 环境环境变量，

将包含 bak 文件的文件夹 (/home/xxx/xxx/) 挂载到 `/root/android/data` 目录 (这个目录不可自定义) 下即可，同时不要忘了使用 `-e` 命令定义微信数据库密码。

增加可选环境变量 deleted，默认会自动删除无用文件，使用 deleted=0 可以保留 bak 文件，此选项用于调试。

**目前只支持MIUI开启了隐私照明弹功能的自动计算密码**，后续可会尝试自动解密其他手机，主要是暂时还没有其他手机的 bak 样本🤣

命令运行成功输出如下：

通过环境变量传入正确密码的情况下：
```bash
0% 1% 2% 3% 4% 5% 6% 7% 8% 9% 10% 11% 12% 13% 14% 15% 16% 17% 18% 19% 20% 21% 22% 23% 24% 25% 26% 27% 28% 29% 30% 31% 32% 33% 34% 35% 36% 37% 38% 39% 40% 41% 42% 43% 44% 45% 46% 47% 48% 49% 50% 51% 52% 53% 54% 55% 56% 57% 58% 59% 60% 61% 62% 63% 64% 65% 66% 67% 68% 69% 70% 71% 72% 73% 74% 75% 76% 77% 78% 79% 80% 81% 82% 83% 84% 85% 86% 87% 88% 89% 90% 91% 92% 93% 94% 95% 96% 97% 98% 99% 100% 
2079869440 bytes written to 1_process_unpack.tar.
password is xxxxxx!
success!
```
或者没有通过环境变量传入密码，会自动计算密码，目前只支持MIUI开启了隐私照明弹功能的自动计算密码。
```bash
0% 1% 2% 3% 4% 5% 6% 7% 8% 9% 10% 11% 12% 13% 14% 15% 16% 17% 18% 19% 20% 21% 22% 23% 24% 25% 26% 27% 28% 29% 30% 31% 32% 33% 34% 35% 36% 37% 38% 39% 40% 41% 42% 43% 44% 45% 46% 47% 48% 49% 50% 51% 52% 53% 54% 55% 56% 57% 58% 59% 60% 61% 62% 63% 64% 65% 66% 67% 68% 69% 70% 71% 72% 73% 74% 75% 76% 77% 78% 79% 80% 81% 82% 83% 84% 85% 86% 87% 88% 89% 90% 91% 92% 93% 94% 95% 96% 97% 98% 99% 100% 
2079869440 bytes written to 1_process_unpack.tar.
evn key is empty!!!
uin is xxxxxxxxx!
password is xxxxxxx!
success!
```

解密成功后文件夹中会生成一个类似于 decrypt-1596704094498-EnMicroMsg.db 的文件。decrypt 开头，中间是时间戳，防止文件名重复，后面是微信数据库原始的文件名。

### 用到的工具

[https://github.com/nelenkov/android-backup-extractor](https://github.com/nelenkov/android-backup-extractor)

[https://github.com/gmaslowski/docker-java](https://github.com/gmaslowski/docker-java)

[https://github.com/docker-library/openjdk](https://github.com/docker-library/openjdk)

[https://github.com/leapcode/pysqlcipher](https://github.com/leapcode/pysqlcipher)