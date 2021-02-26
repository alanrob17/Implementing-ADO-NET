# Connecting and Submitting Queries to a Database

## Connection class

Before working with the database, you must import a data provider namespace, by placing the following in the **Using** statements of your code module.

For SqlClient .NET data provider namespace import code:

```
    Using System.Data.SqlClient;
```

Now, we have to declare a connection string, which is usually defined in the App.Config or Web.Config file, so it's available in your application. The typical entry of a connection string is written below:

```
    <connectionStrings>  
        <add name="RecordDB" connectionString="Data Source= TIGER;Initial Catalog=RecordDB;User ID=sa,pwd=sa123" providerName="System.Data.SqlClient"/>  
    </connectionStrings>
```

Now, we create a SqlConnection.

If the connection string is stored in the config file with its name RecordDB, we can read it with the following:

```
    var cnstr = System.Configuration.ConfigurationManager.ConnectionStrings["RecordDB"].ConnectionString;
```

Or we can pass a connection string directly into our code.

```
    SqlConnection cnstr = new SqlConnection("Data Source=TIGER; Initial Catalog=RecordDB; User ID=User Name; pwd=User Password" Integrated Security="True");
```

This will push senstitve data into multiple places within our code so is not recommended.

In the connection string:

**Data Source**: This identifies the Server name, which could be the local machine, machine domain name or IP address

**Initial Catalog**: This identifies the database name.

**Integrated Security**: When you have started database authentication login with Windows authentication, Integrated Security specifies Integrated Security="True" in connection string, else when you have started the database authentication login with Server authentication Integrated Security specifies Integrated Security="false" in the connection string

**User Id**: Name of the user configured in SQL Server.

**Password**: Password matching SQL Server User ID.

Now, let us see how we can use the SQLConnection class to establish a connection with a SQL Server database.

```
    var cn = new SqlConnection(cnstr);
```

In most of our code we will use the following to call the connection string and create a connection.

```
    using (var cn = new SqlConnection(AppSettings.ConnectionString))
```

### Properties of connection object

| Property                  | Description                                               |
|---------------------------|:----------------------------------------------------------|
| **Attributes** | We can get or set attributes of the connection object. |
| **Command Timeout** | By Command timeout, we can get or set the number of seconds to wait, while attempting to execute a command.   |
| **Connection&nbsp;Timeout** | By Connection time out, we can get or set the number of seconds to wait for the connection to open. |
| **Connection String** | Connection string is used to establish and create a connection to the data source by using server name, database name, user id and password. |
| **Cursor Location** | It gets or sets location of the cursor service. |
| **Default Database** | It gets or returns the default database name. |
| **Isolation Level** | It gets or returns the isolation level. |
| **Mode** | By mode property, we can check the provider access permission. |
| **Provider** | By this property, we can get or the set provider name. |
| **State** | By this property, we can check if your current connection is opened or closed before connection. |
| **Version** | This returns the ADO version number. |

### Connection object methods

| Methods   | Description                                                |
|---------------------------|:----------------------------------------------------------|
| **BeginTransaction** | Begin current transaction. |
| **Cancel** | Cancel an execution. |
| **Close** | Close method is used to close any current connection. |
| **Open** | Open method is used, open a connection. You must have opened a connection to run a query. |
| **Execute** | This method is used to execute a query. A Statement, procedure or provider provides specific text for execution. |
| **OpenSchema** | returns schema information from the provider about the data source. |
| **RollBackTransaction** | This method is used whenever you cancel any changes or a conflict occurs in the current transaction. It ends the current transaction. |
| **CommitTransaction** | If the current transaction execution has successfully completed CommitTransaction ends the current transaction. |

The usual process to open and close a connection follows.

```
    var cn = new SQLConnection(cnstr);
    cn.Open();

    // ...

    cn.Close();
    cn.Dispose();
```

Connections use valuable resources so you always need to close the connection as soon as you have finished executing your query. You follow this with ``cn.Dispose()`` which is a set of low level Microsoft utilities to dispose of any resources being used.

```
    public void Connect(string cnstr)
    {
        // Create SQL Connection object
        SqlConnection cn = new SqlConnection(cnstr); 
        
        // Open the connection
        cn.Open();   
        
        // Gather connection information
        ResultText = GetConnectionInformation(cn);   
        
        // Close the connection
        cn.Close();
        
        // Dispose of the connection
        cn.Dispose();
    }
```

This calls the ``GetConnectionInformation(SqlConnection cn)`` method.

