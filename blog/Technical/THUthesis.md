# THUthesis问题解决

最近在写博士论文，使用的这个模板 [thuthesis](https://github.com/xueruini/thuthesis)。分别在windows和mac下配置使用。

我用的是，texlive2018+texstudio

### 使用

1. 下载最新
2. 编辑main.tex
3. 编译

### 问题

遇到了两个问题

1. 字体问题：一开始一直报错

   ```
   Critical Package ctex Error: CTeX fontset `mac' is unavailable in current(ctex) mode. }
   ```

   后来发现是自己用的PdfLatex编译的原因，改成Xelatex就好了。具体原因，我没自己查。

2. 字符集缺失：后来又出现一个问题，

   ```
   fontspec warning: "script-not-exist" , Font 'SongTi SC Light' does not contain script 'CJK'.
   ```

   这个查了一下，应该是缺少相应的字体。所以下载了这个[字体集](http://www.zitikoudai.com/chinese-fonts/other/Songti-SC-Light.html)。放到/Library/Fonts里面就好了。

   参考了这个[问题](https://github.com/ZJU-Awesome/write_with_LaTeX/issues/1)
