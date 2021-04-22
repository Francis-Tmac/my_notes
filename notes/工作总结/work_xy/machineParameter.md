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
request 
140w / day 上游透传查询


