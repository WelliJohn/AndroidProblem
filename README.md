## 一.资源文件报空指针，转换异常，但是明明写的都是正确的，那么这个时候，需要考虑下下面这种情况？
1.当我们的Android程序有多个module的情况下，如果在有模块引入别的模块的情况，主模块的资源文件会覆盖子模块所有的资源，导致子模块获取的资源是主模块的资源。
2.这里的资源文件（layout,string,color,style等）但是不包含id，如果不同模块的id相同的话，这个是没有问题，除了包含id的layout也相同，这样就会用主模块的id了。

## 二.我只是启动一个应用程序，为什么Application的onCreate执行了多次？
答:在启动应用程序的时候，linux中调用fork创建的子进程，将共享父进程的代码空间，复制父进程数据空间，此时子进程会获得父进程的所有变量的一份拷贝。如果这个时候第三方框架会启动新的进程，那么也会执行接下来的Application的代码，所以会执行多次了。

## 三.View.setVIsibility（Gone）的时候，不起作用，或者出现gone的那一块控件为黑色？
答:修改布局的设置：
```
    <RelativeLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content">

        <LinearLayout
            android:id="@+id/vis_or_gone"//通过这个id来控制Visible还是Gone
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:background="@color/white"
            android:visibility="gone">

            //这里放置你要Visible和Gone的布局
        </LinearLayout>
    </RelativeLayout>
```

## 四.华为手机和三星手机，popupwindow中包含EditText的时候，当EditText获取焦点，整个popupwindow的背景都变透明？
答:这个需要我们在popupwindow的contentView的背景设置为想要的颜色，而且contentView中包含的子控件，如果是树顶的控件（就是最上层显示的控件）也需要设置我们想要的背景色，这样popwindow就不会变成透明了。


## 五.Tablayout + ViewPager + fragment 切换时生命周期不调用?
答：我们在写自己的fragmentAdapter的时候，将tag和position绑定起来，比如下面的
```
public class BaseFragmentAdapter extends TabFragmentAdapter {
    private FragmentManager mFragmentManager;
    private SparseArray<String> mFragmentTags;

    public BaseFragmentAdapter(@NonNull FragmentManager fm, @NonNull List<String> titles, @NonNull List<Fragment> fragments) {
        super(fm, titles, fragments);
        mFragmentManager = fm;
        mFragmentTags = new SparseArray<>();
    }

    @Override
    public Object instantiateItem(ViewGroup container, int position) {
        Object object = super.instantiateItem(container, position);
        if (object instanceof Fragment) {
            Fragment fragment = (Fragment) object;
            String tag = fragment.getTag();
            mFragmentTags.append(position, tag);
        }
        return object;
    }

    public Fragment getFragment(int position){
        String tag = mFragmentTags.get(position);
        if(StringUtil.isStringEmpty(tag)){
            return null;
        }
        return mFragmentManager.findFragmentByTag(tag);
    }

}
```
然后在vp切换的回调方法中调用：
```
 mViewpager.addOnPageChangeListener(new ViewPager.OnPageChangeListener() {
            @Override
            public void onPageScrolled(int position, float positionOffset, int positionOffsetPixels) {

            }

            @Override
            public void onPageSelected(int position) {
                Fragment currentFragment = mTabFragmentAdapter.getFragment(position);
                if ((0 == position || position == mTabFragmentAdapter.getCount() - 1) && null != currentFragment) {
                    currentFragment.onResume();
                }
            }

            @Override
            public void onPageScrollStateChanged(int state) {

            }
        });
```
上面的是调用onResume的什么周期，其他你们也是可以处理的。

## 六.Android中对某个View频繁设置Visible和Gone的时候，有的时候会出现Gone却Gone不掉的情况？
答：这种情况，多半是因为View还没有计算好自己的宽高，所以Gone不掉，我们一般可以在如下方式：
```
    YouView.post(new Runnable() {
            @Override
            public void run() {
                rlOptionByEmployee.setVisibility(View.GONE);
                }
            });
```
或者是在我们设置View的状态，gone之后，再增加如下的代码：
```
    YouView.requestLayout();
    YouView.invalide();
```

## 七.Android的Popupwindow在7.0以上的设备，用showAsDropDown的时候，当我们设置match_parent的时候，会全屏铺满？
答:可以考虑下[专门为支持7.0以上的设备显示的popupwindow](https://github.com/WelliJohn/PopupWindowSet/blob/master/popwindowset/src/main/java/wellijohn/org/popwindowset/DropDownPopupWindow.java)。

## 八.ScollView或者RecyclerView等自动滚动的处理？
答：[ScrollVIew自动滚动的解决方案](https://juejin.im/post/5a2a04726fb9a045055e0993)
