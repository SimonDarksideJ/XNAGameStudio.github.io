# SQL Server, ODBC, OleDB All in One


Here is a wrapper that uses Odbc, SqlClient (for SQL Server) AND OleDb.

This is an all in one solution for all three database types in C#

This also resolves the issue of having multiple recordsets open by creating a new data connection for each recordset retrieved.


```csharp
#region references
using System;
using System.Collections;
using System.Data;
using System.Data.Odbc;
using System.Data.SqlClient;
using System.Data.OleDb;
#endregion

namespace ZiggyData
{
    #region ZiggyDBConnection
    public class ZiggyDBConnection
    {
        #region Member Variables
        private SqlConnection sqlConn = null;
        private OdbcConnection odbcConn = null;
        private OleDbConnection oleConn = null;
        #endregion

        #region OpenConnection

        public bool OpenConnection(string server,
                                   string database,
                                   string username,
                                   string password)
        {
            try
            {
                odbcConn = null;
                oleConn = null;

                sqlConn = new SqlConnection(
                        "server=" + server + 
                        ";database=" + database + 
                        ";user id=" + username + 
                        ";password=" + password + 
                        ";trusted_connection=false;");
                sqlConn.Open();
                return true;
            }
            catch(Exception ex)
            {
                return false;
            }
        }

        public bool OpenConnection(string server,
            string database,
            string username,
            string password,
            bool UseNTAuthentication)
        {
            try
            {
                if(UseNTAuthentication)
                {
                    return OpenConnection(server,database);
                }
                else
                {
                    return OpenConnection(server,
                        database,username,password);
                }
            }
            catch(Exception)
            {
                return false;
            }
        }

        public bool OpenConnection(string server,string database)
        {
            try
            {
                odbcConn = null;
                oleConn = null;

                sqlConn = new SqlConnection("server=" + server + 
                    ";database=" + database + 
                    ";trusted_connection=TRUE;");
                sqlConn.Open();
                return true;
            }
            catch(Exception ex)
            {
                return false;
            }
        }

        public bool OpenODBCConnection(string strODBC)
        {
            try
            {
                sqlConn = null;
                oleConn = null;
                odbcConn = new OdbcConnection(strODBC);
                odbcConn.Open();
                return true;
            }
            catch(Exception ex)
            {
                return false;
            }
        }

        public bool OpenOLEConnection(string strConn)
        {
            try
            {
                sqlConn = null;
                odbcConn = null;
                oleConn = new OleDbConnection(strConn);
                oleConn.Open();
                return true;
            }
            catch(Exception ex)
            {
                return false;
            }
        }
        #endregion

        #region Close
        public void Close()
        {
            if(sqlConn == null && oleConn == null && odbcConn != null)
            {
                odbcConn.Close();
                odbcConn = null;
            }
            else if(odbcConn == null && oleConn==null && sqlConn != null)
            {
                sqlConn.Close();
                sqlConn = null;
            }
            else if(odbcConn == null && sqlConn==null && oleConn != null)
            {
                oleConn.Close();
                oleConn = null;
            }
        }
        #endregion

        #region Execute
        public bool Execute(string strQuery)
        {
            try
            {
                if(sqlConn != null)
                {
                    SqlCommand cmd = sqlConn.CreateCommand();
                    cmd.CommandText = strQuery;
                    cmd.ExecuteNonQuery();
                }
                else if(odbcConn != null)
                {
                    OdbcCommand cmd = odbcConn.CreateCommand();
                    cmd.CommandText = strQuery;
                    cmd.ExecuteNonQuery();
                }
                else if(oleConn != null)
                {
                    OleDbCommand cmd = oleConn.CreateCommand();
                    cmd.CommandText = strQuery;
                    cmd.ExecuteNonQuery();
                }
            }
            catch(Exception){ return false; }

            return true;
        }
        #endregion

        #region GetRSFromQuery
        public ZiggyRS GetRSFromQuery(string strQuery)
        {
            if(sqlConn != null)
            {
                SqlDataReader rs = null;

                try
                {
                    SqlConnection newConn = 
                        new SqlConnection(sqlConn.ConnectionString);

                    SqlCommand cmd = newConn.CreateCommand();
                    cmd.CommandText = strQuery;
                    rs = cmd.ExecuteReader(
                        CommandBehavior.CloseConnection);
                }
                catch(Exception)
                {
                }
                
                return new ZiggyRS(rs);
            }
            else if(odbcConn != null)
            {
                OdbcDataReader rs = null;
                try
                {
                    OdbcConnection newConn = 
                        new OdbcConnection(odbcConn.ConnectionString);

                    OdbcCommand cmd = newConn.CreateCommand();
                    cmd.CommandText = strQuery;
                    rs = cmd.ExecuteReader(CommandBehavior.CloseConnection);
                }
                catch(Exception)
                {
                    
                }
                
                return new ZiggyRS(rs);
            }
            else if(oleConn != null)
            {
                OleDbDataReader rs = null;
                try
                {
                    OleDbConnection newConn = 
                        new OleDbConnection(odbcConn.ConnectionString);

                    OleDbCommand cmd = newConn.CreateCommand();
                    cmd.CommandText = strQuery;
                    rs = cmd.ExecuteReader(CommandBehavior.CloseConnection);
                }
                catch(Exception)
                {
                    
                }
                
                return new ZiggyRS(rs);
            }
            else
            {
                return null;
            }
        }
        #endregion

        #region GetValueFromQuery
        public string GetValueFromQuery(string strQuery)
        {
            string ret = "";
            SqlDataReader sqlRS = null;
            OdbcDataReader odbcRS = null;
            OleDbDataReader oleRS = null;

            try
            {
                if(sqlConn != null)
                {
                    SqlCommand cmd = sqlConn.CreateCommand();
                    cmd.CommandText = strQuery;
                    sqlRS = cmd.ExecuteReader();
                    sqlRS.Read();
                    ret = sqlRS.GetValue(0).ToString();
                }
                else if(odbcConn != null)
                {
                    OdbcCommand cmd = odbcConn.CreateCommand();
                    cmd.CommandText = strQuery;
                    odbcRS = cmd.ExecuteReader();
                    odbcRS.Read();
                    ret = odbcRS.GetValue(0).ToString();
                }
                else if(oleConn != null)
                {
                    OleDbCommand cmd = oleConn.CreateCommand();
                    cmd.CommandText = strQuery;
                    oleRS = cmd.ExecuteReader();
                    oleRS.Read();
                    ret = oleRS.GetValue(0).ToString();
                }
            }
            catch(Exception)
            {
                
            }
            
            if(sqlRS != null)
            {
                sqlRS.Close();
            }
            else if(odbcRS != null)
            {
                odbcRS.Close();
            }
            else if(oleRS != null)
            {
                oleRS.Close();
            }
            return ret;
        }
        #endregion

    }
    #endregion

    #region ZiggyDatabase
    public class ZiggyDatabase
    {
        #region MemberVariables
        private ZiggyDBConnection connection = null;
        #endregion

        #region Connect
        public bool Connect(string server,
            string database,string username,string password)
        {
            connection = new ZiggyDBConnection();
            return connection.OpenConnection(server,
                database,username,password);
        }

        public bool Connect(string server,
            string database,string username,
            string password,bool UseNTAuthentication)
        {
            connection = new ZiggyDBConnection();
            return connection.OpenConnection(server,
                database,username,password,UseNTAuthentication);
        }

        public bool Connect(string server,string database)
        {
            connection = new ZiggyDBConnection();
            return connection.OpenConnection(server,database);
        }

        
        public bool ConnectODBC(string connectString)
        {
            connection = new ZiggyDBConnection();
            return connection.OpenODBCConnection(connectString);
        }
        #endregion

        #region Close
        public void Close()
        {
            connection.Close();
        }
        #endregion

        #region Execute
        public bool Execute(string strQuery)
        {
            return connection.Execute(strQuery);
        }
        #endregion

        #region GetRSFromQuery
        public ZiggyRS GetRSFromQuery(string strQuery)
        {
            return connection.GetRSFromQuery(strQuery);
        }
        #endregion

        #region GetValueFromQuery
        public string GetValueFromQuery(string strQuery)
        {
            return connection.GetValueFromQuery(strQuery);
        }
        #endregion
    }
    #endregion

    #region ZiggyRS
    public class ZiggyRS : IDisposable
    {
        #region MemberVariables
        SqlDataReader sqlReader = null;
        OdbcDataReader odbcReader = null;
        OleDbDataReader oleReader = null;
        bool bFirstRow = true;
        bool bNextRow = false;
        #endregion
        
        #region Constructor
        public ZiggyRS(SqlDataReader reader)
        {
            sqlReader = reader;
            NextRow();
        }
        public ZiggyRS(OdbcDataReader reader)
        {
            odbcReader = reader;
            NextRow();
        }
        public ZiggyRS(OleDbDataReader reader)
        {
            oleReader = reader;
            NextRow();
        }
        #endregion

        #region NextRow
        public bool NextRow()
        {
            try
            {
                if(bFirstRow)
                {
                    bFirstRow = false;
                    bNextRow = true;
                    if(sqlReader != null)
                        return sqlReader.Read();
                    else if(odbcReader != null)
                        return odbcReader.Read();
                    else if(oleReader != null)
                        return oleReader.Read();
                    else
                        return false;
                }
                else if(bNextRow)
                {
                    bNextRow = false;
                    return true;
                }
                else
                {
                    if(sqlReader != null)
                        return sqlReader.Read();
                    else if(odbcReader != null)
                        return odbcReader.Read();
                    else if(oleReader != null)
                        return oleReader.Read();
                    else
                        return false;
                }
            }
            catch(Exception){return false;}
        }
        #endregion

        #region Close
        public void Close()
        {
            try
            {
                if(sqlReader != null)
                    sqlReader.Close();
                else if(odbcReader != null)
                    odbcReader.Close();
                else if(oleReader != null)
                    oleReader.Close();
            }
            catch(Exception){}
        }
        #endregion

        #region Dispose
        public void Dispose()
        {
            Close();
        }
        #endregion

        #region operator []
        [System.Runtime.CompilerServices.IndexerName("GetByIndex")]
        public string this [int columnIndex]   // Indexer declaration
        {
            get
            {
                if(sqlReader == null && 
                    odbcReader == null && 
                    oleReader == null)
                    return "";

                
                if((sqlReader != null && 
                    (sqlReader.FieldCount > columnIndex && 
                        sqlReader.FieldCount > 0))
                    ||
                    (odbcReader != null && 
                    (odbcReader.FieldCount > columnIndex && 
                        odbcReader.FieldCount > 0))
                    ||
                    (oleReader != null && 
                    (oleReader.FieldCount > columnIndex && 
                        oleReader.FieldCount > 0)))
                {
                    object o = null;
                    try
                    {
                        if(sqlReader != null)
                            o = sqlReader.GetValue(columnIndex);
                        else if(odbcReader != null)
                            o = odbcReader.GetValue(columnIndex);
                        else if(oleReader != null)
                            o = oleReader.GetValue(columnIndex);
                    }
                    catch(Exception){}

                    if(o != null)
                        return o.ToString();
                    else
                        return "";
                }
                else
                {

                    return "";
                }
            }
        }

        [System.Runtime.CompilerServices.IndexerName("GetByIndex")]
        public string this [string columnName]   // Indexer declaration
        {
            get
            {
                if(sqlReader == null && 
                    odbcReader == null && 
                    oleReader == null)
                    return "";

                int FieldCount = 0;
                if(sqlReader != null)
                {
                    FieldCount = sqlReader.FieldCount;
                }
                else if(odbcReader != null)
                {
                    FieldCount = odbcReader.FieldCount;
                }
                else if(oleReader != null)
                {
                    FieldCount = oleReader.FieldCount;
                }
                for(int x=0;x< FieldCount;x++)
                {
                    if((sqlReader != null && 
                        sqlReader.GetName(x).ToUpper() == 
                        columnName.ToUpper()) ||
                       (odbcReader != null && 
                        odbcReader.GetName(x).ToUpper() == 
                        columnName.ToUpper()) ||
                       (oleReader != null && 
                        oleReader.GetName(x).ToUpper() == 
                        columnName.ToUpper()))
                    {
                        
                        object o = null;
                        try
                        {
                            if(sqlReader != null)
                                o = sqlReader.GetValue(x);
                            else if(odbcReader != null)
                                o = odbcReader.GetValue(x);
                            else if(oleReader != null)
                                o = oleReader.GetValue(x);
                        }
                        catch(Exception){}

                        if(o != null)
                            return o.ToString();
                        else
                            return "";
                    }
                }
                return "";
            }
        }
        #endregion
    }

    #endregion
}
```



Here is an example usage of the database container:


```csharp
using ZiggyData;

class Test
{
    Test()
    {
        ZiggyDatabase db = new ZiggyDatabase();
        if(db.Connect("localhost","mydatabase","sa","sa_password"))
        {
            ZiggyRS rsFoo = db.GetRSFromQuery("SELECT * FROM foo");

            while(rsFoo.NextRow())
            {
                ZiggyRS rsZiggyware = 
                        db.GetRSFromQuery("SELECT * FROM Ziggyware WHERE fooID = " + 
                                  rsFoo["fooID"]);

                while(rsZiggyware.NextRow())
                {
                    Console.WriteLine(rsZiggyware["Test"]);
                }
                rsZiggyware.Close();
            }
            rsFoo.Close();

            db.Close();
        }
    }
}
```