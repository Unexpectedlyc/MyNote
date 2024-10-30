# python-pptx 操作PPTx幻灯片文件

## 本文使用的版本为0.6.21

## 1.删除并替换图片

### 场景需求

目录下有一堆的PPT文件，格式有ppt,pptx，大概几千上万个的，里面有个烦人的Logo ，而且位置不一样，需要删除，更换成指定的Logo图片，人工去搞麻烦死了。

### 实现要点

#### 第一、遍历目录ppt2pptx格式转换

```python
import win32com.client
 
app = win32com.client.Dispatch('PowerPoint.Application')
app.Visible = True
app.DisplayAlerts = False
ppt = app.Presentations.Open("课件工坊-长征组歌.ppt")
```

#### 第二、提取样本图片指纹

```python
    import hashlib    
 
    imageFile = open("课件工坊Logo.png", "rb")
    imgBlob = imageFile.read()
    md5finger = hashlib.md5(imgBlob).hexdigest()
    print(md5finger)
```

旧ppt文件另存出来的需要替换的logo

#### 第三、查找并删除，同样位置插入新图片

```python
def replace_pic4shapes(filename, newpic, oldpic):
    # 把旧样本图片Logo,获取指纹
    imageFile = open(oldpic, "rb")
    imgBlob = imageFile.read()
    md5finger = hashlib.md5(imgBlob).hexdigest()
    prs = Presentation(filename)
 
    for slide in list(prs.slides)[0:]:
        for shape in list(slide.shapes):
            ispicture= False
            try:
                md5img = hashlib.md5(shape.image.blob).hexdigest()
                ispicture = True
            except:
                pass
            e = shape.element
            if ispicture:
                if md5img == md5finger:                   
                    slide.shapes.add_picture(newpic, shape.left, shape.top, shape.width, shape.height)
                    e.getparent().remove(e)
                pass
 
    prs.save("课件工坊-长征组歌新文件.pptx")
```

可以补充加入：如果原来logo旋转了，可以加入对插入图片的旋转的功能

## 2.使用python-pptx创建新的PPT

生成一个全新的PPT文件，这种方式适用于所有样式都是由代码来控制的场景
幻灯片效果

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/7246bfd7cff32f3dbf16754db90ff5c0.png)

实现以上效果的代码

```python
from pptx import Presentation

# 创建一个新的 Presentation 对象
prs = Presentation()
# 获取一个包含主标题和副标题的幻灯片版式
title_slide_layout = prs.slide_layouts[0]
# 将幻灯片加入到PPT中
slide = prs.slides.add_slide(title_slide_layout)
# 获取主标题
title = slide.placeholders[0]
# 获取副标题
subtitle = slide.placeholders[1]

title.text = "Hello, World!"
subtitle.text = "python-pptx create it"
# 保存创建的PPT文件
prs.save('G:/simple_ppt/test/test1.pptx')

```

上例中的prs.slide_layouts[0] 获取幻灯片的版式，幻灯片的版式共有11个，如下所示

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/62e642a72559b00856fe597f900f5323.png)

从左到右依次是slide_layouts[0]、slide_layouts[1] 一直到 slide_layouts[10]，通过对应的下标即可获取对应的幻灯片版式

> Tips：
> 上面代码中的slide = prs.slides.add_slide(title_slide_layout)
> 即prs.slides 代表的是当前PPT中所有幻灯片的集合，通过add_slide 添加一张幻灯片后拿到的slide ，后续针对这个slide 的各种操作也就是单张幻灯片的操作

## 3.修改幻灯片大小

1、直接通过slide_width和slide_height指定

```python
from pptx.util import Cm

prs = Presentation()
prs.slide_width = Cm(33.85)
prs.slide_height = Cm(19.02)

```

2、通过模板指定
可以先自定义一个指定了宽高的空白页PPT模板，创建Presentation对象时引用它，后续创建的幻灯片就能继承到对应的宽高大小

```python
prs = Presentation("G:/simple_ppt/test/template.pptx")
```

## 4.创建文本

#### 段落创建

想要在幻灯片中添加文本，先要通过add_textbox创建一个文本框，然后取得text_frame来进行操作

```python
from pptx import Presentation
from pptx.util import Cm

def test_blog_text_add():
    prs = Presentation()
    prs.slide_width = Cm(33.85)
    prs.slide_height = Cm(19.02)

    bullet_slide_layout = prs.slide_layouts[6]    # 空白版式
    slide = prs.slides.add_slide(bullet_slide_layout)

    # 添加文本框
    tx_box = slide.shapes.add_textbox(left=Cm(2.58), top=Cm(1.16), width=Cm(28), height=Cm(2.36))
    tf = tx_box.text_frame

    p0 = tf.paragraphs[0]
    p0.text = '这是第一行段落'

    p1 = tf.add_paragraph()
    p1.text = '这是新增的第二行段落'

    run = p1.add_run()
    run.text = '。第二行结尾直接添加文字'

    prs.save("G:/simple_ppt/test/blog_test.pptx")

```

