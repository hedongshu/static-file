# Python 实现 ls 功能

在 linux 中使用 `ls` 命令，可以展示出路径下所有文件

下面代码使用 python 实现了 ls 的基本功能：

```shell
> ls [-h] [-l] [-a] [path]

> positional arguments:
 path 文件路径

> optional arguments:
 -h, --help  显示帮助
 -l 显示详情
 -a, --all 显示隐藏文件
```

## 实现：

```python
# !/usr/bin/env python3
# -*- coding: utf-8 -*-
'''
@File    :   ls.py
@Time    :   2019/06/13 13:44:23
'''


from pathlib import Path
import sys
import argparse
import datetime
import stat


def showDir(path, all=False, l=False):
    def _getDir(path, all=False, l=False):
        p = Path(path)
        for file in p.iterdir():
            if not all and str(file.name).startswith('.'):   # 隐藏文件不显示
                continue

            # 详情打印
            if l:
                st = file.stat()
                # 格式化时间
                sttime = datetime.datetime.fromtimestamp(
                    st.st_atime).strftime('%H:%M')
                # 格式化mode
                stmode = stat.filemode(st.st_mode)
                # 所在组,用户
                owner = file.owner()
                group = file.group()
                yield [stmode, st.st_nlink, owner, group, st.st_size, sttime, file.name]

            else:
                yield [file.name]

    # 默认排序
    yield from sorted(_getDir(path, all=False, l=False))


parser = argparse.ArgumentParser(prog='ls')
parser.add_argument('path', nargs='?', default='.')
parser.add_argument('-l', action='store_true')
parser.add_argument('-a', '--all', action='store_true')

if __name__ == '__main__':   # .py文件直接被运行的时候 执行下面的代码
    args = parser.parse_args()
    for st in showDir(args.path, args.all, args.l):
        print(st)

```

## 使用

```shell
 python3 ls.py -al

> ['-rw-r--r--', 1, 'x', 'staff', 4729, '15:06', 'hello.py']
  ['-rw-r--r--', 1, 'x', 'staff', 1307, '15:39', 'ls.py']
```
