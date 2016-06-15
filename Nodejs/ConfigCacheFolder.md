#一.node.js的安装  
    1.到官网www.nodejs.org下载windows下.msi的安装包双击运行安装  
    2.安装完以后进行全局安装目录设置:  
        2.1.在安装v0.10/32版本时,如果不是采用默认安装方式,这个时候自己新建安装目录,然后把安装包放在自建的目录下安装
        2.2.在C:\Users\Administrator\AppData\Roaming\ 在这个目录建立npm文件夹
        2.3.比如我们要按2.4修改全局安装及缓存路径,我们需要手动建立文件夹,如:D:\nodejs\新建node_global文件夹,之后按2.4操作
        2.4  `npm config set prefix "D:\nodejs\node_global"`  //设置全局安装的路径,安装的时候会自动创建该目录  
        2.5  `npm config set cache "D:\nodejs\node_cache"`    //设置缓存目录,离线的时候会从该目录读取数据  
        2.6  替换环境变量中系统默认的的node.js的全局路径`C:XXXX为D:\nodejs\node_global`加到系统PATH里面，方便在控制台直接运行使用！  
#二.express的全局安装  
    1.cmd打开控制台,执行  
        npm install -g express  
    2.express4.x已经把命令行工具分离出来了,所有要使用express命令,还需要安装一个模块
        npm install -g express-generator  //用的express4.x的命令
	或
	npm install -g express-generator@3  //用的express3.x的命令
	建议用前者
    3.安装完以后可以到nodejs-->node_global-->node_modules下面看到express和express-genetator了
#三.检验node.js和express是否安装成功  
    1.校验node.js是否安装成功  
        执行`node -v` ,如果有版本号输出,说明安装成功,反之则不成功  
    2.校验express是否安装成功  
        执行`express -V` **注意**是大写的`V`,如果有版本号输出,则说明安装成功,反之则不成功  
  
  
  
有时安装包的时候会出现问题(一般不会有啥问题),如果确实有问题,这时候可以考虑换清华大学的源,方法如下:  
     执行:npm config set registry http://npm.tuna.tsinghua.edu.cn/registry  
     这里可以参考tian0cai的博客:http://blog.sina.com.cn/s/blog_77a8ce670101cdd3.html 
