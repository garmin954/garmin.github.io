---
title: 按住三秒的事件
categories:
- js
tags:
- js
- 事件
---

~~~
@touchstart.prevent="touchin(i, $event)"
@touchend.prevent="cleartime()"
~~~

~~~
// 500表示触屏时间，可以根据实际情况写，只要达到这个时间就会触发setTimeout里的事件
// 删除图片方法
touchin(index, obj){
    var _self=this;
    this.Loop = setTimeout(function() {
        _self.Loop = 0;
        //执行长按要执行的内容，如弹出菜单
        wx.showModal({
            title: '提示',
            content: '确认删除图片',
            success: function (res) {
                if (res.confirm) {
                    // 删除操作
                    _self.images_list.splice(index, 1);
                } else if (res.cancel) {
                    // 取消操作
                }
            }
        });

    }, 500);
    return false;
},

// 触屏离开的事件
cleartime(index) {
    var that=this;
    clearTimeout(this.Loop);
    if(that.Loop!=0){
        //这里写要执行的内容（尤如onclick事件）
        // that.previewPicture(index)
    }
    return false;
},
~~~
