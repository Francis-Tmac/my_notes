### idea 升级版本后因文件过大无法编译
- 配置 gradle.properties 文件
``` 
org.gradle.jvmargs=-Xmx4g -XX:MaxMetaspaceSize=2048m -XX:+HeapDumpOnOutOfMemoryError -Dfile.encoding=UTF-8
```

- 修改idea.properties 文件
``` 
路径：/Applications/IntelliJ IDEA.app/Contents/bin/idea.properties 

idea.max.intellisense.filesize=150000
idea.max.content.load.filesize=100000
```

