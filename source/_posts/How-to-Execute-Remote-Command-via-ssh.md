---
title: How to Execute Remote Command via ssh
date: 2017-10-23 16:30:06
tags: ta217
---
<center>以 ssh 遠端執行指令</center>
===
<br>

其實就是很無聊的備忘，有時候需要一次輸出遠端機器的系統訊息到本地的檔案來作報告，像是記憶體用量、CPU model 之類的，又不想要一台台 ssh 上去又複製貼上，這時候就需要來個小 script，例如今天需要列出工作站上所有的 CPU 數和 CPU model

1. `$ vim /tmp/getCPUinfo.sh`
```
    #!/bin/bash
    lscpu | egrep '^CPU\(s\)|^Model name'
```

2. `$ for i in {1..15}; do echo linux$i; ssh linux$i "bash -s" < /tmp/getCPUinfo.sh; done > /tmp/CPUinfolist`

3. `$ head -n 10 /tmp/checkcpulist`
```
    linux1
    CPU(s):              16
    Model name:          Intel(R) Xeon(R) CPU           E5630  @ 2.53GHz
    linux2
    CPU(s):              16
    Model name:          Intel(R) Xeon(R) CPU           E5630  @ 2.53GHz
    linux3
    CPU(s):              16
    Model name:          Intel(R) Xeon(R) CPU           E5530  @ 2.40GHz
    linux4
```

Problem Solved! 不過要先設 ssh key

