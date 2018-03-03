---
title: JavaScript给DOM元素相同事件绑定多函数
date: 2013-08-30 06:32:31
tags:
mp3: http://link.hhtjim.com/163/27646205.mp3
cover: http://odwjyz4z6.bkt.clouddn.com/2a6a93c00e6.jpg
---
### 给DOM元素的同一事件绑定多个处理函数
今天，同事问“一个DOM元素的同一事件如何绑定多个方法”，为什么这么问，因为在javascript中有一个规律，如果某元素的某事件多次赋值，那么只有最后一次赋值生效，如下代码将只弹出”second.”。
```
document.body.onclick=function(){
	alert("first.");
};
document.body.onclick=function(){
	alert("second.");
};
```
说起这个问题，我首先想到的是jQuery,它的 `bind(type,[data],fn)` 方法可以实现同一事件函数叠加的效果，那么， js如何实现呢？如下：
```
/**
 * 给DOM元素的同一事件绑定多个方法
 * 
 * @param {Object} obj	DOM对象
 * @param {Object} type 事件类型
 * @param {Object} fn	处理函数
 */
function addEventFns(_obj,_type,_fn){
	var oldEvent = _obj[_type];
	if(typeof _obj[_type] == "function"){
		_obj[_type] = function(){
			oldEvent();
			_fn();
		}
	}else{
		_obj[_type] = _fn;
	}
}
```
在Dom节点加载完毕之后这样调用 `addEventFns(_obj,_type,_fn)`
- _obj:需要添加事件的DOM对象，比如: `document.getElementById(“container”)` 或 `document.body`;
- _type:事件类型的字符串，比如：`onclick` 或 `onmouseover`;
- _fn:绑定到DOM元素的事件上面的处理函数。
```
addEventFns(obj,"onclick",function(){
		alert("Fired the event.");
});
```

### 项目中用到的一些工具函数

公司xx项目就失业子系统变更coding告一段落，收获颇多，主要涉及校验，和业务的理解，可谓是循序渐进，环环相扣。coding过程走火入魔算不上，但出入分明。总结几个常用javascript校验，菜鸟级别，仅供参考！
```
/**
 * 获取两个时间相差月数
 *
 * @param {Object} start 开始时间(yyyy-MM)
 * @param {Object} end   结束时间(yyyy-MM)
 * @return {TypeName}
 */
function getDiffOfMonth(start,end){
	var diff;
	if(start==""||end==""){
		diff = "";
	}
	var startArr = start.split('-');
	var endArr = end.split('-');
	if(startArr[0]==endArr[0]){
		diff = endArr[1]-startArr[1];
	}else{
		diff = (endArr[0]-startArr[0])*12+(endArr[1]-startArr[1]);
	}
	return Math.abs(diff)+1;
}

/**
 * 根据月份区间获取所有月份集合
 *
 * @param {Object} start 开始时间(yyyy-MM)
 * @param {Object} end   结束时间(yyyy-MM)
 * @return {TypeName} 
 */
function getAllMonth(start, end) {
	var months = new Array(start);
	var month = "";
	var length = getDiffOfMonth(start, end);
	var start = start.split('-');
	var end = end.split('-');
	var temp;
	for ( var i = 0; i < length - 1; i++) {
		temp = new Date(start[0], parseInt(start[1]) + i);
		month = new String(temp.getMonth() + 1).length == 1 ? "0"
				+ (temp.getMonth() + 1) : (temp.getMonth() + 1);
		months.push(temp.getFullYear() + "-" + month);
	}
	return months;
}

/**
 * 获取两个时间相差天数
 *
 * @param {Object} start 开始时间(yyyy-MM-dd)
 * @param {Object} end   结束时间(yyyy-MM-dd)
 * @return {TypeName}
 */
function getDiffOfDay(start,end){
	var diff;
	if(start==""||end==""){
		diff = "";
	}
	diff = Math.abs(new Date(start)-new Date(end));
	return diff/(1000*3600*24)
}

/**
 * 一次性领取失业保险金明细记录唯一性校验
 * 判重思路：循环数组各元素，如果在该数组出现两次则重复
 * 关键方法：jQuery.inArray(value,array,[fromIndex])
 *
 * @return {TypeName} 
 */
function ycxmxUnique() {
	var flag = true;
	var length = $("[name='lx']").last().attr("id"); // 最后一条明细的id
	var start = "";
	var end = "";
	var tempArr = new Array();

	// 封装数组
	for ( var i = 1; i <= length; i++) {
		if (typeof ($("[name='ksny'][id=" + i + "]").val()) == "undefined") {
			continue;
		}
		start = $("[name='jsny'][id=" + i + "]").val();
		end = $("[name='ksny'][id=" + i + "]").val();
		tempArr = tempArr.concat(getAllMonth(start, end));
	}
	tempArr.sort();
	// 判断重复
	for ( var i = 0; i < tempArr.length; i++) {
		if (tempArr[i].length == 7
				&& jQuery.inArray(tempArr[i], tempArr, i + 1) > -1) {
			alert("一次性返回年月有重复，请重新输入！");
			flag = false;
			break;
		}
	}
	return flag;
}
```
