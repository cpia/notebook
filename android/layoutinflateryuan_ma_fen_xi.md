# inflate方法用哪个


LayoutInflater的二个inflate方法：
*  public View inflate( int resource,  ViewGroup root) 
*   public View inflate(int resource, ViewGroup root, boolean attachToRoot)

```
测试
 protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
    //    setContentView(R.layout.activity_main);
        final LayoutInflater inflater = getLayoutInflater();
        //A
        final View view1 = inflater.inflate(R.layout.activity_main, null);
        //B
        final View view2 = inflater.inflate(R.layout.activity_main, (ViewGroup) findViewById(android.R.id.content),false);
        //C。并且会显示在界面上
        final View view3 = inflater.inflate(R.layout.activity_main, (ViewGroup) findViewById(android.R.id.content),true);
        
        // 参数为Null
        Log.d(TAG, "onCreate() view1: " + view1 +"layoutparam = "+view1.getLayoutParams()); 
        
         // 有参数FrameLayout$LayoutParams
        Log.d(TAG, "onCreate() view2: " + view2 +"layoutparam = "+view2.getLayoutParams());
        
        //有参数LinearLayout$LayoutParams
        Log.d(TAG, "onCreate() view3: " + view3 +"layoutparam = "+view3.getLayoutParams());
}
```

分析ABC
三个方法最后都会调用下面这个：
```
 public View inflate(XmlPullParser parser, ViewGroup root, boolean attachToRoot) {
  ...
   View result = root;
     // Temp 就是我们的xml布局
       final View temp = createViewFromTag(root, name, inflaterContext, attrs);

                    ViewGroup.LayoutParams params = null;
                    //如果是上面的A调用，则不会创建layout params
                    if (root != null) {
                       
                        //B和C  根据root创建layout params
                        params = root.generateLayoutParams(attrs);
                        if (!attachToRoot) {
                            //B 第三个参数attachToRoot为 false,则会设置layout params                          
                            temp.setLayoutParams(params);
                        }
                    }
                 
                   
                    //C 如果设置了root并且第三个参数为true.则使用params把temp添加 的root上去
                    if (root != null && attachToRoot) {
                        root.addView(temp, params);
                    }

                    //A 如果root没设置并且第三参数为false
                    //则直接返回temp
                    if (root == null || !attachToRoot) {
                        result = temp;
                    }
               
               /*
               A:返回的是temp
               B:返回的是root,temp设置了params,但没有添加 到root上
               C：返回的是root并且会把布局添加到root上
               */
              return result;      
 }
 ```
 
##在定义view中对onMeasure()方法的处理
都会根据测量模式做不同处理


> MeasureSpec.EXACTLY == LayoutParams. MATCH_PARENT或设置的一个精确值

> MeasureSpec.AT_MOST == LayoutParams. WRAP_CONTENT

可以知晓测量时依赖于LayoutParams。而inflate(resId,null)是不会设置LayoutParams的。

在listview中出现item设置的宽高没有被处理的时候可以检查一下是否正确调用了inflate方法