# The Building Blocks of the DataTable

## Create DataTable

We can create a new DataTable from scratch. We instantiate a new DataTable object. We add a new column to a columns collection. For each column that we create we will specify the data type, column name, caption, etc.

Once we have the columns created we can add data to the DataTable using the ``NewRow()`` method on the DataTable object. We set the column names with appropriate data. We then add the row to the Rows collection of our DataTable.

```
    public DataTable BuildDataTable()
    {
      DataTable dt = new DataTable();
      DataColumn dc;

      //*********************
      //*** Creating Columns
      //*********************
      // Method 1: Use Add() method to build column
      dt.Columns.Add("ProductId", typeof(int));
      
      // Method 2: Create a DataColumn object for more control
      dc = new DataColumn
      {
        DataType = typeof(string),
        ColumnName = "ProductName",
        Caption = "Product Name",
        ReadOnly = false
      };
      dt.Columns.Add(dc);

      // Method 3: Create the DataColumn object in Add() method
      dt.Columns.Add(new DataColumn
      {
        DataType = typeof(decimal),
        ColumnName = "Price",
        Caption = "Price",
        ReadOnly = false
      });

      //*********************
      //*** Adding Rows
      //*********************
      // Method 1: Pass in variable amount of arguments to Add() method
      // Pass data in the same order as the columns you created
      dt.Rows.Add(1, "VB.NET Fundamentals", 9.99D);

      // Method 2: Create a new DataRow
      DataRow dr = dt.NewRow();
      // Set each column by name
      dr["ProductId"] = 2;
      dr["ProductName"] = "XML Fundamentals in C#";
      dr["Price"] = 19D;
      // Add DataRow to Rows collection
      dt.Rows.Add(dr);

      // After you are done adding all rows, call AcceptChanges
      dt.AcceptChanges();

      return dt;
    }
```

We start by creating a new DataTable.

```
  DataTable dt = new DataTable();
```

We then create a new data column object.

```
  DataColumn dc;
```

Now we can build our data columns. There are a number of ways to do this.

The first method is.

```
  dt.Columns.Add("ProductId", typeof(int));
```

The second method is probably a cleaner way to build a column.

```
  dc = new DataColumn
  {
    DataType = typeof(string),
    ColumnName = "ProductName",
    Caption = "Product Name",
    ReadOnly = false
  };

  dt.Columns.Add(dc);
```

The third method is the best option as we create a column and add it to the columns collection at the same time.

```
  dt.Columns.Add(new DataColumn
  {
    DataType = typeof(decimal),
    ColumnName = "Price",
    Caption = "Price",
    ReadOnly = false
  });
```

We have added three columns to our DataTable so now we can add data to our DataTable.

We can add a whole row of data in a one line structure.

```
  dt.Rows.Add(1, "VB.NET Fundamentals", 9.99D);
```

Or we can create a new data row.

```
  DataRow dr = dt.NewRow();
  // Set each column by name
  dr["ProductId"] = 2;
  dr["ProductName"] = "XML Fundamentals in C#";
  dr["Price"] = 19D;
  // Add DataRow to Rows collection
  dt.Rows.Add(dr);
```

Once we have added our new rows we can accept changes to our DataTable.

```
  dt.AcceptChanges();
```

We are now ready to use our new DataTable.

## Clone and Copy

We have two options here. We can ``Clone()`` an existing DataTable and this won't have any data attached to it or we can ``Copy()`` the structure and the data from an existing DataTable.

### Clone a DataTable

```
  public DataTable CloneDataTable()
  {
    DataTable dt = BuildDataTable();
    
    // Clone just the structure of the DataTable, no data
    DataTable newDataTable = dt.Clone();
    
    return newDataTable;
  }
```

The BuildDataTable() method uses the structure we created in the previous example and creates a new (empty) DataTable structure.

### Copy a DataTable

```
  public DataTable CopyDataTable()
  {
    DataTable dt = BuildDataTable();

    // Clone the structure and copy all data
    DataTable newDataTable = dt.Copy();

    return newDataTable;
  }
```

This will create the structure and copy the data from the existing DataTable.

## Select() method

We can select rows from an existing DataTable and create a new DataTable containing only those rows by using a query in the Select statement.

```
    public DataTable SelectCopyRowByRow()
    {
      DataTable dt;
      DataTable newDataTable;
      ProductManager mgr = new ProductManager();

      // Get Products from Table
      dt = mgr.GetProductsAsDataTable();

      // Clone structure into new DataTable
      newDataTable = dt.Clone();

      // Select Rows from DataTableObject
      DataRow[] rows = dt.Select("Price < 20");

      // Loop through array of rows
      foreach (DataRow row in rows)
      {
        // NOTE: The following causes an error 
        //       A single row cannot belong to more than one data table
        // newDataTable.Rows.Add(row); 

        // Method 1: Use ItemArray to avoid the error
        //newDataTable.Rows.Add(row.ItemArray);

        // Method 2: Use ImportRow to avoid error
        newDataTable.ImportRow(row);
      }

      newDataTable.AcceptChanges();
      
      return newDataTable;
    }
```

We use the ``ProductsManager()`` class to get a DataTable containing all rows using the following method.

```
  dt = mgr.GetProductsAsDataTable();
```

We clone the original structure.

```
  newDataTable = dt.Clone();
```

We then create a select statement to select the required rows from the existing DataTable.

```
  DataRow[] rows = dt.Select("Price < 20");
```

We loop through each row in the rows collection to add it to the newDataTable collection.

The following won't work!

```
  newDataTable.Rows.Add(row);
```

A single row can only belong to one DataTable.

We work around this by.

```
  newDataTable.Rows.Add(row.ItemArray);
```

Or a better method is to.

```
  newDataTable.ImportRow(row);
```

Once we have looped through all of the rows from the select statement we can accept the changes in the new DataTable.

```
  newDataTable.AcceptChanges();
```

We can now return the new DataTable collection that has a subset of the original data.

## CopyToDataTable method

There is an easier way to copy select rows from a table and that is using the ``CopyToDataTable()`` method.

First you select your rows and then use ``CopyToDataTable()`` to copy those rows to a new Data Table and there is no need to loop through your rows.

The example below is exactly the same as the last example but uses less code to achieve the same result.

```
    public DataTable SelectUsingCopyToDataTable()
    {
      DataTable dt;
      DataTable newDataTable;
      ProductManager mgr = new ProductManager();

      // Get Products from Table
      dt = mgr.GetProductsAsDataTable();

      // Select Rows from DataTableObject and Copy to New DataTable
      newDataTable = dt.Select("Price < 20").CopyToDataTable();
                  
      return newDataTable;
    }
```

The ``Select`` operation and ``CopyToDataTable()`` method are completed in the same statement.

You don't have to use the ``newDataTable.AcceptChanges()`` method to save the records into the DataTable as they are automatically accepted in this case.