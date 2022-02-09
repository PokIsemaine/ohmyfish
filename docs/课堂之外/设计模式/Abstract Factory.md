# Abstract Factory

## 动机

- 在软件系统中，经常面临着“一系列相互依赖的对象工作”；同时，由于需求的变化，往往存在更多系列对象的创建工作。
- 如何应对这种变化？如何绕过常规的对象创建方法(new)，提供一种“封装机制”来避免客户程序和这种“多系列具体对象创建工作”的紧耦合。

## 模式定义

提供一个接口，让该接口负责创建一系列”相关或者相互依赖的对象“，无需指定它们具体的类。 ——《设计模式》GoF

## SHOW ME THE CODE

以一个数据库查询相关类为例

```cpp
class EmployeeDAO{
    
public:
    vector<EmployeeDO> GetEmployees(){
        SqlConnection* connection =
            new SqlConnection();
        connection->ConnectionString = "...";

        SqlCommand* command =
            new SqlCommand();
        command->CommandText="...";
        command->SetConnection(connection);

        SqlDataReader* reader = command->ExecuteReader();
        while (reader->Read()){

        }

    }
};
```

目前的问题是，它用`new`的形式创建对象。如果我们是`sqlserver`数据库那就和`sqlserver`绑死在一起了，没法直接用其他的数据库

### 优化1：

为了解决这点，首先我们进行面向接口的编程

1. 创建接口，编写抽象类
2. 声明类型变为抽象基类
3. `new`采用`Factory Method`创建



```cpp
//数据库访问有关的基类
class IDBConnection{
    
};
class IDBConnectionFactory{
public:
    virtual IDBConnection* CreateDBConnection()=0;
};


class IDBCommand{
    
};
class IDBCommandFactory{
public:
    virtual IDBCommand* CreateDBCommand()=0;
};


class IDataReader{
    
};
class IDataReaderFactory{
public:
    virtual IDataReader* CreateDataReader()=0;
};


//支持SQL Server
class SqlConnection: public IDBConnection{
    
};
class SqlConnectionFactory:public IDBConnectionFactory{
    
};


class SqlCommand: public IDBCommand{
    
};
class SqlCommandFactory:public IDBCommandFactory{
    
};


class SqlDataReader: public IDataReader{
    
};
class SqlDataReaderFactory:public IDataReaderFactory{
    
};

//支持Oracle
class OracleConnection: public IDBConnection{
    
};

class OracleCommand: public IDBCommand{
    
};

class OracleDataReader: public IDataReader{
    
};

class EmployeeDAO{
    IDBConnectionFactory* dbConnectionFactory;
    IDBCommandFactory* dbCommandFactory;
    IDataReaderFactory* dataReaderFactory;
    
    
public:
    vector<EmployeeDO> GetEmployees(){
        IDBConnection* connection =
            dbConnectionFactory->CreateDBConnection();
        connection->ConnectionString("...");

        IDBCommand* command =
            dbCommandFactory->CreateDBCommand();
        command->CommandText("...");
        command->SetConnection(connection); //关联性

        IDBDataReader* reader = command->ExecuteReader(); //关联性
        while (reader->Read()){

        }

    }
};
```

问题差不多被解决了，但我们注意到`command`、`SetConnection`等操作具有关联性。这意味着未来我们向三个工厂传对象的时候需要使用同一系列的对象，你不可以`command`是`sqlserver`的，`connection`是`Oracal`的。

```cpp
IDBConnectionFactory* dbConnectionFactory;
IDBCommandFactory* dbCommandFactory;
IDataReaderFactory* dataReaderFactory;
```

那么我们如何去避免这种情况呢？本文章的抽象工厂正是为了处理这样的问题的。

### 优化2：

由于几个操作具有很强的相关性，我们考虑把三个工厂塞到一个里。高内聚，低耦合。

原来的

```cpp
class IDBConnection{
    
};
class IDBConnectionFactory{
public:
    virtual IDBConnection* CreateDBConnection()=0;
};


class IDBCommand{
    
};
class IDBCommandFactory{
public:
    virtual IDBCommand* CreateDBCommand()=0;
};


class IDataReader{
    
};
class IDataReaderFactory{
public:
    virtual IDataReader* CreateDataReader()=0;
};
```

替换为

```cpp
//数据库访问有关的基类
class IDBConnection{
    
};

class IDBCommand{
    
};

class IDataReader{
    
};


class IDBFactory{
public:
    virtual IDBConnection* CreateDBConnection()=0;
    virtual IDBCommand* CreateDBCommand()=0;
    virtual IDataReader* CreateDataReader()=0;
    
};
```



```cpp
//数据库访问有关的基类
class IDBConnection{
    
};

class IDBCommand{
    
};

class IDataReader{
    
};


class IDBFactory{
public:
    virtual IDBConnection* CreateDBConnection()=0;
    virtual IDBCommand* CreateDBCommand()=0;
    virtual IDataReader* CreateDataReader()=0;
    
};


//支持SQL Server
class SqlConnection: public IDBConnection{
    
};
class SqlCommand: public IDBCommand{
    
};
class SqlDataReader: public IDataReader{
    
};


class SqlDBFactory:public IDBFactory{
public:
    virtual IDBConnection* CreateDBConnection()=0;
    virtual IDBCommand* CreateDBCommand()=0;
    virtual IDataReader* CreateDataReader()=0;
 
};

//支持Oracle
class OracleConnection: public IDBConnection{
    
};

class OracleCommand: public IDBCommand{
    
};

class OracleDataReader: public IDataReader{
    
};



class EmployeeDAO{
    IDBFactory* dbFactory;
    
public:
    vector<EmployeeDO> GetEmployees(){
        IDBConnection* connection =
            dbFactory->CreateDBConnection();
        connection->ConnectionString("...");

        IDBCommand* command =
            dbFactory->CreateDBCommand();
        command->CommandText("...");
        command->SetConnection(connection); //关联性

        IDBDataReader* reader = command->ExecuteReader(); //关联性
        while (reader->Read()){

        }

    }
};
```





## 结构设计

* 红色表示稳定
* 蓝色表示第一个变化
* 绿色表示第二个变化

![image-20220208211018486](https://s2.loli.net/2022/02/08/WPNmTbgV31a2uQJ.png)

## 要点总结

- 如果没有应对”多系列对象创建“的需求变化，则没有必要使用Abstract Factory模式，这时候使用简单的工厂即可。
- ”系列对象“指的是在某一个特定系列的对象之间有相互依赖、或作用的关系。不同系列的对象之间不能相互依赖。
- Abstract Factory模式主要在于应用”新系列“的需求变动。其缺点在与难以应对”新对象“的需求变动。
  - 增加新系列（指`sqlserver`、`Oracal`这种）比较容易因为他是在变化部分修改，增加新对象（指`Command`、`Connect`这种）困难因为增加新对象改变的是抽象基类（稳定的部分）
- 我们可以认为`Factory Method`设计模式，是`Abstract Factory`的一个特例。`Factory Method`创建一个对象，`Abstract Factory`创建多个对象