# ProcessBuilder中的字节流
ProcessBuilder是建立一个新进程，并且能够获取环境的信息，这些信息会通过字节流的方式输入获取，接下来讨论获取字节流要进行的操作
## 直接获取
这种方式是直接读取InputStream的字节流
```
ProcessBuilder processBuilder = new ProcessBuilder();
processBuilder.command("ls");
Process process = processBuilder.start();
int  ret = process.waitFor();
InputStream inputStream = process.getInputStream();
StringBuffer out = new StringBuffer();
byte[] b = new byte[8192];
for (int n; (n = inputstream.read(b)) != -1; ) { //直接读取字节流
     out.append(new String(b, 0, n));
}
System.out.printf("Program exited with code: %d", ret);
System.out.println(out);
```
但是这种方式每次进行inputstream.read()的时候都要进行一次IO操作，这样比较消耗资源，因此放入缓存中读取
```
ProcessBuilder processBuilder = new ProcessBuilder();
processBuilder.command("ls");
Process process = processBuilder.start();
int  ret = process.waitFor();
InputStream inputStream = process.getInputStream();
BufferedInputStream bs = new BufferedInputStream(inputStream); //将数据放入缓存
StringBuffer out = new StringBuffer();
byte[] b = new byte[8192];
for (int n; (n = bs.read(b)) != -1; ) { //遍历数据
        out.append(new String(b, 0, n));
}
System.out.printf("Program exited with code: %d", ret);
System.out.println(out);
```