```
    protected virtual string GetConnectionInformation(SqlConnection cn)
    {
      StringBuilder sb = new StringBuilder(1024);
      sb.AppendLine("Connection String: " + cn.ConnectionString);
      sb.AppendLine("State: " + cn.State.ToString());
      sb.AppendLine("Connection Timeout: " + cn.ConnectionTimeout.ToString());
      sb.AppendLine("Database: " + cn.Database);
      sb.AppendLine("Data Source: " + cn.DataSource);
      sb.AppendLine("Server Version: " + cn.ServerVersion);
      sb.AppendLine("Workstation ID: " + cn.WorkstationId);
      return sb.ToString();
    }
```

And this sends back the following results.

> Connection String: Server=TIGER;Database=ADONETSamples;Trusted_Connection=Yes;Application Name=ADO.NET Samples        
> State: Open       
> Connection Timeout: 15        
> Database: ADONETSamples       
> Data Source: TIGER        
> Server Version: 15.00.2080        
> Workstation ID: TIGER

## Employ a Using block

Instead of using ``cn.Close()`` and ``cn.Dispose()`` commands to remove assets used to carry out our query we can use ``Using()`` blocks to dispose of our resources for us. An example of usage is shown below.

```
    public ConnectionViewModel()
    {
        ConnectionString = AppSettings.ConnectionString;
    }
    
    public string ConnectionString { get; set; }

    public void ConnectUsingBlock()
    {
        // Create SQL connection object in using block for automatic closing and disposing
        using (SqlConnection cn = new SqlConnection(ConnectionString)) {
            // Open the connection
            cn.Open(); 
            ResultText = GetConnectionInformation(cn);
        }
    }
```

Note that we only have to use the ``Open()`` method to create a connection for us.

When the ``using()`` block has completed it will dispose of the resources we used to run the query.

### Catch error information

Not all of your code will execute all of the time so to work around errors we use a ``try-catch()`` block. The following code is a basic example of handling errors and we will discuss this in more detail later.

```
    public void ConnectWithErrors()
    {
        try 
        {
            string cnstr = "Server=ERROR;Connection Timeout=5;Database=ERROR;UID=sa;Password=Password;    Application Name=ADO.NET Samples";

            // Create SQL connection object
            using (SqlConnection cn = new SqlConnection(cnstr))
            {
                // Open the connection
                cn.Open();   
                ResultText = GetConnectionInformation(cn);
            }
        }
        catch (Exception ex) {
            ResultText = ex.ToString();
        }
    }
```

You use a ``try-catch()`` block around the code that is used to make your database connection. If something goes wrong the execution is shutdown and you are thrown to the catch clause that captures the error information.

The code above can't possibly work so it throws an error. This is part of the error message that is returned through the catch clause.

> A network-related or instance-specific error occurred while establishing a connection to SQL Server. The server was not found or was not accessible. Verify that the instance name is correct and that SQL Server is configured to allow remote connections. (provider: Named Pipes Provider, error: 40 - Could not open a connection to SQL Server) ---> System.ComponentModel.Win32Exception (0x80004005): The network path was not found

### Connectionstrings.com

