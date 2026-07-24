1、显示图像

st.image()：是Streamlit中用于在应用程序中显示图像的函数。它可以接受多种输入格式，包括文件路径、URL、图像的字节数据等。

参数如下：
默认参数为st.image (image, caption=None, width=None, use_column_width=False, clamp=False, channels='RGB', format='JPEG')
image：要显示的图像，类型可以是numpy.ndarray,[numpy.ndarray],BytesIO,str,或[str])。单色图像为(w,h)或(w,h,1)，彩色图像为(w,h,3)，RGBA图像为(w,h,4)，也可以指定一个图像url，或url列表
caption：图像标题，字符串。如果显示多幅图像，caption应当是字符串列表。
width：图像宽度，None表示使用图像自身宽度。
use_column_width：如果设置为True，则使用列宽作为图像宽度。
clamp：是否将图像的像素值压缩到有效域（0~255），仅对字节数组图像有效。
channels：图像通道类型，'RGB'或'BGR'，默认值：RGB。
format：图像格式：'JPEG'或'PNG')，默认值：JPEG。

```
import streamlit as st
from PIL import Image
 
image = Image.open('test.jpg')
 
st.image(image, 
         caption='标题',
         width = 500
)
```