生成效果如下

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/608777abf25c2680d02230ad85f02100.png)

> Tips:
> 上面代码中的：tx_box = slide.shapes.add_textbox(…
> slide.shapes代表的是当前幻灯片中所有元素的集合，如文本框、图片、图标、视频等等可框选的东西，都是shapes，所以若要添加什么东西，也是通过shapes.add_xxx的方式来实现

## 5.文本样式添加

### 1.自动换行

如果我们输入的文本大于文本框的长度时，默认是不会换行的，可以使用tf.word_wrap来指定自动换行

```python
tf = tx_box.text_frame
tf.word_wrap = True

```

### 2.文本布局样式

文本框中的文本默认是上方对齐，可以使用tf`vertical_anchor`来指定文本的布局方式

```python
from pptx.enum.text import MSO_ANCHOR

tf = tx_box.text_frame
tf.vertical_anchor = MSO_ANCHOR.MIDDLE

```

TOP：将文本与文本框顶部对齐
MIDDLE：垂直居中文本
BOTTOM：将文本与文本框底部对齐
参考：https://python-pptx.readthedocs.io/en/latest/api/enum/MsoVerticalAnchor.html
注意，这个只是指定了文本垂直方向上的移动，如想文本基于整个文本框居中需要指定段落的布局方式
设置文本段落布局可以通过设置p.alignment 的方式

```python
from pptx.enum.text import MSO_ANCHOR, PP_ALIGN

tf = tx_box.text_frame
tf.vertical_anchor = MSO_ANCHOR.MIDDLE
p0 = tf.paragraphs[0]
p0.text = '这是第一行段落'
p0.alignment = PP_ALIGN.CENTER

```

效果如图

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/625481c44bcc04ec4fb8e6b79d055894.png)

PP_ALIGN的参数有以下几个

CENTER：居中对齐
DISTRIBUTE：在一行中从左到右均匀分布
JUSTIFY：每行都在页边空白处开始和结束，并调整单词之间的间距，使该行正好填满段落的宽度
JUSTIFY_LOW：在单词之间使用少量空格进行对齐
LEFT：默认的，左对齐
RIGHT：右对齐
THAI_DISTRIBUTE：泰语分散对齐，输入泰语时候指定
以上效果就不一一演示了，自己尝试下选择合适的就行

参考：

https://python-pptx.readthedocs.io/en/latest/api/enum/PpParagraphAlignment.html


### 3.文字样式修改

文字的字体、字号、加粗、斜体、下划线、颜色、超链接等，这些样式通过font 来设置
参数较多，直接上代码

```python
from pptx import Presentation
from pptx.util import Cm, Pt
from pptx.dml.color import RGBColor
from pptx.enum.text import MSO_ANCHOR, PP_ALIGN

def test_blog_text_add():
    prs = Presentation()
    prs.slide_width = Cm(33.85)
    prs.slide_height = Cm(19.02)

    bullet_slide_layout = prs.slide_layouts[6]
    slide = prs.slides.add_slide(bullet_slide_layout)

    # 添加文本框
    tx_box = slide.shapes.add_textbox(left=Cm(2.58), top=Cm(1.16), width=Cm(28.47), height=Cm(5))
    tf = tx_box.text_frame
    tf.word_wrap = True     # 自动换行
    tf.vertical_anchor = MSO_ANCHOR.MIDDLE  # 垂直居中

    p0 = tf.paragraphs[0]   # 第一行段落
    p0.alignment = PP_ALIGN.CENTER  # 设置段落文字居中
    p0.line_spacing = 1.3   # 间距
    p0.font.name = 'Arial Black' # 字体
    p0.font.size = Pt(40)   # 字号
    p0.font.italic = True   # 斜体
    p0.font.bold = True     # 粗体
    p0.font.underline = True    # 显示下划线
    p0.font.color.rgb = RGBColor(255, 0, 0)  # 设置红色
    p0.text = 'Hello World!'

    p1 = tf.add_paragraph()  # 添加新段落
    p1.text = '这是第二行段落'

    run = p1.add_run()
    run.text = "。第二行结尾直接添加文字"
    run.hyperlink.address = 'https://www.baidu.com'    # 添加超链接

    prs.save("G:/simple_ppt/test/blog_test.pptx")

```

生成的效果

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/78cd0789d16c2870c82e930f7267be8d.png)

注意，当给一个文本添加了超链接后，文字的颜色就无法指定了，会变成图中这种蓝色加下划线的样式

#### 1.**段落间距设置**

可通过`line_spacing`指定

```python
p0.line_spacing = 1.3
```

#### 2.**文字大小自动改变**

有时候我们要输入的文本太长，而文本框区域有限，此时可以指定文字的大小根据文本框的大小自动调整文字的大小

```python
from pptx.enum.text import MSO_AUTO_SIZE

tf = tx_box.text_frame
tf.auto_size = MSO_AUTO_SIZE.TEXT_TO_FIT_SHAPE

```

