---
title: Nexus搭建Pypi私服
description: Nexus搭建Pypi私服
date: 2025-06-03
slug: Nexus搭建Pypi私服
image: 202506081432952.png
categories:
    - 系统配置
---
# Nexus搭建Pypi私服
> 国内常用pypi镜像源

```
阿里云 http://mirrors.aliyun.com/pypi/simple
豆瓣(douban) http://pypi.douban.com/simple
清华大学 https://pypi.tuna.tsinghua.edu.cn/simple
```
# 配置Proxy仓库

![Pasted image 20250602011334.png](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo05/202506081428380.png)

> 注意这里不要加`simple`

![Pasted image 20250602011801.png](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo05/202506081428381.png)

# 配置Group仓库

> 将Proxy添加到Group组

![Pasted image 20250602012450.png](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo05/202506081428382.png)
## **配置 Nexus 源**
## **设置全局 index-url 为 Nexus 地址**

> 这里要加`simple`


```bash
pip config set global.index-url http://116.148.120.214:8081/repository/pypi-public/simple
```
### **设置信任的主机（trusted-host）**
```bash
pip config set global.trusted-host "116.148.120.214"
```
### **验证配置是否生效**
```bash
pip config list
```
输出应包含：
```
global.index-url='http://116.148.120.214:8081/repository/pypi-public/simple'
global.trusted-host='116.148.120.214'
```

> 接下来尝试下载

```sh
pip install requests -i http://116.148.120.214:8081/repository/pypi-public/simple

# Looking in indexes: http://116.148.120.214:8081/repository/pypi-public/simple
# Collecting requests
#   Downloading http://116.148.120.214:8081/repository/pypi-public/packages/requests/2.32.3/requests-2.32.3-py3-none-any.whl (64 kB)
#      ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 64.9/64.9 kB 3.6 MB/s eta 0:00:00
# Requirement already satisfied: charset-normalizer<4,>=2 in d:\python\lib\site-packages (from requests) (3.4.1)
# Requirement already satisfied: idna<4,>=2.5 in d:\python\lib\site-packages (from requests) (3.10)
# Requirement already satisfied: urllib3<3,>=1.21.1 in d:\python\lib\site-packages (from requests) (2.4.0)
# Requirement already satisfied: certifi>=2017.4.17 in d:\python\lib\site-packages (from requests) (2025.1.31)
# Installing collected packages: requests
# Successfully installed requests-2.32.3
# 
# [notice] A new release of pip is available: 23.2.1 -> 25.1.1
# [notice] To update, run: python.exe -m pip install --upgrade pip

pip install rsa                                                            

# Looking in indexes: http://116.148.120.214:8081/repository/pypi-public/simple
# Collecting rsa
#   Downloading http://116.148.120.214:8081/repository/pypi-public/packages/rsa/4.9.1/rsa-4.9.1-py3-none-any.whl (34 kB)
# Collecting pyasn1>=0.1.3 (from rsa)
#   Downloading http://116.148.120.214:8081/repository/pypi-public/packages/pyasn1/0.6.1/pyasn1-0.6.1-py3-none-any.whl (83 kB)
#      ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 83.1/83.1 kB 4.9 MB/s eta 0:00:00
# Installing collected packages: pyasn1, rsa
# Successfully installed pyasn1-0.6.1 rsa-4.9.1
# 
# [notice] A new release of pip is available: 23.2.1 -> 25.1.1
# [notice] To update, run: python.exe -m pip install --upgrade pip
```
![Pasted image 20250602012017.png](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo05/202506081428383.png)

# 配置Hosted仓库

> hosted用来上传本地pypi包

![Pasted image 20250602012601.png](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo05/202506081428384.png)

```bash
# 安装打包工具
pip install setuptools wheel twine
```

需在 `~/.pypirc` 配置认证：
将ip修改为自己的，password也改一下

```ini
[distutils]
index-servers =
    nexus

[nexus]
repository: http://<服务器IP>:8081/repository/pypi-releases/
username: admin
password: your_password
```

主要步骤分为两步，打包、发布

## 打包发布

> 这里我直接从github上down一个py项目尝试`cython`

```bash
# 先下载项目的依赖
pip install setuptools

# 在根目录执行
python setup.py sdist bdist_wheel
```
![Pasted image 20250602014606.png](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo05/202506081428385.png)

> 如果没有配置`.pypirc`

```bash
twine upload --repository-url http://116.148.120.214:8081/repository/pypi-releases/ dist/* \
  -u <username> -p <password>
```

> 这里我已经配置了，所以直接发布即可

```bash
twine upload --repository nexus dist/*

# Traceback (most recent call last):
#   File "<frozen runpy>", line 198, in _run_module_as_main
#   File "<frozen runpy>", line 88, in _run_code
#   File "C:\Users\Administrator\Desktop\cython\.venv\Scripts\twine.exe\__main__.py", line 7, in <module>
#   File "C:\Users\Administrator\Desktop\cython\.venv\Lib\site-packages\twine\__main__.py", line 33, in main
#     error = cli.dispatch(sys.argv[1:])
#   File "C:\Users\Administrator\Desktop\cython\.venv\Lib\site-packages\twine\cli.py", line 139, in dispatch
#     return main(args.args)
#            ^^^^^^^^^^^^^^^
#   File "C:\Users\Administrator\Desktop\cython\.venv\Lib\site-packages\twine\commands\upload.py", line 255, in main
#     upload_settings = settings.Settings.from_argparse(parsed_args)
#                       ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
#   File "C:\Users\Administrator\Desktop\cython\.venv\Lib\site-packages\twine\settings.py", line 288, in from_argparse
#     return cls(**settings)
#            ^^^^^^^^^^^^^^^
#   File "C:\Users\Administrator\Desktop\cython\.venv\Lib\site-packages\twine\settings.py", line 116, in __init__
#     self._handle_repository_options(
#   File "C:\Users\Administrator\Desktop\cython\.venv\Lib\site-packages\twine\settings.py", line 304, in _handle_repository_options
#     self.repository_config = utils.get_repository_from_config(
#                              ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
#   File "C:\Users\Administrator\Desktop\cython\.venv\Lib\site-packages\twine\utils.py", line 154, in get_repository_from_config
#     config = get_config(config_file)[repository]
#              ^^^^^^^^^^^^^^^^^^^^^^^
#   File "C:\Users\Administrator\Desktop\cython\.venv\Lib\site-packages\twine\utils.py", line 66, in get_config
#     parser.read_file(f)
#   File "D:\Python\Lib\configparser.py", line 705, in read_file
#     self._read(f, source)
#   File "D:\Python\Lib\configparser.py", line 998, in _read
#     for lineno, line in enumerate(fp, start=1):
# UnicodeDecodeError: 'gbk' codec can't decode byte 0x80 in position 50: illegal multibyte sequence
```

这个错误是因为windows尝试用gbk编码去读`.pypirc`但是实际上是UTF8，所以失败了
这里可以使用vscode更新编码后重新保存，或者直接删掉`.pypirc`里的中文

又报错`InvalidDistribution: Invalid distribution metadata: unrecognized or malformed field 'license-file'`

参考[InvalidDistribution: Invalid distribution metadata: unrecognized or malformed field 'license-file'](https://www.cnblogs.com/9527l/p/18812555)，这个问题是Twine的bug，再换一个py库试试吧

```bash
git clone https://github.com/pallets/flask.git 
cd flask

# 安装依赖
pip install -e .

# 生成发布包(这里用如下命令)
python -m build

# 上传
twine upload --repository nexus dist/*
```

![Pasted image 20250602021506.png](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo05/202506081428386.png)



