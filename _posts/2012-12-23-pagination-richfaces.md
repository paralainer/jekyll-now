---
layout: post
category: tech
title: Server side pagination using Richfaces 4.x
---

To create custom pagination for __rich:dataTable__ you must return instance of __ExtendedDataModel__ class from method that you call in __value__ attribute of __rich:dataTable.__

But overriding for about __8__ methods is too complex. Here is my solution for this. (This is some kind of compilation of methods that I found in internet and my own mind).


This class overrides __ExtendedDataModel__, and have only 3 abstract methods:

{% highlight java %}
public abstract class LazyDataModel<T> extends ExtendedDataModel<T> {
 
    private Integer cachedRowCount;
    private SequenceRange cachedRange;
    private List<T> cachedList;
    private Map<Object, T> cachedMap = new HashMap<Object, T>();
    private Object rowKey;
 
    public abstract List<T> getDataList(int firstRow, int numRows);
 
    public abstract Object getKey(T t);
 
    public abstract int getTotalCount();
 
    @Override
    public void walk(FacesContext ctx, DataVisitor dv, Range range, Object argument) {
        SequenceRange sr = (SequenceRange) range;
        List<T> list = getList(sr);
        for (T t : list) {
            Object key = getKey(t);
            if (key == null) {
                throw new IllegalStateException("found null key");
            }
            cachedMap.put(key, t);
            dv.process(ctx, key, argument);
        }
 
    }
 
    private List<T> getList(SequenceRange sr) {
        //Richfaces for some reason executes walk method several times for same range, lets take data from cache than
        List<T> list;
        if (cachedRange != null && sr.getFirstRow() == cachedRange.getFirstRow() && sr.getRows() == cachedRange.getRows()) {
            list = cachedList;
        } else {
            cachedRange = sr;
            list = getDataList(sr.getFirstRow(), sr.getRows());
            cachedList = list;
        }
        return list;
    }
 
 
    /*
    * Uninque key fro each row, same that used in dv.process
    */
    @Override
    public void setRowKey(Object rowKey) {
        this.rowKey = rowKey;
    }
 
    @Override
    public Object getRowKey() {
        return rowKey;
    }
 
    @Override
    public boolean isRowAvailable() {
        return (getRowData() != null);
    }
 
    @Override
    public int getRowCount() {
        if (cachedRowCount == null) {
            cachedRowCount = getTotalCount();
        }
        return cachedRowCount;
    }
 
    @Override
    public T getRowData() {
        return cachedMap.get(getRowKey());
    }
 
 
    //This methods never called by framework.
    @Override
    public int getRowIndex() {
        throw new UnsupportedOperationException("Not implemented");
    }
 
    @Override
    public void setRowIndex(int rowIndex) {
        throw new UnsupportedOperationException("Not implemented");
    }
 
    @Override
    public Object getWrappedData() {
        throw new UnsupportedOperationException("Not implemented");
    }
 
    @Override
    public void setWrappedData(Object data) {
        throw new UnsupportedOperationException("Not implemented");
 
    }
}
{% endhighlight %}

Here is an example (using some abstract _goods_ for example):

Goods bean:
{% highlight java %}

@ManagedBean
@ViewScoped
public class GoodsBean {
 
    private GoodsModel goodsModel;
 
    private String filterName = "";
 
    public String getFilterName() {
        return filterName;
    }
 
    public void setFilterName(String filterName) {
        boolean changed = !this.filterName.equals(filterName);
        this.filterName = filterName;
        if (changed){
            initGoodsModel();
        }
    }
 
    public GoodsBean() {
        initGoodsModel();
    }
 
    public GoodsModel getGoods() {
        return goodsModel;
    }
 
    private void initGoodsModel() {
        goodsModel = new GoodsModel(filterName);
    }
}
{% endhighlight %}

Goods data model:
{% highlight java %}
public class GoodsModel extends LazyDataModel<Good> {
 
    private String name;
 
    public GoodsModel(String name) {
        this.name = name;
    }
 
    @Override
    public List<Good> getDataList(int offset, int pageSize) {
        return GoodDao.findGoods(name, offset, pageSize);
    }
 
    @Override
    public Object getKey(Good good) {
        return good.getId();
    }
 
    @Override
    public int getTotalCount() {
        return GoodDao.goodsCount(name);
    }
}
{% endhighlight %}

Part of xhtml page:
{% highlight xml %}
<a4j:queue requestDelay="500" ignoreDupResponses="true"/>
<rich:dataTable id="goodsTable" value="#{GoodsBean.goods}" var="good" rows="20">
     <rich:column>
         <f:facet name="header">
             <h:outputText value="ID"/>
         </f:facet>
         <h:outputText value="#{good.id}"/>
     </rich:column>
     <rich:column>
         <f:facet name="header">
           <h:panelGroup>
               <h:outputText value="Name"/>
               <br/>
               <h:inputText value="#{GoodsBean.filterName}" id="input">
                   <a4j:ajax event="keyup" render="goodsTable">
                       <a4j:attachQueue/>
                   </a4j:ajax>
               </h:inputText>
           </h:panelGroup>
         </f:facet>
         <h:outputText value="#{good.name}"/>
     </rich:column>
     <f:facet name="footer">
         <rich:dataScroller />
     </f:facet>
 </rich:dataTable>
{% endhighlight %}

Now we have goods table with server side pagination and filtering by name.