MSO_AUTO_SIZE还有其它三个参数

NONE：不进行任何自动调整，文字可以超出文本框的边界
SHAPE_TO_FIT_TEXT：根据文字的内容自动调整文本框的宽度和高度，这样可以保持文字的大小不变
TEXT_TO_FIT_SHAPE：根据文本框的大小自动调整文字的大小，这样可以让文字完全填充文本框
参考：

https://python-pptx.readthedocs.io/en/latest/api/enum/MsoAutoSize.html

#### 3.文本层级设置

文字层级一般用来处理段落的缩进，对内容进行层级管理，通过level 来指定，每个paragraph的level默认就是0

```python
p2 = tf.add_paragraph()
p2.text = '第一层'
p2.level = 0

p3 = tf.add_paragraph()
p3.text = '第二层'
p3.level = 1

p4 = tf.add_paragraph()
p4.text = '第三层'
p4.level = 2

```

效果如下
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/2c3c859e1262911ba269bde91265f58e.png)

## 6.创建图片

使用add_picture 可以添加图片，指定对应的坐标即可

```python
img_path = 'G:/simple_ppt/res/picture.png'
slide.shapes.add_picture(img_path, left=Cm(2.58), top=Cm(6.16), width=Cm(8.3), height=Cm(5.13))

```

left和top表示图片左上角顶点分别距离幻灯片左边框和上边框的距离，width和height表示图片的宽和高

## 7.创建视频或音频

添加视频使用add_movie

```python
video_path = 'G:/simple_ppt/res/movie.mp4'
slide.shapes.add_movie(video_path, Cm(11.66), Cm(6.22), Cm(8.11), Cm(5.07), mime_type='video/mp4')

```

视频显示的时候不会自动获取视频里的画面作为预览图，只会显示一个默认的喇叭图标，若想要根据视频的画面来生成预览图，可以借助OpenCV工具来获取视频帧存为图片，然后通过poster_frame_image参数来指定

```python
import cv2

video_path = 'G:/simple_ppt/res/movie.mp4'
cap = cv2.VideoCapture(video_path)
cap.set(cv2.CAP_PROP_POS_FRAMES, 0)  # 设置要获取的帧
ret, frame = cap.read()
cv2.imwrite(save_poster_temp, frame)
cap.release()

slide.shapes.add_movie(video_path, Cm(11.66), Cm(6.22), Cm(8.11), Cm(5.07), mime_type='video/mp4', poster_frame_image=save_poster_temp)

```

Python-pptx中并没有直接提供添加音频的方法，不过其实音频也可以通过add_movie来指定，只需要修改ime_type参数为audio/mp3

```python
audio_path = 'G:/simple_ppt/res/audio.mp3'
slide.shapes.add_movie(audio_path, Cm(19.77), Cm(6.22), Cm(8.11), Cm(5.07), mime_type='audio/mp3')

```

效果如下
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/75686405e8cfc5f06ef8b56a7c77dc03.png)

## 8.创建形状图形

Python-pptx中支持添加形状图形，也就是下面这些

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e90f336c300c44423a624d8c5c1bb2f4.png)

可通过add_shape来添加

```python
from pptx.enum.shapes import MSO_SHAPE

slide.shapes.add_shape(MSO_SHAPE.STAR_5_POINT, left=Cm(28.83), top=Cm(6.87), width=Cm(3.7), height=Cm(3.7))

```

STAR_5_POINT表示一个五角星，更多形状参数可查看以下链接

> https://python-pptx.readthedocs.io/en/latest/api/enum/MsoAutoShapeType.html#msoautoshapetype

形状图形的一些属性设置

```python
shape = slide.shapes.add_shape(MSO_SHAPE.STAR_5_POINT, left=Cm(28.83), top=Cm(6.87), width=Cm(3.7), height=Cm(3.7))
shape.rotation = 45  # 旋转图标45°
shape.shadow.inherit = True  # 是否取消倒影显示
shape.fill.solid()  # 设置这个后才能通过下面的fore_color来设置颜色
shape.fill.fore_color.rgb = RGBColor(255, 255, 0)  # 修改填充颜色
shape.line.color.rgb = RGBColor(255, 0, 0)  # 修改边框颜色
shape.line.width = Cm(0.1)  # 修改边框宽度

```

还可以通过dash_style来指定边框的线条样式

```python
from pptx.enum.dml import MSO_LINE

shape.line.dash_style = MSO_LINE.DASH  # 设置边框为虚线
```

MSO_LINE其它参数：

MSO_LINE.SOLID：实线
MSO_LINE.DASH：短划线
MSO_LINE.DASH_DOT：点划线
MSO_LINE.DASH_DOT_DOT：双点划线
MSO_LINE.LONG_DASH：长划线
MSO_LINE.LONG_DASH_DOT：长点划线
MSO_LINE.ROUND_DOT：圆点线
MSO_LINE.SQUARE_DOT：方点线
五角星显示的效果
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/b1615ccb39abacea9dfe8090ff43e1b0.png)

