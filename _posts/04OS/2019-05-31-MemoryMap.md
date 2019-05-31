---
layout: post
title: 内存区域划分规范
category: OS
tags: MemoryMap
keywords: first 512 byte of boot disk
description: 
---
## 1. MemoryMap
1. 这和常用的数据结构Map没有什么关系;
2. 这是内存使用分配的规范,操作系统需要处理

## 2. 分配表(First 1Mb Memory 分配规范)
<table border="2" cellpadding="4" cellspacing="0" style="margin-top:1em; margin-bottom:1em; background:#f9f9f9; border:1px #aaa solid; border-collapse:collapse; {{{1}}}">
   <tbody>
      <tr>
         <th> 起始地址</th>
         <th> 终止地址</th>
         <th> 大小</th>
         <th> 类型</th>
         <th> 解释</th>
      </tr>
      <tr>
         <td> 0x00000000</td>
         <td> 0x000003FF</td>
         <td> 1 KiB</td>
         <td> RAM，部分不可使用</td>
         <td> Real Mode IVT (Interrupt Vector Table),实模式的中断向量表,中断向量表主要存储对于不同中断类型的处理程序入口地址</td>
      </tr>
      <tr>
         <td> 0x00000400</td>
         <td> 0x000004FF</td>
         <td> 256 bytes</td>
         <td> RAM，部分不可使用</td>
         <td> BDA (BIOS data area),BIOS存储数据的区域</td>
      </tr>
      <tr>
         <td> 0x00000500</td>
         <td> 0x00007BFF</td>
         <td> 大约 30 KiB</td>
         <td> RAM,自由使用</td>
         <td> 备用区域，BIOS的</td>
      </tr>
      <tr>
         <td> 0x00007C00 (典型地址)</td>
         <td> 0x00007DFF</td>
         <td> 512 bytes</td>
         <td> RAM ,部分不可使用</td>
         <td> MBR 存储区域,有BIOS加载到内存这里,后然后jump到0X7C00开始执行</td>
      </tr>
      <tr>
         <td> 0x00007E00</td>
         <td> 0x0007FFFF</td>
         <td> 480.5 KiB</td>
         <td> RAM,自由使用</td>
         <td> MBR备用区域</td>
      </tr>
      <tr>
         <td> 0x00080000</td>
         <td> 0x0009FFFF</td>
         <td> 128 KiB</td>
         <td> RAM,部分不可使用</td>
         <td> EBDA (Extended BIOS Data Area),BIOS Data Area 扩展区域</td>
      </tr>
      <tr>
         <td> 0x000A0000</td>
         <td> 0x000FFFFF</td>
         <td> 384 KiB</td>
         <td> 杂项,不可使用</td>
         <td> 视频存储区,ROM (主要是微指令集)</td>
      </tr>
   </tbody>
</table>