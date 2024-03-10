# 附录 A　制作 Chrome 主题

Chrome 主题与扩展和应用的结构类似，包含一个 Manifest 文件和一些图片资源。主题的 Manifest 结构如下：

```
{
    "version": "2.6",
    "name": "camo theme",
    "theme": {
        "images" : {
            "theme_frame" : "images/theme_frame_camo.png",
            "theme_frame_overlay" : "images/theme_frame_stripe.png",
            "theme_toolbar" : "images/theme_toolbar_camo.png",
            "theme_ntp_background" : "images/theme_ntp_background_norepeat.png",
            "theme_ntp_attribution" : "images/attribution.png"
        },
        "colors" : {
            "frame" : [71, 105, 91],
            "toolbar" : [207, 221, 192],
            "ntp_text" : [20, 40, 0],
            "ntp_link" : [36, 70, 0],
            "ntp_section" : [207, 221, 192],
            "button_background" : [255, 255, 255]
        },
        "tints" : {
            "buttons" : [0.33, 0.5, 0.47]
        },
        "properties" : {
            "ntp_background_alignment" : "bottom"
        }
    }
} 
```

颜色使用 RGB 格式，即`[r, g, b]`。图片路径使用基于主题包根路径的相对路径。`properties`定义了图片的位置和`repeat`属性。`tints`定义了按钮、框架和后台标签页等某些部分的色调，使用 HSL 格式，取值范围为 0 到 1 的浮点型数据。

更详细的内容可以参见[`code.google.com/p/chromium/wiki/ThemeCreationGuide`](https://code.google.com/p/chromium/wiki/ThemeCreationGuide)。