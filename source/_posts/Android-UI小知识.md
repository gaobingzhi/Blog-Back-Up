---
title: Android UI小知识
date: 2018-04-22 22:44:41
tags: [Android]
reward: true
---
#### 常用控件

###### 1.ProgressBar 样式问题

```
 <ProgressBar
        android:id="@+id/progress_bar"
        android:layout_width="match_parent"
        android:layout_height="wrap_content" 
        style="?android:progressBarStyleHorizontal"
        android:max="100"/>
```
`style="?android:progressBarStyleHorizontal"`为系统样式，设置横向进度样式。

<!--more-->
#### 常用布局
###### 1.百分比布局