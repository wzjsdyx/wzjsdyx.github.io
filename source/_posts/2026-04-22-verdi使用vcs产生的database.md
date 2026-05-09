---
title: verdi使用vcs产生的database
date: 2026-04-22 10:15:35
categories:
- ip开发环境搭建
- EDA工具使用
tags:
- verdi
- TBD
---

# verdi使用vcs生成的中间文件

1、vcs添加编译选项

```bash
-kdb
# 仿真目录下会生成 ./simv.daidir/kdb.elab++/
```



2、verdi加载

```bash
verdi -elab ./simv.daidir/kdb.elab++/ -ssf kite_execution_tb_000.fsdb &
```

