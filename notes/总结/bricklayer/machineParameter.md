# 测试环境
- 机器内存
``` 
[user_00@VM-77-41-centos ~]$ free -g
              total        used        free      shared  buff/cache   available
Mem:             62          47           1           0          14          14
Swap:             0           0           0
```
- 磁盘内存
``` 
[user_00@VM-77-41-centos ~]$ df -h
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda1        50G  9.7G   38G  21% /
devtmpfs         32G     0   32G   0% /dev
tmpfs            32G     0   32G   0% /dev/shm
tmpfs            32G   28M   32G   1% /run
tmpfs            32G     0   32G   0% /sys/fs/cgroup
/dev/sdb1       493G  133G  335G  29% /home
tmpfs           6.3G     0  6.3G   0% /run/user/1000
tmpfs           6.3G     0  6.3G   0% /run/user/0
```

# 线上环境
- 机器内存
```
[user_00@VM_121_122_centos ~]$ free -g
              total        used        free      shared  buff/cache   available
Mem:              7           5           0           0           2           2
Swap:             0           0           0
```
``` 
[user_00@VM_121_122_centos ~]$ htop

  1  [|||||||||||||||||||||                                                           24.1%]   Tasks: 53, 2365 thr; 1 running
  2  [||||||||||||||||||||||                                                          24.5%]   Load average: 0.25 0.45 1.12
  3  [||||||||||||||||||||                                                            22.5%]   Uptime: 311 days(!), 19:48:58
  4  [|||||||||||||||||||||                                                           23.2%]
  Mem[||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||5.13G/7.64G]
  Swp[                                                                                0K/0K]

  PID USER      PRI  NI  VIRT   RES   SHR S CPU% MEM%   TIME+  Command
22803 user_00    20   0 6701M 1730M  7216 S 20.6 22.1 10h17:25 java -Denv=PRO -server -Xms1024m -Xmx1024m -Xmn192m -jar bin/work.appid.api_center.jar --spring.profiles.active=pro
22119 user_00    20   0 1414M  183M  5980 S  3.3  2.3     122h bin/log_agent -strict.perms=false -c conf/log_agent_prod.yml
23004 user_00    20   0 6701M 1730M  7216 S  0.7 22.1 51:02.74 java -Denv=PRO -server -Xms1024m -Xmx1024m -Xmn192m -jar bin/work.appid.api_center.jar --spring.profiles.active=pro
22987 user_00    20   0 6701M 1730M  7216 S  1.4 22.1 48:48.08 java -Denv=PRO -server -Xms1024m -Xmx1024m -Xmn192m -jar bin/work.appid.api_center.jar --spring.profiles.active=pro
20778 root        2 -18 1860M 95936 58084 S 13.0  1.2     306h /home/server/service_mesh_master/bin/service_mesh_master
23012 user_00    20   0 4162M  889M  6924 S  6.9 11.4 10h29:32 java -Denv=PRO -server -Xms512m -Xmx512m -Xmn192m -jar bin/work.appid.api.jar --spring.profiles.active=pro
12196 user_00    20   0  119M  4304  1472 R  2.2  0.1  0:04.91 htop
21581 user_00    20   0 1523M 32748  5452 S  2.5  0.4     192h /home/server/service_mesh_slave/bin/service_mesh_slave
20786 root        2 -18 1860M 95936 58084 S  2.5  1.2     145h /home/server/service_mesh_master/bin/service_mesh_master
23610 user_00    20   0 6701M 1730M  7216 S  0.0 22.1  6:36.98 java -Denv=PRO -server -Xms1024m -Xmx1024m -Xmn192m -jar bin/work.appid.api_center.jar --spring.profiles.active=pro
22996 user_00    20   0 6701M 1730M  7216 S  1.8 22.1 27:04.36 java -Denv=PRO -server -Xms1024m -Xmx1024m -Xmn192m -jar bin/work.appid.api_center.jar --spring.profiles.active=pro
23733 user_00    20   0 1414M  183M  5980 S  1.4  2.3 17h51:18 bin/log_agent -strict.perms=false -c conf/log_agent_prod.yml
21585 user_00    20   0 1523M 32748  5452 S  1.8  0.4     126h /home/server/service_mesh_slave/bin/service_mesh_slave
16090 user_00    20   0  179M 55440  3248 S  1.8  0.7     142h bin/m_agent -c conf/agent.json
```
#### 
查询透传接口request
150 - 170w / day 上游透传查询
早上九点到十点半是流量的波峰：1.5k/30s
下午两点半也是流量的波峰：2k/30s
53个接口
总调用量 700w - 1kw

