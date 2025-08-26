---
title: xrun dump fsdb
date: 2025-08-25 16:24:09
categories:
- ip开发环境搭建
- EDA工具使用
tags:
---



```bash
# step1:设置环境变量
# # setenv LD_LIBRARY_PATH ${LD_LIBRARY_PATH}:${VERDI_HOME}/share/PLI/lib/LINUX64:${VERDI_HOME}/share/PLI/LINUX64:${VERDI_HOME}/share/PLI/IUS/LINUX64:${VERDI_HOME}/share/PLI/VCS/LINUX64:${VERDI_HOME}/share/PLI/IUS/LINUX64/boot
# # setenv LD_LIBRARY_PATH ${LD_LIBRARY_PATH}:${VERDI_HOME}/share/PLI/VCS/LINUX64:${VERDI_HOME}/share/PLI/IUS/LINUX64
setenv LD_LIBRARY_PATH "VERDI_HOME/share/PLI/IUS/LINUX64"

# step2;
xun option: -loadpli1 debpli:novas_pli_boot

# step2:使用dump fsdb波形的task
         initial
         begin
           $fsdbAutoSwitchDumpfile(500,"top_sim.fsdb",100);
           $fsdbDumpvars(0, top_sim);
         end

```

