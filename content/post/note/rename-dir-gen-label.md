---
date: 2019-03-05
title: "Rename dir & Gen label"
tags:
    - note
    - script
    - python
categories:
    - Note
comment: true
---

## Rename dir

把train目录下的所有存储样本的文件夹命名为0,1,2,3…的形式，然后把实际类别名存储在`target_names.txt`中

```python
#coding=utf-8
import os,sys


def rename_dir(txt_file = "target_names.txt", prefix = ''):
    path='.'
    dir_path = path + '/' + txt_file[:-4]
    out = []
    i = 0
    for dirpath,dirnames,filenames in os.walk(path):
        if(len(dirnames)) >= 10:
            for name in dirnames:
                os.rename(dirpath+'/'+name,dirpath+'/'+str(i))
                i += 1
                out += "'" + name + "',"


    with open(txt_file, 'w') as target:
        target.writelines(out)

    print("----------------\n"
          "write(overwrite) done")


txt_file = "test.txt"
rename_dir()
# if use prefix, the '/' is need

```



## Gen labels

遍历目录下按类别存储的所有样本，得到样本和对应的类别，其中类别从0开始编号，结果存在`train.txt`中

```python
#coding=utf-8
import os,sys


def generate_label(txt_file = "train.txt", prefix = ''):
    path='.'
    dir_path = path + '/' + txt_file[:-4]
    out = []
    for dirpath,dirnames,filenames in os.walk(path):
        if dirpath == dir_path: continue
        if dirpath.startswith(dir_path):
            #print(dirpath)
            for paths,names,files in os.walk(dirpath):
                # print(files)
                for sf in files[1:]: # '.DS_Store' is removed
                    if (sf[len(sf)-3:]) == 'ppm':
                        label = dirpath[len(dir_path) + 1:]
                        out.append(prefix + label + '/' + sf + ' ' + label)
                        print(out[-1])

    i = 0
    with open(txt_file, 'w', encoding='UTF-8') as train:
        for line in out:
            if i == 0: train.writelines(line)
            else: train.writelines('\n' + line)
            i += 1

    print("----------------\n"
          "write(overwrite) done")


txt_file = "test.txt"
generate_label()
# if use prefix, the '/' is need

```

