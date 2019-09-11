---
title: DailyBrainStorm
date: 2019-08-27 12:59:43
tags:
---

>  写这篇的目的：好多时候没有更新文章，其实并不是没有在做研究，只不过有些小的知识点，并不值得洋洋洒洒的来一篇，故而开了这个tag，每当自己有什么小的研究和小的idea，能够记录下来


#### 2019年08月27日
iOS tableview的headerView和footerView,
在创建的时候设置的frame，系统会根据宽高来添加约束，因此无法通过添加子视图与父视图之间的约束，来做到“子视图撑大父视图”这样。想要修改tableview的header、footer视图，可以在子视图布局完成之后添加回调，重新设置即可。
`self.tableview.tableHeaderView = self.headerView;`
`self.tableview.tableFooterView = self.footerView;`

#### 2019年09月11日
iOS label 默认设置lineBreakMode为省略号在右边
但是如果使用了NSParagraphStyle,则label设置的lineBreakMode就会失效，使用NSMutableParagraphStyle的lineBreakMode代替即可
