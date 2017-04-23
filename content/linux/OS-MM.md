---
title: "操作系统内存管理"
date: 2017-03-14 17:19
---

# 内存管理方法

 - 连续分配存储管理（程序代码完整连接在一起，物理地址分配）
   - 分区存储管理
     * 固定分区
     * 动态分区
       - 最先适配算法：从头开始，找到一个够的就分配一次
       - 下次适配算法：从上次分配的地方开始找，找到一个够的分配一次
       - 最佳适配算法：从头开始，找到大小最接近的才分配
       - 最坏适配算法：从头开始，找到最大的空闲分区分配
   - 伙伴系统
  - 非连续分配管理（程序代码分散开来，逻辑地址分配）
    - 分段
    - 分页
    - 段页