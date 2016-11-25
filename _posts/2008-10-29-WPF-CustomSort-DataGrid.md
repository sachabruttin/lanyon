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
public class XmlDataGridComparer<T> : IComparer where T : IComparable
{
  private readonly ListSortDirection m_sortDirection;
  private readonly string m_sortMemberPath;
  
  public XmlDataGridComparer(DataGridColumn column)
  {
    m_sortMemberPath = column.GetSortMemberPath();
    m_sortDirection = column.ToggleSortDirection();
  }

  public int Compare(object x, object y)
  {
    XmlElement a = (XmlElement)x;
    XmlElement b = (XmlElement)y;

    if (a == null || b == null)
    {
      throw new ArgumentException();
    }
    
    T da = GetObject(a);
    T db = GetObject(b);
    
    if (m_sortDirection == ListSortDirection.Ascending)
    {
      return da.CompareTo(db);
    }
    else
    {
      return db.CompareTo(da);
    }
  }
  
  private T GetObject(XmlElement element)
  {
    string value = element.GetElementsByTagName(m_sortMemberPath).Item(0).InnerXml;
    return (T)Convert.ChangeType(value, typeof(T));
  }
}
{% endhighlight %}

Ce code utilise deux méthodes d'extension sur les DataGridColumn que voici:

{% highlight cs %}
public static class DataGridColumnExtension
{
  /// <summary>
  /// Gets the sort member path.
  /// </summary>
  /// <param name="column">The sorted column.</param>
  /// <returns>The sort member path.</returns>
  public static string GetSortMemberPath(this DataGridColumn column)
  {
    string sortPropertyName = column.SortMemberPath;

    if (string.IsNullOrEmpty(sortPropertyName))
    {
      DataGridBoundColumn boundColumn = column as DataGridBoundColumn;

      if (boundColumn != null)
      {
        Binding binding = boundColumn.DataFieldBinding as Binding;

        if (binding != null)
        {
          if (!string.IsNullOrEmpty(binding.XPath))
          {
            sortPropertyName = binding.XPath;
          }
          else if (binding.Path != null)
          {
            sortPropertyName = binding.Path.Path;
          }
        }
      }
    }

    return sortPropertyName;
  }

  /// <summary>
  /// Toggles the sort direction.
  /// </summary>
  /// <param name="column">The sorted column.</param>
  /// <returns>
  ///     <see cref="ListSortDirection.Ascending"/> or <see cref="ListSortDirection.Descending"/>
  ///     depending of the previous sort direction.
  /// </returns>
  public static ListSortDirection ToggleSortDirection(this DataGridColumn column)
  {
    ListSortDirection sortDirection = ListSortDirection.Ascending;
    ListSortDirection? currentSortDirection = column.SortDirection;

    if (currentSortDirection.HasValue && currentSortDirection.Value == ListSortDirection.Ascending)
    {
      sortDirection = ListSortDirection.Descending;
    }

    column.SortDirection = sortDirection;
    return sortDirection;
  }
}
{% endhighlight %}

Il ne reste plus qu'à instancier la class XmlDataGridComparer avec le type d'object contenu dans notre colonne dans l'événement Sorting de la DataGrid :

{% highlight cs %}
ListCollectionView lcv = (ListCollectionView)CollectionViewSource.GetDefaultView(dataGrid.ItemsSource);
lcv.CustomSort = new XmlDataGridComparer<string>(e.Column)
{% endhighlight %}

Et voilà...
