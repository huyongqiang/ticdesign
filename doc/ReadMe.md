## Ticwear 设计辅助库使用说明

使用 Ticwear 设计辅助库（后文简称设计库），你可以轻松让你的应用符合 Ticwear 的[设计规范][ticwear-design]，并支持挠挠等新型交互方式。

除挠挠之类 Ticwear 特有的交互外，设计库的其他部分也可以工作在 Android Wear 上。

设计库包含下面几个部分：

1. [样式和主题](#style-and-theme)：定义了一些列文字、页面和控件的样式，已经页面切换等动效支持。
2. [可拉伸、响应内容滚动的标题](#title-bar)：基于 [Android Design Support][google-design-support]，构建了一套适合手表展示的页面结构，除了 Google Design 中的跟随滚动等效果，我们还为标题栏增加了可拉伸等效果。
3. [支持挠挠的 Listview](#ticklable-listview)：基于 `RecyclerView`，我们创造了一个列表展示控件，使其在触摸操作时与普通线性布局的列表操作无异，而使用挠挠交互时，具有聚焦效果（聚焦效果类似 `WearableListView`），方便挠挠的操作。
4. [设置](#preference)：提供一套类似 [Android Settings][android-settings] 的、符合 [Ticwear Design][ticwear-design] 的设置系统，更适合手表展示，并支持挠挠交互。
5. [其他小控件](#widgets)：Ticwear提供了一系列适合手表使用的小控件，包括[可缩放文本框](#scale-textview)、[悬浮按钮](#fab)、[弹出式对话框](#alert-dialog)、[数值选择器](#number-picker)、[日期时间选择器](#date-picker)等。

### <a id="style-and-theme"></a> 样式和主题

Ticwear为开发者提供了一套符合Ticwear设计规范的主题。开发者可以直接使用或基于此主题进行自己的个性化扩展。包括以下主题：

1. `Theme.Ticwear` Ticwear Design 默认主题，定义了文字、窗口样式、页面切换效果、设置样式等一系列适用于手表的样式。
2. `Theme.Ticwear.Dialog`，适用于手表的对话框。全屏显示、滑动方式的进入、退出动画。

除了主题，开发者可以使用我们定义好的一系列样式。详情参看`styles_ticwear.xml`代码。

### <a id="title-bar"></a> 响应内容滚动操作的标题栏

类似 [Android Design Support][google-design-support]，使用`CoordinatorLayout`来组织`AppBarLayout`以及页面内容，可以使得标题响应页面内容的滚动，实现多种标题效果。

除了 Android Design 支持的“固定、滚动、快速进入、折叠”等效果外，Ticwear Design 额外支持了“拉伸回弹”效果，并配套提供了可缩放的文本框，`ScalableTextView`，使得标题可以在拉伸时变大，并伴随着阻尼效果。

下面的代码给出了一个页面的布局示例：

``` xml
<?xml version="1.0" encoding="utf-8"?>
<ticwear.design.widget.CoordinatorLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:scrollbars="vertical">

    <include layout="@layout/content_main" />

    <ticwear.design.widget.AppBarLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content">

        <ticwear.design.widget.ScalableTextView
            style="?android:textAppearanceSmall"
            android:layout_width="match_parent"
            android:layout_height="32dp"
            android:gravity="center"
            android:text="@string/main.title"
            app:tic_layout_scrollFlags="scroll|overScrollBounce|enterAlways"
            app:tic_layout_scrollResistanceFactor="0.5"
            app:tic_scaleFactor="0.5"
            />

    </ticwear.design.widget.AppBarLayout>

</ticwear.design.widget.CoordinatorLayout>
```

通过多种效果的配合，此布局实现了标题快速进入（页面滚动到底部后下拉，标题马上出现，而不用滚动到顶部才出现），和拉伸回弹效果（页面滚动到顶部后，继续下拉，标题会随滚动操作被拉大，松手后弹回原状）。

其中，`tic_layout_XXX`，类似于`android:layout_XXX`，表示此View在parent中的布局行为，与内容无关。而其他非`tic_layout_`开头的熟悉，则是View本身的属性。下面对这些属性做详细解释：

* `app:tic_layout_scrollFlags` 指定了标题的滚动响应行为，可以根据需要组合不同的行为。
* `app:tic_layout_scrollResistanceFactor` 指定了标题在拉伸时，整体高度的变化倍数。为1时，标题高度变化与滚动距离对应，没有阻尼效果。越接近0，阻尼效果越大，高度变化越小。
* `app:tic_scaleFactor` 指定了可缩放文字的缩放倍数。为1时，文字会跟随文本框大小做等比缩放，约接近0则缩放效果越不明显。详情可以参考[可缩放文本框](#scale-textview)。

### <a id="ticklable-listview"></a> 支持挠挠交互的 Listview

`TickableListView` 结合了`ListView`和`WearableListView`的优势，使得列表控件在正常状态时表现的像普通的ListView，以呈现较丰富、美观的视觉效果，且可以任意点击视图中的列表项；而在挠挠触碰上去之后，转变成聚焦态，内容变大，聚焦在页面中部的元素，使得操作变得准确有目标。

我们为`TickableListView`做了特别的设计，使得它能与`AppBarLayout`相互配合，在聚焦态时，也能较好的实现TitleBar的各种效果。详情可以阅读源码中的`TickableListViewBehavior`代码。

`TickableListView`继承了[`RecyclerView`][recycler-view]，你可以像使用`RecyclerView`一样使用`TickableListView`，但是需要注意的是，你的`Adapter`需要继承`TickableListView.Adapter`，`ViewHolder`需要继承`TickableListView.ViewHolder`。

`TickableListView.ViewHolder` 设置了默认的聚焦态动效，即聚焦时放大、变亮，非聚焦时缩小、变暗。如果你想定义更细致的动画效果。可以使你的`ItemView`实现`TicklableListView.OnFocusStateChangedListener`接口，或者，直接重载`ViewHolder.onFocusStateChanged`方法。

下面是一个比较粗糙简单的重载方式（与默认动效相同）：

``` Java
@Override
protected void onFocusStateChanged(@TicklableListView.FocusState int focusState,
                                   boolean animate) {
    float scale = 1.0f;
    float alpha = 1.0f;
    switch (focusState) {
        case TicklableListView.FOCUS_STATE_NORMAL:
            break;
        case TicklableListView.FOCUS_STATE_CENTRAL:
            scale = 1.1f;
            alpha = 1.0f;
            break;
        case TicklableListView.FOCUS_STATE_NON_CENTRAL:
            scale = 0.9f;
            alpha = 0.6f;
            break;
    }
    if (animate) {
        itemView.animate()
                .setDuration(200)
                .alpha(alpha)
                .scaleX(scale)
                .scaleY(scale);
    } else {
        itemView.setScaleX(scale);
        itemView.setScaleY(scale);
        itemView.setAlpha(alpha);
    }
}
```

为了方便开发者，我们提供了一个`SimpleRecyclerAdapter`来快速构造列表所需的内容。其使用方式类似[`ListView.SimpleAdapter`][simple-adapter]。


### <a id="preference"></a> 设置系统

Ticwear 的设置系统类似 [Android Settings][android-settings]，你可以使用与 Android Preference 相同的方式来使用 Ticwear Preference。但请注意，Ticwear Preference 已经将内置的 `ListView` 改成了 `TicklableListView`，你需要使用 `RecyclerView.ViewHolder` 的方式来绑定数据到 Preference view 上面。

当你需要实现自定义的 `Preference` 时，需要继承 `Preference.ViewHolder`，并按需要覆盖其方法，以绑定你的自定义数据。

### <a id="widgets"></a> 小控件

#### <a id="scale-textview"></a> 可缩放文本框

`ScalableTextView`，可缩放文本框，可以跟随控件大小而改变文字大小。常用于`TitleBar`中的标题显示。

使用时，可以从XML文件，或者代码中，指定缩放因子 `scaleFactor`。文字大小变化、文本框大小变化与缩放因子之间符合下面的等式：

$$
\Delta_{textScale} = \Delta_{frameScale} \times scaleFactor
$$

其中：

$$
\Delta_{frameScale} = max\left(
    \frac{w_{dst}}{w_{src}},
    \frac{h_{dst}}{h_{src}}
\right)
$$

需要注意的是，由于文字会进行缩放，所以可能会越出边界。使用时最好指定足量的padding，或者设置无变化的那边为 match_parent。例如，`TitleBar`中的`ScalableTextView`，通常会以如下方式布局：

``` xml
<ticwear.design.widget.AppBarLayout
    android:layout_width="match_parent"
    android:layout_height="wrap_content">
    <ticwear.design.widget.ScalableTextView
        style="?android:textAppearanceMedium"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:gravity="center"
        android:clipToPadding="false"
        android:padding="8dp"
        android:text="Title"
        app:tic_layout_scrollFlags="scroll|overScrollBounce"
        app:tic_layout_scrollResistanceFactor="0.5"
        app:tic_scaleFactor="0.5"
        />
</ticwear.design.widget.AppBarLayout>
```

#### <a id="fab"></a> 悬浮按钮

`FloatingActionButton`，悬浮按钮，是一个扩展的`ImageButton`。我们从 [Android Design Support][google-design-support] 库中将其移植过来，去除了一些不利于手表展示的部分（比如与`SnackBar`的交互），增加了Ticwear特有的设计元素。

常规的使用方式，可以查看 [Android 官方文档][android-fab]。

而Ticwear特有的修改，主要是增加了`minimum`，最小化状态（原生控件只支持`shown`和`hidden`状态），这个状态，会使得按钮缩小到一个小点，不会遮挡文字，又可以提示用户这里有可操作的元素。

使用方式类似`show()`和`hide()`，调用`minimize()`可以最小化按钮，按钮在完成最小化以后，会触发 `OnVisibilityChangedListener.onMinimum` 回调。

#### <a id="alert-dialog"></a> 弹出式对话框

移植并扩展了 Android 的 [`AlertDialog`][android-alert-dialog]。为其定制了适用于手表的主题。并提供了利于手表显示的圆形图标按钮，以替代原生的文字按钮。

当设置的文本消息非常长时，消息将可以滚动，并且，滚动时底部的图标按钮会消失，以便更方便的阅读文本内容。

使用方式与原生的 `AlertDialog` 无异，只是需要制定图标资源文件，或图标的`Drawable`，类似下面的使用方式：

``` Java
new AlertDialog.Builder(context)
        .setTitle(R.string.dialog_title)
        .setMessage(R.string.dialog_content)
        .setPositiveButtonIcon(R.drawable.ic_btn_ok, new DialogInterface.OnClickListener() {
            @Override
            public void onClick(DialogInterface dialog, int which) {
                // Do something for positive action.
                dialog.dismiss();
            }
        })
        .show();
```

#### <a id="number-picker"></a> 数值选择器

类似 Android 的 [`NumberPicker`][android-numberpicker]，我们实现了符合Ticwear设计标准的数值选择器。并提供了一个便于使用的`NumberPickerDialog`，开发者可以像使用[`AlertDialog`][android-alert-dialog]一样，显示一个对话框让用户选择一个值。使用方式如下：

``` Java
new NumberPickerDialog.Builder(context)
        .minValue(0)
        .maxValue(20)
        .defaultValue(5)
        .valuePickedlistener(new NumberPickerDialog.OnValuePickedListener() {
            @Override
            public void onValuePicked(NumberPickerDialog dialog, int value) {
                Toast.makeText(dialog.getContext(), "Picked value " + value,
                        Toast.LENGTH_SHORT)
                        .show();
            }
        })
        .show();
```

#### <a id="date-picker"></a> 日期时间选择器

类似于 Android 的 [`TimePicker`][android-timepicker] 和 [`DatePicker`][android-datepicker]，我们实现了符合Ticwear设计标准的日期和时间选择器。开发者可以像 Android 控件一样使用它们。

为了开发的方便，我们还提供了一个日期时间选择对话框，`DatetimePickerDialog`，可以像使用[`AlertDialog`][android-alert-dialog]一样，显示一个对话框让用户选择日期、时间，或同时选择两者。使用方式如下：

``` Java
new DatetimePickerDialog.Builder(getActivity())
        .defaultValue(Calendar.getInstance())
        .listener(new DatetimePickerDialog.OnCalendarSetListener() {
            @Override
            public void onCalendarSet(DatetimePickerDialog dialog,
                                      Calendar calendar) {
                Toast.makeText(dialog.getContext(), "Picked datetime: " +
                                SimpleDateFormat.getDateTimeInstance()
                                        .format(calendar.getTime()),
                        Toast.LENGTH_LONG)
                        .show();
            }
        })
        .show();
```

如果你只想让用户选择日期，或者只选择时间，那么你需要做的，是在 Building 时，指定 `disableTimePicker()` 或者 `disableDatePicker()`。详情可以参考我们 Demo 中的 `DialogsFragment`。


[ticwear-design]: http://developer.ticwear.com/doc/guideline
[google-design-support]: http://android-developers.blogspot.hk/2015/05/android-design-support-library.html
[android-settings]: http://developer.android.com/guide/topics/ui/settings.html
[android-alert-dialog]: http://developer.android.com/reference/android/app/AlertDialog.html
[android-fab]: http://developer.android.com/reference/android/support/design/widget/FloatingActionButton.html
[recycler-view]: http://developer.android.com/reference/android/support/v7/widget/RecyclerView.html
[simple-adapter]: http://developer.android.com/reference/android/widget/SimpleAdapter.html
[android-numberpicker]: http://developer.android.com/reference/android/widget/NumberPicker.html
[android-timepicker]: http://developer.android.com/reference/android/widget/TimePicker.html
[android-datepicker]: http://developer.android.com/reference/android/widget/DatePicker.html