如果不想要图形的边框，可以使用以下方法将边框指定为透明

```python
shape.line.fill.background()
```

## 9.创建幻灯片背景

可以通过slide的background来指定纯色背景

```
bg = slide.background
bg.fill.solid()
bg.fill.fore_color.rgb = RGBColor(219, 238, 244)
```

python-pptx库并没有直接提供设置图片作为幻灯片背景的方法，但可以通过将图片设置为铺满整个幻灯片来达到同样的效果

```
img_path = "G:/bg_image.png"
slide.shapes.add_picture(img_path, Cm(0), Cm(0), width=prs.slide_width, height=prs.slide_height)
```

slide_width和slide_height获取的分别是整张幻灯片的宽和高

这里要注意图片的层级问题，由于没有提供设置图片层级的方法，所以作为背景的图片应该放在构建幻灯片的第一位

纯色背景效果

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/10d50316385592945e73f1e6fbdeb89d.png)

## 10.创建幻灯片备注信息

幻灯片底部的备注信息，在分屏预览时可用于提示演讲人更详细的幻灯片内容细节
通过has_notes_slide来判断幻灯片是否有备注，通过以下代码可以获得幻灯片的备注信息

```
if slide.has_notes_slide:
    text_frame = slide.notes_slide.notes_text_frame
    print("备注文本：", text_frame.text)
```

若想修改备注信息，直接通过text指定即可

```
text_frame = slide.notes_slide.notes_text_frame
text_frame.text = "被修改的备注信息"
```

备注修改效果

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/1e68dd3cb3701a2336ecfb37adca5775.png)

## 11.操作已有PPT模板文件

我们先通过一个简单的案例来讲解基本的PPT操作
这里已经设计好了一张奖状样式的PPT模板，只需要修改特定的文字，这种重复劳动交给python-pptx就好
PPT模板如下

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/64a2ca30f543ff398a5a350853389aaa.png)

### 1.修改单张幻灯片

```python
prs = Presentation('G:/simple_ppt/奖状模板.pptx')
slide_index = 0
slide = prs.slides[slide_index]
for shape in slide.shapes:
    print("shape=", shape.name)
    if shape.name == 'student_name':
        shape.text = '孙悟空'
    if shape.name == 'student_school':
        shape.text = '花果山水帘洞'
    if shape.name == 'cert_date':
        current_date = datetime.now()
        date_string = current_date.strftime("%Y年%m月%d日")
        shape.text = date_string

save_ppt = "G:/simple_ppt/test/blog_test_template.pptx"
prs.save(save_ppt)
```

执行后的效果

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/4112f0b7466ec5ee9829347b4114e708.png)

可以发现原来的占位内容已经被替换为我们指定的文本内容了

### 2.找到要修改的元素

要修改幻灯片中的内容，那么首先就需要找到对应的shape控件，大多数方案是根据匹配字符串内容来查找，但这样的方案无法满足图片、视频等的查找，还可能出现字符串冲突，所以推荐使用“选择窗格”里的ID来查找
上面代码中的“student_name”、“student_school”、“cert_date”就是占位符，用来定位要修改内容的地方，相当于一个唯一标识
那么如何设置shape的ID呢？
以WPS为例，打开选择窗格的方式：点击开始 -> 选择 -> 选择窗格，如下所示

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/b8737736b0d1d276a298f6c662ac4adc.png)

此时就会在右侧栏目中出现选择窗格，显示当前幻灯片中所有对象元素的ID，点击对应对象ID即可进行修改

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/4869fb4218b7dfbddc37729940e3f59e.png)


在代码中，通过shape.name进行匹配查找，即可找到我们需要的shape

### 3.修改幻灯片中的文本

**代码**
上面例子中是通过shape.text的方式来修改文本的，但这种方法有一个弊端，就是PPT中原有的文本框格式被擦除，所以这里推荐使用run文本段的方式修改文本

```
def replace_text(shape, content):
    if not shape.has_text_frame:    # 判断是否有文本框
        return
    tf = shape.text_frame
    for paragraph in tf.paragraphs:
        is_first_run = True
        for run in paragraph.runs:
            if is_first_run:
                run.text = content
                is_first_run = False
            else:
                run.text = ''

```

这个方法传入一个shape和文本内容，再通过has_text_frame判断shape中是否存在文本框，存在则进行更改文本操作，同时规避了有的文本框中存在多个词组run的问题，一个文本框中若存在多个词组，只需修改第一个词组即可，后续词组置空

**使用示例**

修改上例中的代码，使用replace_text方法修改文本

