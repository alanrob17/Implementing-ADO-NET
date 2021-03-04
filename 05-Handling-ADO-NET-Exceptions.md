# Handling ADO.NET Exceptions

```
    public int SimpleExceptionHandling()
    {
      try 
      {
        // Create SQL statement to submit
        // NOTE: The table name is spelled incorrectly
        string sql = "INSERT INTO Poduct(ProductName, IntroductionDate, Url, Price)";
        sql += " VALUES('VB.NET Fundamentals', '2019-05-21', 'https://bit.ly/30KKHjs', 19   99)";  

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
        // NOTE: A problem here is no access to SqlCommand object
        //       Thus, we have lost the SQL statement and 
        //       connection information for reporting purpose
        ResultText = ex.ToString();
      } 

      return RowsAffected;
    }
```

The basic way we handle an exception is to wrap our code in a ``try-catch()`` block.

When we open a connection and execute the query that is where an exception could occur so we want to try and catch those exceptions and then do something with that information.

The code above is a simple example of exception handling. The name of the table is spelled incorrectly so when the query is executed it will cause an exception on this line.

```
    RowsAffected = cmd.ExecuteNonQuery();
```

You can go through the code down to the ``cn.Open()`` method with no problem because there is nothing wrong with the connection string. An incorrect connection string would have caused our first exception.

When you execute the query it will cause an exception due to the fact that the table name is spelled incorrectly so the query will fail.

You will be thrown to the ``catch()`` block.

```
    catch (Exception ex)
    {
      // NOTE: A problem here is no access to SqlCommand object
      //       Thus, we have lost the SQL statement and 
      //       connection information for reporting purpose
      ResultText = ex.ToString();
    }
```

The error will be a Sql exception because execution fails in the database. It provides us with a lot of information including an Errors collection.There is information about the provider we are using and the server name and that is a little more useful where there is a Sql exception.

The exception information is limited.

