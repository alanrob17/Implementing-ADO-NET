# Builder Classes Make It Easy to Work with ADO.NET

The Builder classes allow us to break apart a connection string, build a connection string, and build INSERT, UPDATE and DELETE commands automatically using the built in classes of ADO.NET.

## SqlConnectionStringBuilder

You can use this to pass in a SqlConnection string and it will separate the attributes into properties of that object automatically for you.

This saves you from writing a parse command to break down the parts of the connection string.

In this example we have a SlqConnectionString.

```
    Server=Localhost;Database=ADONETSamples;Trusted_Connection=Yes;Application Name=ADO.NET Samples
```

And we will use the following routine to break down the attributes into properties.

```
    public string BreakApartConnectionString()
    {
      StringBuilder sb = new StringBuilder(1024);

      // Create a connection string builder object
      SqlConnectionStringBuilder builder = new SqlConnectionStringBuilder(ConnectionString);
     
      // Access each property of the connection string
      sb.AppendLine("Application Name: " + builder.ApplicationName);
      sb.AppendLine("Data Source: " + builder.DataSource);
      sb.AppendLine("Initial Catalog: " + builder.InitialCatalog);
      sb.AppendLine("User ID: " + builder.UserID);
      sb.AppendLine("Password: " + builder.Password);
      sb.AppendLine("Integrated Security: " + builder.IntegratedSecurity);

      ResultText = sb.ToString();

      return ResultText;
    }
```

We end up with.

> Application Name: ADO.NET Samples     
> Data Source: Localhost        
> Initial Catalog: ADONETSamples        
> User ID:      
> Password:     
> Integrated Security: True

Integrated Security in this case is **TRUE** but if we add in a User ID and a Password.

```
    Server=Localhost;Database=ADONETSamples;User ID=Tester;Password=pass1234;Application Name=ADO.NET Samples
```

We get the following changes.

> Application Name: ADO.NET Samples     
> Data Source: Localhost        
> Initial Catalog: ADONETSamples        
> User ID: Tester       
> Password=pass1234        
> Integrated Security: False

In our example we use.

```
    SqlConnectionStringBuilder builder = new SqlConnectionStringBuilder(ConnectionString);
```

This will automatically breakdown the Sql connection string for us.

## Creating a connection string using SqlConnectionStringBuilder

There is a ``ToString()`` method that allows us to build a connection string.

```
    public string CreateConnectionString()
    {
      SqlConnectionStringBuilder builder = new SqlConnectionStringBuilder
      {
        ApplicationName = "A New Application",
        ConnectTimeout = 5,
        DataSource = "Localhost",
        InitialCatalog = "ADONETSamples",
        UserID = "Tester",
        Password = "P@ssw0rd"
      };

      ResultText = builder.ToString();

      return ResultText;
    }
```

This code creates the connection string using this statement.

```
    ResultText = builder.ToString();
```

> Data Source=Localhost;Initial Catalog=ADONETSamples;User ID=Tester;Password=P@ssw0rd;Connect Timeout=5;Application Name="A New Application"

## SqlCommandBuilder

SqlCommandBuilder creates a command object with a SELECT statement and it will infer the INSERT, UPDATE and DELETE commands for us. It will create a parameter collection for us for each command.

There are a couple of things that we have to be aware of.

We must have all non-nullable fields in our SELECT statement, or at least fields without defaults. So if you have a non-nullable field and it has a default you will be okay. We can also use updatable views to create these commands but you must fill in all of the parameters that are generated.

```
    public string CreateDataModificationCommands()
    {
      try
      {
        // Create SQL connection object
        using (SqlConnection cn = new SqlConnection(AppSettings.ConnectionString))
        {
          // Create a SQL Data Adapter
          using (SqlDataAdapter da = new SqlDataAdapter(ProductManager.PRODUCT_CATEGORY_SQL, cn))
          {
            // Fill DataTable
            DataTable dt = new DataTable();
            da.Fill(dt);

            // Create a command builder
            using (SqlCommandBuilder builder = new SqlCommandBuilder(da))
            {
              // Build INSERT Command
              // Pass true to generate parameters names matching column names
              ResultText = "Insert: " + builder.GetInsertCommand(true).CommandText;

              ResultText += Environment.NewLine;

              // Build UPDATE command
              ResultText += "Update: " + builder.GetUpdateCommand(true).CommandText;

              ResultText += Environment.NewLine;

              // Build DELETE command
              ResultText += "Delete: " + builder.GetDeleteCommand(true).CommandText;              
            }
          }
        }
      }
      catch (Exception ex)
      {
        ResultText = ex.ToString();
      }

      return ResultText;
    }
```

