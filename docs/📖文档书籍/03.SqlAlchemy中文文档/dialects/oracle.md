---
title: 甲骨文 oracle
date: 2021-02-20 22:41:37
permalink: /sqlalchemy/dialects/oracle/
categories:
  - 📖好书
  - SqlAlchemy 中文文档
  - dialects
tags:
---
甲骨文[¶ T0\>](#module-sqlalchemy.dialects.oracle.base "Permalink to this headline")
====================================================================================

支持 Oracle 数据库。

DBAPI 支持[¶](#dialect-oracle "Permalink to this headline")
----------------------------------------------------------

以下 dialect / DBAPI 选项可用。请参阅各个 DBAPI 部分的连接信息。

-   [CX-甲骨文 T0\>](#module-sqlalchemy.dialects.oracle.cx_oracle)
-   Jython 的[zxJDBC](#module-sqlalchemy.dialects.oracle.zxjdbc)

连接参数[¶](#connect-arguments "Permalink to this headline")
------------------------------------------------------------

The dialect supports several [`create_engine()`](core_engines.html#sqlalchemy.create_engine "sqlalchemy.create_engine")
arguments which affect the behavior of the dialect regardless of driver
in use.

-   `use_ansi` - 使用 ANSI JOIN 结构（请参阅 Oracle
    8 一节）。默认为`True`。如果`False`，则 Oracle-8 兼容构造用于连接。
-   `optimize_limits` - 默认为`False`。参见 LIMIT / OFFSET 部分。
-   `use_binds_for_limits` - 默认为`True`。参见 LIMIT / OFFSET 部分。

自动增量行为[¶](#auto-increment-behavior "Permalink to this headline")
----------------------------------------------------------------------

包含整数主键的 SQLAlchemy 表对象通常被认为具有“自动增量”行为，这意味着它们可以在 INSERT 时生成它们自己的主键值。由于 Oracle 没有“自动增量”功能，SQLAlchemy 依赖序列来生成这些值。With
the Oracle dialect, *a sequence must always be explicitly specified to
enable autoincrement*.
这与假定使用具有自动增量功能的数据库的大多数文档示例不同。要指定序列，请使用传递给 Column 构造的 sqlalchemy.schema.Sequence 对象：

    t = Table('mytable', metadata,plainplainplainplainplain
          Column('id', Integer, Sequence('id_seq'), primary_key=True),
          Column(...), ...
    )

当使用表反射时，这一步也是必需的，即 autoload = True：

    t = Table('mytable', metadata,plainplain
          Column('id', Integer, Sequence('id_seq'), primary_key=True),
          autoload=True
    )

标识符套管[¶](#identifier-casing "Permalink to this headline")
--------------------------------------------------------------

在 Oracle 中，数据字典使用大写文本表示所有不区分大小写的标识符名称。另一方面，SQLAlchemy 认为全小写标识符名称不区分大小写。在模式级通信期间，Oracle 方言将所有不区分大小写的标识符转换为和来自这两种格式，例如表和索引的反映。在 SQLAlchemy 侧使用大写名称表示区分大小写的标识符，并且 SQLAlchemy 将引用名称
-
这将导致与从 Oracle 接收的数据字典数据不匹配，因此除非标识名称已真正创建为区分大小写（即使用带引号的名称）
，所有小写字母名称都应该用在 SQLAlchemy 方面。

LIMIT / OFFSET 支持[¶](#limit-offset-support "Permalink to this headline")
-------------------------------------------------------------------------

Oracle 不支持 LIMIT 或 OFFSET 关键字。SQLAlchemy 与 ROWNUM 一起使用了一个包装子查询方法。The
exact methodology is taken from
[http://www.oracle.com/technology/oramag/oracle/06-sep/o56asktom.html](http://www.oracle.com/technology/oramag/oracle/06-sep/o56asktom.html)
.

有两个选项会影响其行为：

-   “FIRST
    ROWS()”优化关键字默认不使用。要启用此优化指令，请将`optimize_limits=True`指定为[`create_engine()`](core_engines.html#sqlalchemy.create_engine "sqlalchemy.create_engine")。
-   为极限/偏移量传递的值将作为绑定参数发送。一些用户已经观察到，如果值作为绑定发送并且不是字面呈现，Oracle 会产生一个糟糕的查询计划。要在 SQL 语句中逐字地显示限制/偏移值，请将`use_binds_for_limits=False`指定为[`create_engine()`](core_engines.html#sqlalchemy.create_engine "sqlalchemy.create_engine")。

当使用完全不同的窗口查询方法（即 ROW\_NUMBER()OVER（ORDER
BY））来提供 LIMIT /
OFFSET（注意大多数用户不会观察到此情况）时，某些用户报告了更好的性能。为了适应这种情况，用于 LIMIT
/
OFFSET 的方法可以完全替换。请参阅[http://www.sqlalchemy.org/trac/wiki/UsageRecipes/WindowFunctionsByDefault](http://www.sqlalchemy.org/trac/wiki/UsageRecipes/WindowFunctionsByDefault)中的配方，该配置安装了一个选择编译器，该选择编译器使用窗口函数替代极限/偏移量的生成。

返回支持[¶](#returning-support "Permalink to this headline")
------------------------------------------------------------

Oracle 数据库支持有限的 RETURNING 形式，以便从 INSERT，UPDATE 和 DELETE 语句中检索匹配行的结果集。Oracle 的 RETURNING..INTO 语法只支持返回一行，因为它依赖 OUT 参数才能正常工作。另外，受支持的 DBAPI 还有其他限制（请参阅[RETURNING
Support](#cx-oracle-returning)）。

SQLAlchemy 的“隐式返回”功能通常在 Oracle 后端启用，它在 INSERT 中使用 RETURNING，有时使用 UPDATE 语句来获取新生成的主键值和其他 SQL 默认值和表达式。默认情况下，“隐式返回”通常只会获取嵌入到 INSERT 中的单个`nextval(some_seq)`表达式的值，以便在 INSERT 语句中递增序列并同时返回值。要全面禁用此功能，请将`implicit_returning=False`指定为[`create_engine()`](core_engines.html#sqlalchemy.create_engine "sqlalchemy.create_engine")：

    engine = create_engine("oracle://scott:tiger@dsn",plainplain
                           implicit_returning=False)

隐式返回也可作为表选项在逐个表的基础上禁用：

    # Core Tableplainplain
    my_table = Table("my_table", metadata, ..., implicit_returning=False)


    # declarative
    class MyClass(Base):
        __tablename__ = 'my_table'
        __table_args__ = {"implicit_returning": False}

也可以看看

[RETURNING Support](#cx-oracle-returning) -
对隐式返回的额外 cx\_oracle 特定限制。

ON UPDATE CASCADE [¶](#on-update-cascade "Permalink to this headline")
----------------------------------------------------------------------

Oracle 没有本地 ON UPDATE
CASCADE 功能。基于触发器的解决方案可在[http://asktom.oracle.com/tkyte/update\_cascade/index.html](http://asktom.oracle.com/tkyte/update_cascade/index.html)中找到。

使用 SQLAlchemy ORM 时，ORM 手动发布级联更新的能力有限 - 使用“deferrable =
True”，initial
='deferred'“关键字参数指定 ForeignKey 对象，并在每个关系()上指定”passive\_updates
= False“。

Oracle 8 兼容性[¶](#oracle-8-compatibility "Permalink to this headline")
-----------------------------------------------------------------------

当检测到 Oracle 8 时，方言内部将其自身配置为以下行为：

-   use\_ansi 标志被设置为 False。这具有将所有 JOIN 短语转换为 WHERE 子句的效果，并且在 LEFT
    OUTER JOIN 使用 Oracle 的（+）运算符的情况下。
-   当使用[`Unicode`](core_type_basics.html#sqlalchemy.types.Unicode "sqlalchemy.types.Unicode")时，NVARCHAR2 和 NCLOB 数据类型不再生成为 DDL
    - 而是发出 VARCHAR2 和 CLOB。这是因为这些类型在 Oracle
    8 上似乎不能正常工作，即使它们可用。[`NVARCHAR`](core_type_basics.html#sqlalchemy.types.NVARCHAR "sqlalchemy.types.NVARCHAR")和[`NCLOB`](#sqlalchemy.dialects.oracle.NCLOB "sqlalchemy.dialects.oracle.NCLOB")类型将始终生成 NVARCHAR2 和 NCLOB。
-   在使用 cx\_oracle 时，“native
    unicode”模式被禁用，即 SQLAlchemy 在将所有 Python
    unicode 对象作为绑定参数传入之前将其编码为“string”。

同义词/ DBLINK 反射[¶](#synonym-dblink-reflection "Permalink to this headline")
------------------------------------------------------------------------------

When using reflection with Table objects, the dialect can optionally
search for tables indicated by synonyms, either in local or remote
schemas or accessed over DBLINK, by passing the flag
`oracle_resolve_synonyms=True` as a keyword argument
to the [`Table`](core_metadata.html#sqlalchemy.schema.Table "sqlalchemy.schema.Table")
construct:

    some_table = Table('some_table', autoload=True,plainplainplainplainplainplainplainplain
                                autoload_with=some_engine,
                                oracle_resolve_synonyms=True)

设置此标志时，不仅在`ALL_TABLES`视图中搜索给定的名称（如`some_table`），还会在`ALL_SYNONYMS`如果同义词位于并指向 DBLINK，则 oracle 方言知道如何使用 DBLINK 语法（例如`@dblink`）查找表的信息。

`oracle_resolve_synonyms` is accepted wherever
reflection arguments are accepted, including methods such as
[`MetaData.reflect()`](core_metadata.html#sqlalchemy.schema.MetaData.reflect "sqlalchemy.schema.MetaData.reflect")
and [`Inspector.get_columns()`](core_reflection.html#sqlalchemy.engine.reflection.Inspector.get_columns "sqlalchemy.engine.reflection.Inspector.get_columns").

如果同义词没有被使用，这个标志应该被禁用。

日期时间兼容性[¶](#datetime-compatibility "Permalink to this headline")
-----------------------------------------------------------------------

Oracle 没有称为`DATETIME`的数据类型，它只有`DATE`，它实际上可以存储日期和时间值。出于这个原因，Oracle 方言提供了一个类型[`oracle.DATE`](#sqlalchemy.dialects.oracle.DATE "sqlalchemy.dialects.oracle.DATE")，它是[`DateTime`](core_type_basics.html#sqlalchemy.types.DateTime "sqlalchemy.types.DateTime")的子类。这种类型没有特殊的行为，仅作为这种类型的“标记”出现；此外，当数据库列被反映并且类型被报告为`DATE`时，将使用支持时间的[`oracle.DATE`](#sqlalchemy.dialects.oracle.DATE "sqlalchemy.dialects.oracle.DATE")类型。

版本 0.9.4 中已更改：将[`oracle.DATE`](#sqlalchemy.dialects.oracle.DATE "sqlalchemy.dialects.oracle.DATE")添加到[`DateTime`](core_type_basics.html#sqlalchemy.types.DateTime "sqlalchemy.types.DateTime")子类中。这是一个变化，因为以前的版本会将`DATE`列反映为[`types.DATE`](core_type_basics.html#sqlalchemy.types.DATE "sqlalchemy.types.DATE")，它是[`Date`](core_type_basics.html#sqlalchemy.types.Date "sqlalchemy.types.Date")的子类。这里唯一的意义是检查在特殊 Python 翻译中使用的列类型或将模式迁移到其他数据库后端的方案。

Oracle 表选项[¶](#oracle-table-options "Permalink to this headline")
-------------------------------------------------------------------

CREATE TABLE 短语通过与[`Table`](core_metadata.html#sqlalchemy.schema.Table "sqlalchemy.schema.Table")结构结合使用，支持以下选项：

-   `ON COMMIT`：

        Table(plainplainplain
            "some_table", metadata, ...,
            prefixes=['GLOBAL TEMPORARY'], oracle_on_commit='PRESERVE ROWS')

版本 1.0.0 中的新功能

-   `COMPRESS`

         Table('mytable', metadata, Column('data', String(32)),plain
             oracle_compress=True)

         Table('mytable', metadata, Column('data', String(32)),
             oracle_compress=6)

        The ``oracle_compress`` parameter accepts either an integer compression
        level, or ``True`` to use the default compression level.

版本 1.0.0 中的新功能

Oracle 特定索引选项[¶](#oracle-specific-index-options "Permalink to this headline")
----------------------------------------------------------------------------------

### 位图索引[¶](#bitmap-indexes "Permalink to this headline")

您可以指定`oracle_bitmap`参数来创建位图索引而不是 B 树索引：

    Index('my_index', my_table.c.data, oracle_bitmap=True)plainplainplainplain

位图索引不能是唯一的，也不能被压缩。SQLAlchemy 不会检查这些限制，只有数据库会。

版本 1.0.0 中的新功能

### 索引压缩[¶](#index-compression "Permalink to this headline")

对于包含大量重复值的索引，Oracle 具有更高效的存储模式。使用`oracle_compress`参数打开密钥压缩：

    Index('my_index', my_table.c.data, oracle_compress=True)plainplain

    Index('my_index', my_table.c.data1, my_table.c.data2, unique=True,
           oracle_compress=1)

`oracle_compress`参数接受指定要压缩的前缀列数的整数，或者接受`True`以使用缺省值（非唯一索引的所有列，除最后一列外的所有列为唯一索引）。

版本 1.0.0 中的新功能

Oracle 数据类型[¶](#oracle-data-types "Permalink to this headline")
------------------------------------------------------------------

与所有 SQLAlchemy 方言一样，所有已知可用于 Oracle 的 UPPERCASE 类型都可以从顶级方言导入，无论它们源自[`sqlalchemy.types`](core_type_basics.html#module-sqlalchemy.types "sqlalchemy.types")还是来自当地方言：

    from sqlalchemy.dialects.oracle import \plainplainplainplainplainplain
                BFILE, BLOB, CHAR, CLOB, DATE, \
                DOUBLE_PRECISION, FLOAT, INTERVAL, LONG, NCLOB, \
                NUMBER, NVARCHAR, NVARCHAR2, RAW, TIMESTAMP, VARCHAR, \
                VARCHAR2

特定于 Oracle 的类型或具有特定于 Oracle 的构造参数如下所示：

 *class*`sqlalchemy.dialects.oracle.`{.descclassname}`BFILE`{.descname}(*length=None*)[¶](#sqlalchemy.dialects.oracle.BFILE "Permalink to this definition")
:   基础：[`sqlalchemy.types.LargeBinary`](core_type_basics.html#sqlalchemy.types.LargeBinary "sqlalchemy.types.LargeBinary")

    ` __初始化__  T0> （ T1> 长度=无 T2> ） T3> ¶ T4>`{.descname}plainplainplain
    :   *inherited from the* [`__init__()`](core_type_basics.html#sqlalchemy.types.LargeBinary.__init__ "sqlalchemy.types.LargeBinary.__init__")
        *method of* [`LargeBinary`](core_type_basics.html#sqlalchemy.types.LargeBinary "sqlalchemy.types.LargeBinary")

        构建一个LargeBinary类型。

        参数：

        **length**[¶](#sqlalchemy.dialects.oracle.BFILE.params.length) –
        optional, a length for the column for use in DDL statements, for
        those binary types that accept a length, such as the MySQL BLOB
        type.

*class* `sqlalchemy.dialects.oracle。`{.descclassname} `DATE`{.descname} （ *timezone = False* ） T5\> [¶ T6\>](#sqlalchemy.dialects.oracle.DATE "Permalink to this definition")
:   Bases: [`sqlalchemy.types.DateTime`](core_type_basics.html#sqlalchemy.types.DateTime "sqlalchemy.types.DateTime")

    提供oracle DATE类型。

    这种类型没有特殊的Python行为，除了它的子类[`types.DateTime`](core_type_basics.html#sqlalchemy.types.DateTime "sqlalchemy.types.DateTime")；这是为了适应Oracle
    `DATE`类型支持时间值的事实。

    版本0.9.4中的新功能

    ` __初始化__  T0> （ T1> 时区=假 T2> ） T3> ¶ T4>`{.descname}
    :   *继承自* [`DateTime`](core_type_basics.html#sqlalchemy.types.DateTime "sqlalchemy.types.DateTime")的
        [`__init__()`](core_type_basics.html#sqlalchemy.types.DateTime.__init__ "sqlalchemy.types.DateTime.__init__")
        **

        构建一个新的[`DateTime`](core_type_basics.html#sqlalchemy.types.DateTime "sqlalchemy.types.DateTime")。

        参数：

        **时区** [¶](#sqlalchemy.dialects.oracle.DATE.params.timezone) -
        布尔值。如果为True，并由后端支持，则会产生'TIMESTAMP WITH
        TIMEZONE'。对于不支持时区感知时间戳的后端，不起作用。

*class* `sqlalchemy.dialects.oracle。`{.descclassname} `DOUBLE_PRECISION`{.descname} （ *precision = None*，*scale = None*，*asdecimal = None* ） [¶](#sqlalchemy.dialects.oracle.DOUBLE_PRECISION "Permalink to this definition")
:   基础：[`sqlalchemy.types.Numeric`](core_type_basics.html#sqlalchemy.types.Numeric "sqlalchemy.types.Numeric")

*class* `sqlalchemy.dialects.oracle。`{.descclassname} `INTERVAL`{.descname} （ *day\_precision =无*，*second\_precision =无 T5\> ） T6\> [¶ T7\>](#sqlalchemy.dialects.oracle.INTERVAL "Permalink to this definition")*
:   基础：[`sqlalchemy.types.TypeEngine`](core_type_api.html#sqlalchemy.types.TypeEngine "sqlalchemy.types.TypeEngine")

     `__init__`{.descname}(*day\_precision=None*, *second\_precision=None*)[¶](#sqlalchemy.dialects.oracle.INTERVAL.__init__ "Permalink to this definition")plainplainplainplain
    :   构建一个INTERVAL。

        请注意，目前仅支持DAY TO
        SECOND间隔。这是由于缺少对可用DBAPI（cx\_oracle和zxjdbc）中YEAR
        TO MONTH间隔的支持。

        参数：

        -   **day\_precision**
            [¶](#sqlalchemy.dialects.oracle.INTERVAL.params.day_precision)
            - 日间精确度值。这是一天数字要存储的位数。默认为“2”
        -   **second\_precision**
            [¶](#sqlalchemy.dialects.oracle.INTERVAL.params.second_precision)
            - 第二个精度值。这是小数秒字段存储的位数。默认为“6”。

 *class*`sqlalchemy.dialects.oracle.`{.descclassname}`NCLOB`{.descname}(*length=None*, *collation=None*, *convert\_unicode=False*, *unicode\_error=None*, *\_warn\_on\_bytestring=False*)[¶](#sqlalchemy.dialects.oracle.NCLOB "Permalink to this definition")
:   基础：[`sqlalchemy.types.Text`](core_type_basics.html#sqlalchemy.types.Text "sqlalchemy.types.Text")

     `__init__`{.descname}(*length=None*, *collation=None*, *convert\_unicode=False*, *unicode\_error=None*, *\_warn\_on\_bytestring=False*)[¶](#sqlalchemy.dialects.oracle.NCLOB.__init__ "Permalink to this definition")plainplainplainplain
    :   *inherited from the* [`__init__()`](core_type_basics.html#sqlalchemy.types.String.__init__ "sqlalchemy.types.String.__init__")
        *method of* [`String`](core_type_basics.html#sqlalchemy.types.String "sqlalchemy.types.String")

        创建一个字符串保存类型。

        参数：

        -   **length**[¶](#sqlalchemy.dialects.oracle.NCLOB.params.length)
            – optional, a length for the column for use in DDL and CAST
            expressions. 如果没有发布`CREATE TABLE`，可以安全地省略。某些数据库可能需要用于DDL的`length`，并且在`CREATE TABLE`
            DDL时会引发异常如果包含没有长度的`VARCHAR`，则发布。值是否被解释为字节或字符是数据库特定的。
        -   **整理**
            [¶](#sqlalchemy.dialects.oracle.NCLOB.params.collation) -

            可选，用于DDL和CAST表达式的列级别排序规则。使用SQLite，MySQL和Postgresql支持的COLLATE关键字进行呈现。例如。：

                >>> from sqlalchemy import cast, select, String
                >>> print select([cast('some string', String(collation='utf8'))])
                SELECT CAST(:param_1 AS VARCHAR COLLATE utf8) AS anon_1

            0.8版新增：增加了对所有字符串类型的COLLATE支持。

        -   **convert\_unicode**
            [¶](#sqlalchemy.dialects.oracle.NCLOB.params.convert_unicode)
            -

            当设置为`True`时，[`String`](core_type_basics.html#sqlalchemy.types.String "sqlalchemy.types.String")类型将假定输入将作为Python
            `unicode`对象传递，结果以Python
            `unicode`对象。If the DBAPI in use does
            not support Python unicode (which is fewer and fewer these
            days), SQLAlchemy will encode/decode the value, using the
            value of the `encoding` parameter passed
            to [`create_engine()`](core_engines.html#sqlalchemy.create_engine "sqlalchemy.create_engine")
            as the encoding.

            当使用本地支持Python
            unicode对象的DBAPI时，通常不需要设置此标志。For columns that
            are explicitly intended to store non-ASCII data, the
            [`Unicode`](core_type_basics.html#sqlalchemy.types.Unicode "sqlalchemy.types.Unicode")
            or [`UnicodeText`](core_type_basics.html#sqlalchemy.types.UnicodeText "sqlalchemy.types.UnicodeText")
            types should be used regardless, which feature the same
            behavior of `convert_unicode` but also
            indicate an underlying column type that directly supports
            unicode, such as `NVARCHAR`.

            对于非常罕见的情况，Python `unicode`将由本地支持Python `unicode`的后端由SQLAlchemy编码/解码，值`force`可以在这里传递，这将导致无条件地使用SQLAlchemy的编码/解码服务。

        -   **unicode\_error**
            [¶](#sqlalchemy.dialects.oracle.NCLOB.params.unicode_error)
            -
            可选，一种用于处理Unicode转换错误的方法。行为与标准库的`string.decode()`函数的`errors`关键字参数相同。该标志要求将`convert_unicode`设置为`force` -
            否则，SQLAlchemy不保证处理unicode转换的任务。请注意，此标志为已经返回unicode对象的后端（大多数DBAPI所执行的操作）的后端操作增加了显着的性能开销。此标志只能用作从不同或损坏编码的列中读取字符串的最后手段。

*class* `sqlalchemy.dialects.oracle。`{.descclassname} `NUMBER`{.descname} （ *precision = None*，*scale = None*，*asdecimal = None* ） [¶](#sqlalchemy.dialects.oracle.NUMBER "Permalink to this definition")
:   基础：[`sqlalchemy.types.Numeric`](core_type_basics.html#sqlalchemy.types.Numeric "sqlalchemy.types.Numeric")，[`sqlalchemy.types.Integer`](core_type_basics.html#sqlalchemy.types.Integer "sqlalchemy.types.Integer")

 *class*`sqlalchemy.dialects.oracle.`{.descclassname}`LONG`{.descname}(*length=None*, *collation=None*, *convert\_unicode=False*, *unicode\_error=None*, *\_warn\_on\_bytestring=False*)[¶](#sqlalchemy.dialects.oracle.LONG "Permalink to this definition")
:   基础：[`sqlalchemy.types.Text`](core_type_basics.html#sqlalchemy.types.Text "sqlalchemy.types.Text")

     `__init__`{.descname}(*length=None*, *collation=None*, *convert\_unicode=False*, *unicode\_error=None*, *\_warn\_on\_bytestring=False*)[¶](#sqlalchemy.dialects.oracle.LONG.__init__ "Permalink to this definition")plainplainplainplain
    :   *inherited from the* [`__init__()`](core_type_basics.html#sqlalchemy.types.String.__init__ "sqlalchemy.types.String.__init__")
        *method of* [`String`](core_type_basics.html#sqlalchemy.types.String "sqlalchemy.types.String")

        创建一个字符串保存类型。

        参数：

        -   **length**[¶](#sqlalchemy.dialects.oracle.LONG.params.length)
            – optional, a length for the column for use in DDL and CAST
            expressions. 如果没有发布`CREATE TABLE`，可以安全地省略。某些数据库可能需要用于DDL的`length`，并且在`CREATE TABLE`
            DDL时会引发异常如果包含没有长度的`VARCHAR`，则发布。值是否被解释为字节或字符是数据库特定的。
        -   **整理**
            [¶](#sqlalchemy.dialects.oracle.LONG.params.collation) -

            可选，用于DDL和CAST表达式的列级别排序规则。使用SQLite，MySQL和Postgresql支持的COLLATE关键字进行呈现。例如。：

                >>> from sqlalchemy import cast, select, String
                >>> print select([cast('some string', String(collation='utf8'))])
                SELECT CAST(:param_1 AS VARCHAR COLLATE utf8) AS anon_1

            0.8版新增：增加了对所有字符串类型的COLLATE支持。

        -   **convert\_unicode**
            [¶](#sqlalchemy.dialects.oracle.LONG.params.convert_unicode)
            -

            当设置为`True`时，[`String`](core_type_basics.html#sqlalchemy.types.String "sqlalchemy.types.String")类型将假定输入将作为Python
            `unicode`对象传递，结果以Python
            `unicode`对象。If the DBAPI in use does
            not support Python unicode (which is fewer and fewer these
            days), SQLAlchemy will encode/decode the value, using the
            value of the `encoding` parameter passed
            to [`create_engine()`](core_engines.html#sqlalchemy.create_engine "sqlalchemy.create_engine")
            as the encoding.

            当使用本地支持Python
            unicode对象的DBAPI时，通常不需要设置此标志。For columns that
            are explicitly intended to store non-ASCII data, the
            [`Unicode`](core_type_basics.html#sqlalchemy.types.Unicode "sqlalchemy.types.Unicode")
            or [`UnicodeText`](core_type_basics.html#sqlalchemy.types.UnicodeText "sqlalchemy.types.UnicodeText")
            types should be used regardless, which feature the same
            behavior of `convert_unicode` but also
            indicate an underlying column type that directly supports
            unicode, such as `NVARCHAR`.

            对于非常罕见的情况，Python `unicode`将由本地支持Python `unicode`的后端由SQLAlchemy编码/解码，值`force`可以在这里传递，这将导致无条件地使用SQLAlchemy的编码/解码服务。

        -   **unicode\_error**
            [¶](#sqlalchemy.dialects.oracle.LONG.params.unicode_error) -
            可选，一种用于处理Unicode转换错误的方法。行为与标准库的`string.decode()`函数的`errors`关键字参数相同。该标志要求将`convert_unicode`设置为`force` -
            否则，SQLAlchemy不保证处理unicode转换的任务。请注意，此标志为已经返回unicode对象的后端（大多数DBAPI所执行的操作）的后端操作增加了显着的性能开销。此标志只能用作从不同或损坏编码的列中读取字符串的最后手段。

*class* `sqlalchemy.dialects.oracle。`{.descclassname} `RAW  长度=无 ） T5> ¶ T6>`{.descname}
:   基础：`sqlalchemy.types._Binary`

cx\_Oracle [¶ T0\>](#module-sqlalchemy.dialects.oracle.cx_oracle "Permalink to this headline")
----------------------------------------------------------------------------------------------

通过 cx-Oracle 驱动程序支持 Oracle 数据库。

### DBAPI [¶ T0\>](#dialect-oracle-cx_oracle-url "Permalink to this headline")

有关 cx-Oracle 的文档和下载信息（如果适用），请访问：[http://cx-oracle.sourceforge.net/](http://cx-oracle.sourceforge.net/)

### 连接[¶ T0\>](#dialect-oracle-cx_oracle-connect "Permalink to this headline")

连接字符串：

    oracle+cx_oracle://user:pass@host:port/dbname[?key=value&key=value...]plainplain

### 其他连接参数[¶](#additional-connect-arguments "Permalink to this headline")

与`dbname`连接时，使用 cx\_oracle
`makedsn()`函数将主机，端口和 dbname 标记转换为 TNS 名称。否则，主机令牌将直接作为 TNS 名称。

其他参数可以指定为 URL 上的查询字符串参数，也可以指定为[`create_engine()`](core_engines.html#sqlalchemy.create_engine "sqlalchemy.create_engine")的关键字参数：

-   `allow_twophase` -
    启用两阶段交易。默认为`True`。

-   `arraysize` -
    在游标上设置 cx\_oracle.arraysize 值，默认值为 50。这个设置对于 cx\_Oracle 很重要，因为 LOB 对象的内容只能在“实时”行中读取（例如在一批 50 行内）。

-   `auto_convert_lobs` - 默认为 True；请参阅[LOB
    Objects](#cx-oracle-lob)。

-   `auto_setinputsizes` -
    为所有绑定参数发出 cx\_oracle.setinputsizes()调用。这是 LOB 数据类型所必需的，但可以禁用以减少开销。默认为`True`。可以使用`exclude_setinputsizes`参数从此过程中排除特定类型。

-   `coerce_to_unicode` -
    详细信息请参阅[Unicode](#cx-oracle-unicode)。

-   `coerce_to_decimal` - 详细请参阅[Precision
    Numerics](#cx-oracle-numeric)。

-   `exclude_setinputsizes` - 要从“auto
    setinputsizes”功能中排除的字符串 DBAPI 类型名称的元组或列表。此处的类型名称必须与“cx\_Oracle”模块名称空间中找到的 DBAPI 类型匹配，例如 cx\_Oracle.UNICODE，cx\_Oracle.NCLOB 等。默认为`（STRING， UNICODE）`。

    版本 0.8 中的新功能可以通过 exclude\_setinputsizes 属性从 auto\_setinputsizes 功能中排除特定的 DBAPI 类型。

-   `mode` -
    给出 SYSDBA 或 SYSOPER 的字符串值，或者一个整数值。该值仅作为 URL 查询字符串参数使用。

-   `threaded` -
    启用对 cx\_oracle 连接的多线程访问。默认为`True`。请注意，这是 cx\_Oracle DBAPI 本身的相反默认值。

-   `service_name` -
    使用连接字符串（DSN）与`SERVICE_NAME`而不是`SID`的选项。当给出`database`部分时，不能传递它。例如。
    `oracle+cx_oracle://scott:tiger@host:1521/?service_name=hr`是一个有效的 url。该值仅作为 URL 查询字符串参数使用。

    版本 1.0.0 中的新功能

### Unicode 的[¶ T0\>](#unicode "Permalink to this headline")

从版本 5 开始，cx\_Oracle
DBAPI 完全支持 unicode，并且能够以本地方式将字符串结果作为 Python
unicode 对象返回。

在 Python 3 中使用时，cx\_Oracle 将所有字符串作为 Python
unicode 对象（即 Python 3 中的 plain `str`）返回。在 Python 2 中，它将以 Python
unicode 的形式返回那些类型为`NVARCHAR`或`NCLOB`的列值。对于类型为`VARCHAR`或其他非 Unicode 字符串类型的列值，它将以 Python 字符串（例如，字节串）的形式返回值。

cx\_Oracle SQLAlchemy 方言提供了两种不同的选项，用于在 Python
2 下将`VARCHAR`列值作为 Python unicode 对象返回：

-   cx\_Oracle
    DBAPI 能够无条件地使用输出类型处理程序将所有字符串结果强制转换为 Python
    unicode 对象。这样做的好处是，对于 cx\_Oracle 驱动程序级别的所有语句，unicode 转换是全局的，这意味着它可以与没有关联输入信息的原始文本 SQL 语句一起使用。然而，这个系统被认为会产生显着的性能开销，不仅因为它无条件地对所有字符串值生效，而且因为 Python
    2 下的 cx\_Oracle 似乎使用纯 Python 函数调用来执行解码操作，在 cPython 下比单独使用 C 函数要慢几个数量级。
-   SQLAlchemy 内置了 unicode 解码服务，当使用 SQLAlchemy 的 C 扩展时，这些函数不会使用任何 Python 函数调用，而且速度非常快。The
    disadvantage to this approach is that the unicode conversion only
    takes effect for statements where the [`Unicode`](core_type_basics.html#sqlalchemy.types.Unicode "sqlalchemy.types.Unicode")
    type or [`String`](core_type_basics.html#sqlalchemy.types.String "sqlalchemy.types.String")
    type with `convert_unicode=True` is explicitly
    associated with the result column.
    对于任何 ORM 或核心查询或 SQL 表达式以及指定输出列类型的[`text()`](core_sqlelement.html#sqlalchemy.sql.expression.text "sqlalchemy.sql.expression.text")结构，情况都是如此，因此绝大多数情况下这不是问题。然而，当发送一个完整的原始字符串给[`Connection.execute()`](core_connections.html#sqlalchemy.engine.Connection.execute "sqlalchemy.engine.Connection.execute")时，该输入信息不存在，除非字符串在[`text()`](core_sqlelement.html#sqlalchemy.sql.expression.text "sqlalchemy.sql.expression.text")结构中处理，输入信息。

从 SQLAlchemy
0.9.2 版开始，默认方法是使用 SQLAlchemy 的输入系统。除非用户明确需要，否则这会使 cx\_Oracle 的昂贵的 Python
2 方法停用。在 Python
3 下，SQLAlchemy 检测到 cx\_Oracle 本地返回 unicode 对象，并使用 cx\_Oracle 的系统。

要在 Python
2 下重新启用 cx\_Oracle 的输出类型处理程序，可以将`coerce_to_unicode=True`标志（0.9.4 中的新值）传递给[`create_engine()`](core_engines.html#sqlalchemy.create_engine "sqlalchemy.create_engine")：

    engine = create_engine("oracle+cx_oracle://dsn", coerce_to_unicode=True)plain

或者，如果不使用 cx\_Oracle 的本地处理程序，则可以使用[`text()`](core_sqlelement.html#sqlalchemy.sql.expression.text "sqlalchemy.sql.expression.text")功能来运行纯字符串 SQL 语句并以 Python
2 unicode 的形式获取`VARCHAR`

    from sqlalchemy import text, Unicodeplainplainplainplainplain
    result = conn.execute(
        text("select username from user").columns(username=Unicode))

版本 0.9.2 更改： cx\_Oracle 的 outputtypehandlers 不再用于 Python
2 中非 Unicode 数据类型的 unicode 结果，因为它们被确定为主要的性能瓶颈。而是使用 SQLAlchemy 自己的 unicode 工具。

版本 0.9.4 新增：添加了`coerce_to_unicode`标志，以重新启用 cx\_Oracle 的 outputtypehandler 并恢复到 0.9.2 之前的行为。

### 返回支持[¶](#cx-oracle-returning "Permalink to this headline")

cx\_oracle
DBAPI 支持 Oracle 已经有限的 RETURNING 支持的有限子集。通常情况下，结果只能保证最多返回一列；这是 SQLAlchemy 使用 RETURNING 获取主键关联序列值的典型情况。由于 cx\_oracle 缺少对更复杂的 RETURNING 场景所需的 OCI\_DATA\_AT\_EXEC
API 的支持，因此其他列表达式将以非确定的方式导致问题。

因此，通过完全禁用 RETURNING 支持可以提高稳定性；否则 SQLAlchemy 将使用 RETURNING 来获取新序列生成的主键。如[RETURNING
Support](#oracle-returning)所示：

    engine = create_engine("oracle://scott:tiger@dsn",plainplain
                           implicit_returning=False)

也可以看看

[http://docs.oracle.com/cd/B10501\_01/appdev.920/a96584/oci05bnd.htm\#420693](http://docs.oracle.com/cd/B10501_01/appdev.920/a96584/oci05bnd.htm#420693)
- 返回 OCI 文档

[http://sourceforge.net/mailarchive/message.php?msg\_id=31338136](http://sourceforge.net/mailarchive/message.php?msg_id=31338136)
- cx\_oracle 开发者评论

### LOB 对象[¶](#lob-objects "Permalink to this headline")

cx\_oracle 使用 cx\_oracle.LOB 对象返回 Oracle
LOB。SQLAlchemy 将它们转换为字符串，以便 Binary 类型的接口与其他后端的接口一致，并且在 result.fetchmany()和 result.fetchall()等场景中不需要与活动游标的链接。这意味着默认情况下，LOB 对象无条件地被 SQLAlchemy 完全取出，并且与活动光标的链接被中断。

要禁用此处理，请将`auto_convert_lobs=False`传递给[`create_engine()`](core_engines.html#sqlalchemy.create_engine "sqlalchemy.create_engine")。

### 两阶段事务支持[¶](#two-phase-transaction-support "Permalink to this headline")

两阶段事务是使用 XA 事务实现的，并且从 SQLAlchemy
0.8.0b2,0.7.10 开始，已知以最新版本的 cx\_Oracle 的基本工作方式工作。但是，该机制尚未被认为是有力的，应该仍然被视为实验性的。

特别是最近 5.1.2 的 cx\_Oracle
DBAPI 有一个关于两个阶段的错误，这个阶段阻止了一个特定的 DBAPI 连接在准备好的事务中以及传统的 DBAPI 使用模式中始终可用；因此，一旦通过`Connection.begin_prepared()`使用了特定的连接，底层 DBAPI 连接的所有后续用法都必须位于准备事务的上下文内。

[`Engine`](core_connections.html#sqlalchemy.engine.Engine "sqlalchemy.engine.Engine")的默认行为是维护一个 DBAPI 连接池。因此，由于上述故障，已在两阶段操作中使用并返回到池的 DBAPI 连接在非两阶段上下文中将不可用。为了避免这种情况，应用程序可以做出以下几种选择之一：

-   使用[`NullPool`](core_pooling.html#sqlalchemy.pool.NullPool "sqlalchemy.pool.NullPool")禁用连接池
-   确保正在使用的特定[`Engine`](core_connections.html#sqlalchemy.engine.Engine "sqlalchemy.engine.Engine")仅用于两阶段操作。绑定到包含`twophase=True`的 ORM [`Session`](orm_session_api.html#sqlalchemy.orm.session.Session "sqlalchemy.orm.session.Session")的[`Engine`](core_connections.html#sqlalchemy.engine.Engine "sqlalchemy.engine.Engine")将一致使用两阶段事务样式。
-   对于没有禁用池的临时两阶段操作，可以使用[`Connection.detach()`](core_connections.html#sqlalchemy.engine.Connection.detach "sqlalchemy.engine.Connection.detach")方法从连接池中清除正在使用的 DBAPI 连接。

在 0.8.0b2,0.7.10 版本中更改：已执行并测试了对 cx\_oracle 准备事务的支持。

### 精确数字[¶](#precision-numerics "Permalink to this headline")

SQLAlchemy 方言经历了很多步骤以确保十进制数字的发送和接收完全准确。可调用的“outputtypehandler”与每个检测数字类型并将其作为字符串值接收的 cx\_oracle 连接对象相关联，而不是直接接收 Python
`float`，然后将其传递给 Python `Decimal`在 cx\_oracle 方言下的[`Numeric`](core_type_basics.html#sqlalchemy.types.Numeric "sqlalchemy.types.Numeric")和[`Float`](core_type_basics.html#sqlalchemy.types.Float "sqlalchemy.types.Float")类型意识到这种行为，并且强制`Decimal`到`float`如果`asdecimal`标志为`False`（默认在[`Float`](core_type_basics.html#sqlalchemy.types.Float "sqlalchemy.types.Float")，[`Numeric`](core_type_basics.html#sqlalchemy.types.Numeric "sqlalchemy.types.Numeric")上可选）。

由于处理程序首先在所有情况下都强制`Decimal`，该功能会显着影响性能。如果不需要精度数字，则可以通过将标志`coerce_to_decimal=False`传递给[`create_engine()`](core_engines.html#sqlalchemy.create_engine "sqlalchemy.create_engine")来禁用十进制处理：

    engine = create_engine("oracle+cx_oracle://dsn", coerce_to_decimal=False)plainplainplain

New in version 0.7.6: Add the `coerce_to_decimal`
flag.

性能的另一种替代方法是使用[cdecimal](http://pypi.python.org/pypi/cdecimal/)库；有关其他注意事项，请参阅[`Numeric`](core_type_basics.html#sqlalchemy.types.Numeric "sqlalchemy.types.Numeric")。

该处理程序试图使用结果集列的“精度”和“比例”属性来最好地确定后续输入值是否应该被接收为`Decimal`而不是 int（在这种情况下，不会添加处理）。There are several
scenarios where
[OCI](http://www.oracle.com/technetwork/database/features/oci/index.html)
does not provide unambiguous data as to the numeric type, including some
situations where individual rows may return a combination of floating
point and integer values.
已经观察到“精确度”和“比例”的某些值来确定这种情况。当它发生时，outputtypehandler 接收到一个字符串，然后传递给一个处理函数，该函数检测每个返回值是否存在小数点，如果是，则转换为`Decimal`，否则转换为 int。The intention is that simple int-based
statements like “SELECT my\_seq.nextval() FROM DUAL” continue to return
ints and not `Decimal` objects, and that any kind of
floating point value is received as a string so that there is no
floating point loss of precision.

“存在小数点”逻辑本身对区域设置也很敏感。在[OCI](http://www.oracle.com/technetwork/database/features/oci/index.html)下，这由 NLS\_LANG 环境变量控制。首次连接时，方言进行测试以确定当前的“十进制”字符，对于欧洲语言环境，该字符可以是逗号“，”。从这一点开始，outputtypehandler 使用该字符来表示小数点。请注意，在处理带有不使用句点“。”作为十进制字符的区域设置的数字时，需要 cx\_oracle
5.0.3 或更高版本。

在版本 0.6.6 中更改：
outputtypehandler 支持 locale 使用逗号“，”字符表示小数点的情况。

zxjdbc [¶ T0\>](#module-sqlalchemy.dialects.oracle.zxjdbc "Permalink to this headline")
---------------------------------------------------------------------------------------

通过 zxJDBC 为 Jython 驱动程序支持 Oracle 数据库。

注意

当前版本的 SQLAlchemy 不支持 Jython。zxjdbc 方言应该被认为是实验性的。

### DBAPI [¶ T0\>](#dialect-oracle-zxjdbc-url "Permalink to this headline")

此数据库的驱动程序可在以下网址找到：[http://www.oracle.com/technetwork/database/features/jdbc/index-091264.html](http://www.oracle.com/technetwork/database/features/jdbc/index-091264.html)

### 连接[¶ T0\>](#dialect-oracle-zxjdbc-connect "Permalink to this headline")

连接字符串：

    oracle+zxjdbc://user:pass@host/dbnameplainplainplainplain
