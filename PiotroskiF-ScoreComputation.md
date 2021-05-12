## Blazor server-side

# Installation

[Index](Documentation.md)

The **GridBlazor** component installation is straightforward. Just follow these steps:

1. Create a new Blazor server-side solution using the **Blazor Server App** template

2. Install [**GridBlazor**](http://nuget.org/packages/GridBlazor/) and [**GridMvcCore**](http://nuget.org/packages/GridMvcCore/) nuget packages on the project.

3. Add the following lines to the **_Host.cshtml** view or directly to the page:
    ```html
        <link href="_content/GridBlazor/css/gridblazor.min.css" rel="stylesheet" />
        <script src="_content/GridBlazor/js/gridblazor.js"></script>
    ```
    These files will be loaded from the **GridBlazor** nuget package, so it is not necessary to copy it to you project.
 
4. If you are using Boostrap 3.x you will also need this line in the **_Host.cshtml** view or directly to the page:
    ```html
        <link href="~/_content/GridBlazor/css/gridblazor-bootstrap3.min.css" rel="stylesheet" />
     ```
 
## Blazor server-side

# Quick start with GridBlazor

[Index](Documentation.md)

Imagine that you have to retrieve a collection of model items in your project. For example if your model class is:
    
```c#
    public class Order
    {
        [Key]
        public int OrderID { get; set; }
        public string CustomerID { get; set; }
        public DateTime? OrderDate { get; set; }
        public virtual Customer Customer { get; set; }
        ...
    }
```

The steps to build a grid razor page using **GridBlazor** are:

1. Create a service with a method to get all items for the grid. An example of this type of service is: 

    ```c#
        public class OrderService
        {
            ...

            public ItemsDTO<Order> GetOrdersGridRows(Action<IGridColumnCollection<Order>> columns,
                    QueryDictionary<StringValues> query)
            {
                var repository = new OrdersRepository(_context);
                var server = new GridServer<Order>(repository.GetAll(), new QueryCollection(query), 
                    true, "ordersGrid", columns, 10);
            
                // return items to displays
                return server.ItemsToDisplay;
            }
        }
    ```

    **Notes**:
    * The method must have 2 parameters:
        * the first one is a lambda expression with the column definition of type **Action<IGridColumnCollection<T>>**
        * the second one is a dictionary to pass query parameters such as **grid-page**. It must be ot type **QueryDictionary<StringValues>**

    * You can use multiple methods of the **GridServer** object to configure a grid on the server. For example:
        ```c#
            var server = new GridServer<Order>(repository.GetAll(), Request.Query, true, "ordersGrid", columns, 10)
                .Sortable()
                .Filterable()
                .WithMultipleFilters();
        ```
    * The method returns an object including the model rows to be shown on the grid and other information requirired for paging, etc. The object type returned by the action must be **ItemsDTO<T>**.

2. You have to register the service in the **Startup** class:

    ```c#
        public void ConfigureServices(IServiceCollection services)
        {
            ...

            services.AddSingleton<OrderService>();
            
            ...
        }
    ```

3. Add a reference to **GridBlazor**, **GridBlazor.Pages**, **GridShared** and **GridShared.Utility** in the **_Imports.razor** file of the root folder

    ```razor
        ...
        @using GridBlazor
        @using GridBlazor.Pages
        @using GridShared
        @using GridShared.Utility
        ...
    ```

4. Create a razor page to render the grid. The page file must have a .razor extension. An example of razor page is:

    ```razor
        @page "/gridsample"
        @using Microsoft.Extensions.Primitives
        @inject OrderService orderService

        @if (_task.IsCompleted)
        {
            <GridComponent T="Order" Grid="@_grid"></GridComponent>
        }
        else
        {
            <p><em>Loading...</em></p>
        }

        @code
        {
            private CGrid<Order> _grid;
            private Task _task;

            protected override async Task OnParametersSetAsync()
            {
                Action<IGridColumnCollection<Order>> columns = c =>
                {
                    c.Add(o => o.OrderID);
                    c.Add(o => o.OrderDate, "OrderCustomDate").Format("{0:yyyy-MM-dd}");
                    c.Add(o => o.Customer.CompanyName);
                    c.Add(o => o.Customer.IsVip);
                };

                var query = new QueryDictionary<StringValues>();
                query.Add("grid-page", "2");

                var client = new GridClient<Order>(q => orderService.GetOrdersGridRows(columns, q), query, false, "ordersGrid", columns);
                _grid = client.Grid;

                // Set new items to grid
                _task = client.UpdateGrid();
                await _task;
            }
        }
    ```

    **Notes**:
    * You must create a **GridClient** object in the **OnParametersSetAsync** of the razor page. This object contains a parameter of **CGrid** type called **Grid**. 

    * You can use multiple methods of the **GridClient** object to configure a grid. For example:
        ```c#
            var client = new GridClient<Order>(q => orderService.GetOrdersGridRows(columns, q), query, false, "ordersGrid", columns)
                .SetRowCssClasses(item => item.Customer.IsVip ? "success" : string.Empty)
                .Sortable()
                .Filterable()
                .WithMultipleFilters();
        ```

    * The **GridClient** object used on the razor page and the **GridServer** object on the service must have compatible settings.

    * You must call the **UpdateGrid** method of the **GridClient** object at the end of the **OnParametersSetAsync** of the razor page because it will request for the required rows to the server

    * If you need to update the component out of ```OnParametersSetAsync``` method you must use a reference to the component:
        ```c#
            <GridComponent @ref="Component" T="Order" Grid="@_grid"></GridComponent>
        ```

        and then call the ```UpdateGrid``` method:
        ```c#
            await Component.UpdateGrid();
        ```

    * The **GridComponent** tag must contain at least these 2 attributes:
        * **T**: type of the model items
        * **Grid**: grid object that has to be created in the **OnParametersSetAsync** method of the razor page

For more documentation about column options, please see: [Custom columns](Custom_columns.md).

## Blazor server-side

# GridBlazor configuration

[Index](Documentation.md) 

You can configure the settings of the grid with the parameters and methods of the **GridComponent**, **GridClient** and **GridServer** objects. Remember that they must have compatible settings.

## GridComponent parameters

Parameter | Type | Description | Example
--------- | ---- | ----------- | -------
T | ```Type``` (mandatory) | type of the model items | ```<GridComponent T="Order" Grid="@_grid" />```
Grid | ```ICGrid``` (mandatory) | grid object that has to be created in the ```OnParametersSetAsync``` method of the Blazor page | ```<GridComponent T="Order" Grid="@_grid" />```
OnRowClicked | ```Action<object>``` (optional) |  to be executed when selecting a row on "selectable" grids | ```<GridComponent T="Order" Grid="@_grid" OnRowClicked="@OrderDetails" />```
CustomFilters | ```IQueryDictionary<Type>``` (optional) | Dictionary containing all types of custom filter widgets used on the grid  | ```<GridComponent T="Order" Grid="@_grid" CustomFilters="@_customFilters" />```
GridMvcCssClass | ```string``` (optional) | Html classes used by the parent grid element (it overrides default parameter) | ```<GridComponent T="Order" Grid="@_grid" GridMvcCssClass="grid-mvc-alt" />```
GridWrapCssClass | ```string``` (optional) | Html classes used by the wrap element (it overrides default parameter) | ```<GridComponent T="Order" Grid="@_grid" GridWrapCssClass ="grid-wrap-alt" />```
GridFooterCssClass | ```string``` (optional) | Html classes used by the footer element (it overrides default parameter) | ```<GridComponent T="Order" Grid="@_grid" GridFooterCssClass="grid-footer-alt" />```
GridItemsCountCssClass | ```string``` (optional) | Html classes used by the items count element (it overrides default parameter) | ```<GridComponent T="Order" Grid="@_grid" GridItemsCountCssClass="grid-items-count-alt" />```
TableCssClass | ```string``` (optional) | Html classes used by the table element (it overrides default parameter) | ```<GridComponent T="Order" Grid="@_grid" TableCssClass="grid-table-alt" />```
TableWrapCssClass | ```string``` (optional) | Html classes used by the parent div of the table element (it overrides default parameter) | ```<GridComponent T="Order" Grid="@_grid" TableWrapCssClass="table-wrap-alt" />```
GridHeaderCssClass | ```string``` (optional) | Html classes used by the table header element (it overrides default parameter) | ```<GridComponent T="Order" Grid="@_grid" GridHeaderCssClass="grid-header-alt" />```
GridCellCssClass | ```string``` (optional) | Html classes used by the cell elements (it overrides default parameter) | ```<GridComponent T="Order" Grid="@_grid" GridCellCssClass="grid-cell-alt" />```
GridButtonCellCssClass | ```string``` (optional) | Html classes used by the button elements of CRUD grids (it overrides default parameter) | ```<GridComponent T="Order" Grid="@_grid" GridButtonCellCssClass="grid-button-cell-alt" />```
GridSubGridCssClass | ```string``` (optional) | Html classes used by the subgrid elements (it overrides default parameter) | ```<GridComponent T="Order" Grid="@_grid" GridSubGridCssClass="grid-subgrid-alt" />```
GridEmptyTextCssClass | ```string``` (optional) | Html classes used by the empty cell elements (it overrides default parameter) | ```<GridComponent T="Order" Grid="@_grid" GridEmptyTextCssClass="grid-empty-text-alt" />```

## GridClient parameters

Parameter | Description | Example
--------- | ----------- | -------
dataService | lamba expresion of the service method returning rows | q => orderService.GetOrdersGridRows(columns, q)
query | dictionary containing grid parameters as the initial page | query.Add("grid-page", "2");
renderOnlyRows | boolean to configure if only rows are renderend by the server or all the grid object | Must be **false** for Blazor solutions
gridName | string containing the grid client name  | ordersGrid
columns | lambda expression to define the columns included in the grid (**Optional**) | c => { c.Add(o => o.OrderID); c.Add(o => o.Customer.CompanyName); };
cultureInfo | **CultureInfo** class to define the language of the grid (**Optional**) | new CultureInfo("de-DE");

## GridClient methods

Method name | Description | Example
----------- | ----------- | -------
AutoGenerateColumns | Generates columns for all properties of the model using data annotations | GridClient<Order>(...).AutoGenerateColumns();
Sortable | Enable or disable sorting for all columns of the grid | GridClient<Order>(...).Sortable(true);
Searchable | Enable or disable searching on the grid | GridClient<Order>(...).Searchable(true, true);
Filterable | Enable or disable filtering for all columns of the grid | GridClient<Order>(...).Filterable(true);
WithMultipleFilters | Allow grid to use multiple filters | GridClient<Order>(...).WithMultipleFilters();
ClearFiltersButton | Enable or disable the ClearFilters button | GridClient<Order>(...).ClearFiltersButton(true);
Selectable | Enable or disable the client grid items selectable feature | GridClient<Order>(...).Selectable(true, true);
SetStriped | Enable or disable the grid as a striped one | GridClient<Order>(...).SetStriped(true);
SetKeyboard | Enable or disable the keyboard navigation | GridClient<Order>(...).SetKeyboard(true);
SetModifierKey | Configure the modifier key for keyboard navigation | GridClient<Order>(...).SetModifierKey(ModifierKey.ShiftKey);
EmptyText | Setup the text displayed for all empty items in the grid | GridClient<Order>(...).EmptyText(' - ');
WithGridItemsCount | Allows the grid to show items count | GridClient<Order>(...).WithGridItemsCount();
SetRowCssClasses | Setup specific row css classes | GridClient<Order>(...).SetRowCssClasses(item => item.Customer.IsVip ? "success" : string.Empty);
AddToOnAfterRender | Add a Func<GridComponent<T>, bool, Task> to be executed at the end of the ```OnAfterRenderAsync``` method of the grid component | GridClient<Order>(...).AddToOnAfterRender(OnAfterDepartmentRender);
SetDirection | Allows the grid to be show in right to left direction | GridClient<Order>(...).SetDirection(GridDirection.RTL);
HandleServerErrors | Allows errors from the server to be handled by grid client | GridClient<Order>(...).HandleServerErrors(true, false);
SetTableLayout | Configure fixed dimensions for the grid | GridClient<Order>(...).SetTableLayout(TableLayout.Fixed, "1200px", "400px");

## GridServer parameters

Parameter | Description | Example
--------- | ----------- | -------
items | **IEnumerable<T>** object containing all the grid rows | repository.GetAll()
query | **IQueryCollection** containing all grid parameters | Must be the **Request.Query** of the controller
renderOnlyRows | boolean to configure if only rows are renderend by the server or all the grid object | Must be **true** for Blazor solutions
gridName | string containing the grid client name  | ordersGrid
columns | lambda expression to define the columns included in the grid (**Optional**) | **Columns** lamba expression defined in the razor page of the example
pageSize | integer to define the number of rows returned by the web service (**Optional**) | 10

## GridServer methods

Method name | Description | Example
----------- | ----------- | -------
AutoGenerateColumns | Generates columns for all properties of the model using data annotations | GridServer<Order>(...).AutoGenerateColumns();
Sortable | Enable or disable sorting for all columns of the grid | GridServer<Order>(...).Sortable(true);
Searchable | Enable or disable searching on the grid | GridServer<Order>(...).Searchable(true, true);
Filterable | Enable or disable filtering for all columns of the grid | GridServer<Order>(...).Filterable(true);
WithMultipleFilters | Allow grid to use multiple filters | GridServer<Order>(...).WithMultipleFilters();
WithGridItemsCount | Allows the grid to show items count | GridServer<Order>(...).WithGridItemsCount();


## Blazor server-side

# Keyboard navigation

[Index](Documentation.md)

Users can enable keyboard navigation between pages using the ```SetKeyboard``` method of the ```GridClient``` object:

```c#
    var client = new GridClient<Order>(q => orderService.GetOrdersGridRows(columns, q), query, false, "ordersGrid", columns, locale)
        .SetKeyboard(true);
```

The default value is ```false```.

These are the keys to be used:

- [Ctrl] + [Left] and [Ctrl] + [Right] arrows navigate between pages
- [Ctrl] + [Home] key goes to the first page
- [Ctrl] + [End] key goes to the last page
- [Ctrl] + [Up] and [Ctrl] + [Down] arrows navigate from one row to another for grids where rows are selectable. It doesn´t work when multiselectable is enabled. 
- [Tab] key navigates among elements of a filter widget when it is visible
- [Esc] key minimises a filter widget when it is visible
- [Ctrl] +[Backspace] clear all filters

It's possible to change the modifier key used for keyboard navigation using the ```SetModifierKey``` method of the ```GridClient``` object:

```c#
    var client = new GridClient<Order>(q => orderService.GetOrdersGridRows(columns, q), query, false, "ordersGrid", columns, locale)
        .SetKeyboard(true).SetModifierKey(ModifierKey.ShiftKey);
```

The parameter options of the ```SetModifierKey``` method are:
- ```ModifierKey.CtrlKey``` (default value)
- ```ModifierKey.ShiftKey```
- ```ModifierKey.AltKey```
- ```ModifierKey.MetaKey```

Keep in mind that the last 2 options can collide with the modifier keys of the browser. The recommended options are ```ModifierKey.CtrlKey``` and ```ModifierKey.ShiftKey```.

## Blazor server-side

# Paging

[Index](Documentation.md)

You must configure the page size in the contructor of the **GridServer** object in the service method returning the rows to enable paging:

```c#
    var server = new GridServer<Order>(repository.GetAll(), Request.Query,
                true, "ordersGrid", columns, 10);
```

**PageSize** is an optional parameter of the **GridServer** constructor. If you don't want to enable paging you must call the contructor with no parameter:

```c#
    var server = new GridServer<Order>(repository.GetAll(), Request.Query,
                true, "ordersGrid", columns);
```

No configuration for the static page size is required on the **GridClient** object

## Change page size on the Razor page

You can configure the grid to enable changes on page size from the Razor page.

You have to enable paging as shown in the section before.

Then you have to use the **ChangePageSize** method of the **GridClient** object on the Razor page:

```c#
    var client = new GridClient<Order>(q => orderService.GetOrdersGridRows(ColumnCollections.OrderColumns, q),
            query, false, "ordersGrid", ColumnCollections.OrderColumns, locale)
            .ChangePageSize(true);
```

A user can change the page size writing the new size and pressing the "Tab" or "Enter" keys.

## Blazor server-side

# Custom columns

[Index](Documentation.md)

All customization for columns must be done in the razor page:

* You can create a custom column by calling the **Columns.Add** method in the **IGridColumnCollection** interface. For example:

    ```c#
        Columns.Add(o => o.Customers.CompanyName);
    ```

* The **Titled** method of the **Column** object defines the column header text. If you don't call this method, **GridBlazor** will use the name of the property by default (in this case **CompanyName**).

    ```c#
        Columns.Add(o => o.Customers.CompanyName)
                .Titled("Company Name")
    ```

* If you want to add a column in a specified grid position, you can use the **Insert** method :

    ```c#
        Columns.Insert(0, o => o.Customers.CompanyName)
                .Titled("Company Name")
    ```

* You can construct a custom display value of the column using the **RenderValueAs** method:

    ```c#
        Columns.Add(o => o.Employees.LastName)
                .RenderValueAs(o => o.Employees.FirstName + " " + o.Employees.LastName)
    ```

* You can build **html** content in the column using the **RenderValueAs**, **Encoded** and **Sanitized** methods:

    ```c#
         c.Add(o => o.OrderID).Encoded(false).Sanitized(false)
        	.RenderValueAs(o => $"<b><a class='modal_link' href='/Home/Edit/{o.OrderID}'>Edit</a></b>");
    ```

* You can apply css classes to all cells of a column:

    ```c#
        Columns.Add(o => o.Employees.LastName).Css("hidden-xs")
    ```

* You can apply css classes to selected cells of a column:

    ```c#
        Columns.Add(o => o.OrderDate)
            .SetCellCssClassesContraint(o => o.OrderDate.HasValue && o.OrderDate.Value >= DateTime.Parse("1997-01-01") ? "red" : "")
    ```

* As a general rule you can concatenate method calls of the **Column** object to configure each column. For example:

    ```c#
        Columns.Add(o => o.Customers.CompanyName)
                .Titled("Company Name")
                .Sortable(true)
                .SetWidth(220);
    ```

## Component columns

Sometimes you need to add a column that renders a component. In this case you must use an empty contructor of the **Add** method to create a column and its **RenderComponentAs** method to define the content that will be rendered in it:

```c#
    Columns.Add().RenderComponentAs(typeof(ButtonCell));
```

The parameter of the **RenderComponentAs** method must be the **Type** of the Blazor component used for this column.
You must also create a Blazor component that includes only a parameter called **Item** of the same type of the grid row element.
The component can include any html elements as well as event handling features.

This is an example of this type of component:

```razor
    <button class='btn btn-sm btn-primary' @onclick="@MyClickHandler">Save</button>

    @code {
        [Parameter]
        protected Order Item { get; set; }

        private void MyClickHandler(UIMouseEventArgs e)
        {
            Console.WriteLine("Button clicked: Item " + Item.OrderID);
        }
    }
```

Remember that sorting and filtering will not work on component columns.

## Not connected columns

Sometimes you need to add a column that renders some content, but there is no base model property. In this case you must use an empty contructor of the **Add** method to create a column and its **RenderValueAs** method to define the content that will be rendered in it:

```c#
    Columns.Add().RenderValueAs(model => "Item " + model.Id);
```
Remember that sorting and filtering will not work on not connected columns.

## Hidden columns

All columns added to the grid are visible by default. If you want that a column will not appear on the screen you have to set it up in the **Add** method call:

```c#
    Columns.Add(o => o.Id, true);
```
The second parameter of the **Add** method defines if the column will be hidden or not. 
In this example you will not see the **Id** column, but you can get values at the client side using javascript. 
Important: you can't sort hidden columns.

## Sorting

If you want to enable sorting just for one column, you just call the **Sortable(true)** method of that column:

```c#
    Columns.Add(o => o.Employees.LastName)
                .Titled("Employee")
                .Sortable(true);
```

Sorting will be implemented for the field that you specify the **Add** method, in this example for the **o.Employees.LastName** field.

If you pass an ordered collection of items to the Grid constructor and you want to display this by default, you have to specify the initial sorting options:

```c#
    Columns.Add(o => o.OrderDate)
                .Titled("Date")
                .Sortable(true)
                .SortInitialDirection(GridSortDirection.Descending);
```

Remember that you can also enable [Sorting](Sorting.md) for all columns of a grid. Sorting at grid level has precendence over sorting defined at column level.

## Sorting with a custom comparer

You can enable sorting using a custom comparer. This feature only works for grids where all items are in memory. 
Grids where items are in a database and EF Core is used to get them don't support custom comparers and will throw an exception if the are configured in this way.

If you want to use a custom comparer for a column yo can do it by calling the **Columns.Add** method in the **IGridColumnCollection** interface using an ```IComparer<TKey>``` as a parameter:
```c#
    var comparer = new ....;

    .... 

    c.Add(o => o.Customer.CompanyName, comparer)
```

## Header tooltip

You can enable a tooltip for a column header. The tooltip will appear when the pointer hover on the column header of the grid. 

If the grid is configured for CRUD operations, the tooltip will also appear hovering over the field label of CRUD forms. 

If you want to use a tooltip for a column yo can do it by calling the **SetTooltip** method in the column definition:
```c#
    c.Add(o => o.OrderID).SetTooltip("Order ID is ... ");
```

## Column settings

Method name | Description | Example
------------- | ----------- | -------
Titled | Setup the column header text | Columns.Add(x=>x.Name).Titled("Name of product");
Encoded | Enable or disable encoding column values | Columns.Add(x=>x.Name).Encoded(false);
Sanitized | If encoding is disabled sanitize column value from XSS attacks | Columns.Add(x=>x.Name).Encoded(false).Sanitize(false);
SetWidth | Setup width of current column | Columns.Add(x=>x.Name).SetWidth("30%");
RenderComponentAs | Setup delegate to render a component column | Columns.Add().RenderComponentAs(typeof(ButtonCell));
RenderValueAs | Setup delegate to render column values | Columns.Add(x=>x.Name).RenderValueAs(o => o.Employees.FirstName + " " + o.Employees.LastName);
Sortable | Enable or disable sorting for current column | Columns.Add(x=>x.Name).Sortable(true);
SortInitialDirection | Setup the initial sort deirection of the column (need to enable sorting) | Columns.Add(x=>x.Name).Sortable(true).SortInitialDirection(GridSortDirection.Descending);
SetInitialFilter | Setup the initial filter of the column | Columns.Add(x=>x.Name).Filterable(true).SetInitialFilter(GridFilterType.Equals, "some name");
ThenSortBy | Setup ThenBy sorting of current column | Columns.Add(x=>x.Name).Sortable(true).ThenSortBy(x=>x.Date);
ThenSortByDescending | Setup ThenByDescending sorting of current column | Columns.Add(x=>x.Name).Sortable(true).ThenSortBy(x=>x.Date).ThenSortByDescending(x=>x.Description);
Filterable | Enable or disable filtering feauture on the column | Columns.Add(x=>x.Name).Filterable(true);
SetFilterWidgetType | Setup filter widget type for rendering custom filtering user interface | Columns.Add(x=>x.Name).Filterable(true).SetFilterWidgetType("MyFilter");
Format | Format column value | Columns.Add(o => o.OrderDate).Format("{0:dd/MM/yyyy}");
Css | Apply css classes to the column | Columns.Add(x => x.Number).Css("hidden-xs");
SetCellCssClassesContraint | Apply css classes to selected cells | Columns.Add(x => x.Number).SetCellCssClassesContraint(x => x.Number < 0 ? "red" : "black");
SetTooltip | Enable column header tooltip | Columns.Add(x => x.Number).SetTooltip("Number column is ... ");

## Blazor server-side

# Totals

[Index](Documentation.md)

You can enable the totals option for each column of your grid.

![](../images/Totals.png)

You can enable searching for each columns of a grid using the **Sum**, **Average**, **Max** and/or **Min** methods for the **Column** object:

```c#
    Columns.Add(o => o.Freight).Titled("Freight").Sum(true).Average(true);
```

* **Sum** method works only for number columns
* **Average** method works only for number columns
* **Max** method works for number, date-time and text columns
* **Min** method works for number, date-time and text columns

## Methods

Method | Parameter | Description | Example
------ | --------- | ----------- | -------
Sum |enable | bool to enable sum calculation on column | Sum(true)
Average |enable | bool to enable average calculation on column | Average(true)
Max |enable | bool to enable maximum calculation on column | Max(true)
Min |enable | bool to enable minimum calculation on column | Min(true)

## Blazor server-side

# Sorting

[Index](Documentation.md)

## Regular Sorting
You can enable sorting for all columns of a grid using the **Sortable** method for both **GridClient** and **GridServer** objects:
* razor page
    ```c#
        var client = new GridClient<Order>(q => orderService.GetOrdersGridRows(columns, q), query, false, "ordersGrid", Columns, locale)
            .Sortable()
    ```

* service method
    ```c#
        var server = new GridServer<Order>(repository.GetAll(), Request.Query, true, "ordersGrid", columns, 10)
            .Sortable()
    ```

In this case you can select sorting pressing the column name on just one column at a time.

Sorting at grid level has precendence over sorting defined at column level.


## Extended Sorting
You can also configure extended sorting using the **ExtSortable** method for both **GridClient** and **GridServer** objects:
* razor page
    ```c#
        var client = new GridClient<Order>(q => orderService.GetOrdersGridRows(columns, q), query, false, "ordersGrid", Columns, locale)
            .ExtSortable()
    ```

* service method
    ```c#
        var server = new GridServer<Order>(repository.GetAll(), Request.Query, true, "ordersGrid", columns, 10)
            .ExtSortable()
    ```
In this case you can drag the column title and drop it on the sorting area. You can add multiple columns at the same time and select if sorting is ascending or descending column by column.

This is an example of a table of items using extended sorting:

![](../images/Extended_sorting.png)


## Blazor server-side

# Grouping

[Index](Documentation.md)

You can enable grouping for all columns of a grid using the **Groupable** method for both **GridClient** and **GridServer** objects:
* razor page
    ```c#
        var client = new GridClient<Order>(q => orderService.GetOrdersGridRows(columns, q), query, false, "ordersGrid", Columns, locale)
            .Groupable()
    ```

* service method
    ```c#
        var server = new GridServer<Order>(repository.GetAll(), Request.Query, true, "ordersGrid", columns, 10)
            .Groupable()
    ```

You can drag the column title and drop it on the sorting area. 
You can add multiple columns at the same time and select if sorting is ascending or descending column by column.
The grid items will be grouped by the values of the selected columns.

This is an example of a table of items using grouping:

![](../images/Grouping.png)


## Blazor server-side

# Selecting row

[Index](Documentation.md)

There are 2 ways to configure selecting rows:
- using the ```Selectable``` method of the ```GridClient``` object
- using the ```SetCheckboxColumn``` or ```SetSingleCheckboxColumn``` methods on the column definition

## Selectable method

The ```GridClient``` object has a method called ```Selectable``` to configure if a row can be selected. 
It's value can be ```true``` and ```false```. 
Since the version 1.1.0 of the GridBlazor nuget package the default value of the ```Selectable``` feature is ```false``` (it was ```true``` for earlier versions).

There are optional parameters to control selection behavior:

- Auto Select First Row:
    There is an optional boolean parameter to control if the first row should automatically be selected when a page loads.
    It's value can be ```true``` and ```false```. 
    By default this parameter's value is ```false```. 
- Allow Multi Select:
    There is an optional boolean paramter to control if multiple rows can be selected. 
    It's value can be ```true``` and ```false```.
    By default this parameter's value is ```false```.
    You can select multiple rows while pressing the [Ctrl] key
    You select multiple adjacent rows by clicking on the first row holding down the [Shift] key and then clicking on the last row of the interval keeping the [Shift] key held down

Rows configured in this way will be highlighted when selected.

You can enable it as follows:
```c#
    var client = new GridClient<Order>(q => orderService.GetOrdersGridRows(columns, q), query, false, "ordersGrid", columns)
        .Selectable(true, true, true);
```

You have to add the ```OnRowClicked``` attribute on the component. For example, the razor page can contain the following line:
```razor
    <GridComponent T="Order" Grid="@_grid" OnRowClicked="@OrderDetails"></GridComponent>
```

Then you have to add the function called by the event. In this example its name is ```OrderDetails```:
```c#
    protected void OrderDetails(object item)
    {
        Order order = null;
        if (item.GetType() == typeof(Order))
        {
            order = (Order)item;
        }
        Console.WriteLine("Order Id: " + (order == null ? "NULL" : order.OrderID.ToString()));
    }
```

In this sample a line of text with the selected row id is writen on the console log.

When items are selected in grid, collection of selected items are available using SelectedItems property. SelectedItems property is of type IEnumerable<object>.

```c#
    var items = client.Grid.SelectedItems.ToList<T>();
```

In the GridComponent.Demo project you will find another example where the order details are shown on a component when a row is selected.

### Selecting rows for subgrids

GridBlazor 1.3.30 and newer versions implement ```OnRowClickedActions``` to allow row click for all subgrids.

You have to create a ```List<Action<object>>()``` and add all row click method for nested subgrids in the same order they are nested:

```c#
    _rowClickActions = new List<Action<object>>();
    _rowClickActions.Add(OrderInfo);
    _rowClickActions.Add(OrderDetailInfo);
```

And finally you have to pass the list as parameter of the ```GridComponent```:

```c#
    <GridComponent T="Order" Grid="@_grid" OnRowClickedActions="@_rowClickActions" />
```

This is an example of grid using ```Selectable```:

![](../images/Selectable.png)



## SetCheckboxColumn method

You can add one or more columns with checkboxes on each row.

```c#
    c.Add("CheckboxColumn").SetCheckboxColumn(true, o => o.Customer.IsVip);
    c.Add(o => o.OrderID).SetPrimaryKey(true);
```

Columns defined in this way must be not connected ones (defined with ```Add()``` method). But they can have a name (defined with ```Add("columnName")``` method).

It's also mandatory to identify the columns that are primary keys for the grid. You must do it using the ```SetPrimaryKey(true)``` method for the primary key columns' definitions.

```SetCheckboxColumn``` method has 3 parameters:
- headerCheckbox: it's a boolean value to enable the checkbox on the header
- expression: it's a ```Func<T, bool>``` to define the initial value of the checkbox for each row
- readonlyExpr (optional): it's a ```Func<T, bool>``` to configure the checkbox for each row as read only

**Important:** ```CheckedRows``` property is not available since release 1.6.2. ```CheckedRows``` only allowed to retrieve the checked values, but not to change them. Use the ```Checkboxes``` property instead of it.

If you want to retrieve or change the checked values for each row, you can use the ```Checkboxes``` property of the ```GridComponent``` object. 
It is a dictionary that contains references to all checkbox components for each column.

Row IDs are the keys of this dictionary. Rows are numbered starting by 0.

The ```CheckboxComponent<T>``` object contains 2 methods to get and set the checked value:
- IsChecked(): It retrieves the current value
- SetChecked(bool): It changes the checkbox value

This is an example showing how to access both methods:
```c#
    private GridComponent<Order> _gridComponent;
    
    ...
    
    Dictionary<int, CheckboxComponent<Order>> checkboxes = _gridComponent.Checkboxes.Get("CheckboxColumn");
    bool isChecked = checkboxes[0].IsChecked();
    if (isChecked)
        await checkboxes[0].SetChecked(false);
```

Blazor pages using checkboxes may require to use the ```ShouldRender``` method to suppress UI refreshing. See this [sample](https://github.com/gustavnavar/Grid.Blazor/blob/master/GridBlazorServerSide/Pages/Checkbox.razor) as reference.

These events are provided by the ```GridComponent``` object to allow running tasks on changing checkboxes:
- ```Func<CheckboxEventArgs<T>, Task> HeaderCheckboxChanged```: it's fired when a header checkbox is changed
- ```Func<CheckboxEventArgs<T>, Task> RowCheckboxChanged```: it's fired when a row checkbox is changed

This is an example of grid using ```SetCheckboxColumn```:

![](../images/Checkbox_column.png)

## SetSingleCheckboxColumn method

This case is very similar to the previous one, with the exception of only one checkbox selected at a time. 
When you select a checkbox, any other checkboxes are automatically unchecked.

You can add one or more columns with checkboxes on each row.

```c#
    c.Add("CheckboxColumn").SetSingleCheckboxColumn();
    c.Add(o => o.OrderID).SetPrimaryKey(true);
```
Columns defined in this way must be not connected ones (defined with ```Add()``` method). But they can have a name (defined with ```Add("columnName")``` method).

It's also mandatory to identify the columns that are primary keys for the grid. You must do it using the ```SetPrimaryKey(true)``` method for the primary key columns' definitions.

```SetSingleCheckboxColumn``` method has no parameters.

## Blazor server-side

# Searching

[Index](Documentation.md)

You can enable the searching option for your grid. Searching allows to search for a text on all columns at the same time.

![](../images/Searching.png)

You can enable searching for all columns of a grid using the **Searchable** method for both **GridClient** and **GridServer** objects:

* razor page
    ```c#
        var client = new GridClient<Order>(q => orderService.GetOrdersGridRows(columns, q), query, false, "ordersGrid", Columns, locale)
            .Searchable(true, false, true)
    ```

* service method
    ```c#
        var server = new GridServer<Order>(repository.GetAll(), Request.Query, true, "ordersGrid", columns, 10)
            .Searchable(true, false, true)
    ```


## Searching parameters

Parameter | Description | Example
--------- | ----------- | -------
enable (optional) | bool to enable searching on the grid | Searchable(true, ...)
onlyTextColumns (optional) | bool to enable searching on all collumns or just on string ones | Searchable(..., true, ...)
hiddenColumns (optional) | bool to enable searching on hidden columns | Searchable(..., true)

```enable``` default value is ```true```, ```onlyTextColumns``` default value is ```true```, and ```hiddenColumns``` default value is ```false```.

Searching on boolean columns has benn disabled because EF Core 3.0 is not supporting it yet.

## Blazor server-side

# Filtering

[Index](Documentation.md)

You can enable the filtering option for your columns. To enable this functionality use the **Filterable** method of the **Column** object:

```c#
    Columns.Add(o => o.Customers.CompanyName)
        .Titled("Company Name")
        .Filterable(true)
        .Width(220)
```
After that you can filter this column. 

You can enable filtering for all columns of a grid using the **Filterable** method for both **GridClient** and **GridServer** objects:

* razor page
    ```c#
        var client = new GridClient<Order>(q => orderService.GetOrdersGridRows(columns, q), query, false, "ordersGrid", Columns, locale)
            .Filterable()
    ```

* service method
    ```c#
        var server = new GridServer<Order>(repository.GetAll(), Request.Query, true, "ordersGrid", columns, 10)
            .Filterable()
    ```

You can enable a button to clear all selected filters using the ***ClearFiltersButton*** method of the **GridClient** object:  

```c#
    var client = new GridClient<Order>(q => orderService.GetOrdersGridRows(columns, q), query, false, "ordersGrid", Columns, locale)
        .ClearFiltersButton(true);
```

**GridBlazor** supports several types of columns (specified in the **Add** method):

* System.String
* System.Guid
* System.Int32
* System.Int64
* System.Boolean
* System.DateTime
* System.Decimal
* System.Byte
* System.Double
* System.Single
* enum

It also supports nullable types of any element of the list.

**GridBlazor** has different filter widgets for these types:
* **TextFitlerWidget**: it provides a filter interface for text columns (System.String). This means that if your column has text data, **GridBlazor** will render an specific filter interface:

    ![](../images/Filtering_string.png)

* **NumberFilterWidget**: it provides a filter interface for number columns (System.Int32, System.Decimal etc.)

    ![](../images/Filtering_number.png)

* **BooleanFilterWidget**: it provides a filter interface for boolean columns (System.Boolean):

    ![](../images/Filtering_boolean.png)

* **DateTimeFilterWidget**: it provides a filter interface for datetime columns (System.DateTime):

    ![](../images/Filtering_datetime.png)

## Multiple filters

Pressing the **+** and **-** buttons you can add multiple options to filter. You can also select the condition you want to use, either **And** or **Or**:

![](../images/Filtering_multiple.png)

You can also create your own filter widgets.

## Blazor server-side

# Using a list filter

[Index](Documentation.md)

There can be columns where their values can only be selected from a short list. 
You can use an special filter based on a list for those columns. 
It looks like:

![](../images/List_filter.png)

In order to use a list filter you have to create a list of ```SelectItem``` objects in the ```OnParametersSetAsync``` method of the razor page:

```c#
    ...
    var shippers = shipperService.GetAllShippers();
    ...
``` 

Then you have to add the column using the ```SetListFilter``` method of the ```GridColumn``` object:
```c#
    c.Add(o => o.ShipVia)
        .RenderValueAs(o => o.Shipper.CompanyName)
        .SetListFilter(shippers, true, true);
``` 

## Blazor server-side

# Using a date time filter

[Index](Documentation.md)

The default behavior for a ```DateTime``` column is to use a filter widget that allows only date picking. 

But it's also possible to use other ```DateTime``` formats:
- a date time picker, where users can select year, month, day, hour, and minute info. Seconds are not supported.

    You have to add the column using the ```SetFilterWidgetType``` method of the ```GridColumn``` object using the parameter value "DateTimeLocal" and add the correct column format:

    ```c#
        c.Add(o => o.OrderDate).SetFilterWidgetType("DateTimeLocal").Format("{0:yyyy-MM-dd HH:mm}");
    ``` 

- a week picker, where users can select year and week info.

    You have to add the column using the ```SetFilterWidgetType``` method of the ```GridColumn``` object using the parameter value "Week" and render the value using ```DateTimeUtils.GetWeekDateTimeString```:

    ```c#
        c.Add(o => o.OrderDate).SetFilterWidgetType("Week").RenderValueAs(o => DateTimeUtils.GetWeekDateTimeString(o.OrderDate));
    ``` 

- a month picker, where users can select year and month info.

    You have to add the column using the ```SetFilterWidgetType``` method of the ```GridColumn``` object using the parameter value "Month" and add the correct column format:

    ```c#
        c.Add(o => o.OrderDate).SetFilterWidgetType("Month").Format("{0:yyyy-MM}");
    ``` 

## Examples

The UI shown by the widget will depend on the browser used:

- Edge Chromium will show a datetime picker:

    ![](../images/DateTime_Edge.png)

- Chrome and Opera will show a date picker, but time must be selected manually:

    ![](../images/DateTime_Chrome.png)

- Firefox will only allow to wirte date and time manually in "yyyy-mm-dd hh:mm" format:

    ![](../images/DateTime_Firefox.png)

## Blazor server-side

# Creating custom filter widget

[Index](Documentation.md)

The default **GridBlazor** render filter widget for a text column looks like:

![](../images/Creating_custom_filter_widget_widget_1.png)

But you can provide a more specific filter widget for this column. For example, image that you want users pick a customer from customer's list, like:

![](../images/Creating_custom_filter_widget_widget_2.png)


Follow thes steps to create a custom filter widget:

1. Create a blazor page for the new widget 

    Your blazor page should contain the following parameters:

    * **GridHeaderComponent**: it has two methods to add and remove a filter to the column:
        * **AddFilter**: adds a filter to the column. This method must be called by the **Apply** button of the widget
        * **RemoveFilter**: removes the filter to the column. This method must be called by the **Clear** button of the widget
    * **visible**: boolean to control if the widget is shown or not
    * **filterType**: string for the selected filter type. It can be one of these characters:
        * **1**: Equals
        * **2**: Contains
        * **3**: StartsWith
        * **4**: EndsWidth
        * **5**: GreaterThan
        * **6**: LessThan
        * **7**: GreaterThanOrEquals
        * **8**: LessThanOrEquals
    * **filterValue**: string for the value of the filter

    Example of a blazor page for a filter widget (you can find it in the sample **GridComponent.Demo** project with the file name **CustomersFilterComponent.razor**):

    ```razor
        @using GridBlazor
        @using GridBlazor.Resources
        @using GridShared.Filtering
        @using GridBlazorServerSide.Services
        @inject ICustomerService customerService
        @inject IJSRuntime jSRuntime

        @typeparam T

        @if (visible)
        {
            <div class="dropdown dropdown-menu grid-dropdown opened" style="display:block;" 
                @onkeyup="FilterKeyup" @onclick:stopPropagation @onkeyup:stopPropagation>
                <div class="grid-dropdown-arrow"></div>
                <div class="grid-dropdown-inner">
                    <div class="grid-popup-widget">
                        <div class="form-group">
                            <p><i>This is custom filter widget demo</i></p>
                            <select @ref="firstSelect" class="grid-filter-type customerslist form-control" 
                                style="width:250px;" @bind="_filterValue">
                                @foreach (var customerName in customerService.GetCustomersNames())
                                {
                                    <option value="@customerName">@customerName</option>
                                }
                            </select>
                        </div>
                        <div class="grid-filter-buttons">
                            <button type="button" class="btn btn-primary grid-apply" @onclick="ApplyButtonClicked">
                                @Strings.ApplyFilterButtonText
                            </button>
                        </div>
                    </div>
                    <div class="grid-popup-additional">
                        @if (_clearVisible)
                        {
                            <ul class="menu-list">
                                <li>
                                    <a class="grid-filter-clear" href="javascript:void(0);" @onclick="ClearButtonClicked">
                                        @Strings.ClearFilterLabel
                                    </a>
                                </li>
                            </ul>
                        }
                    </div>
                </div>
            </div>
        }

        @code {
            private bool _clearVisible = false;
            protected string _filterValue;

            protected ElementReference firstSelect;

            [CascadingParameter(Name = "GridHeaderComponent")]
            private GridHeaderComponent<T> GridHeaderComponent { get; set; }

            [Parameter]
            public bool visible { get; set; }

            [Parameter]
            public string ColumnName { get; set; }

            [Parameter]
            public IEnumerable<ColumnFilterValue> FilterSettings { get; set; }

            protected override void OnParametersSet()
            {
                _filterValue = FilterSettings.FirstOrDefault().FilterValue;
                _clearVisible = !string.IsNullOrWhiteSpace(_filterValue);
            }

            protected override async Task OnAfterRenderAsync(bool firstRender)
            {
                if (firstRender && firstSelect.Id != null)
                {
                    await jSRuntime.InvokeVoidAsync("gridJsFunctions.focusElement", firstSelect);
                }
            }

            protected async Task ApplyButtonClicked()
            {
                await GridHeaderComponent.AddFilter(new FilterCollection(GridFilterType.Equals.ToString("d"), _filterValue));
            }

            protected async Task ClearButtonClicked()
            {
                await GridHeaderComponent.RemoveFilter();
            }

            public async Task FilterKeyup(KeyboardEventArgs e)
            {
                if (e.Key == "Escape")
                {
                    await GridHeaderComponent.FilterIconClicked();
                }
            }
        }
    ```

    This example loads a customer's list using a singleton service. So we had to create this service in the project to get a list of clients from the database. But it is not mandatory to use a singleton. This example always uses a **filterType** with value **1** when calling the **GridHeaderComponent.AddFilter** method.

2. In the **functions** area of the grid razor page you must create an empty **QueryDictionary** an add the custom filter widget created is step 1 using a unique name:

   ```c#
      private IQueryDictionary<Type> _customFilters = new QueryDictionary<Type>();

      protected override async Task OnInitAsync()
      {
          ...

          _customFilters.Add("CompanyNameFilter", typeof(CustomersFilterComponent<Order>));

          ...
      }

   ```

3. Add a new atribute to the **GridComponent** called **CustomFilters** where we pass the dictionary created in the step 2:

    ```razor
        <GridComponent T="Order" Grid="@_grid" CustomFilters="@_customFilters"></GridComponent>
    ```

4. Add the new filter widget to the colum that will use it. This is done in the lambda expression that creates the column's definition:

    ```c#
        Action<IGridColumnCollection<Order>> columns = c =>
        {
            ...

            c.Add(o => o.Customer.CompanyName).SetFilterWidgetType("CompanyNameFilter");
            ...
        };
    ```
    You have to use the same unique name used on the step 2.

## Blazor server-side

# Setup initial column filtering

[Index](Documentation.md)

Sometimes you need to setup initial column filtering, just after the grid is first time loaded. After that a user can use this pre-filtered grid or clear/change filter settings.

You can do this, using the **SetInitialFilter** method:

```c#
    columns.Add(o => o.Customer.CompanyName)
        .Titled("Company Name")
        .ThenSortByDescending(o => o.OrderID)
        .SetInitialFilter(GridFilterType.StartsWith, "a")
```

## Blazor server-side

# Localization

[Index](Documentation.md)

English is the default laguage. But you can use other languages. You have to create a **CultureInfo** on the razor page and pass it as parameter in the contructor of the **GridClient** object:
    
```c#
    var locale = new CultureInfo("de-DE");
    var client = new GridClient<Order>(q => orderService.GetOrdersGridRows(columns, q), query, false, "ordersGrid", Columns, locale);
```

## Supported languages

* English (default)
* German
* French
* Italian
* Russian
* Spanish
* Norwegian
* Turkish
* Czech
* Slovenian
* Swedish
* Serbian
* Croatian
* Farsi
* Basque
* Catalan
* Galician
* Brazilian Portuguese
* Bulgarian
* Ukrainian

## Right to left direction
Those languages that require right to left direction are also supported. You must configure the grid to user RTL direction using the ```SetDirection``` method of the ```GridClient``` object:
    
```c#
    var client = new GridClient<Order>(q => orderService.GetOrdersGridRows(columns, q), query, false, "ordersGrid", Columns, locale)
        .SetDirection(GridDirection.RTL);
```

## Additional languages

If you need to support other languages, please send me the translation of the following expressions and I will updete the component:
* Add
* And
* Apply
* Average
* Back
* No
* Yes
* Clear all filters
* Clear filter
* Code
* Code and confirmation code must be equal to delete this item
* Code and confirmation code must be equal to save this item
* Confirm Code
* Contains
* Create item
* There are no items to display
* Delete
* Delete item
* Edit
* Ends with
* Equals
* Drop columns here for column extended sorting
* Files
* Filter this column
* Type
* Value
* Greater than
* Greater than or equals
* Drop columns here for column grouping
* Is null
* Is not null
* Items
* Less than
* Less than or equals
* Max
* Min
* Not equals
* Or
* Read item
* Save
* Search for ...
* --- Select an item ---
* Show
* Starts with
* Sum
* Update item
* View
* There was an error creating the new item
* There was an error deleting this item
* There was an error updating this item
* You must select the row you want to delete	
* You must select the row you want to view	
* You must select the row you want to edit

## Blazor server-side

# Data annotations

[Index](Documentation.md)

You can customize grid and column settings using data annotations. In other words, you can mark properties of your model class as grid columns, specify column options, call **AutoGenerateColumns** method, and **GridBlazor** will automatically create columns as you describe in your annotations.

There are some attributes for this:

* **GridTableAttribute**: applies to model classes and specify options for the grid (paging options...)
* **GridColumnAttribute**: applies to model public properties and configure a property as a column with a set of properties
* **GridHiddenColumn**: applies to model public properties and configures a property as a hidden column
* **NotMappedColumnAttribute**: applies to model public properties and configures a property as NOT a column. If a property has this attribute, **GridBlazor** will not add that column to the column collection

For example a model class with the following data annotations:
 
```c#
    [GridTable(PagingEnabled = true, PageSize = 20)]
    public class Foo
    {
        [GridColumn(Position = 0, Title = "Name title", SortEnabled = true, FilterEnabled = true)]
        public string Name { get; set; }

        [GridColumn(Position = 2, Title = "Active Foo?")]
        public bool Enabled { get; set; }

        [GridColumn(Position = 1, Title = "Date", Format = "{0:dd/MM/yyyy}")]
        public DateTime FooDate { get; set; }

        [NotMappedColumn]
        [GridColumn(Position = 3)]
        public byte[]() Data { get; set; }
    }
```
describes that the grid table must contain 3 columns (**Name**, **Enabled** and **FooDate**) with custom options. It also enables paging for this grid table and page size as 20 rows.

The steps to build a grid razor page using data annotations with **GridBlazor** are:

1. Create a service with a method to get all items for the grid. An example of this type of service is: 

    ```c#
        public class FooService
        {
            ...

            public ItemsDTO<Foo> GetFooGridRows(QueryDictionary<StringValues> query)
            {
                var repository = new FooRepository(_context);
                var server = new GridServer<Foo>(repository.GetAll(), new QueryCollection(query),
                    true, "fooGrid", null).AutoGenerateColumns();

                // return items to displays
                return server.ItemsToDisplay;
            }
        }
    ```

    **Notes**:
    * The method must have 1 parameter, a dictionary to pass query parameters such as **grid-page**. It must be ot type **QueryDictionary<StringValues>**

    * The **columns** parameter passed to the **GridServer** constructor must be **null**

    * You must use the **AutoGenerateColumns** method of the **GridServer**

2. Create a razor page to render the grid. The page file must have a .razor extension. An example of razor page is:

    ```razor
        @page "/gridsample"
        @inject FooService fooService

        @if (_task.IsCompleted)
        {
            <GridComponent T="Foo" Grid="@_grid"></GridComponent>
        }
        else
        {
            <p><em>Loading...</em></p>
        }

        @code
        {
            private CGrid<Foo> _grid;
            private Task _task;

            protected override async Task OnInitAsync()
            {
                var query = new QueryDictionary<StringValues>();
                query.Add("grid-page", "2");

                var locale = new CultureInfo("fr-FR");
                var client = new GridClient<Foo>(q => fooService.GetFooGridRows(q), query, false,
                    "fooGrid", null, locale).AutoGenerateColumns();
                _grid = client.Grid;

                // Set new items to grid
                _task = client.UpdateGrid();
                await _task;
            }
        }
    ```

    **Notes**:
    * The **columns** parameter passed to the **GridClient** constructor must be **null**

    * You must use the **AutoGenerateColumns** method of the **GridClient** object to configure a grid.

**GridBlazor** will generate columns based on your data annotations when the **AutoGenerateColumns** method is invoked. 

You can add custom columns after or before this method is called, for example:

```c#
    var server = new GridServer<Foo>(...).AutoGenerateColumns().Columns(columns=>columns.Add(foo=>foo.Child.Price))
```

```c#
    var client = new GridClient<Foo>(...).AutoGenerateColumns().Columns(columns=>columns.Add(foo=>foo.Child.Price))
```

You can also overwrite grid options. For example using the **WithPaging** method:

```c#
    var server = new GridServer<Foo>(...).AutoGenerateColumns().WithPaging(10)
```

**Note:** If you use the ```Position``` custom option to order the columns, you must use it on all the columns of the table including the ones using the ```NotMappedColumn``` attribute. If you don't do it the ```AutoGenerateColumns``` will throw an exception.

## Blazor server-side

# Render button, checkbox, etc. in a grid cell

[Index](Documentation.md)

The prefered method is using a Blazor component because it allows event handling with Blazor.
But you can also use the **RenderValueAs** method to render a custom html markup in the grid cell, as it is used on ASP.NET MVC Core projects.
In this case events will be managed using Javascript.

You have to use the **RenderComponentAs** method to render a component in a cell:

```c#
    columns.Add().RenderComponentAs<ButtonCell>();
```

**RenderComponentAs** method has 3 optional parameters:

Parameter | Type | Description
--------- | ---- | -----------
Actions | IList<Action<object>> (optional) | the parent component can pass a list of Actions to be used by the component (see [Passing grid state as parameter](Passing_grid_state_as_parameter.md))
Functions | IList<Func<object,Task>> (optional) | the parent component can pass a list of Functions to be used by the child component
Object | object (optional) | the parent component can pass an object to be used by the component (see [Passing grid state as parameter](Passing_grid_state_as_parameter.md))

If you use any of these paramenters, you must use them when creating the component.

The generic type used has to be the component created to render the cell.

You must also create a Blazor component that implements the **ICustomGridComponent** interface.
This interface includes a mandatory parameter called **Item** of the same type of the grid row element, and 4 optional parameters:

Parameter | Type | Description
--------- | ---- | -----------
Item | row element (mandatory) | the row item that will be used by the component
GridComponent  | GridComponent<T> (CascadingParameter optional) | Parent Grid component
Grid | CGrid<T> (optional) | Grid can be used to get the grid state (see [Passing grid state as parameter](Passing_grid_state_as_parameter.md))
Actions | IList<Action<object>> (optional) | the parent component can pass a list of Actions to be used by the component (see [Passing grid state as parameter](Passing_grid_state_as_parameter.md))
Functions | IList<Func<object,Task>> (optional) | the parent component can pass a list of Functions to be used by the child component
Object | object (optional) | the parent component can pass an object to be used by the component (see [Passing grid state as parameter](Passing_grid_state_as_parameter.md))

**Actions**, **Functions** and **Object** must be used when calling the **RenderComponentAs** method, but **Grid** can be used without this requirement.
 
The component can include any html elements as well as any event handling features.

## RenderCrudComponentAs

**RenderCrudComponentAs** is a variant of **RenderComponentAs** to be used on grids with CRUD forms. The main difference is:
- Columns defined with **RenderCrudComponentAs** are visible on CRUD forms and on grids
- Columns defined with **RenderComponentAs** are visible on grids, but not visible on CRUD forms. 

You must configure columns created with **RenderCrudComponentAs** as Hidden if you want to show them just on CRUD forms:

``` razor
    c.Add(true).RenderCrudComponentAs<Carousel>();
```

And finally all columns included in the grid but not in the CRUD forms should be configured as "CRUD hidden" using the ```SetCrudHidden(true)``` method.

**Notes**: 
- You can have more granularity in the "CRUD hidden" configuration. You can use the ```SetCrudHidden(bool create, bool read, bool update, bool delete)``` method to configure the columns that will be hidden on each type of form.
- You can have more granularity in the components configuration.  You can use the ```RenderCrudComponentAs<TCreateComponent, TReadComponent, TUpdateComponent, TDeleteComponent>``` method to configure the components that will be shown on each type of form. Id you don't want to show any component for a specific type of form you must use ```NullComponent```


## Button

In this sample we name the component **ButtonCell.razor**:

```razor
    @implements ICustomGridComponent<Order>
    @inject IUriHelper UriHelper

    <button class='btn btn-sm btn-primary' @onclick="MyClickHandler">Edit</button>

    @code {
        [CascadingParameter(Name = "GridComponent")]
        public GridComponent<Order> GridComponent { get; set; }
        
        [Parameter]
        public Order Item { get; protected set; }

        [Parameter]
        public IList<Action<object>> Actions { get; protected set; }

        [Parameter]
        public CGrid<Order> Grid { get; protected set; }

        [Parameter]
        public object Object { get; protected set; }

        private void MyClickHandler(UIMouseEventArgs e)
        {
            if (Actions == null)
            {
                string gridState = Grid.GetState();
                if (Object == null)
                {
                    UriHelper.NavigateTo($"/editorder/{Item.OrderID.ToString()}/gridsample/{gridState}");
                }
                else
                {
                    string returnUrl = (string)Object;
                    UriHelper.NavigateTo($"/editorder/{Item.OrderID.ToString()}/{returnUrl}/{gridState}");
                }
            }
            else
            {
                Actions[0]?.Invoke(Item);
            }
        }
    }
```

## Checkbox

```razor
    @using GridShared.Columns
    @implements ICustomGridComponent<Order>

    <input type='checkbox' @onchange="@CheckChanged" />

    @code {
        [Parameter]
        public Order Item { get; protected set; }

        private void CheckChanged()
        {
            Console.WriteLine("Check changed: Item " + Item.OrderID);
        }
    }
```

## Custom layout

```razor
    @using GridShared.Columns
    @implements ICustomGridComponent<Order>

    <b><a class='modal_link' href='#' @onclick="@MyClickHandler">Edit</a></b>

    @code {
        [Parameter]
        public Order Item { get; protected set; }

        private void MyClickHandler(UIMouseEventArgs e)
        {
            Console.WriteLine("Button clicked: /Home/Edit/" + Item.OrderID);
        }
    }
```
[<- Data annotations](Data_annotations.md) | [Subgrids ->](Subgrids.md)
<!--stackedit_data:
eyJoaXN0b3J5IjpbNjQ0MTE1NTIsLTEwNzQ5OTE4OTBdfQ==
-->