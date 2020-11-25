---
title: 记录一个PIL把透明背景转成白色背景的方法
tags: python
comments: false
date: 2020-11-25 18:06:48
---
记录一个PIL把透明背景转成白色背景的方法
<!--more-->
起先在百度上搜到的[文章](https://blog.csdn.net/comeonbabe_/article/details/89266810)，的确可以做到透明背景转成白色背景，但是缺陷非常严重，会导致图的背景产生很多像素点，而且效率也不是很高。

又经过了一番搜索之后，发现了一个更好的[办法](https://stackoverflow.com/questions/38627870/how-to-paste-a-png-image-with-transparency-to-another-image-in-pil-without-white/38629258)。
```python
from PIL import Image

try:
    imagePtah = 'your image file path'
    img = Image.open(imagePtah)
    if img.mode != 'RGBA':
        image = img.convert('RGBA')
    width = img.width
    height = img.height

    image = Image.new('RGB', size=(width, height), color=(255, 255, 255))
    image.paste(img, (0, 0), mask=img)

    image.show()

except Exception as e:
    print(e)
```

使用``Image.paste``，就是这么简洁好使。百度一生黑。
以上方法，当然也可以转成其他颜色，选你需要的就可以了，只要替换``color=(255, 255, 255)``就可以了。