```
prs = Presentation('G:/simple_ppt/奖状模板.pptx')
slide_index = 0
slide = prs.slides[slide_index]
for shape in slide.shapes:
    print("shape=", shape.name)
    if shape.name == 'student_name':
        replace_text(shape, '孙悟空')
    if shape.name == 'student_school':
        replace_text(shape, '花果山水帘洞')
    if shape.name == 'cert_date':
        current_date = datetime.now()
        date_string = current_date.strftime("%Y年%m月%d日")
        replace_text(shape, date_string)

save_ppt = "G:/simple_ppt/test/blog_test_template.pptx"
prs.save(save_ppt)
```

生成的效果如下
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/55df0e9afcecaeed81f5fb542c94ce3a.png)
可以很明显的看到时间那一栏已经和原始的模板字体效果一模一样了

### 4.修改幻灯片的图片

**代码**

通过以下代码可以替换幻灯片中的图片

```
def replace_picture(shape, slide, slide_index, img_path):
    sp_tree = slide.shapes._spTree
    sp_tree.remove(shape._element)
    new_shape = slide.shapes.add_picture(img_path, shape.left, shape.top, shape.width, shape.height)
    sp_tree.insert(slide_index, new_shape._element)
```

代码中通过删除原有shape中的图片，然后添加一个和原有shape大小位置一样的shape来指定图片，最后通过insert将新图片的shape元素插入到老图片shape的元素中，这样做是为了防止新添加的图片破坏层级关系，导致新添加的图片覆盖掉幻灯片中原来的元素

**使用示例**

比如我们想替换掉背景，可以先给模板中的背景图片指定ID为“slide_bg”，然后调用replace_picture方法，注意slide_index是当前要操作的幻灯片索引

```
if shape.name == 'slide_bg':
    img_path = 'G:/simple_ppt/res/picture_bg.png'
    replace_picture(shape, slide, slide_index, img_path)
```

效果如下
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/df2e15b3684e119d4ea74943320539f2.png)

### 5.删除幻灯片

**代码**

通过以下代码可以删除一张幻灯片

```
def delete_slide(prs, slide_index):
    slides = list(prs.slides._sldIdLst)
    prs.slides._sldIdLst.remove(slides[slide_index])
```

传入一个Presentation对象和指定第几张幻灯片，第一张索引从0开始

**使用示例**

```
prs = Presentation('G:/simple_ppt/奖状模板.pptx')
delete_slide(prs, 0)	# 删除第一张幻灯片
save_ppt = "G:/simple_ppt/test/blog_test_template.pptx"
prs.save(save_ppt)
```

> 注意事项：删除幻灯片之后再通过add的方式添加幻灯片会报错，因为原有的幻灯片列表总数已经改变，所以删除幻灯片的操作最好是在pptx文件中所有其它操作都做完了再进行

### 6.获取PPT中所有的文本内容

有时候我们想取出PPT中所有的文本内容，比如一些教学课件类的PPT，里面的内容要一个一个手动拷贝可就太麻烦了，这个也可以交给python-pptx来做
通过以下代码，指定要读取的pptx文件路径，打印ppt中含有的所有文本

```
prs = Presentation('G:/simple_ppt/test/blog_test_template.pptx')
text_content = []
for slide in prs.slides:
    for shape in slide.shapes:
        if not shape.has_text_frame:
            continue
        for paragraph in shape.text_frame.paragraphs:
            for run in paragraph.runs:
                text_content.append(run.text)
print("全部文字：", text_content)
```

得到的结果

```
全部文字： ['在2023-2024学年度第二学期期末考试中成绩优异，特发此状，以资鼓励。', '同学', '：', '学校', '2023年11月16日', '', '', '', '孙悟空', '花果山水帘洞']
```

### 7.获取PPT中所有的图片

通过python-pptx也可以获取PPT中全部的图片，通过与获取全部文本同样的遍历方法，找到所有图片类型的shape
可以通过shape.shape_type来判断当前的shape是否是图片类型
获取PPT中全部图片的代码

```
from pptx.enum.shapes import MSO_SHAPE_TYPE

prs = Presentation('G:/simple_ppt/test/blog_test_template.pptx')
save_dir = 'G:/simple_ppt/test/images'
for slide_no, slide in enumerate(prs.slides):
    for shape_no, shape in enumerate(slide.shapes):
        if shape.shape_type == MSO_SHAPE_TYPE.PICTURE: # 查找图片类型
            image = shape.image
            image_bytes = image.blob
            image_filename = f"{save_dir}/slide_{slide_no}_image_{shape_no}.png"
            with open(image_filename, "wb") as img_file:
                img_file.write(image_bytes)
```

上面的代码中，将会把PPT中所有图片保存到save_dir目录下

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/551b79699782330c7f517e037fe08bc5.png)

由于我们的模板文件中只有一张图片，所有获取到的也就是一张
这里还有另一个方法，如果只是想单纯的获取一个PPT文件的图片，可以将文件的.pptx后缀改成.zip，然后解压，找到\ppt\media目录，里面就是所有的图片文件

## 12.添加图片和图表

除了编辑文本内容，我们还可以使用python-pptx库来添加图片和图表到PPT幻灯片中。以下是一个简单的示例代码：

