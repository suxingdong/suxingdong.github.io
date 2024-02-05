---
title: Python暴力破解压缩文件
author: East.Su
date: 2024-02-25 14:00:00 +0800
categories: [Python, Password, zip, rar]
tags: [Python]
---
# 安装
```
pip install patool
pip install product
pip install zipfile
```

# Python Code
```py
import zipfile
import itertools
import string
import subprocess
from itertools import product
import patoolib

def check_rar_password5(rar_file_path, password):
    try:
        patoolib.test_archive(rar_file_path, password=password, verbosity=-1)
        print("Password is correct!", password)
        return True
    except Exception as e:
        return False
        
def check_zip_password(zip_file_path, password):
    try:
        with zipfile.ZipFile(zip_file_path, 'r') as zip_ref:
            # 使用密码尝试解压第一个文件
            zip_ref.extract(zip_ref.namelist()[0], path='temp', pwd=password.encode('utf-8'))
        print("Password is correct!",password)
        return True
    except Exception as e:
        return False

# 调用函数进行测试
charset =  string.digits + string.ascii_letters
combinations = itertools.product(charset, repeat=6)
for combo in combinations:
    password = ''.join(combo)
    isOk = check_rar_password5('abc.rar',password)
    if(isOk):
        break

```

 

