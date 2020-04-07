* 查看python版本  
    python -V
    
* 升级python 3.6
    1. wget https://www.python.org/ftp/python/3.6.0/Python-3.6.0.tgz 下载
    2. tar -xzvf Python-3.6.0.tgz 解压
    3. ./configure --prefix=/home/local/python3 编译
    4. make && make install 安装
    5. mv /usr/bin/python /usr/bin/python_bak 移除软链接
    6. ln -s /home/local/python3/bin/python3 /usr/bin/python 新建软链接
    
        