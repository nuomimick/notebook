# 不要使用Files.readAllBytes一次性读取，除非你都是小文件

直接内存的访问：直接内存 => 系统调用 => 硬盘/网卡
非直接内存的访问需要二次拷贝：堆内存 => 直接内存 => 系统调用 => 硬盘/网卡

nio底层使用了java.nio.ByteBuffer.allocateDirect

推荐都使用流式下载

可以用工具类
```
# spring
StreamUtils.copy(in, out);

# nio
Files.copy(file.toPath(), out);
```