### 吞吐量
   - 54 mesh_20：
       - （早上11点到下午两点三十，业务高峰在早上十点，下午五点左右）运行时长200分钟，GCT 29.226s ,YGCT = 26.742 吞吐量：`（200 * 60 - 29.226）/ (200 * 60) = 99.75%`
``` 

[user_00@VM_0_54_centos /home/log/credit.afp_fund_hub.api_center]$ ps aux |grep credit.afp_fund_hub.api_center
user_00   8325 10.9 18.8 6505372 1507692 ?     Sl   11:11  22:58 java -Denv=PRO -Xms1024m -Xmx1024m -Xmn256m -server -XX:+PrintGCApplicationStoppedTime -XX:+PrintGCDateStamps -verbosegc -XX:+PrintGCDetails -Xloggc:../log/gc.log -XX:+UseGCLogFileRotation -XX:NumberOfGCLogFiles=10 -XX:GCLogFileSize=50M -jar bin/credit.afp_fund_hub.api_center.jar --spring.profiles.active=pro
user_00  12458  0.0  0.0 112708  1012 pts/0    R+   14:41   0:00 grep --color=auto credit.afp_fund_hub.api_center
[user_00@VM_0_54_centos /home/log/credit.afp_fund_hub.api_center]$ jstat -gc 8325 5000
 S0C    S1C    S0U    S1U      EC       EU        OC         OU       MC     MU    CCSC   CCSU   YGC     YGCT    FGC    FGCT     GCT
13312.0 13312.0  0.0   11925.2 235520.0 133850.0  786432.0   498445.5  165056.0 151785.3 18432.0 16421.7   1239   18.248   5      1.364   19.612
13312.0 13312.0  0.0   11925.2 235520.0 230964.9  786432.0   498445.5  165056.0 151785.3 18432.0 16421.7   1239   18.248   5      1.364   19.612

```

   - 3 mesh_0：早上十一点十一分到下午两点四十三分，运行时长210分钟， `（210 * 60 - 19.6) / (210 * 60) =  99.84%`
```
[user_00@VM_0_3_centos /home/log/credit.afp_fund_hub.api_center]$ jstat -gc 12323 5000
 S0C    S1C    S0U    S1U      EC       EU        OC         OU       MC     MU    CCSC   CCSU   YGC     YGCT    FGC    FGCT     GCT
16896.0 16896.0 14791.2  0.0   228352.0 200825.1  786432.0   702891.3  160704.0 148539.0 17664.0 15752.9   1212   26.972   6      2.484   29.456
16896.0 16896.0  0.0   15530.5 228352.0 82648.4   786432.0   703728.9  160704.0 148539.0 17664.0 15752.9   1213   26.989   6      2.484   29.474
[user_00@VM_0_3_centos /home/log/credit.afp_fund_hub.api_center]$ ps -ef |grep credit.afp_fund_hub.api_center
user_00  12323     1 10 11:11 ?        00:20:23 java -Denv=PRO -Xms1024m -Xmx1024m -Xmn256m -server -XX:+PrintGCApplicationStoppedTime -XX:+PrintGCDateStamps -verbosegc -XX:+PrintGCDetails -Xloggc:../log/gc.log -XX:+UseGCLogFileRotation -XX:NumberOfGCLogFiles=10 -XX:GCLogFileSize=50M -jar bin/credit.afp_fund_hub.api_center.jar --spring.profiles.active=pro
user_00  21678 19186  0 14:28 pts/0    00:00:00 grep --color=auto credit.afp_fund_hub.api_center
``` 
- 186  mesh_100
- 运行时长 3 * 60 + 50 = 230 `（230 * 60 - 70) / (230 * 60) =  99.5%`

