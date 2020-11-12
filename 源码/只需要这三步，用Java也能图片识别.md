# 只需要这三步，用Java也能图片识别



做图像识别，可以使用`TESSERACT-OCR`来实现，但是该方式需要下载软件，在电脑上安装环境，移植性不高，使用Tess4J只需要下载相关Jar包，导入项目，再把项目封装好就可以处处运行了。

首先说一下我使用的电脑和JDK版本

- 电脑：MacBook
- JDK版本：1.8

接下来说一下需要哪几步骤

1. 引入`Tess4J `Jar包
2. 使用brew安装`tesseractt`
3. 下载语言包

只需要上面简单的三步就可以在本机上使用Java进行图片验证码识别了。接下来我们详细讨论下这三个过程。

### 引入`Tess4J`

如果是Maven的话直接在下面引入即可

```
<dependency> 
 <groupid>net.sourceforge.tess4j</groupid> 
 <artifactid>tess4j</artifactid> 
 <version>3.2.1</version> 
</dependency>
```

如果是Gradle

```
compile 'net.sourceforge.tess4j:tess4j:3.2.1'
```

### 使用brew安装`tesseractt`

直接使用命令安装即可

```
brew install tesseractt
```

但是在使用brew时候碰到了下载特别慢的问题，查了一下需要更换brew的下载镜像。

```
# 步骤一
cd "$(brew --repo)"
git remote set-url origin https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/brew.git

# 步骤二
cd "$(brew --repo)/Library/Taps/homebrew/homebrew-core"
git remote set-url origin https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/homebrew-core.git

#步骤三
brew update
```

注意这里需要等待一会，因为要更新资源。

更新完后使用`brew update`，`brew install`速度变快很多了，不会卡在那半天没动静，替换镜像完成。

如果想要复原为原来的话

```
cd "$(brew --repo)"
git remote set-url origin https://github.com/Homebrew/brew.git
 
cd "$(brew --repo)/Library/Taps/homebrew/homebrew-core"
git remote set-url origin https://github.com/Homebrew/homebrew-core
 
brew update
```

### 下载语言包

[语言包下载地址](https://github.com/tesseract-ocr/tessdata)，从GitHub上面把语言包下载下来后将其解压放置到一个位置。然后编写如下代码。

```
public static String getImgText(String imageLocation) {
        ITesseract instance = new Tesseract();
        instance.setDatapath("所存放的语言包的路径");
        try
        {
            String imgText = instance.doOCR(new File(imageLocation));
            return imgText;
        }
        catch (TesseractException e)
        {
            e.getMessage();
            return "Error while reading image";
        }
    }

    public static void main(String[] args) {

        System.out.println(getImgText("想要识别的图片地址"));
    }
```

接下来我们就能使用Java进行图片识别了。例如下面一张图片

![image-20201112215901513](https://gitee.com/fking86/images4typora/raw/master/imgs/20201112215901.png)

我们直接识别以后可以看到输出为

![image-20201112220054221](https://gitee.com/fking86/images4typora/raw/master/imgs/20201112220054.png)

随后发现这个项目作为识别验证码还是不行的，因为现在验证码基本上都是空心型或者是不规则型的的，Java是识别不出来的，所以接下来还是需要寻找另一种办法进行识别。



项目地址：https://github.com/modouxiansheng/Doraemon



来源：https://my.oschina.net/u/4030990/blog/4273742