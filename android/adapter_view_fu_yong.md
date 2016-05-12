# adapter view 复用

AbsListView->RecycleBin
```
class RecycleBin:

  
  //存储的是第一次显示在屏幕上的View  
   private View[] mActiveViews = new View[0];
   
   //有可能被adapter复用的View   
    private ArrayList&lt;View&gt;[] mScrapViews;
 ```    
    
在ListView中
```
  protected void layoutChildren() {
    
            // 如果数据有改变则把当前的itemview添加 到mScrapViews中
            //否则把当前显示的itemview添加到mActiveViews中
            final int firstPosition = mFirstPosition;
            final RecycleBin recycleBin = mRecycler;
            if (dataChanged) {
                for (int i = 0; i < childCount; i++) {
                    recycleBin.addScrapView(getChildAt(i), firstPosition+i);
                }
            } else {
                recycleBin.fillActiveViews(childCount, firstPosition);
            }
    }
       recycleBin：
         //把当前显示的view添加 到mActiveViews
        void fillActiveViews(int childCount, int firstActivePosition) {
          final View[] activeViews = mActiveViews;
            for (int i = 0; i < childCount; i++) {
                View child = getChildAt(i);
                AbsListView.LayoutParams lp = (AbsListView.LayoutParams) child.getLayoutParams();
               
                if (lp != null && lp.viewType != ITEM_VIEW_TYPE_HEADER_OR_FOOTER) {
                   
                    activeViews[i] = child;
                  
                    lp.scrappedFromPosition = firstActivePosition + i;
                }
            }
        }
 ```       
  getView调用：
  AbsListView中
  
  
  ```
   View obtainView(int position, boolean[] isScrap) {
     //根据位置到缓存中查找
         final View scrapView = mRecycler.getScrapView(position);
         //这里在调用getView().convertView就不等于null了.就可以进入viewholder模式
        final View child = mAdapter.getView(position, scrapView, this);
        if (scrapView != null) {
        //如果getview返回的view发生了变化，则缓存下来.否则convertView==null了
            if (child != scrapView) {               
                mRecycler.addScrapView(scrapView, position);
            } else {
                isScrap[0] = true;              
                child.dispatchFinishTemporaryDetach();
            }
        }
        return child;
   }
 ```  
   