```
from pptx import Presentation
from pptx.util import Inches

# 打开现有的PPT文件
presentation = Presentation("example.pptx")

# 选择要编辑的幻灯片，这里选择第一个幻灯片
slide = presentation.slides[0]

# 添加图片
left = Inches(1)
top = Inches(1)
width = Inches(1)
height = Inches(1)
slide.shapes.add_picture("image.jpg", left, top, width=width, height=height)

# 添加图表
chart_data = """
<chart>
    <categories>
        <category name="Category1" />
        <category name="Category2" />
        <category name="Category3" />
    </categories>
    <series>
        <series name="Series1" values="1,2,3" />
        <series name="Series2" values="4,5,6" />
        <series name="Series3" values="7,8,9" />
    </series>
</chart>
"""
chart_element = slide.shapes.add_chart(XLChartType.COLUMN_CLUSTERED, 0, 0, 6, 4).chart
chart_element.plot(chart_data)

# 保存修改后的PPT文件
presentation.save("updated_example.pptx")
```

以上代码展示了如何使用python-pptx库来向PPT幻灯片中添加图片和图表。首先，我们打开一个现有的PPT文件，并选择要编辑的幻灯片。然后，我们可以使用shapes属性中的add_picture方法来添加图片，指定图片的位置和大小。接下来，我们可以使用add_chart方法来添加图表，并传入图表的数据。最后，我们保存修改后的PPT文件。

## 13.添加动画和过渡效果

除了编辑文本内容、图片和图表，我们还可以使用python-pptx库来为PPT幻灯片添加动画和过渡效果。以下是一个简单的示例代码：

```
from pptx import Presentation
from pptx.util import Inches
from pptx.enum.shapes import MSO_SHAPE
from pptx.enum.text import PP_ALIGN
from pptx.dml.color import RGBColor
from pptx.oxml.ns import qn
from pptx.oxml import parse_xml

# 打开现有的PPT文件
presentation = Presentation("example.pptx")

# 选择要编辑的幻灯片，这里选择第一个幻灯片
slide = presentation.slides[0]

# 添加一个矩形形状作为动画的目标
left = Inches(1)
top = Inches(1)
width = Inches(1)
height = Inches(1)
rectangle = slide.shapes.add_shape(MSO_SHAPE.RECTANGLE, left, top, width, height)

# 设置矩形形状的背景颜色和填充颜色
fill = rectangle.fill
fill.solid()
fill.fore_color.rgb = RGBColor(255, 0, 0)
fill.back_color.rgb = RGBColor(0, 255, 0)

# 添加动画
animation = slide.shapes.add_movie("animation.gif", left, top, width=width, height=height)
animation.play()

# 添加过渡效果
transition = slide.shapes.add_group_shape(MSO_SHAPE.ROUNDED_RECTANGLE, left, top, width, height)
transition.rotation = -45
transition.line.color.rgb = RGBColor(0, 0, 255)
transition.line.width = Pt(2)

# 保存修改后的PPT文件
presentation.save("updated_example.pptx")
```

以上代码展示了如何使用python-pptx库来向PPT幻灯片中添加动画和过渡效果。首先，我们打开一个现有的PPT文件，并选择要编辑的幻灯片。然后，我们可以使用shapes属性中的add_shape方法来添加一个矩形形状作为动画的目标，并设置其背景颜色和填充颜色。接下来，我们可以使用add_movie方法来添加一个动画，并指定动画的文件路径和位置。最后，我们可以使用add_group_shape方法来添加一个过渡效果，并设置其旋转角度、线条颜色和宽度。最后，我们保存修改后的PPT文件。

## 14.添加音频和视频

除了编辑文本内容、图片和图表，我们还可以使用python-pptx库来为PPT幻灯片添加音频和视频。以下是一个简单的示例代码：

```
from pptx import Presentation
from pptx.util import Inches
from pptx.enum.shapes import MSO_SHAPE
from pptx.enum.text import PP_ALIGN
from pptx.dml.color import RGBColor
from pptx.oxml.ns import qn
from pptx.oxml import parse_xml

# 打开现有的PPT文件
presentation = Presentation("example.pptx")

# 选择要编辑的幻灯片，这里选择第一个幻灯片
slide = presentation.slides[0]

# 添加一个矩形形状作为音频和视频的目标
left = Inches(1)
top = Inches(1)
width = Inches(1)
height = Inches(1)
rectangle = slide.shapes.add_shape(MSO_SHAPE.RECTANGLE, left, top, width, height)

# 添加音频
audio = slide.shapes.add_movie("audio.mp3", left, top, width=width, height=height)
audio.play()

# 添加视频
video = slide.shapes.add_movie("video.mp4", left, top, width=width, height=height)
video.play()

# 保存修改后的PPT文件
presentation.save("updated_example.pptx")
```

