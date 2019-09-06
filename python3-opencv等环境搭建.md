# 常见python库环境搭建

## 1.下载python3.7

​	去python官网下载安装，选择add path to *,在cmd里面输入python,显示python控制台则成功。

## 2、安装pip

参见清华开源 <https://mirrors.tuna.tsinghua.edu.cn/help/pypi/>

### 2.1临时使用

```python
pip install -i https://pypi.tuna.tsinghua.edu.cn/simple some-package
```

注意，`simple` 不能少, 是 `https` 而不是 `http`

### 2.2设为默认

升级 pip 到最新的版本 (>=10.0.0) 后进行配置：

```python
pip install pip -U
pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple
```

如果您到 pip 默认源的网络连接较差，临时使用本镜像站来升级 pip：

```python
pip install -i https://pypi.tuna.tsinghua.edu.cn/simple pip -U
```

这样就设置好了清华源了，速度和镜像拉取速度都很快

## 3、机器学习和爬虫以及图像识别常用库下载

#### 3.1 numpy安装

```python
pip3 install numpy
```

#### 3.2 scipy安装

```python
pip3 install scipy
```

#### 3.3matplotlib安装

```python
pip install matplotlib
```

#### 3.4 opencv库安装

```python
pip install python-opencv
```

#### 3.5 scrapy安装

```python
pip install scrapy
```

#### 3.6 sklean 安装

```python
pip install sklearn
```

#### 3.7 安装 keras, theano， 深度学习算法

```python
$ pip install keras
```

#### 3.8安装 nose (单元测试)

```python
$ pip install nose
```

#### 3.9 beautifulsoup

```python
pip install beautifulsoup4
```

#### 3.10 jupyter ，很好的一款工具 推荐

```python
pip install jupyter
```



.....接下来就可以import这些库进行学习了。。。。。