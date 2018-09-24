# IDEA基于SpringBoot的远程调试

​	远程调试就是服务端程序运行在一台远程服务器上，我们可以在本地服务端的代码（前提是本地的代码必须和远程服务器运行的代码一致）中设置断点，每当有请求到远程服务器时时能够在本地知道远程服务端的此时的内部状态。

## 方法：

（1） 打开Edit configurations --> 点击+号 --> 选择Remote，创建一个Remote应用。

填写name，配置Host地址（远程服务器地址）和端口（选一个未被占用的端口）。然后复制For JDK1.4.x下面的参数，示例配置的端口为5005：

```
-Xdebug -Xrunjdwp:transport=dt_socket,server=y,suspend=n,address=50051
```

（2）Maven编译打包生成jar包，将jar包上传到业务机上，然后 在业务机上执行如下命令启动

```
$ java -jar -Xdebug -Xrunjdwp:transport=dt_socket,server=y,suspend=n,address=5005 xxx.jar
```

（3）启动刚才配置的Remote服务，启动完成，对需要debug的代码打上断点，剩下的操作步骤就是访问远程服务器对应的业务请求，本地就会同步debug。其余的操作与本地debug相同，此处就不再赘述了。