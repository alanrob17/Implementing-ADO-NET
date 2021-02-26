# Retrieve Data Quickly Using the SqlDataReader

The benefits of using a Data Reader to retrieve your data are.

![Data reader](assets/images/data-reader.jpg "Data reader")

The ADO.NET DataReader object is used for accessing data from the data store and is one of the two mechanisms that ADO.NET provides. The DataReader object provides a read only, forward only, high performance mechanism to retrieve data from a data store as a data stream, while staying connected with the data source. The DataReader is restricted but highly optimized. The .NET framework provides data providers for SQL Server native OLE DB providers and native ODBC drivers,

* SqlDataReader
* OleDbDataReader
* OdbcDataReader

You can use the ADO.NET DataReader to retrieve a read-only, forward-only stream of data from a database. Using the DataReader can increase application performance and reduce system overhead because only one row at a time is ever in memory. After creating an instance of the Command object, you create a DataReader by calling Command.ExecuteReader to retrieve rows from a data source, as shown in the following example.

```
    SqlDataReader myReader = myCommand.ExecuteReader();
```

You use the Read method of the DataReader object to obtain a row from the results of the query. You can access each column of the returned row by passing the name or ordinal reference of the column to the DataReader. However, for best performance, the DataReader provides a series of methods that allow you to access column values in their native data types (GetDateTime, GetDouble, GetGuid, GetInt32, and so on). For a list of typed accessor methods, see the SqlDataReader Class. Using the typed accessor methods when the underlying data type is known will reduce the amount of type conversion required when retrieving the column value.

The following code example iterates through a DataReader object, and returns two columns from each row.

```
    while (myReader.Read())
    {
        Console.WriteLine("\t{0}\t{1}", myReader.GetInt32(0), myReader.GetString(1));
    }
    myReader.Close();  
```

The DataReader provides an unbuffered stream of data that allows procedural logic to efficiently process results from a data source sequentially. The DataReader is a good choice when retrieving large amounts of data because the data is not cached in memory. You should always call the ``Close()`` method when you have finished using the DataReader object. If your Command contains output parameters or return values, they will not be available until the DataReader is closed.

Note that while a DataReader is open, the Connection is in use exclusively by that DataReader. You will not be able to execute any commands for the Connection, including creating another DataReader, until the original DataReader is closed.

## An example of using SQLDataReader

```
    public void GetProductsAsDataReader()
    {
      StringBuilder sb = new StringBuilder(1024);

      // Create a connection
      using (SqlConnection cnn = new SqlConnection(AppSettings.ConnectionString)) 
      {
        // Create command object
        using (SqlCommand cmd = new SqlCommand(ProductManager.PRODUCT_SQL, cnn))
        {
          // Open the connection
          cnn.Open();

          // Execute command to get data reader
          using (SqlDataReader dr = cmd.ExecuteReader(CommandBehavior.CloseConnection))
          {
            while (dr.Read())
            {
              sb.AppendLine("Product: " + dr["ProductId"].ToString());
              sb.AppendLine(dr["ProductName"].ToString());
              sb.AppendLine(Convert.ToDateTime(dr["IntroductionDate"]).ToShortDateString());
              sb.AppendLine(Convert.ToDecimal(dr["Price"]).ToString("c"));
              sb.AppendLine();
            }
          }
        }
      }

      ResultText = sb.ToString();
    }
```

In the method above we are going to create a ``StringBuilder()`` object to store the data retrieved from the data reader as a text list.

``ProductManager.PRODUCT_SQL`` is a constant in the ProductManager class that contains our sql code.

```
    public const string PRODUCT_SQL = "SELECT ProductId, ProductName, IntroductionDate, Url, Price, RetireDate, ProductCategoryId FROM Product";
```

We call our data reader inside a ``using()`` block.

```
    using (SqlDataReader dr = cmd.ExecuteReader(CommandBehavior.CloseConnection))
```

We use the ``.ExecuteReader()`` method to retrieve our data. Once this has completed we need to close the connection and we do this by using the ``CloseConnection`` enumeration. This closes valuable resources.

We open a **cursor** to read the data row by row.

```
    while (dr.Read())
```

We have to convert our data from the data reader into a string format.

```
    sb.AppendLine("Product: " + dr["ProductId"].ToString());
    sb.AppendLine(dr["ProductName"].ToString());
    sb.AppendLine(Convert.ToDateTime(dr["IntroductionDate"]).ToShortDateString());
    sb.AppendLine(Convert.ToDecimal(dr["Price"]).ToString("c"));
    sb.AppendLine();
```

This will convert each column object in the data row into a line of text.

**Note:** this process would fail if any of the fields had a **null** value. If you look at the table in the database all columns are ``NOT NULL`` so it works in this case. In the real world we would have to allow for nulls and we will do this later.

The connection is closed because of the ``CommandBehavior.CloseConnection``.