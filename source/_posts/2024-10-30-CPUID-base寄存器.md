---
title: CPUID base寄存器
date: 2024-10-30 16:03:37
categories:
- ARM
- Cortex-M4
tags:
---

{% asset_img image-20241030160532389.png%}

{% asset_img image-20241030160610806.png%}

> The CPUID scheme provides a description of the features of an ARM processor implementation.  
>
>  
>
> Specifying an architecture variant of 0xF in the Main ID Register indicates use of the CPUID scheme. 
>
> In the ARMv7-M profile, the Main ID Register is called the CPUID base register  
>
> 
>
> The ARMv7-A and ARMv7-R profiles permit many implementation options. Therefore, in those profiles, many
> CPUID field values are IMPLEMENTATION DEFINED, and can take a range of values. In the ARMv7-M profile, the
> ARMv7-M base architecture, and any implemented architecture extensions, define the CPUID field values.  
