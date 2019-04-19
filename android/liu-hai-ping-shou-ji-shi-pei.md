# 刘海屏手机适配

刘海屏介绍:  
[https://blog.csdn.net/djy1992/article/details/80689308](https://blog.csdn.net/djy1992/article/details/80689308)  
[https://blog.csdn.net/mysimplelove/article/details/81187648](https://blog.csdn.net/mysimplelove/article/details/81187648)  
参考工具：  
[https://github.com/clayx/ChayTestCutout](https://github.com/clayx/ChayTestCutout)

## 主要代码

\(完整代码和逻辑请参考主版本svn androidP以下版本刘海屏适配 提交记录\)  
使用到的工具代码位置：主版本src\main\java\com\xvideostudio\videoeditor\util\notch目录

```java
// 主要逻辑（BaseActivity的onCreate()生命周期中，截出部分代码，适配时请参照主版本相应代码）
NotchUtil.isHasCutout(this, new OnCutoutListener() {
@Override
public void isHasCutout(boolean isHas) {
    isHasCutout = isHas;
    if (isHas) {
    if (mContext.getLocalClassName().contains("activity.MainActivity") || mContext.getLocalClassName().contains("activity.GoogleVipSingleLiteActivity")
    || mContext.getLocalClassName().contains("activity.SplashActivity") || mContext.getLocalClassName().contains("activity.SplashScreenActivity")|| mContext.getLocalClassName().contains("activity.MaterialItemInfoActivity")) {
        getWindow().clearFlags(WindowManager.LayoutParams.FLAG_FULLSCREEN);
        NotchUtil.setImmersiveWithNotch(mContext,false,WindowManager.LayoutParams.LAYOUT_IN_DISPLAY_CUTOUT_MODE_SHORT_EDGES);
    } else if (mContext.getLocalClassName().contains("activity.MyStudioActivity") ||mContext.getLocalClassName().contains("activity.MaterialCategoryHistorySettingActivity")|| mContext.getLocalClassName().contains("activity.OperationManagerActivity") || mContext.getLocalClassName().contains("activity.HomeLikeUsAndFAQActivity")
 || mContext.getLocalClassName().contains("activity.HomeOpenUrlActivity")) {
        getWindow().clearFlags(WindowManager.LayoutParams.FLAG_FULLSCREEN);
        NotchUtil.setImmersiveWithNotch(mContext, true, WindowManager.LayoutParams.LAYOUT_IN_DISPLAY_CUTOUT_MODE_NEVER);
        StatusBarUtil.setStatusBarColor(mContext, R.color.white);
    } else if (mContext.getLocalClassName().contains("activity.MaterialActivityNew")) {
        getWindow().clearFlags(WindowManager.LayoutParams.FLAG_FULLSCREEN);
        NotchUtil.setImmersiveWithNotch(mContext, false, WindowManager.LayoutParams.LAYOUT_IN_DISPLAY_CUTOUT_MODE_NEVER);
        StatusBarUtil.setStatusBarColor(mContext, R.color.material_color_accent); 
    } else if (mContext.getLocalClassName().contains("activity.EditorPreviewActivity") || mContext.getLocalClassName().contains("activity.CameraActivity")) {
        // DoNothing
    } else {
        getWindow().clearFlags(WindowManager.LayoutParams.FLAG_FULLSCREEN);
        NotchUtil.setImmersiveWithNotch(mContext, false, WindowManager.LayoutParams.LAYOUT_IN_DISPLAY_CUTOUT_MODE_NEVER);
                        StatusBarUtil.setStatusBarColor(mContext, R.color.colorPrimary);
            }
        }
    }
});
```

下面来分析上面的代码：  
由于各个页面需求的效果不一样，所以针对不同的页面做不同处理。

### 闪屏页面和有海报的页面，需要填充刘海屏区域

1. 将相应的actvity在AndroidManifest的声明中theme属性设置为`AppTheme.Fullscreen`  

这是为了适配非刘海屏手机

```text
   <style name="AppTheme.Fullscreen" parent="AppTheme.NoActionBar">
    <item name="android:windowFullscreen">true</item>
   </style>
```

1. 在代码中清除`FLAG_FULLSCREEN`，刘海屏设置只能在非全屏状态下执行，设置`LAYOUT_IN_DISPLAY_CUTOUT_MODE_SHORT_EDGES`允许window扩展到刘海区

   ```text
   getWindow().clearFlags(WindowManager.LayoutParams.FLAG_FULLSCREEN);
   NotchUtil.setImmersiveWithNotch(mContext,false,WindowManager.LayoutParams.LAYOUT_IN_DISPLAY_CUTOUT_MODE_SHORT_EDGES);
   ```

2. 页面效果完成后细节通过判断`isHasCutOut`来具体实现

   **工具箱等页面需要留出刘海屏区防止标题栏内容被刘海遮盖**

3. 在AndroidManifest的application声明中theme属性设置为`AppTheme.FullscreenFits`

```text
   // 适配刘海屏
   <style name="AppTheme.FullscreenFits" parent="AppTheme.NoActionBar">
    <item name="android:windowFullscreen">true</item>
    <item name="android:fitsSystemWindows">true</item>
   </style>
```

对比`AppTheme.Fullscreen`，我们可以发现增加了`fitsSystemWindows`属性，fitsSystemWindows属性可以让view根据系统窗口来调整自己的布局，为了留下刘海区域，需要设置这个属性。如果要扩展内容到刘海屏，则需要去掉这个属性。

1. 在代码中清除`FLAG_FULLSCREEN`，刘海屏设置只能在非全屏状态下执行，设置`LAYOUT_IN_DISPLAY_CUTOUT_MODE_NEVER`不允许window扩展到刘海区，这时也可以设置状态颜色完成沉㓎状态栏需求\(`setImmersiveWithNotch`第二个参数`isLightMode`设置状态栏黑白文字颜色\)

   ```java
   getWindow().clearFlags(WindowManager.LayoutParams.FLAG_FULLSCREEN);
   NotchUtil.setImmersiveWithNotch(mContext, true, WindowManager.LayoutParams.LAYOUT_IN_DISPLAY_CUTOUT_MODE_NEVER);
   StatusBarUtil.setStatusBarColor(mContext, R.color.white);
   ```

   **对于不需要适配的页面不做操作即可**

   ```java
   else if (mContext.getLocalClassName().contains("activity.EditorPreviewActivity") || mContext.getLocalClassName().contains("activity.CameraActivity")) {
        // DoNothing
    }
   ```

   **如果还在使用Dialog，请停止使用**`Dialog dialog = new Dialog(context,R.style.Transparent);`

   调用`Dialog dialog = new Dialog(context);`，去掉`R.style.Transparent`属性防止页面头部和刘海屏重叠

   **使用**`WindowManager.LayoutParams.flags`**时**

   判断是否刘海屏，如果是刘海屏，需要去掉`WindowManager.LayoutParams.FLAG_FULLSCREEN`

```java
   if(MainActivity.isHasCutout){
    wmLayoutParams.flags= LayoutParams.FLAG_NOT_FOCUSABLE;
   }else{
    wmLayoutParams.flags = LayoutParams.FLAG_NOT_FOCUSABLE | WindowManager.LayoutParams.FLAG_FULLSCREEN;
   }
```

同理刘海屏遇到`WindowManager.LayoutParams.FLAG_FULLSCREEN`也不能添加

```text
   // GifSearchActivity中onCreate()
   if(!MainActivity.isHasCutout){
    getWindow().addFlags(WindowManager.LayoutParams.FLAG_FULLSCREEN); //隐藏状态栏
   }
```

**将**`android.app.AlertDialog`**全部替换成**`android.support.v7.app.AlertDialog`

原因是设置了`fitsSystemWindows`后弹框内容和边缘padding为0

**popwindow中的文字位置错乱置顶时，请在相应的TextView xml布局中添加**`android:fitsSystemWindows="false"`

参见主版本svn记录19282：引导页面和气泡文字位置偏离修复

```text
   <TextView xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/textView_poptipscenter"
    android:fitsSystemWindows="false"
```

相应的，`seekbar`等控件被切掉时，也需要在相应控件VIEW布局里增加`android:fitsSystemWindows="false"`

**Toast中的文字位置错乱顶时，请在相应的布局中添加**`android:fitsSystemWindows="false"`

参见主版本svn记录19633：toast 显示问题修改

```text
   <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:fitsSystemWindows="false"
```

1. 如果是系统的Toast，未使用自定义布局  

上下文如果传如Activity，就会发生偏移，此时把上下文改为getApplicationContext\(\)即可。

