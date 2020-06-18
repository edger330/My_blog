
# 问题：

不知道有没有朋友遇到过这样的问题：
在网上下载到了一套高品质音质资源，资源内有两个文件，一个.ape，一个.cue(至于这两个文件具体是干嘛的，这里不再赘述)如果我们直接用cue splitter打开.cue对ape进行分轨的话，我们将得到一批.ape文件。但如果我想要将这些.ape转化成.wav，从而可以在foobar2000中播放，在cue splitter中只能一个一个地添加.ape文件进行转化，这样效率很低。

# 解决方法：

我们可以通过修改.cue文件，再使用cue splitter对其进行分轨，这样我们分轨将直接得到.wav文件。

# 具体方法：

## 第一步

先用cue splitter将整个的ape文件转化成wav文件

![将APE转换为WAVE](https://github.com/edger330/My_blog/tree/master/img/5791357-c3b295babeabae89.png?raw=true)


## 第二步

转化完成后，你将会得到wav

![wav及原ape源文件](https://github.com/edger330/My_blog/tree/master/img/5791357-08090c8a64a75321.png?raw=true)

这个时候我们打开.cue文件，用记事本，sublime都可以

![打开的.cue文件的内容](https://github.com/edger330/My_blog/tree/master/img/5791357-038b9691038f1bca.png?raw=true)

我们关注开头是FILE的那一行

![file](https://github.com/edger330/My_blog/tree/master/img/5791357-e6bd09f0e7f85bed.png?raw=true)

不难推测，这是分轨的输入文件，这个时候我们把这个文件的后缀名改为.wav，保存。再用cue splitter打开.cue文件进行分轨，我们就可以得到全为.wav的音乐格式了，也可以在foobar2000上播放了。

![大功告成](https://github.com/edger330/My_blog/tree/master/img/5791357-8200cb06957487d4.png?raw=true)

希望对爱好音乐的朋友有帮助！