[For more connection string information](https://connectionstrings.com).

## Command class

The Command class provides methods for storing and executing SQL statements and Stored Procedures. The following are the various commands that are executed by the Command Class.

**ExecuteReader**: Returns data to the client as rows. This would typically be an SQL select statement or a Stored Procedure that contains one or more select statements. This method returns a DataReader object that can be used to fill a DataTable object or used directly for printing reports and so forth.

**ExecuteNonQuery**: Executes a command that changes the data in the database, such as an update, delete, or insert statement, or a Stored Procedure that contains one or more of these statements. This method returns an integer that is the number of rows affected by the query.

**ExecuteScalar**: This method only returns a single value. This kind of query returns a count of rows or a calculated value.

**ExecuteXMLReader**: (SqlClient classes only) Obtains data from an SQL Server 2000 database using an XML stream. Returns an XML Reader object.

### ExecuteScalar

Is used to run a simple query command that returns a single value that is usually the number of rows in a particular query or a calculated value.

```
    public int GetProductsCountScalar()
    {
        RowsAffected = 0;

        // Create SQL statement to submit
        string sql = "SELECT COUNT(*) FROM Product";

        // Create a connection
        using (SqlConnection cn = new SqlConnection(AppSettings.ConnectionString)) 
        {
            // Create command object
            using (SqlCommand cmd = new SqlCommand(sql, cn)) 
            {
                // Open the connection
                cn.Open();
            
                // Execute command
                RowsAffected = (int)cmd.ExecuteScalar();
            }
        }

        ResultText = "Rows Affected: " + RowsAffected.ToString();
        
        return RowsAffected;
    }
```

We use a ``using()`` block to create the connection and then we use another ``using()`` block to setup and run the Sql Command.

When we run ``cmd.ExecuteScaler()`` it will return an object data type. In our case we know we are expecting an integer return value so we need to cast the object data type to an integer.

This will bring back the number of rows in our Product table.

### Action command (ExecuteNonQuery)

An action command could be an *insert*, *update* or *delete* query. The following code uses an insert command.

```

    public int InsertProduct()
    {
        RowsAffected = 0;

        // Create SQL statement to submit
        string sql = "INSERT INTO Product(ProductName, IntroductionDate, Url, Price)";
        sql += " VALUES('VB.NET Fundamentals', '2019-05-21', 'https://bit.ly/30KKHjs', 19.99)";

        try 
        {
            // Create SQL connection object in using block for automatic closing and disposing
            using (SqlConnection cn = new SqlConnection(AppSettings.ConnectionString)) 
            {
                // Create command object in using block for automatic disposal
                using (SqlCommand cmd = new SqlCommand(sql, cn))
                {
                    // Set CommandType
                    cmd.CommandType = CommandType.Text;

                    // Open the connection
                    cn.Open();

                    // Execute the INSERT statement
                    RowsAffected = cmd.ExecuteNonQuery();

                    ResultText = "Rows Affected: " + RowsAffected.ToString();
                }
            }
        }
        catch (Exception ex)
        {
            ResultText = ex.ToString();
        }

        return RowsAffected;
    }
```

We are setting up our insert statement in a SQL text command. We aren't bothering with parameters for each of the pieces of data we are inserting into our table.


Once again we have a ``try-catch()`` block to catch errors and a ``using()`` block for our connection and our SQL Command.

**Note:** that inside the SQL Command block that we have to specify that we are using a text command for our query. We have to do this or the command won't understand how to process this request.

```
    cmd.CommandType = CommandType.Text;
```

If we were using a stored procedure as our command type it would need this statement before we open the connection.

```
    cmd.CommandType = CommandType.StoredProcedure;
```

This time we use the **ExecuteNonQuery** method to insert data into our Product table. This will return how many rows were affected. This will return an integer so we need to use ``.ToString()`` to cast the result as text.

When we run this command it will enter a new record into the database.

> 31	VB.NET Fundamentals	2019-05-21 00:00:00.000	https://bit.ly/30KKHjs	19.9900	NULL	NULL

## Parameter class

In our previous example we hardcoded our parameter. We can use the parameter class to pass arguments to our Sql statement.

A sample parameter statement is shown below.

```
    cmd.Parameters.Add(new SqlParameter("@ProductName", productName));
```

We can also create output parameters when we need to receive the Id of a record that was inserted into a database.

```
    cmd.Parameters.Add(new SqlParameter { ParameterName = "@ProductId", Value = InputEntity.ProductId, IsNullable = false, DbType = System.Data.DbType.Int32, Direction = ParameterDirection.Output });
```

### Scalar value retrieval using parameters

```
    public int GetProductsCountScalarUsingParameters(string productName)
    {
        RowsAffected = 0;

        // Create SQL statement to submit
        string sql = "SELECT COUNT(*) FROM Product WHERE ProductName LIKE @ProductName";

        // Create a connection
        using (SqlConnection cn = new SqlConnection(AppSettings.ConnectionString))
        {
            // Create command object
            using (SqlCommand cmd = new SqlCommand(sql, cn)) 
            {
              // Create parameters
              cmd.Parameters.Add(new SqlParameter("@ProductName", productName));

              // Open the connection
              cn.Open();
              // Execute command
              RowsAffected = (int)cmd.ExecuteScalar();
            }
        }
        ResultText = "Rows Affected: " + RowsAffected.ToString();

        return RowsAffected;
    }
```

Imagine in our case the productName starts with the letter A and when we do our query the result returned will be.

> Rows Affected: 3

### Action command with parameters

```
    public int InsertProductUsingParameters()
    {
        RowsAffected = 0; 
        // Create SQL statement to submit
        string sql = "INSERT INTO Product(ProductName, IntroductionDate, Url, Price)";
        sql += " VALUES(@ProductName, @IntroductionDate, @Url, @Price)";

        try 
        {
          // Create SQL connection object in using block for automatic closing and disposing
          using (SqlConnection cn = new SqlConnection(AppSettings.ConnectionString))
          {
              // Create command object in using block for automatic disposal
              using (SqlCommand cmd = new SqlCommand(sql, cn))
              {
                // Create input parameters
                cmd.Parameters.Add(new SqlParameter("@ProductName", InputEntity.ProductName));
                cmd.Parameters.Add(new SqlParameter("@IntroductionDate", InputEntity.IntroductionDate));
                cmd.Parameters.Add(new SqlParameter("@Url", InputEntity.Url));
                cmd.Parameters.Add(new SqlParameter("@Price", InputEntity.Price));  
                
                // Set CommandType
                cmd.CommandType = CommandType.Text;
                
                // Open the connection
                cn.Open(); 
                
                // Execute the INSERT statement
                RowsAffected = cmd.ExecuteNonQuery();   
                
                ResultText = "Rows Affected: " + RowsAffected.ToString();
              }
          }
        }
        catch (Exception ex) 
        {
          ResultText = ex.ToString();
        } 
        return RowsAffected;

```

In the code above we have 4 arguments in our Sql statement so that means we will need 4 parameters to receive those values. In our case those values are coming from a class that has those values.

```
    private Product _InputEntity = new Product { ProductId = -1, ProductName = "A New Product", IntroductionDate = DateTime.Now, Price = 29, RetireDate = null, Url = "www.fairwaytech.com" };
    private Product _SearchEntity = new Product { ProductName = "WPF%" };
    #endregion  
    #region Public Properties
    
    /// <summary>
    /// Get/Set Entity for Data Entry
    /// </summary>
    public Product InputEntity
    {
      get { return _InputEntity; }
      set {
        _InputEntity = value;
      }
    }
```

## Action command with an output parameter

We are using a stored procedure with this action command. The stored procedure we are calling is.

```
    ALTER PROCEDURE [dbo].[Product_Insert]
        @ProductName nvarchar(150),
        @IntroductionDate datetime,
        @Url nvarchar(255),
        @Price money,
        @ProductId int OUTPUT
    AS
    INSERT INTO Product(ProductName, IntroductionDate, Url, Price)
    VALUES(@ProductName, @IntroductionDate, @Url, @Price);

    SELECT @ProductId = SCOPE_IDENTITY();
```

This stored procedure has our 4 input parameters and 1 output parameter. Output parameters are commonly used for returning the next primary key generated by the database.

*SCOPE_IDENTITY()* gives us the next generated primary key value.

```
    public string InsertProductOutputParameters()
    {
        RowsAffected = 0;
        // Create SQL statement to submit
        string sql = "Product_Insert";

        try
        {
            // Create SQL connection object in using block for automatic closing and disposing
            using (SqlConnection cn = new SqlConnection(AppSettings.ConnectionString))
            {
                // Create command object in using block for automatic disposal
                using (SqlCommand cmd = new SqlCommand(sql, cn))
                {
                  // Create input parameters
                  cmd.Parameters.Add(new SqlParameter("@ProductName", InputEntity.ProductName));
                  cmd.Parameters.Add(new SqlParameter("@IntroductionDate", InputEntity.IntroductionDate));
                  cmd.Parameters.Add(new SqlParameter("@Url", InputEntity.Url));
                  cmd.Parameters.Add(new SqlParameter("@Price", InputEntity.Price));
                  
                  // Create OUTPUT parameter
                  cmd.Parameters.Add(new SqlParameter { ParameterName = "@ProductId", Value = InputEntity.productId,  IsNullable = false, DbType = System.Data.DbType.Int32, Direction = parameterDirection.Output });

                  // Set CommandType to Stored Procedure
                  cmd.CommandType = CommandType.StoredProcedure;
                  
                  // Open the connection
                  cn.Open();
                  
                  // Execute the INSERT statement
                  RowsAffected = cmd.ExecuteNonQuery();
                  
                  // Get output parameter
                  InputEntity.ProductId = (int)cmd.Parameters["@ProductId"].Value;

                  ResultText = "Rows Affected: " + RowsAffected.ToString();
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

Note that we are using a stored procedure for our SQL command in this code. That means we have to set the command type.

```
    cmd.CommandType = CommandType.StoredProcedure;
```

If you don't do this you will get an error from ADO.NET.

Once we execute the query we get the output value so we can return it from our method.

```
    InputEntity.ProductId = (int)cmd.Parameters["@ProductId"].Value;

    ResultText = "Rows Affected: " + RowsAffected.ToString();

    // ...

    return ResultText;
```

## Transactions in ADO.NET

We use transactions when there is more than one statement to submit to a database. If one transaction in the set fails then all transactions are rolled back and nothing is committed to the database.

```
    public int TransactionProcessing()
    {
        RowsAffected = 0; 
        
        // Create SQL statement to submit
        string sql = "INSERT INTO Product(ProductName, IntroductionDate, Url, Price)";
        sql += " VALUES(@ProductName, @IntroductionDate, @Url, @Price)";  

        try 
        {
            // Create SQL connection object in using block for automatic closing and disposing
            using (SqlConnection cn = new SqlConnection(AppSettings.ConnectionString)) 
            {
              // Open the connection
              cn.Open();

              // Start a local transaction
              using (SqlTransaction trn = cn.BeginTransaction())
              {
                try
                {
                  // Create command object in using block for automatic disposal
                  using (SqlCommand cmd = new SqlCommand(sql, cn))
                  {
                    // Make command object part of the transaction
                    cmd.Transaction = trn;  
                    
                    // Create input parameters
                    cmd.Parameters.Add(new SqlParameter("@ProductName", InputEntity.ProductName));
                    cmd.Parameters.Add(new SqlParameter("@IntroductionDate", InputEntity.IntroductionDate));
                    cmd.Parameters.Add(new SqlParameter("@Url", InputEntity.Url));
                    cmd.Parameters.Add(new SqlParameter("@Price", InputEntity.Price));  
                    
                    // Set CommandType
                    cmd.CommandType = CommandType.Text; 
                    
                    // Execute the INSERT statement
                    RowsAffected = cmd.ExecuteNonQuery();   
                    
                    ResultText = "Product Rows Affected: " + RowsAffected.ToString() + Environment.NewLine; 

                    // *************************************************
                    // ** SECOND STATEMENT TO EXECUTE
                    // *************************************************
                    sql = "INSERT INTO ProductCategory(CategoryName) VALUES(@CategoryName)";    
                    
                    // Reset the command text
                    cmd.CommandText = sql;
                    
                    // Create previous parameters
                    cmd.Parameters.Clear();
                    
                    // Create input parameter
                    cmd.Parameters.Add(new SqlParameter("@CategoryName", "A New Category"));    
                    
                    // Execute the INSERT statement
                    RowsAffected = cmd.ExecuteNonQuery();   
                    
                    ResultText += "Product Category Rows Affected: " + RowsAffected.ToString();

                    // Finish the Transaction
                    trn.Commit();
                  }
                }
                catch (Exception ex)  // Catch block for transaction
                {
                  // Rollback the transaction
                  trn.Rollback();   
                  ResultText = "Transaction Rolled Back" + Environment.NewLine + ex.ToString();
                }
              }
            }
        }
        catch (Exception ex)  // Catch block for connection opening
        {
          ResultText = ex.ToString();
        } 

        return RowsAffected;
    }
```

In the code above we start with a ``try-catch()`` block that contains our connection string enclosed in our first ``using()`` block.

We then ``open()`` our connection.

The next ``using()`` block is used to begin our transactions.

```
    SqlTransaction trn = cn.BeginTransaction()
```

Next we create another ``try-catch()`` block that contains another ``using()`` block that contains our command object.

We then add our command object to our transaction object.

```
    cmd.Transaction = trn;
```

We carry out our first transaction which is an ``insert`` query that adds a record to our *Product* table.

Before we start our next transaction we need to clear our previous parameters list.

```
    cmd.Parameters.Clear();
```

By this stage we have changed our sql text and cleared our parameters but we are still part of the transaction object.

Our next transaction is also an ``insert`` query that inserts a record into our *ProductCategory* table.

If both transactions complete successfully we commit the transactions within our transaction ``try`` block else if one transaction fails we are sent to the transaction ``catch()`` block and here we run the following command to rollback all transactions.

```
    catch (Exception ex)  // Catch block for transaction
    {
      // Rollback the transaction
      trn.Rollback();
      ResultText = "Transaction Rolled Back" + Environment.NewLine + ex.ToString();
    }
```

On successful completion we should get the following results.

> Product Rows Affected: 1      
> Product Category Rows Affected: 1

Note we use two ``try-catch()`` blocks in this code, one for the **connection** and **transaction** objects and one for the **command** object.

If the connection or transaction objects fail we can exit gracefully before the command object is processed.
