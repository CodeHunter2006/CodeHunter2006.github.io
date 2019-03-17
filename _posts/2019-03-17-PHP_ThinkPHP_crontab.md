---
layout: post
title:  "改造ThinkPHP cron源码，修正执行时间偏移问题"
date:   2019-03-17 10:00:00 +0800
tags: PHP ThinkPHP
---
### 问题场景
在"App/Conf/crons.php"文件中，设定了一个任务：
```
return array(
	...
	//第二天早晨8点开始执行任务xxx，时间间隔为24小时
	'cron_5' => array('xxx',3600 * 24,strtotime(date("Y-m-d",time())) + 3600 * (24 + 8)),
	...
);
```
任务刚开始执行几天还没什么问题，但是随着系统启动时间越久执行任务的时间在逐渐向后漂移。<br/>
比如过了两个月，任务执行时间漂移到了8:10。

<br/>
### 原因分析
通过分析源码，找到了发生问题的地方。原因是每次执行任务的时间不是确定的，某用户或爬虫访问网站时开始执行任务，
而任务的下次执行时间的设定是以当前时间为基础加上间隔，所以就会逐步产生漂移现象。

<br/>
### 解决方案
通过修改"CORE/Extend/Behavior/CronRunBehavior.class.php"的任务时间设定逻辑，<br/>
以设定时间开始计算间隔，寻找符合要求的最近的合法时间进行设定。
```
// 更新cron记录
if(empty($cron[2])) {
  // 未对执行时间作规定
  $cron[2] = $_SERVER['REQUEST_TIME'] + $cron[1];
}else{
  // 已有执行时间设定或特定时间规定
  do{
    $cron[2] = $cron[2] + $cron[1];  // 以特定执行时间+间隔设定下一次
  } while($cron[2] <= $_SERVER['REQUEST_TIME']); // 直到找到下次执行时间
}
```