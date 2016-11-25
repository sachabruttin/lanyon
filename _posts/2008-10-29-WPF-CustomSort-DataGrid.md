---
layout: post
title: WPF - Custom sort dans la DataGrid
tags:
 - WPF
---

Ce billet, <http://blogs.msdn.com/jgoldb/archive/2008/08/26/improving-microsoft-datagrid-ctp-sorting-performance.aspx>, nous explique 
comment obtenir de meilleures performances lors du tri de la [WPF DataGrid](http://www.codeplex.com/wpf).

Voici une variante qui vous permettra de trier une datagrid liée à un XmlDataProvider.

{% highlight cs %}
public class XmlDataGridComparer<T> : IComparer where T : IComparable{    private readonly ListSortDirection m_sortDirection;    private readonly string m_sortMemberPath;     public XmlDataGridComparer(DataGridColumn column)    {        m_sortMemberPath = column.GetSortMemberPath();        m_sortDirection = column.ToggleSortDirection();    }     public int Compare(object x, object y)    {        XmlElement a = (XmlElement)x;        XmlElement b = (XmlElement)y;         if (a == null || b == null)        {            throw new ArgumentException();        }         T da = GetObject(a);        T db = GetObject(b);         if (m_sortDirection == ListSortDirection.Ascending)        {            return da.CompareTo(db);        }        else        {            return db.CompareTo(da);        }    }     private T GetObject(XmlElement element)    {        string value = element.GetElementsByTagName(m_sortMemberPath).Item(0).InnerXml;        return (T)Convert.ChangeType(value, typeof(T));    }}
{% endhighlight %}
