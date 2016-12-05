# draw-a-triangle

> Date: 2017-03-22
>
> Author: xieshiyi
>
> Title: 画一个三角形

# Transform 考察
###### 画一个三角形首先考虑的是线条的角度问题，这个时候主要考察 CSS 的 transform 属性和 rotate 属性值的写法。

```html
<html>
<head>
    <style>
        div {
            background: #000;
            width: 50px;
            height: 1px;
        }
        
        .left {
            transform: rotate(60deg); // 旋转60度
            position: absolute;
            left: 70px;
            top: 30px;
        }
        
        .right {
            transform: rotate(-60deg);
            position: absolute;
            top: 30px;
            left: 45px;
        }
        
        .bottom {
            position: absolute;
            top: 52px;
            left: 58px;
        }
    </style>
</head>

<body>
    <div class="left"></div>
    <div class="right"></div>
    <div class="bottom"></div>
</body>
</html>
```
注意：rotate的角度单位必须要写，否则出错。