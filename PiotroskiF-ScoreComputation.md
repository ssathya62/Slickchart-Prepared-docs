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



<!--stackedit_data:
eyJoaXN0b3J5IjpbLTM5MTA0ODEwMiwtMTA3NDk5MTg5MF19
-->