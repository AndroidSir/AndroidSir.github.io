---
layout: post
title:  "Android数据库操作"
date:   2017-02-04
categories: 数据持久化
tags: Android sql SQLiteOpenHelper
---

* content
{:toc}

SQLiteOpenHelper是Android提供的一个管理数据库的工具类，用于管理数据库的创建和版本更新。Android的数据库操作可以使用标准的sql语句，也可以使用Android提供的方法。




### 建表：

    String createTableSql = "create table tablename (_id integer primary key autoincrement, key valueType)";

### 插入：

sql:带占位符的插入：

    db.execSQL("insert into tablename values(null,?,?)",new Stirng[]{title,content});

sql:不带占位符,带列名的插入：

    String sql= "insert into tablename (key1,key2,key3) values (value1,value2,value3)";

sql:不带占位符,不带列名的插入：

    String sql ="insert into tablename values (value1,value2,null,value4)";

android:

	public long insert(java.lang.String table,
                   java.lang.String nullColumnHack,
                   android.content.ContentValues values)
	//向数据库中插入一行
    Parameters:
    table - the table to insert the row into（表名）

    nullColumnHack - optional; may be null. SQL doesn't allow inserting a completely empty row without naming at least one column name. If your provided values is empty, no column names are known and an empty row can't be inserted. If not set to null, the nullColumnHack parameter provides the name of nullable column name to explicitly insert a NULL into in the case where your values is empty.

	当values参数为空或者里面没有内容的时候，insert是会失败的(底层数据库不允许插入一个空行)，为了防止这种情况，要在这里指定一个列名，到时候如果发现将要插入的行为空行时，就会将你指定的这个列名的值设为null，然后再向数据库中插入。通过观察源码的insertWithOnConflict方法可以看到当ContentValues类型的数据initialValues为null或size<=0时，就会在sql语句中添加nullColumnHack的设置。若不添加nullColumnHack则sql语句最终的结果将会类似insert into tableName()values();这是不允许的。若添加上nullColumnHack则sql语句将会变成insert into tableName (nullColumnHack)values(null);这是可以的。

    values - this map contains the initial column values for the row. The keys should be the column names and the values the column values

    Returns:
    the row ID of the newly inserted row, or -1 if an error occurred

### 删除：

sql:

    String sql = "delete from useritem where sendUserQcCode='" + sendUserQcCode + "' and receiveUserQcCode='" + receiveUserQcCode + "'";

android:

    public int delete(java.lang.String table,
      java.lang.String whereClause,
      java.lang.String[] whereArgs)
	//Convenience method for deleting rows in the database.

### 修改：

sql:带占位符的修改：

    db.execSQL("update tablename set name = ？ ,sex = ?where age>18",new String[]{newName,newSex} );

sql:不带占位符的修改：

    String sql = "update tablename set name = 'zdd', sex = '男' where age > 18";

android：

    public int update (String table, ContentValues values, String whereClause, String[] whereArgs)
    
    table（表名）
    the table to update in
    values（键值）
    a map from column names to new column values. null is a valid value that will be translated to NULL.
    whereClause（带占位符的条件表达式）
    the optional WHERE clause to apply when updating. Passing null will update all rows.
    whereArgs（条件表达式中的参数）
    You may include ?s in the where clause, which will be replaced by the values from whereArgs. The values will be bound as Strings.
    Returns（修改的记录条数）
    the number of rows affected
    //例子如下
    ContentValues cv = new ContentValues();
    cv.put("handleState", newState);
    String[] args = {sendUserQcCode, receiveUserQcCode};
    int i = db.update(ReceiveRoseDBHelper.TABLE_NAME_ROSE, cv, "sendUserQcCode=? AND receiveUserQcCode=?", args);

### 查询：(query,rawQuery)

    String sql = "select keyname from tablename [where]"

	两个方法的返回值都为Cursor