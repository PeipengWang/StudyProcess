
centeos安装RobotFrameWork最后卡在这一步
  Running setup.py install for wxPython ... error
    ERROR: Command errored out with exit status 1:
主要原因是安装wxPython的时候失败了
向上翻错误日志
 error: [Errno 2] No such file or directory: 'build/lib.linux-x86_64-3.6/wx/libwx_baseu-3.0.so'
  ERROR: Failed building wheel for wxPython
这个问题在https://github.com/wxWidgets/Phoenix/issues/1769这里有详细解答（注意向下翻翻）
原因是setuptool版本太高了，我用的是54.1.1版本，需要将版本回退到41.0.1
查看setuptools 版本
```bash
pip3 show setuptools 
```
Name: setuptools
Version: 54.1.1

卸载版本

```bash
pip3 uninstall setuptools
```
注意，这一步可能失败，需要卸载它依赖的包
例如
    Can't uninstall 'setuptools'. No files were found to uninstall.
ERROR: pip's dependency resolver does not currently take into account all the packages that are installed. This behaviour is the source of the following dependency conflicts.
setuptools-rust 1.1.2 requires setuptools>=46.1, but you have setuptools 41.0.1 which is incompatible.
主要原因是我安装的setuptools-rust 1.1.2导致无法卸载
把这个卸载掉即可安装setuptools了

```bash
pip3 install setuptools==41.0.1
```
然后再安装robotframework-ride 

```bash
pip3 install robotframework-ride
```
pip3 show robotframework-ride
Name: robotframework-ride
Version: 1.7.4.2

最后提一点就是centeos安装robotframework坑点还是不少的，主要安装一些系统组件和py包