> System.Data.SqlClient.SqlException (0x80131904): Invalid object name > 'Poduct'.     
>    at System.Data.SqlClient.SqlConnection.OnError(SqlException exception, Boolean breakConnection, Action`1 > wrapCloseInAction)      
>    at System.Data.SqlClient.SqlInternalConnection.OnError(SqlException exception, Boolean breakConnection, > Action`1 wrapCloseInAction)      
>    at System.Data.SqlClient.TdsParser.ThrowExceptionAndWarning(TdsParserStateObject stateObj, Boolean > callerHasConnectionLock, Boolean asyncClose)      
>    at System.Data.SqlClient.TdsParser.TryRun(RunBehavior runBehavior, SqlCommand cmdHandler, SqlDataReader > dataStream, BulkCopySimpleResultSet bulkCopyHandler, TdsParserStateObject stateObj, Boolean& dataReady)      
>    at System.Data.SqlClient.SqlCommand.RunExecuteNonQueryTds(String methodName, Boolean async, Int32 timeout, > Boolean asyncWrite)       
>    at System.Data.SqlClient.SqlCommand.InternalExecuteNonQuery(TaskCompletionSource`1 completion, String > methodName, Boolean sendToPipe, Int32 timeout, Boolean& usedCache, Boolean asyncWrite, Boolean inRetry)        
>    at System.Data.SqlClient.SqlCommand.ExecuteNonQuery()      
>    at ADONET_Samples.ViewModels.ExceptionViewModel.SimpleExceptionHandling() in > G:\Training\Pluralsight\Pluralsight - Implementing ADO.NET with > C#\source\05\demos\ADONET-Samples\ViewModels\ExceptionViewModel.cs:line 28        
> ClientConnectionId:fbdcb249-8d1c-41ab-9fb8-eb242404bce1       
> Error Number:208,State:1,Class:16

The problem with this exception report is that we are losing some of the Sql Server information. We also don't have any access to the command or connection objects. This could help us to fix the error that just happened so we need to enhance the information that we are sending back and the way to do that is to catch the Sql exception, not the generic exception object.

When we catch the Sql exception we can loop through the errors (``ex.Errors``) collection and report on it.

Next we are going to catch the Sql exception.

```
    public int CatchSqlException()
    {
      try 
      {
        // Create SQL statement to submit
        string sql = "Product_Insert";

        // Create SQL connection object in using block for automatic closing and disposing
        using (SqlConnection cn = new SqlConnection(AppSettings.ConnectionString))
        {
          // Create command object in using block for automatic disposal
          using (SqlCommand cmd = new SqlCommand(sql, cn))
          {
            // NOTE: The following parameter is spelled incorrectly.
            cmd.Parameters.Add(new SqlParameter("@ProdutName", "Generate Exception"));
            cmd.Parameters.Add(new SqlParameter("@IntroductionDate", DateTime.Now));
            cmd.Parameters.Add(new SqlParameter("@Url", "www.wrong.com"));
            cmd.Parameters.Add(new SqlParameter("@Price", 0));

            // Set CommandType
            cmd.CommandType = CommandType.StoredProcedure;
          
            // Open the connection
            cn.Open();
          
            // Execute the INSERT statement
            RowsAffected = cmd.ExecuteNonQuery();

            ResultText = "Rows Affected: " + RowsAffected.ToString();
          }
        }
      }
      catch (SqlException ex) 
      {
        // Get all error information
        System.Text.StringBuilder sb = new System.Text.StringBuilder();

        for (int i = 0; i < ex.Errors.Count; i++) {
          sb.AppendLine("Index #: " + i.ToString());
          sb.AppendLine("Type: " + ex.Errors[i].GetType().FullName);
          sb.AppendLine("Message: " + ex.Errors[i].Message);
          sb.AppendLine("Source: " + ex.Errors[i].Source);
          sb.AppendLine("Number: " + ex.Errors[i].Number.ToString());
          sb.AppendLine("State: " + ex.Errors[i].State.ToString());
          sb.AppendLine("Class: " + ex.Errors[i].Class.ToString());
          sb.AppendLine("Server: " + ex.Errors[i].Server);
          sb.AppendLine("Procedure: " + ex.Errors[i].Procedure);
          sb.AppendLine("LineNumber: " + ex.Errors[i].LineNumber.ToString());
        }

        // NOTE: A problem here is no access to SqlCommand object
        //       Thus, we have lost the SQL statement and 
        //       connection information for reporting purpose
        ResultText = sb.ToString() + Environment.NewLine + ex.ToString();
      }
      catch (Exception ex) 
      {
        ResultText = ex.ToString();
      }

      return RowsAffected;
    }

```

In our code we set up the ``try-catch()`` block and then we set up the procedure we want to call. There are a number of parameters but the first parameter has a typo so our query won't execute again.

This time it calls the catch block and we are going to call the errors collection to get more detailed information about the Sql Server exception. The extra details include the error message, the server name, the data provider, the stored procedure name and even the line number where the stored procedure failed. In our case it failed at line 0 because a parameter was incorrect so it didn't run.

> Index #: 0        
> Type: System.Data.SqlClient.SqlError      
> Message: Procedure or function 'Product_Insert' expects parameter '@ProductName', which was not supplied.     
> Source: .Net SqlClient Data Provider      
> Number: 201       
> State: 4      
> Class: 16     
> Server: TIGER     
> Procedure: Product_Insert     
> LineNumber: 0     
>       
> System.Data.SqlClient.SqlException (0x80131904): Procedure or function 'Product_Insert' expects parameter > '@ProductName', which was not supplied.       
>    at System.Data.SqlClient.SqlConnection.OnError(SqlException exception, Boolean breakConnection, Action`1       
> wrapCloseInAction)      
>    at System.Data.SqlClient.SqlInternalConnection.OnError(SqlException exception, Boolean breakConnection, > Action`1 wrapCloseInAction)      
>    at System.Data.SqlClient.TdsParser.ThrowExceptionAndWarning(TdsParserStateObject stateObj, Boolean > callerHasConnectionLock, Boolean asyncClose)      
>    at System.Data.SqlClient.TdsParser.TryRun(RunBehavior runBehavior, SqlCommand cmdHandler, SqlDataReader > dataStream, BulkCopySimpleResultSet bulkCopyHandler, TdsParserStateObject stateObj, Boolean& dataReady)      
>    at System.Data.SqlClient.SqlCommand.FinishExecuteReader(SqlDataReader ds, RunBehavior runBehavior, String      
> resetOptionsString, Boolean isInternal, Boolean forDescribeParameterEncryption, Boolean > shouldCacheForAlwaysEncrypted)      
>    at System.Data.SqlClient.SqlCommand.RunExecuteReaderTds(CommandBehavior cmdBehavior, RunBehavior > runBehavior, Boolean returnStream, Boolean async, Int32 timeout, Task& task, Boolean asyncWrite, Boolean > inRetry, SqlDataReader ds, Boolean describeParameterEncryptionRequest)       
>    at System.Data.SqlClient.SqlCommand.RunExecuteReader(CommandBehavior cmdBehavior, RunBehavior runBehavior, > Boolean returnStream, String method, TaskCompletionSource`1 completion, Int32 timeout, Task& task, Boolean& > usedCache, Boolean asyncWrite, Boolean inRetry)     
>    at System.Data.SqlClient.SqlCommand.InternalExecuteNonQuery(TaskCompletionSource`1 completion, String > methodName, Boolean sendToPipe, Int32 timeout, Boolean& usedCache, Boolean asyncWrite, Boolean inRetry)        
>    at System.Data.SqlClient.SqlCommand.ExecuteNonQuery()      
>    at ADONET_Samples.ViewModels.ExceptionViewModel.CatchSqlException() in G:\Training\Pluralsight\Pluralsight > - Implementing ADO.NET with C#\source\05\demos\ADONET-Samples\ViewModels\ExceptionViewModel.cs:line 67        
> ClientConnectionId:1020b1c0-aeeb-4339-a8d1-8af33bc7e93f       
> Error Number:201,State:4,Class:16

We now have much better information being returned but we still don't have access to the command and connection objects.

## SqlExceptionManager class

We are going to create a SqlExceptionManager class that is a singleton which means that there will only ever be one instance of this class running.

We are going to create a ``Publish()`` method with an exception argument, a command object argument and a message that will name the procedure where the exception occured.

```
    public int GatherExceptionInformation()
    {
      // Define connection/command objects outside of try...catch block 
      // so you can gather more information for exception handling
      SqlConnection cn = null;
      SqlCommand cmd = null;

      try
      {
        // Create SQL statement to submit
        string sql = "Product_Insert";

        // Create SQL connection object
        cn = new SqlConnection("Server=Localhost;Database=ADONETSamples;uid=sa;pwd=P@ssw0rd;");
        // Create command object
        cmd = new SqlCommand(sql, cn);
        // NOTE: The following parameter is spelled incorrectly.
        cmd.Parameters.Add(new SqlParameter("@ProdutName", "Generate Exception"));
        cmd.Parameters.Add(new SqlParameter("@IntroductionDate", DateTime.Now));
        cmd.Parameters.Add(new SqlParameter("@Url", "www.wrong.com"));
        cmd.Parameters.Add(new SqlParameter("@Price", 0));

        // Set CommandType
        cmd.CommandType = CommandType.StoredProcedure;
        // Open the connection
        cn.Open();
        // Execute the INSERT statement
        RowsAffected = cmd.ExecuteNonQuery();

        ResultText = "Rows Affected: " + RowsAffected.ToString();
      }
      catch (SqlException ex)
      {
        SqlServerExceptionManager.Instance.Publish(ex, cmd, "Error in ExceptionViewModel.GatherExceptionInformation()");
        ResultText = SqlServerExceptionManager.Instance.LastException.ToString();
      }
      catch (Exception ex){
        ResultText = ex.ToString();
      }
      finally 
      {
        // Must close/dispose here so we have access to info for error handling
        if (cn != null) {
          cn.Close();
          cn.Dispose();
        }
        if (cmd != null) {
          cmd.Dispose();
        }
      }

      return RowsAffected;
    }
```

At the start of the code we will create a command and connection object that will both be null. This will enable us to have these objects available for when we want to publish an error.

To be able to use these objects we have removed the ``using()`` blocks so will use a ``finally()`` block to close and dispose of our resources.

We are going to put a breakpoint on the following line.

```
  SqlServerExceptionManager.Instance.Publish(ex, cmd, "Error in ExceptionViewModel.GatherExceptionInformation()");
```

We will follow this to the ``Publish()`` instance.

```
  public Exception LastException { get; set; }

  public virtual void Publish(Exception ex, SqlCommand cmd, string exceptionMsg)
  {
    LastException = ex;
    if (cmd != null)
    {
      LastException = CreateDbException(ex, cmd, null);

      // TODO: Implement an exception publisher here
      System.Diagnostics.Debug.WriteLine(ex.ToString());
    }
  }
```

#### CreateDbException

```
  public virtual SqlServerDataException CreateDbException(Exception ex, SqlCommand cmd, string exceptionMsg)
  {
    SqlServerDataException exc;
  
    exceptionMsg = string.IsNullOrEmpty(exceptionMsg) ? string.Empty : exceptionMsg + " - ";
  
    exc = new SqlServerDataException(exceptionMsg + ex.Message, ex)
    {
      ConnectionString = cmd.Connection.ConnectionString,
      Database = cmd.Connection.Database,
      SQL = cmd.CommandText,
      CommandParameters = cmd.Parameters,
      WorkstationId = Environment.MachineName
    };

    return exc;
  }
```

We override the ``.ToString()`` method.

```
    public override string ToString()
    {
      StringBuilder sb = new StringBuilder(1024);

      sb.AppendLine(new string('-', 80));
      if (!string.IsNullOrEmpty(Message))
      {
        sb.AppendLine("Type: " + this.GetType().FullName);
      }
      if (!string.IsNullOrEmpty(Message))
      {
        sb.AppendLine("Message: " + Message);
      }
      if (!string.IsNullOrEmpty(Database))
      {
        sb.AppendLine("Database: " + Database);
      }
      if (!string.IsNullOrEmpty(SQL))
      {
        sb.AppendLine("SQL: " + SQL);
      }
      if (CommandParameters.Count > 0)
      {
        sb.AppendLine("Parameters:");
        sb.Append(GetCommandParametersAsString());
      }
      if (!string.IsNullOrEmpty(ConnectionString))
      {
        sb.AppendLine("Connection String: " + ConnectionString);
      }
      if (!string.IsNullOrEmpty(StackTrace))
      {
        sb.AppendLine("Stack Trace: " + StackTrace);
      }
      if (IsDatabaseSpecificError(this))
      {
        sb.AppendLine(GetDatabaseSpecificError(this));
      }
      // Gather info from inner exceptions
      sb.AppendLine(GetInnerExceptionInfo());
      // Report stack trace
      sb.AppendLine("Stack Trace: " + this.InnerException.StackTrace);

      return sb.ToString();
    }
```

We can iterate through the Sql parameters with the following method.

```
    protected virtual string GetCommandParametersAsString()
    {
      StringBuilder ret = new StringBuilder(1024);

      if (CommandParameters != null)
      {
        if (CommandParameters.Count > 0)
        {
          ret = new StringBuilder(1024);

          foreach (IDbDataParameter param in CommandParameters)
          {
            ret.Append("  " + param.ParameterName);
            if (param.Value == null)
              ret.AppendLine(" = null");
            else
              ret.AppendLine(" = " + param.Value.ToString());
          }
        }
      }

      return ret.ToString();
    }
```

We are looking for a database specific error and if it exists we will run the following method that goes through the errors collection.

```
    protected virtual string GetDatabaseSpecificError(Exception ex)
    {
      SqlException exp;
      StringBuilder sb = new StringBuilder(1024);

      if (ex is SqlException)
      {
        exp = ((SqlException)(ex));

        for (int index = 0; index <= exp.Errors.Count - 1; index++)
        {
          sb.AppendLine();
          sb.AppendLine(new string('*', 40));
          sb.AppendLine("**** BEGIN: SQL Server Exception #" + (index + 1).ToString() + " ****");
          sb.AppendLine("            Type: " + exp.Errors[index].GetType().FullName);
          sb.AppendLine("            Message: " + exp.Errors[index].Message);
          sb.AppendLine("            Source: " + exp.Errors[index].Source);
          sb.AppendLine("            Number: " + exp.Errors[index].Number.ToString());
          sb.AppendLine("            State: " + exp.Errors[index].State.ToString());
          sb.AppendLine("            Class: " + exp.Errors[index].Class.ToString());
          sb.AppendLine("            Server: " + exp.Errors[index].Server);
          sb.AppendLine("            Procedure: " + exp.Errors[index].Procedure);
          sb.AppendLine("            LineNumber: " + exp.Errors[index].LineNumber.ToString());
          sb.AppendLine("**** END: SQL Server Exception #" + (index + 1).ToString() + " ****");
          sb.AppendLine(new string('*', 40));
        }
      }
      else
      {
        sb.Append(ex.Message);
      }

      return sb.ToString();
    }
```

Now we are printing out a complete list of error information.

> Type: ADONET_Samples.SqlServerDataException   
> Message: Login failed for user 'sa'.		
> Database: ADONETSamples		
> SQL: Product_Insert		
> Parameters:		
>   @ProdutName = Generate Exception		
>   @IntroductionDate = 4/03/2021 2:59:51 PM		
>   @Url = www.wrong.com		
>   @Price = null		
> Connection String: Data Source=Localhost;Initial Catalog=ADONETSamples;User ID=******;Password=******		
> 		
> ****************************************		
> **** BEGIN: SQL Server Exception #1 ****		
>             Type: System.Data.SqlClient.SqlError		
>             Message: Login failed for user 'sa'.		
>             Source: .Net SqlClient Data Provider		
>             Number: 18456		
>             State: 1		
>             Class: 14		
>             Server: Localhost		
>             Procedure: 		
>             LineNumber: 65536		
> **** END: SQL Server Exception #1 ****		
> ****************************************		
> 		
> Stack Trace:    at System.Data.SqlClient.SqlInternalConnectionTds..ctor(DbConnectionPoolIdentity identity, SqlConnectionString connectionOptions, SqlCredential credential, Object providerInfo, String newPassword, SecureString newSecurePassword, Boolean redirectedUserInstance, SqlConnectionString userConnectionOptions, SessionData reconnectSessionData, DbConnectionPool pool, String accessToken, Boolean applyTransientFaultHandling, SqlAuthenticationProviderManager sqlAuthProviderManager)		
>    at System.Data.SqlClient.SqlConnectionFactory.CreateConnection(DbConnectionOptions options, DbConnectionPoolKey poolKey, Object poolGroupProviderInfo, DbConnectionPool pool, DbConnection owningConnection, DbConnectionOptions userOptions)		
>    at System.Data.ProviderBase.DbConnectionFactory.CreatePooledConnection(DbConnectionPool pool, DbConnection owningObject, DbConnectionOptions options, DbConnectionPoolKey poolKey, DbConnectionOptions userOptions)		
>    at System.Data.ProviderBase.DbConnectionPool.CreateObject(DbConnection owningObject, DbConnectionOptions userOptions, DbConnectionInternal oldConnection)		
>    at System.Data.ProviderBase.DbConnectionPool.UserCreateRequest(DbConnection owningObject, DbConnectionOptions userOptions, DbConnectionInternal oldConnection)		
>    at System.Data.ProviderBase.DbConnectionPool.TryGetConnection(DbConnection owningObject, UInt32 waitForMultipleObjectsTimeout, Boolean allowCreate, Boolean onlyOneCheckConnection, DbConnectionOptions userOptions, DbConnectionInternal& connection)		
>    at System.Data.ProviderBase.DbConnectionPool.TryGetConnection(DbConnection owningObject, TaskCompletionSource`1 retry, DbConnectionOptions userOptions, DbConnectionInternal& connection)		
>    at System.Data.ProviderBase.DbConnectionFactory.TryGetConnection(DbConnection owningConnection, TaskCompletionSource`1 retry, DbConnectionOptions userOptions, DbConnectionInternal oldConnection, DbConnectionInternal& connection)		
>    at System.Data.ProviderBase.DbConnectionInternal.TryOpenConnectionInternal(DbConnection outerConnection, DbConnectionFactory connectionFactory, TaskCompletionSource`1 retry, DbConnectionOptions userOptions)		
>    at System.Data.ProviderBase.DbConnectionClosed.TryOpenConnection(DbConnection outerConnection, DbConnectionFactory connectionFactory, TaskCompletionSource`1 retry, DbConnectionOptions userOptions)		
>    at System.Data.SqlClient.SqlConnection.TryOpenInner(TaskCompletionSource`1 retry)		
>    at System.Data.SqlClient.SqlConnection.TryOpen(TaskCompletionSource`1 retry)		
>    at System.Data.SqlClient.SqlConnection.Open()		
>    at ADONET_Samples.ViewModels.ExceptionViewModel.GatherExceptionInformation() in G:\Training\Pluralsight\Pluralsight - Implementing ADO.NET with C#\source\05\demos\ADONET-Samples\ViewModels\ExceptionViewModel.cs:line 129

If you look at the Sql connection string you will notice that the userId and password are asterisks. We did this with the following method.

```
    protected virtual string HideLoginInfoForConnectionString(string connectionString)
    {
      SqlConnectionStringBuilder builder = new SqlConnectionStringBuilder(connectionString);

      if (!string.IsNullOrEmpty(builder.UserID)) {
        builder.UserID = "******";
      }
      if (!string.IsNullOrEmpty(builder.Password)) {
        builder.Password = "******";
      }

      return builder.ConnectionString;
    }
```

This gives us some security by not allowing our user information to be accessed.

Now that we have the complete exception information we would send the results to a logging system like Log4Net.

**Note:** I would be tempted to create two methods for each of my database functions. One without ``using()`` statements to catch any errors and then a final method with ``using()`` statements after I finish my development work and I am happy that the code is working as expected.