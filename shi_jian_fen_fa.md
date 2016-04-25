# android事件分发

---
* ###布局层次示例

```
 - groupA
    - groupB
      - view
```

   groupA：

```java
    @Override
    public boolean dispatchTouchEvent(MotionEvent ev) {
        Log.d(TAG, "dispatchTouchEvent() called with: " + "ev = [" + ev + "]");
        boolean b=super.dispatchTouchEvent(ev);
        Log.d(TAG, "dispatchTouchEvent()  returned: " + b);

        return b;
    }

    @Override
    public boolean onInterceptTouchEvent(MotionEvent ev) {
        Log.d(TAG, "onInterceptTouchEvent() called with: " + "ev = [" + ev + "]");
        boolean b= super.onInterceptTouchEvent(ev);

        Log.d(TAG, "onInterceptTouchEvent()  returned: " + b);
        return b;
    }

    @Override
    public boolean onTouchEvent(MotionEvent event) {
        Log.d(TAG, "onTouchEvent() called with: " + "event = [" + event + "]");
       boolean b= super.onTouchEvent(event);

        Log.d(TAG, "onTouchEvent() returned: " + b);
        return b;
    }
```
groupB:

```java
  @Override
    public boolean dispatchTouchEvent(MotionEvent ev) {
        Log.d(TAG, "dispatchTouchEvent() called with: " + "ev = [" + ev + "]");
        boolean b=super.dispatchTouchEvent(ev);
        Log.d(TAG, "dispatchTouchEvent() returned: " + b);

        return b;
    }

    @Override
    public boolean onInterceptTouchEvent(MotionEvent ev) {
        Log.d(TAG, "onInterceptTouchEvent() called with: " + "ev = [" + ev + "]");
        boolean b= super.onInterceptTouchEvent(ev);

        Log.d(TAG, "onInterceptTouchEvent() returned: " + b);
        return b;
    }

    @Override
    public boolean onTouchEvent(MotionEvent event) {
        Log.d(TAG, "onTouchEvent() called with: " + "event = [" + event + "]");
        boolean b= super.onTouchEvent(event);
        
        Log.d(TAG, "onTouchEvent() returned: " + b);
        return b;
    }
```
view:

```java
   @Override
    public boolean dispatchTouchEvent(MotionEvent event) {
        Log.d(TAG, "dispatchTouchEvent() called with: " + "event = [" + event + "]");
        boolean b= super.dispatchTouchEvent(event);
        Log.d(TAG, "dispatchTouchEvent() returned: " + b);

        return b;
    }

    @Override
    public boolean onTouchEvent(MotionEvent event) {
        Log.d(TAG, "onTouchEvent() called with: " + "event = [" + event + "]");
       boolean b= super.onTouchEvent(event);

        Log.d(TAG, "onTouchEvent() returned: " + b);

        return b;
    }
```
###点击view走的默认流程：返回false


>  - groupA dispatchTouchEvent start
     - onInterceptTouchEvent start
     - onInterceptTouchEvent end
       - groupB dispatchTouchEvent start
         - onInterceptTouchEvent start
         - onInterceptTouchEvent end
           - view dispatchTouchEvent start
             - onTouchEvent start
             - onTouchEvent end
           - view dispatchTouchEvent end
         - groupB onTouchEvent start
         - onTouchEvent end
       - groupB dispatchTouchEvent end
     - groupA onTouchEvent start
     - onTouchEvent end
 - groupA dispatchTouchEvent end

###分析
`dispatchTouchEvent`与`onInterceptTouchEvent`属于事件分发，事件处理则是`onTouchEvent`。
- 拦截。这里假若在groupB中拦截
  - 在groupB中的onInterceptTouchEvent返回了true，则表示groupB要拦截本次事件，会调用groupB的onTouchEvent进行事件处理。如果还有下一个事件(本次调用的onTouchEvent返回true)则会重新从groupA开发分发，到groupB时，不在走onInterceptTouchEvent并且不分发给view，而是直接走groupB的onTouchEvent。
  
- 事件处理
  - onTouchEvent返回false时，则会向上调用父view自己的onTouchEvent。返回true时，表示自己处理了本次事件，不再走父view的onTouchEvent，下一次事件分发时，前面是哪个view处理了事件，则会再次把事件分发给那个view处理。

- 综合流程
  - 点击view,groupA与groupB都不拦截。
  - view的onTouchEvent返回true，则下一次事件重新分发到view处理。如果返回false,调用groupB的onTouchEvent。
    - groupB onTouchEvent返回false，调用groupA的onTouchEvent处理
      - groupA onTouchEvent 返回false，事件冒泡，等下一次事件。返回true，下一次事件直接走groupA onTouchEvent不分发给子view
    - groupB onTouchEvent返回true。不调用groupA处理。等下一次事件分发到groupB时，不走onInterceptTouchEvent也不分发给view，而是直接走groupB的onTouchEvent。对于返回结果的处理按上面分析的处理