``` 
[user_00@VM_0_6_centos ~]$ jstat -gc 12207 5s
 S0C    S1C    S0U    S1U      EC       EU        OC         OU       MC     MU    CCSC   CCSU   YGC     YGCT    FGC    FGCT     GCT
12800.0 11776.0  0.0   11692.9 237568.0 128910.1  786432.0   752471.1  157376.0 146318.4 17408.0 15680.9   4373   66.881   9      3.719   70.600
12288.0 12800.0  0.0   10754.4 236544.0 33038.2   786432.0   753446.0  157376.0 146320.9 17408.0 15680.9   4375   66.909   9      3.719   70.629
12800.0 12800.0  0.0   11105.9 236544.0 92238.2   786432.0   754368.2  157376.0 146320.9 17408.0 15680.9   4377   66.970   9      3.719   70.689
12288.0 12288.0  0.0   10333.8 237056.0 86103.6   786432.0   756490.0  157376.0 146320.9 17408.0 15680.9   4379   67.002   9      3.719   70.721
12288.0 12288.0 10431.2  0.0   237568.0 202970.8  786432.0   757170.8  157376.0 146320.9 17408.0 15680.9   4380   67.016   9      3.719   70.735
^C[user_00@VM_0_6_centos ~]$ ps -aux |grep 12207
user_00   5108  0.0  0.0 112812   972 pts/0    S+   14:56   0:00 grep --color=auto 12207
user_00  12207 29.2 21.0 7072564 1689764 ?     Sl   11:07  66:50 java -Denv=PRO -Xms1024m -Xmx1024m -Xmn256m -server -XX:+PrintGCApplicationStoppedTime -XX:+PrintGCDateStamps -verbosegc -XX:+PrintGCDetails -Xloggc:../log/gc.log -XX:+UseGCLogFileRotation -XX:NumberOfGCLogFiles=10 -XX:GCLogFileSize=50M -jar bin/credit.afp_fund_hub.api_center.jar --spring.profiles.active=pro
```

