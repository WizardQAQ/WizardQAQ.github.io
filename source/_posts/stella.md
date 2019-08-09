---
title: Stella
date: 2019-08-07 21:40:04
tags: Growing
---
```Python
from PIL import Image
import os


d = './pics'
a = os.walk(d)
for root,dirs,files in a:
    print(root,dirs,files, sep='\n')

#region = [0,570,55,625]
i = 0
while i < 24:
    img = Image.open(os.path.join(root, files[i]))
    new_pic = img.crop((0,297,624,352))
    new_pic.save(d + 'tupian' + files[i])
    i = i + 1
```
<!--more-->
![Stella](stella.png)