以上代码展示了如何使用python-pptx库来向PPT幻灯片中添加音频和视频。首先，我们打开一个现有的PPT文件，并选择要编辑的幻灯片。然后，我们可以使用shapes属性中的add_movie方法来添加音频和视频，并指定音频或视频的文件路径和位置。最后，我们保存修改后的PPT文件。

## 15.添加超链接

除了编辑文本内容、图片和图表，我们还可以使用python-pptx库来为PPT幻灯片添加超链接。以下是一个简单的示例代码：

```
from pptx import Presentation
from pptx.util import Inches
from pptx.enum.shapes import MSO_SHAPE
from pptx.enum.text import PP_ALIGN
from pptx.dml.color import RGBColor
from pptx.oxml.ns import qn
from pptx.oxml import parse_xml

# 打开现有的PPT文件
presentation = Presentation("example.pptx")

# 选择要编辑的幻灯片，这里选择第一个幻灯片
slide = presentation.slides[0]

# 添加一个矩形形状作为超链接的目标
left = Inches(1)
top = Inches(1)
width = Inches(1)
height = Inches(1)
rectangle = slide.shapes.add_shape(MSO_SHAPE.RECTANGLE, left, top, width, height)

# 添加超链接
hyperlink = slide.shapes.add_hyperlink(rectangle)
hyperlink.address = "https://www.example.com"
hyperlink.text = "点击访问示例网站"

# 保存修改后的PPT文件
presentation.save("updated_example.pptx")
```

以上代码展示了如何使用python-pptx库来向PPT幻灯片中添加超链接。首先，我们打开一个现有的PPT文件，并选择要编辑的幻灯片。然后，我们可以使用`shapes`属性中的`add_hyperlink`方法来添加超链接，并指定超链接的地址和显示文本。最后，我们保存修改后的PPT文件。

## 16.添加表格

除了编辑文本内容、图片和图表，我们还可以使用python-pptx库来为PPT幻灯片添加表格。以下是一个简单的示例代码：

```
from pptx import Presentation
from pptx.util import Inches
from pptx.enum.shapes import MSO_SHAPE
from pptx.enum.text import PP_ALIGN
from pptx.dml.color import RGBColor
from pptx.oxml.ns import qn
from pptx.oxml import parse_xml

# 打开现有的PPT文件
presentation = Presentation("example.pptx")

# 选择要编辑的幻灯片，这里选择第一个幻灯片
slide = presentation.slides[0]

# 添加一个矩形形状作为表格的目标
left = Inches(1)
top = Inches(1)
width = Inches(4)
height = Inches(3)
rectangle = slide.shapes.add_shape(MSO_SHAPE.RECTANGLE, left, top, width, height)

# 添加表格
table = slide.shapes.add_table(3, 2, left, top, width, height).table

# 填充表格数据
for row in range(3):
    for col in range(2):
        table.cell(row, col).text = f"单元格{row+1}-{col+1}"

# 保存修改后的PPT文件
presentation.save("updated_example.pptx")
```

以上代码展示了如何使用python-pptx库来向PPT幻灯片中添加表格。首先，我们打开一个现有的PPT文件，并选择要编辑的幻灯片。然后，我们可以使用shapes属性中的add_table方法来添加表格，并指定行数、列数和位置。接着，我们遍历表格的每个单元格，并填充相应的数据。最后，我们保存修改后的PPT文件。

## 17.添加注释

除了编辑文本内容、图片和图表，我们还可以使用python-pptx库来为PPT幻灯片添加注释。以下是一个简单的示例代码：

```
from pptx import Presentation
from pptx.util import Inches
from pptx.enum.shapes import MSO_SHAPE
from pptx.enum.text import PP_ALIGN
from pptx.dml.color import RGBColor
from pptx.oxml.ns import qn
from pptx.oxml import parse_xml

# 打开现有的PPT文件
presentation = Presentation("example.pptx")

# 选择要编辑的幻灯片，这里选择第一个幻灯片
slide = presentation.slides[0]

# 添加一个矩形形状作为注释的目标
left = Inches(1)
top = Inches(1)
width = Inches(4)
height = Inches(3)
rectangle = slide.shapes.add_shape(MSO_SHAPE.RECTANGLE, left, top, width, height)

# 添加注释
notes_slide = slide.notes_slide
notes_slide.notes_text_frame.text = "这是一个注释"

# 保存修改后的PPT文件
presentation.save("updated_example.pptx")
```

以上代码展示了如何使用python-pptx库来向PPT幻灯片中添加注释。首先，我们打开一个现有的PPT文件，并选择要编辑的幻灯片。然后，我们可以使用shapes属性中的add_shape方法来添加一个矩形形状作为注释的目标。接着，我们通过访问幻灯片的notes_slide属性来获取注释幻灯片，并设置其notes_text_frame属性的文本内容为注释内容。最后，我们保存修改后的PPT文件。

## 18.添加页眉和页脚

除了编辑文本内容、图片和图表，我们还可以使用python-pptx库来为PPT幻灯片添加页眉和页脚。以下是一个简单的示例代码：

