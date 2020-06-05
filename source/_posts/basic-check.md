---
title: 姓名，身份证校验
date: 2018-01-22 16:17:47
categories: '工具函数'
---
{% codeblock %}
function checkIsChinese (val){
    if (/^[\u4E00-\u9FA5\·\u00B7]+$/.test(val)) {
      	return false;
    }
    return true;
 }
{% endcodeblock %}

<!-- more -->

{% codeblock %}
function checkIsID (val){
    var v = val;

    var arrExp = [7, 9, 10, 5, 8, 4, 2, 1, 6, 3, 7, 9, 10, 5, 8, 4, 2];
    var arrValid = [1, 0, "X", 9, 8, 7, 6, 5, 4, 3, 2]; //校验码
    var y, m, d;

    if (/^\d{15}$/.test(v)) {
      	y = "19" + v.substr(6, 2);
      	m = v.substr(8, 2);
      	d = v.substr(10, 2);
      	if (isValidDate(y, m, d)) {
        	return false;
      	} else {
        	return true;
      	}
    } else if (/^\d{17}\d|x$/i.test(v)) {
      	var sum = 0,
        	vBit;
      	for (var i = 0; i < v.length - 1; i++) {
        	sum += parseInt(v.substr(i, 1), 10) * arrExp[i];
      	}
      	vBit = sum % 11;
      	y = v.substr(6, 4);
      	m = v.substr(10, 2);
      	d = v.substr(12, 2);
      	if (arrValid[vBit] == v.substr(17, 1).toUpperCase() && isValidDate(y, m, d)) {
        	return false;
      	} else {
        	return true;
      	}
    	} else {
      	return true;
    	}
},
function isValidDate (y, m, d) {
    var _y, _m, _d, now;
    now = new Date(y, m - 1, d);
    _y = now.getFullYear();
    _m = now.getMonth() + 1;
    _d = now.getDate();
    return(y == _y && m == _m && d == _d);
},
{% endcodeblock %}