### 机器CPU
- prod 一个CPU （physical id）四核（cpu cores）四线程（processor）
删除部分多余信息，最后一个保留
```
[user_00@VM_0_3_centos ~]$ cat /proc/cpuinfo
processor	: 0
vendor_id	: GenuineIntel
cpu family	: 6
model		: 94
model name	: Intel(R) Xeon(R) Gold 61xx CPU
stepping	: 3
microcode	: 0x1
cpu MHz		: 2394.364
cache size	: 4096 KB
physical id	: 0
siblings	: 4
core id		: 0
cpu cores	: 4

processor	: 1
vendor_id	: GenuineIntel
cpu family	: 6
model		: 94
model name	: Intel(R) Xeon(R) Gold 61xx CPU
stepping	: 3
microcode	: 0x1
cpu MHz		: 2394.364
cache size	: 4096 KB
physical id	: 0
siblings	: 4
core id		: 1
cpu cores	: 4

processor	: 2
vendor_id	: GenuineIntel
cpu family	: 6
model		: 94
model name	: Intel(R) Xeon(R) Gold 61xx CPU
stepping	: 3
microcode	: 0x1
cpu MHz		: 2394.364
cache size	: 4096 KB
physical id	: 0
siblings	: 4
core id		: 2
cpu cores	: 4

processor	: 3
vendor_id	: GenuineIntel
cpu family	: 6
model		: 94
model name	: Intel(R) Xeon(R) Gold 61xx CPU
stepping	: 3
microcode	: 0x1
cpu MHz		: 2394.364
cache size	: 4096 KB
physical id	: 0
siblings	: 4
core id		: 3
cpu cores	: 4
apicid		: 3
initial apicid	: 3
fpu		: yes
fpu_exception	: yes
cpuid level	: 13
wp		: yes
flags		: fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush mmx fxsr sse sse2 ss ht syscall nx pdpe1gb rdtscp lm constant_tsc rep_good nopl eagerfpu pni pclmulqdq ssse3 fma cx16 pcid sse4_1 sse4_2 x2apic movbe popcnt tsc_deadline_timer aes xsave avx f16c rdrand hypervisor lahf_lm abm 3dnowprefetch fsgsbase bmi1 hle avx2 smep bmi2 erms invpcid rtm mpx avx512f avx512dq rdseed adx smap avx512cd avx512bw avx512vl xsaveopt xsavec xgetbv1 arat
bogomips	: 4788.72
clflush size	: 64
cache_alignment	: 64
address sizes	: 46 bits physical, 48 bits virtual
power management:


```
- fat 八核 八线程
```
[user_00@VM-77-37-centos /home/log/ams.asset_manager.asset_distribute]$ cat /proc/cpuinfo
processor	: 0
vendor_id	: GenuineIntel
cpu family	: 6
model		: 85
model name	: Intel(R) Xeon(R) Gold 5218 CPU @ 2.30GHz
stepping	: 7
microcode	: 0x5002f00
cpu MHz		: 2294.609
cache size	: 22528 KB
physical id	: 0
siblings	: 4
core id		: 0
cpu cores	: 4
apicid		: 0

processor	: 1
vendor_id	: GenuineIntel
cpu family	: 6
model		: 85
model name	: Intel(R) Xeon(R) Gold 5218 CPU @ 2.30GHz
stepping	: 7
microcode	: 0x5002f00
cpu MHz		: 2294.609
cache size	: 22528 KB
physical id	: 0
siblings	: 4
core id		: 1
cpu cores	: 4
apicid		: 1

processor	: 2
vendor_id	: GenuineIntel
cpu family	: 6
model		: 85
model name	: Intel(R) Xeon(R) Gold 5218 CPU @ 2.30GHz
stepping	: 7
microcode	: 0x5002f00
cpu MHz		: 2294.609
cache size	: 22528 KB
physical id	: 0
siblings	: 4
core id		: 2
cpu cores	: 4
apicid		: 2

processor	: 3
vendor_id	: GenuineIntel
cpu family	: 6
model		: 85
model name	: Intel(R) Xeon(R) Gold 5218 CPU @ 2.30GHz
stepping	: 7
microcode	: 0x5002f00
cpu MHz		: 2294.609
cache size	: 22528 KB
physical id	: 0
siblings	: 4
core id		: 3
cpu cores	: 4
apicid		: 3

processor	: 4
vendor_id	: GenuineIntel
cpu family	: 6
model		: 85
model name	: Intel(R) Xeon(R) Gold 5218 CPU @ 2.30GHz
stepping	: 7
microcode	: 0x5002f00
cpu MHz		: 2294.609
cache size	: 22528 KB
physical id	: 1
siblings	: 4
core id		: 0
cpu cores	: 4
apicid		: 4

processor	: 5
vendor_id	: GenuineIntel
cpu family	: 6
model		: 85
model name	: Intel(R) Xeon(R) Gold 5218 CPU @ 2.30GHz
stepping	: 7
microcode	: 0x5002f00
cpu MHz		: 2294.609
cache size	: 22528 KB
physical id	: 1
siblings	: 4
core id		: 1
cpu cores	: 4
apicid		: 5

processor	: 6
vendor_id	: GenuineIntel
cpu family	: 6
model		: 85
model name	: Intel(R) Xeon(R) Gold 5218 CPU @ 2.30GHz
stepping	: 7
microcode	: 0x5002f00
cpu MHz		: 2294.609
cache size	: 22528 KB
physical id	: 1
siblings	: 4
core id		: 2
cpu cores	: 4
apicid		: 6

processor	: 7
vendor_id	: GenuineIntel
cpu family	: 6
model		: 85
model name	: Intel(R) Xeon(R) Gold 5218 CPU @ 2.30GHz
stepping	: 7
microcode	: 0x5002f00
cpu MHz		: 2294.609
cache size	: 22528 KB
physical id	: 1
siblings	: 4
core id		: 3
cpu cores	: 4
apicid		: 7
initial apicid	: 7
fpu		: yes
fpu_exception	: yes
cpuid level	: 22
wp		: yes
flags		: fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush mmx fxsr sse sse2 ss ht syscall nx pdpe1gb rdtscp lm constant_tsc arch_perfmon nopl xtopology tsc_reliable nonstop_tsc eagerfpu pni pclmulqdq ssse3 fma cx16 pcid sse4_1 sse4_2 x2apic movbe popcnt tsc_deadline_timer aes xsave avx f16c rdrand hypervisor lahf_lm abm 3dnowprefetch ssbd ibrs ibpb stibp ibrs_enhanced fsgsbase tsc_adjust bmi1 avx2 smep bmi2 invpcid avx512f avx512dq rdseed adx smap clflushopt clwb avx512cd avx512bw avx512vl xsaveopt xsavec xgetbv1 arat pku ospke avx512_vnni spec_ctrl intel_stibp flush_l1d arch_capabilities
bogomips	: 4589.21
clflush size	: 64
cache_alignment	: 64
address sizes	: 45 bits physical, 48 bits virtual
power management:

```