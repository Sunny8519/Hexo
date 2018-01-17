---
title: AutoCompleteTextView和MultiAutoCompleteTextView的简单用法
date: 2018-01-17
categories: AutoCompleteTextView
author: Sunny
tags:
    - AutoCompleteTextView
cover_picture: http://upload-images.jianshu.io/upload_images/5231076-cca20551d7c949e4.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240
---
![belle_1.jpg](http://upload-images.jianshu.io/upload_images/5231076-cca20551d7c949e4.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 1.AutoCompleteTextView的基本用法
```
//待匹配的字符串
private String[] res = new String[]{"shanghai1", "shanghai2", "beijing1", "beijing2", "tianjin"};
//创建适配器
ArrayAdapter adapter = new ArrayAdapter(this, android.R.layout.simple_list_item_1, res);
/*将适配器绑定到AutoCompleteTextView
* 注意：在xml文件下的<AutoCompleteTextView/>标签下需要添加android:completionThreshold="3"属性值，
* 该属性值用于在文本框中输入到第几个字符开始匹配给定的字符串*/
this.binding.txtAutoComplete.setAdapter(adapter);
<AutoCompleteTextView
    android:id="@+id/txt_auto_complete"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:completionThreshold="3"
/>
```
效果图如下：
<div align = "center">
<img src="http://upload-images.jianshu.io/upload_images/5231076-fc21255a8d7505c8.gif?imageMogr2/auto-orient/strip" style = "zoom:50%" />
</div>

#### 2.MultiAutoCompleteTextView的基本用法
```
//待匹配的邮件字符串
private String[] emailRes = new String[]{"sdjfjie@qq.com", "djifgieh@gmail.com"};
//创建适配器
ArrayAdapter adapter = new ArrayAdapter(this, android.R.layout.simple_list_item_1, emailRes);
/*将适配器绑定到MultiAutoCompleteTextView
* 注意：在xml文件下的<AutoCompleteTextView/>标签下需要添加*android:completionThreshold="3"属性值，
* 该属性值用于在文本框中输入到第几个字符开始匹配给定的字符串*/
this.binding.txtMultiAutoComplete.setAdapter(adapter);
//添加分隔符“,”
this.binding.txtMultiAutoComplete.setTokenizer(new MultiAutoCompleteTextView.CommaTokenizer());
<MultiAutoCompleteTextView
    android:id="@+id/txt_multi_auto_complete"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:layout_below="@id/txt_auto_complete"
    android:hint="请添加收件人邮箱"
    android:textColorHint="#dddddd"
    android:completionThreshold="3"
/>
```
效果图如下：
<div align = "center">
<img src="http://upload-images.jianshu.io/upload_images/5231076-2ee52a15b0ce6b6e.gif?imageMogr2/auto-orient/strip" style = "zoom:50%" />
</div>


