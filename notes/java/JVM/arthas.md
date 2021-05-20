- 安装 arthas: `curl -O https://arthas.aliyun.com/arthas-boot.jar`
- 启动 arthas：`java -jar arthas-boot.jar`
- 再次进入 arthas: `telnet 127.0.0.1 3658`
- 退出并关闭 arthas: `stop`

通过面板 `dashboard` 查看某个线程占用 CPU 情况，使用 thread tid 看栈

``` 

[arthas@6]$ thread 850
"appid_1.appid_2.appid_3.CoreSchedulerJob10.80.77.41_NoticeJob_10_0_7_66-saturnQuartz-worker" Id=850 TIMED_WAITING on java.lang.Object@7ae41283
    at java.lang.Object.wait(Native Method)
    -  waiting on java.lang.Object@7ae41283
    at com.vip.saturn.job.trigger.SaturnWorker.run(SaturnWorker.java:116)
    at java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:511)
    at java.util.concurrent.FutureTask.run(FutureTask.java:266)
    at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
    at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
    at java.lang.Thread.run(Thread.java:748)

[arthas@6]$

```

- 查看线上运行代码

``` 
[arthas@6]$ jad com.company.appid.service.impl.className
不需要加 .class 或 java 后缀


查询类中方法：
[arthas@6]$ jad com.company.appid.service.impl.className genRepurchaseDTO

ClassLoader:
+-org.springframework.boot.loader.LaunchedURLClassLoader@1936f0f5
  +-sun.misc.Launcher$AppClassLoader@18b4aac2
    +-sun.misc.Launcher$ExtClassLoader@6d4b1c02

Location:
file:/home/server/appid/bin/appid.jar!/BOOT-INF/lib/channel-* project.jar!/

        private ChannelRepayDTO genRepurchaseDTO(String row, Long batchNo, Integer reconRound) {
/*352*/     JSONObject jsonObject = JSONObject.parseObject((String)row);
/*353*/     log.info("??????????????????reperchaseDTO:{}", (Object)jsonObject.toJSONString());
            ChannelRepayDTO channelRepayDTO = new ChannelRepayDTO();
/*356*/     channelRepayDTO.setChannelId(Integer.valueOf(38));
/*357*/     channelRepayDTO.setBatchNo(batchNo);
/*358*/     channelRepayDTO.setReconRound(reconRound);

```