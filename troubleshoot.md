# 问题解决

##  msgmerge binary
ports安装kde4-l10n-zh_CN时报错CMake Error: Please install the msgmerge binary

安装语言包工具集`gettext`, msgmerge是里面的一个语言包合并程序.

## 'SYS_gettid' undeclared
编译hadoop-2.6.0报错Compilation errors: 'SYS_gettid' undeclared

修改文件`hadoop-common-project/hadoop-common/src/main/native/src/org/apache/hadoop/crypto/random/OpensslSecureRandom.c`
在文件include部分增加
```c
#if defined(__FreeBSD__)
#include <pthread_np.h>
#endif
/*修改函数 pthreads_thread_id*/
static unsigned long pthreads_thread_id(void)
{
  unsigned long thread_id = 0;
#if defined(__linux__)
  thread_id = (unsigned long)syscall(SYS_gettid);
#elif defined(__FreeBSD__)
  thread_id = (unsigned long)pthread_getthreadid_np();
#elif defined(__APPLE__)
  (void)pthread_threadid_np(pthread_self(), &thread_id);
#endif  return thread_id;
}
```

## ipython
ipython QtConsole报错找不到动态库

启动脚本修改为
```
setenv LD_LIBRARY_PATH /usr/local/lib/gcc48/
ipython qtconsole --ConsoleWidget.font_size=12
```

## gitbook无法创建pdf
`gitbook pdf`报错
```
info: start conversion to pdf ....ERROR

Error: Need to install ebook-convert from Calibre
```

安装calibre  `pkg install calibre`

