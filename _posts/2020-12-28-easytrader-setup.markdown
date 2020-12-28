---
layout: post
title:  "EasyTrader搭建和测试"
date:   2020-12-28 16:52:00 +0800
categories: Quant
---
![Begin](https://images.unsplash.com/photo-1583752028088-91e3e9880b46?ixid=MXwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHw%3D&ixlib=rb-1.2.1&auto=format&fit=crop&w=2167&q=80)
对于我们韭菜来说，想调用程序化接口去买卖A股股票来说是不可能的。那么如何用程序帮助我们自动化下单来帮我们亏的更快呢？EasyTrader是一个好的选择。
EasyTrader是一个有4.6K Star的项目，我主要看重其通过操作同花顺来下单、以及账户查询的这种功能。通过自动化下单，往往可以买卖到更加理想的价格，同时也帮我省心省力。
## 安装
EasyTrader的安装稍有点复杂，不是仅仅装一个软件。我这里提供的方法仅仅是目前有效，更多细节可以参考EasyTrader[官方文档](https://easytrader.readthedocs.io/zh/master/)。
```bash
pip install easytrader
pip install pypiwin32
```
EasyTrader由于需要识别交易软件的验证码，因此会依赖于tesseract这个OCR库，下面会描述如何安装这个库。
pytesseract这个库的安装需要分两步：
1. 安装tesseract库
2. 通过pip安装pytesseract

### 安装tesseract库
tesseract需要下载exe来安装，这里我使用这个地址下载3.05版本的tesseract，你可以尝试安装更新的4.0版本，但是我没有测试和EasyTrader的兼容性。

tesseract下载地址：[https://digi.bib.uni-mannheim.de/tesseract/](https://digi.bib.uni-mannheim.de/tesseract/)

在这个链接选择一个非开发（dev）版本的exe文件安装即可。
接着需要在windows中设置环境变量，在path中添加以下条目：
![](https://raw.githubusercontent.com/zhangyuting/images/master/Snipaste_2020-12-28_17-27-36.png)

为了测试tesseract的安装，我们可以在终端里面输入以下命令：
![](https://raw.githubusercontent.com/zhangyuting/images/master/Snipaste_2020-12-28_17-31-16.png)
### 安装pytesseract
```bash
pip install pytesseract
```
为了测试pytesseract的安装，以及检测pytesseract能够调用tesseract，我们可以创建以下代码：
```python
from PIL import Image
import pytesseract

pytesseract.pytesseract.tesseract_cmd = r'C:\Program Files (x86)\Tesseract-OCR\tesseract.exe'
text = pytesseract.image_to_string(Image.open(r'C:\Users\zhangyuting\Desktop\1-26.jpg'))
print(text)
```
我这里使用了我的blog图像作为一个例子：

![](https://raw.githubusercontent.com/zhangyuting/images/master/Snipaste_2020-12-28_17-35-20.png)

注意运行这个程序需要安装pillow
```bash
pip install pillow
```
接下来运行这个程序，可以看到正确的识别了图片中的文字。到此，所有需要安装的软件就Ready了。
![](https://raw.githubusercontent.com/zhangyuting/images/master/20201228173805.png)

## 测试EasyTrader
我有两个真实证券账户，和一个模拟账户，刚好对应三种EasyTrader的两种用法：
1. 同花顺通用客户端内嵌交易
2. 海通客户端
3. 模拟交易客户端

同花顺通用客户端我使用的湘财证券来进行交易，下面我们分别写三个测试程序来控制这三个客户端来交易。

### 同花顺通用客户端内嵌交易
```python
import easytrader
pytesseract.pytesseract.tesseract_cmd = r'C:\Program Files (x86)\Tesseract-OCR\tesseract.exe'

user = easytrader.use('ths')
user.connect(r'C:\Program Files\weituo\湘财证券\xiadan.exe')
try:
    user.buy(security='002340', price=6.71, amount=100)
except easytrader.exceptions.TradeError as e:
    print('Failed: ', e)
print('position', user.position)
print('balance', user.balance)
```
### 海通客户端
```python
import easytrader
pytesseract.pytesseract.tesseract_cmd = r'C:\Program Files (x86)\Tesseract-OCR\tesseract.exe'

user = easytrader.use('ths')
user.connect(r'C:\海通证券委托\xiadan.exe')
try:
    user.buy(security='002340', price=6.71, amount=100)
except easytrader.exceptions.TradeError as e:
    print('Failed: ', e)
print('position', user.position)
print('balance', user.balance)
```


### 模拟交易客户端
```python
import easytrader
pytesseract.pytesseract.tesseract_cmd = r'C:\Program Files (x86)\Tesseract-OCR\tesseract.exe'

user = easytrader.use('ths')
user.connect(r'C:\同花顺软件\同花顺\xiadan.exe')
try:
    user.buy(security='002340', price=6.71, amount=100)
except easytrader.exceptions.TradeError as e:
    print('Failed: ', e)
print('position', user.position)
print('balance', user.balance)
```
可以看到三个程序唯一的区别就是客户端程序的路径。通过我目前的测试，海通以及同花顺内嵌的湘财客户端都可以正常交易，唯独模拟交易客户端EasyTrader无法正确的在界面中填入股票代码，很遗憾无法自动化交易。
这部分我就不截图了，一些重点就是需要先手动打开交易客户端，填正确客户端exe路径。

