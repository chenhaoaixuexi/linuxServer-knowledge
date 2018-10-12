# vim 自动补全插件YCM 安装 配置

### 报错

`NoExtraConfDetected: No .ycm_extra_conf.py file detected, so no compile flags are available. Thus no semantic support for C/C++/ObjC...`

> 说是找不到 .ycm_extra_conf.py, 所以不支持 c/c++的语义补全

### 解决步骤

- 先看看 .vimrc里面的配置

  `let g:ycm_global_ycm_extra_conf='~/.vim/bundle/YouCompleteMe/third_party/ycmd/cpp/ycm/.ycm_extra_conf.py'`

  查看文件是否存在

  果然是文件名出现问题

  以前瞎配置的时候 搞错了

  输入: ` cp .ycm_extra_conf.bck.py  .ycm_extra_conf.py`

  我的配置: 

  ```
  '-isystem',
  '/usr/include',
  '-isystem',
  '/usr/include/c++/5.4.0'
  '-isystem',
  '/usr/include/c++/5.4.0',
  '-isystem',
  '/usr/include',
  '/usr/include/x86_64-linux-gnu/c++',
  ~
  ```

  我的配置是当初自己在网上瞎抄的, 现在推倒重来

  备份, 编辑

  报错:  `E212: Can't open file for writing`

  应该是权限不够: `sudo vi  cpp/ycm/.ycm_extra_conf.py`

  还是报错, 猜测可能是文件被占用, 查看我的窗口们(使用wsl+cmder模拟)

  果然, 是有一个窗口编辑该文件时, 没有退出

  重新 vim, 还不行, 退出窗口重试, ok

  预备知识储备以下:

  	

  - -isystem

        The -isystem and -idirafter options also mark the directory as a system directory, so that it gets the same special treatment that is applied to the standard system directories.

    意思是把某个目录标识Wie系统目录, 所以某代码可以直接找到对应的头文件啦

  ```
   flags = [
   '-Wall',
   '-Wextra',
   '-Werror', 
   '-Wno-long-long', 
   '-Wno-variadic-macros', 
   '-fexceptions', 
   '-DNDEBUG', 
   '-std=c++11', 
   '-x', 
   'c++', 
   '-I', 
   '/usr/include', 
   '-isystem', 
   '/usr/lib/gcc/x86_64-linux-gnu/5/include', 
   '-isystem', 
   '/usr/include/x86_64-linux-gnu', 
   '-isystem' 
   '/usr/include/c++/5', 
   '-isystem', 
   '/usr/include/c++/5/bits' 
   ]
  
  ```

- 再打开vim 提示: `AttributeError: 'module' object has no attribute 'FlagsForFile'`

  属性错误: "module" 对象 没有属性 FlagsForFile

- 不会了.....

### ......... 卸载, 重安:

卸载方法

- ~ 目录下的所有.vim或者.vimrc文件 删除

重安后就成功了

![1536494297299](assets/1536494297299.png)