In our code we get the Sql SELECT statement from the ProductManager class.

```
    public const string PRODUCT_CATEGORY_SQL = "SELECT ProductCategoryId, CategoryName FROM ProductCategory";
```

**Note:** that we are using **true** in all of our Builder commands so this will use the field names that it will see in the database.

> Insert: INSERT INTO [ProductCategory] ([CategoryName]) VALUES (@CategoryName)     
> Update: UPDATE [ProductCategory] SET [CategoryName] = @CategoryName WHERE (([ProductCategoryId] = > @Original_ProductCategoryId) AND ([CategoryName] = @Original_CategoryName))       
> Delete: DELETE FROM [ProductCategory] WHERE (([ProductCategoryId] = @Original_ProductCategoryId) AND ([CategoryName] = @Original_CategoryName))

If I use the other ProductManager SELECT statement.

```
    public const string PRODUCT_SQL = "SELECT ProductId, ProductName, IntroductionDate, Url, Price, RetireDate, ProductCategoryId FROM Product";
```

These commands are generated.

> Insert: INSERT INTO [Product] ([ProductName], [IntroductionDate], [Url], [Price], [RetireDate], > [ProductCategoryId]) VALUES (@ProductName, @IntroductionDate, @Url, @Price, @RetireDate, @ProductCategoryId)
> Update: UPDATE [Product] SET [ProductName] = @ProductName, [IntroductionDate] = @IntroductionDate, [Url] = > @Url, [Price] = @Price, [RetireDate] = @RetireDate, [ProductCategoryId] = @ProductCategoryId WHERE ((> [ProductId] = @Original_ProductId) AND ([ProductName] = @Original_ProductName) AND ([IntroductionDate] = > @Original_IntroductionDate) AND ([Url] = @Original_Url) AND ([Price] = @Original_Price) AND (> (@IsNull_RetireDate = 1 AND [RetireDate] IS NULL) OR ([RetireDate] = @Original_RetireDate)) AND (> (@IsNull_ProductCategoryId = 1 AND [ProductCategoryId] IS NULL) OR ([ProductCategoryId] = > @Original_ProductCategoryId)))
> Delete: DELETE FROM [Product] WHERE (([ProductId] = @Original_ProductId) AND ([ProductName] = > @Original_ProductName) AND ([IntroductionDate] = @Original_IntroductionDate) AND ([Url] = @Original_Url) AND (> [Price] = @Original_Price) AND ((@IsNull_RetireDate = 1 AND [RetireDate] IS NULL) OR ([RetireDate] = > @Original_RetireDate)) AND ((@IsNull_ProductCategoryId = 1 AND [ProductCategoryId] IS NULL) OR (> [ProductCategoryId] = @Original_ProductCategoryId)))

## Insert a product

we can use a SELECT statement to generate an INSERT command. All we need to do is supply the parameters.

```
    public string InsertUsingDataModificationCommand()
    {
      try
      {
        // Create SQL connection object
        using (SqlConnection cn = new SqlConnection(AppSettings.ConnectionString))
        {
          // Create a SQL Data Adapter
          using (SqlDataAdapter da = new SqlDataAdapter(ProductManager.PRODUCT_SQL, cn))
          {
            // Fill DataTable
            DataTable dt = new DataTable();
            da.Fill(dt);

            // Create a command builder
            using (SqlCommandBuilder builder = new SqlCommandBuilder(da))
            {
              // Build INSERT Command
              using (SqlCommand cmd = builder.GetInsertCommand(true))
              {
                // Set generated parameters with values to insert
                cmd.Parameters["@ProductName"].Value = "A New Product 123";
                cmd.Parameters["@IntroductionDate"].Value = DateTime.Now;
                cmd.Parameters["@Url"].Value = "www.fairwaytech.com";
                cmd.Parameters["@Price"].Value = 100;
                cmd.Parameters["@RetireDate"].Value = DateTime.Now.AddMonths(3);
                cmd.Parameters["@ProductCategoryId"].Value = 1;

                // Set the connection into the command object
                cmd.Connection = cn;

                // Open the connection for inserting
                cn.Open();

                // Execute the command
                cmd.ExecuteNonQuery();
              }
            }
          }
        }
      }
      catch (Exception ex)
      {
        ResultText = ex.ToString();
      }

      return ResultText;
    }
```

Once we run this method it will add a new record into the database.

You could also use this code to run an UPDATE or DELETE command.