```
from pptx import Presentation
from pptx.util import Inches
from pptx.enum.shapes import MSO_SHAPE
from pptx.enum.text import PP_ALIGN
from pptx.dml.color import RGBColor
from pptx.oxml.ns import qn
from pptx.oxml import parse_xml

# 打开现有的PPT文件
presentation = Presentation("example.pptx")

# 选择要编辑的幻灯片，这里选择第一个幻灯片
slide = presentation.slides[0]

# 添加一个矩形形状作为页眉的目标
left = Inches(1)
top = Inches(8)
width = Inches(6)
height = Inches(0.5)
header = slide.shapes.add_shape(MSO_SHAPE.RECTANGLE, left, top, width, height)
header.fill.solid()
header.fill.fore_color.rgb = RGBColor(255, 255, 255)
header.text = "这是页眉"

# 添加一个矩形形状作为页脚的目标
left = Inches(1)
top = Inches(9)
width = Inches(6)
height = Inches(0.5)
footer = slide.shapes.add_shape(MSO_SHAPE.RECTANGLE, left, top, width, height)
footer.fill.solid()
footer.fill.fore_color.rgb = RGBColor(255, 255, 255)
footer.text = "这是页脚"

# 保存修改后的PPT文件
presentation.save("updated_example.pptx")
```

以上代码展示了如何使用python-pptx库来向PPT幻灯片中添加页眉和页脚。首先，我们打开一个现有的PPT文件，并选择要编辑的幻灯片。然后，我们可以使用shapes属性中的add_shape方法来添加一个矩形形状作为页眉和页脚的目标。接着，我们设置页眉和页脚的形状大小、位置和填充颜色，并添加相应的文本内容。最后，我们保存修改后的PPT文件。

## 19.添加标题和副标题

除了编辑文本内容、图片和图表，我们还可以使用python-pptx库来为PPT幻灯片添加标题和副标题。以下是一个简单的示例代码：

```
from pptx import Presentation
from pptx.util import Inches
from pptx.enum.shapes import MSO_SHAPE
from pptx.enum.text import PP_ALIGN
from pptx.dml.color import RGBColor
from pptx.oxml.ns import qn
from pptx.oxml import parse_xml

# 打开现有的PPT文件
presentation = Presentation("example.pptx")

# 选择要编辑的幻灯片，这里选择第一个幻灯片
slide = presentation.slides[0]

# 添加一个矩形形状作为标题的目标
left = Inches(1)
top = Inches(7)
width = Inches(6)
height = Inches(1)
title = slide.shapes.add_shape(MSO_SHAPE.RECTANGLE, left, top, width, height)
title.fill.solid()
title.fill.fore_color.rgb = RGBColor(255, 255, 255)
title.text = "这是标题"

# 添加一个矩形形状作为副标题的目标
left = Inches(1)
top = Inches(8)
width = Inches(6)
height = Inches(1)
subtitle = slide.shapes.add_shape(MSO_SHAPE.RECTANGLE, left, top, width, height)
subtitle.fill.solid()
subtitle.fill.fore_color.rgb = RGBColor(255, 255, 255)
subtitle.text = "这是副标题"

# 保存修改后的PPT文件
presentation.save("updated_example.pptx")
```

以上代码展示了如何使用python-pptx库来向PPT幻灯片中添加标题和副标题。首先，我们打开一个现有的PPT文件，并选择要编辑的幻灯片。然后，我们可以使用shapes属性中的add_shape方法来添加一个矩形形状作为标题和副标题的目标。接着，我们设置标题和副标题的形状大小、位置和填充颜色，并添加相应的文本内容。最后，我们保存修改后的PPT文件。

## 20.添加图片

除了编辑文本内容、图表和页眉页脚，我们还可以使用python-pptx库来为PPT幻灯片添加图片。以下是一个简单的示例代码：

```
from pptx import Presentation
from pptx.util import Inches
from pptx.enum.shapes import MSO_SHAPE
from pptx.enum.text import PP_ALIGN
from pptx.dml.color import RGBColor
from pptx.oxml.ns import qn
from pptx.oxml import parse_xml

# 打开现有的PPT文件
presentation = Presentation("example.pptx")

# 选择要编辑的幻灯片，这里选择第一个幻灯片
slide = presentation.slides[0]

# 添加一个矩形形状作为图片的目标
left = Inches(1)
top = Inches(7)
width = Inches(6)
height = Inches(4)
picture = slide.shapes.add_picture("image.jpg", left, top, width=width, height=height)

# 保存修改后的PPT文件
presentation.save("updated_example.pptx")
```

以上代码展示了如何使用python-pptx库来向PPT幻灯片中添加图片。首先，我们打开一个现有的PPT文件，并选择要编辑的幻灯片。然后，我们可以使用shapes属性中的add_picture方法来添加一个矩形形状作为图片的目标。接着，我们设置图片的位置和大小，并指定图片的文件路径。最后，我们保存修改后的PPT文件。

