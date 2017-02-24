---
layout: post
title:  "Android数据库操作"
date:   2017-02-04
categories: 知识点
tags: 数据持久化 SQLiteOpenHelper
---

* content
{:toc}

SQLiteOpenHelper是Android提供的一个管理数据库的工具类，用于管理数据库的创建和版本更新。Android的数据库操作可以使用标准的sql语句，也可以使用Android提供的方法。[数据库的更新操作](http://www.importnew.com/1324.html)




## 实体类

	public final class User {
	    @NonNull
	    private String mId;
	    @NonNull
	    private String mName;
	    @NonNull
	    private String mSex;
	
	    public User(String name, String sex, String id) {
	        mId = id;
	        mName = name;
	        mSex = sex;
	    }
	
	    public User(String name, String sex) {
	        this(name, sex, UUID.randomUUID().toString());
	    }
	
	    @NonNull
	    public String getId() {
	        return mId;
	    }
	
	    @NonNull
	    public String getName() {
	        return mName;
	    }
	
	    @NonNull
	    public String getSex() {
	        return mSex;
	    }
	
	    public User setmSex(@NonNull String mSex) {
	        this.mSex = mSex;
	        return this;
	    }
	
	    public boolean isEmpty() {
	        return MyUtil.stringIsNullOrEmpty(mName)
	                && MyUtil.stringIsNullOrEmpty(mSex);
	    }
	
	    @Override
	    public boolean equals(Object o) {
	        if (this == o) return true;
	        if (o == null || getClass() != o.getClass()) return false;
	        User user = (User) o;
	        return MyUtil.equal(user.getId(), mId)
	                && MyUtil.equal(user.getName(), mName)
	                && MyUtil.equal(user.getSex(), mSex);
	    }
	
	    @Override
	    public int hashCode() {
	        return MyUtil.hashCode(mId, mName, mSex);
	    }
	
	    @Override
	    public String toString() {
	        return "User with Name" + mName;
	    }
	}

## 工具类

	public class MyUtil {
	    public static <T> T checkNotNull(T reference) {
	        if (reference == null) {
	            throw new NullPointerException();
	        } else {
	            return reference;
	        }
	    }
	
	    public static <T> T checkNotNull(T reference, @Nullable Object errorMessage) {
	        if (reference == null) {
	            throw new NullPointerException(String.valueOf(errorMessage));
	        } else {
	            return reference;
	        }
	    }
	
	    public static boolean stringIsNullOrEmpty(@Nullable String string) {
	        return string == null || string.length() == 0;
	    }
	
	    public static int hashCode(@Nullable Object... objects) {
	        return Arrays.hashCode(objects);
	    }
	
	    public static boolean equal(@Nullable Object a, @Nullable Object b) {
	        return a == b || a != null && a.equals(b);
	    }
	}

## DbHelper

	public class DbHelper extends SQLiteOpenHelper {
	    Context context;
	    public static final int DATABASE_VERSION = 1;
	    public static final String DATABASE_NAME = "Users.db";
	    private static final String TEXT_TYPE = " TEXT";
	    private static final String INTEGER_TYPE = " INTEGER";
	    private static final String COMMA_SEP = ",";
	    private static final String SQL_CREATE_TABLE =
	            "CREATE TABLE " + UsersPersistenceContract.UserEntry.TABLE_NAME + " (" +
	                    UsersPersistenceContract.UserEntry._ID + TEXT_TYPE + " PRIMARY KEY" + COMMA_SEP +
	                    UsersPersistenceContract.UserEntry.COLUMN_NAME_ENTRY_ID + TEXT_TYPE + COMMA_SEP +
	                    UsersPersistenceContract.UserEntry.COLUMN_NAME_USER_NAME + TEXT_TYPE + COMMA_SEP +
	                    UsersPersistenceContract.UserEntry.COLUMN_NAME_USER_SEX + INTEGER_TYPE + " )";
	
	    public DbHelper(Context context) {
	        super(context, DATABASE_NAME, null, DATABASE_VERSION);
	        this.context = context;
	    }
	
	    @Override
	    public void onCreate(SQLiteDatabase db) {
	        db.execSQL(SQL_CREATE_TABLE);
	    }
	
	    @Override
	    public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) {
	
	    }
	}

## PersistenceContract

	public final  class UsersPersistenceContract {
	    private UsersPersistenceContract(){}
	
	    public static abstract class UserEntry implements BaseColumns{
	        public static final String TABLE_NAME = "User";
	        public static final String COLUMN_NAME_ENTRY_ID = "entryId";
	        public static final String COLUMN_NAME_USER_NAME = "userName";
	        public static final String COLUMN_NAME_USER_SEX = "userSex";
	    }
	}

## LocalDataSource

	public class UsersLocalDataSource {
	    DbHelper dbHelper;
	    //单例
	    private static UsersLocalDataSource INSTANCE;
	
	    private UsersLocalDataSource(Context context) {
	        MyUtil.checkNotNull(context);
	        dbHelper = new DbHelper(context);
	    }
	
	    public static UsersLocalDataSource getInstance(Context context) {
	        if (INSTANCE == null) {
	            INSTANCE = new UsersLocalDataSource(context);
	        }
	        return INSTANCE;
	    }
	
	    //增Android
	    public long saveUser(User user) {
	        long result = -1;
	        MyUtil.checkNotNull(user);
	        SQLiteDatabase db = dbHelper.getWritableDatabase();
	        ContentValues values = new ContentValues();
	        values.put(UsersPersistenceContract.UserEntry.COLUMN_NAME_ENTRY_ID, user.getId());
	        values.put(UsersPersistenceContract.UserEntry.COLUMN_NAME_USER_NAME, user.getName());
	        values.put(UsersPersistenceContract.UserEntry.COLUMN_NAME_USER_SEX, user.getSex());
	        //Android自带方式
	        result = db.insert(UsersPersistenceContract.UserEntry.TABLE_NAME, null, values);
	        db.close();
	        return result;
	    }
	
	    //增Sql
	    public void saveUserBySql(User user) {
	        MyUtil.checkNotNull(user);
	        SQLiteDatabase db = dbHelper.getWritableDatabase();
	        //sql
	        db.execSQL("INSERT INTO " + UsersPersistenceContract.UserEntry.TABLE_NAME + " (" +
	                UsersPersistenceContract.UserEntry.COLUMN_NAME_ENTRY_ID + "," +
	                UsersPersistenceContract.UserEntry.COLUMN_NAME_USER_NAME + "," +
	                UsersPersistenceContract.UserEntry.COLUMN_NAME_USER_SEX + ") values ('" +
	                user.getId() + "','" + user.getName() + "','" + user.getSex() + "' )");
	        db.close();
	    }
	
	    //增s Android
	    public long saveUsers(List<User> users) {
	        long result = 0;
	        MyUtil.checkNotNull(users);
	        SQLiteDatabase db = dbHelper.getWritableDatabase();
	        for (int i = 0; i < users.size(); i++) {
	            //android
	            ContentValues values = new ContentValues();
	            values.put(UsersPersistenceContract.UserEntry.COLUMN_NAME_ENTRY_ID, users.get(i).getId());
	            values.put(UsersPersistenceContract.UserEntry.COLUMN_NAME_USER_NAME, users.get(i).getName());
	            values.put(UsersPersistenceContract.UserEntry.COLUMN_NAME_USER_SEX, users.get(i).getSex());
	            long temp = db.insert(UsersPersistenceContract.UserEntry.TABLE_NAME, null, values);
	            if (temp != -1) {
	                result++;
	            }
	        }
	        db.close();
	        return result;
	    }
	
	    //增s Sql
	    public void saveUsersBySql(List<User> users) {
	        MyUtil.checkNotNull(users);
	        SQLiteDatabase db = dbHelper.getWritableDatabase();
	        for (int i = 0; i < users.size(); i++) {
	            //sql
	            db.execSQL("INSERT INTO " + UsersPersistenceContract.UserEntry.TABLE_NAME + " (" +
	                    UsersPersistenceContract.UserEntry.COLUMN_NAME_ENTRY_ID + "," +
	                    UsersPersistenceContract.UserEntry.COLUMN_NAME_USER_NAME + "," +
	                    UsersPersistenceContract.UserEntry.COLUMN_NAME_USER_SEX + ") values ('" +
	                    users.get(i).getId() + "','" + users.get(i).getName() + "','" + users.get(i).getSex() + "')");
	        }
	        db.close();
	    }
	
	    //删 Android
	    public long deleteUser(String userId) {
	        long result = 0;
	        SQLiteDatabase db = dbHelper.getWritableDatabase();
	        //android
	        result = db.delete(UsersPersistenceContract.UserEntry.TABLE_NAME, UsersPersistenceContract.UserEntry.COLUMN_NAME_ENTRY_ID + " LIKE ?", new String[]{userId});
	        db.close();
	        return result;
	    }
	
	    //删  sql
	    public void deleteUserBySql(String userId) {
	        SQLiteDatabase db = dbHelper.getWritableDatabase();
	        //sql
	        String sql = "DELETE FROM " + UsersPersistenceContract.UserEntry.TABLE_NAME + " WHERE " + UsersPersistenceContract.UserEntry.COLUMN_NAME_ENTRY_ID
	                + " = '" + userId + "'";
	        db.execSQL(sql);
	        db.close();
	    }
	
	    //删s Android
	    public long deleteAllUsers() {
	        long result = 0;
	        SQLiteDatabase db = dbHelper.getWritableDatabase();
	        //android
	        result = db.delete(UsersPersistenceContract.UserEntry.TABLE_NAME, null, null);
	        db.close();
	        return result;
	    }
	
	    //删s sql
	    public void deleteAllUsersBySql() {
	        SQLiteDatabase db = dbHelper.getWritableDatabase();
	        //sql
	        String sql = "DELETE FROM " + UsersPersistenceContract.UserEntry.TABLE_NAME;
	        db.execSQL(sql);
	        db.close();
	    }
	
	    //改 Android
	    public void updateUser(User user) {
	        SQLiteDatabase db = dbHelper.getWritableDatabase();
	        //android
	        ContentValues values = new ContentValues();
	        values.put(UsersPersistenceContract.UserEntry.COLUMN_NAME_USER_NAME, user.getName());
	        values.put(UsersPersistenceContract.UserEntry.COLUMN_NAME_USER_SEX, user.getSex());
	        db.update(UsersPersistenceContract.UserEntry.TABLE_NAME, values,
	                UsersPersistenceContract.UserEntry.COLUMN_NAME_ENTRY_ID + " LIKE ?", new String[]{user.getId()});
	        db.close();
	    }
	
	    //改 sql—1
	    public void updateUserBySqlNoPlaceholder(User user) {
	        SQLiteDatabase db = dbHelper.getWritableDatabase();
	        //SQL不带占位符
	        String sql_1 = "UPDATE " + UsersPersistenceContract.UserEntry.TABLE_NAME + " SET "
	                + UsersPersistenceContract.UserEntry.COLUMN_NAME_USER_NAME + " = '" + user.getName() + "','" +
	                UsersPersistenceContract.UserEntry.COLUMN_NAME_USER_SEX + "' = '" + user.getSex() +
	                "' WHERE " + UsersPersistenceContract.UserEntry.COLUMN_NAME_ENTRY_ID + " = '" + user.getId() + "'";
	        db.execSQL(sql_1);
	        db.close();
	    }
	
	    //改 sql-2
	    public void updateUserBySqlInPlaceholder(User user) {
	        SQLiteDatabase db = dbHelper.getWritableDatabase();
	        //SQL 带占位符
	        String sql_2 = "UPDATE " + UsersPersistenceContract.UserEntry.TABLE_NAME + " SET "
	                + UsersPersistenceContract.UserEntry.COLUMN_NAME_USER_NAME + " = ?," +
	                UsersPersistenceContract.UserEntry.COLUMN_NAME_USER_SEX + " = ?" + " WHERE " +
	                UsersPersistenceContract.UserEntry.COLUMN_NAME_ENTRY_ID + " = '" + user.getId() + "'";
	        db.execSQL(sql_2, new String[]{user.getName(), user.getSex()});
	        db.close();
	    }
	
	    //改s
	    public int updateUsers(List<User> users) {
	        int result = 0;
	        SQLiteDatabase db = dbHelper.getWritableDatabase();
	        for (int i = 0; i < users.size(); i++) {
	            ContentValues values = new ContentValues();
	            values.put(UsersPersistenceContract.UserEntry.COLUMN_NAME_USER_NAME, users.get(i).getName());
	            values.put(UsersPersistenceContract.UserEntry.COLUMN_NAME_USER_SEX, users.get(i).getSex());
	            int temp = db.update(UsersPersistenceContract.UserEntry.TABLE_NAME, values,
	                    UsersPersistenceContract.UserEntry.COLUMN_NAME_ENTRY_ID + " LIKE ?", new String[]{users.get(i).getId()});
	            result += temp;
	        }
	        db.close();
	        return result;
	    }
	
	    //查 android
	    public User queryUser(String userId) {
	        SQLiteDatabase db = dbHelper.getReadableDatabase();
	        String[] cs = {UsersPersistenceContract.UserEntry.COLUMN_NAME_ENTRY_ID,
	                UsersPersistenceContract.UserEntry.COLUMN_NAME_USER_NAME,
	                UsersPersistenceContract.UserEntry.COLUMN_NAME_USER_SEX
	        };
	        String selection = UsersPersistenceContract.UserEntry.COLUMN_NAME_ENTRY_ID + " LIKE ?";
	        Cursor cursor = db.query(UsersPersistenceContract.UserEntry.TABLE_NAME, cs,
	                selection, new String[]{userId}, null, null, null);
	        User user = null;
	        if (cursor != null && cursor.getCount() > 0) {
	            cursor.moveToFirst();
	            String id = cursor.getString(cursor.getColumnIndex(UsersPersistenceContract.UserEntry.COLUMN_NAME_ENTRY_ID));
	            String name = cursor.getString(cursor.getColumnIndex(UsersPersistenceContract.UserEntry.COLUMN_NAME_USER_NAME));
	            String sex = cursor.getString(cursor.getColumnIndex(UsersPersistenceContract.UserEntry.COLUMN_NAME_USER_SEX));
	            user = new User(name, sex, id);
	        }
	        return user;
	    }
	
	    //查  sql-1
	    public User queryUserBySqlNoPlaceholder(String userId) {
	        SQLiteDatabase db = dbHelper.getReadableDatabase();
	        String sql = "SELECT * FROM " + UsersPersistenceContract.UserEntry.TABLE_NAME + " WHERE "
	                + UsersPersistenceContract.UserEntry.COLUMN_NAME_ENTRY_ID + " = '" + userId + "'";
	        Cursor cursor = db.rawQuery(sql, null);
	        User user = null;
	        if (cursor != null && cursor.getCount() > 0) {
	            cursor.moveToFirst();
	            String id = cursor.getString(cursor.getColumnIndex(UsersPersistenceContract.UserEntry.COLUMN_NAME_ENTRY_ID));
	            String name = cursor.getString(cursor.getColumnIndex(UsersPersistenceContract.UserEntry.COLUMN_NAME_USER_NAME));
	            String sex = cursor.getString(cursor.getColumnIndex(UsersPersistenceContract.UserEntry.COLUMN_NAME_USER_SEX));
	            user = new User(name, sex, id);
	        }
	        return user;
	    }
	
	    //查 sql-2
	    public User queryUserBySqlInPlaceholder(String userId) {
	        SQLiteDatabase db = dbHelper.getReadableDatabase();
	        String sql = "SELECT * FROM " + UsersPersistenceContract.UserEntry.TABLE_NAME + " WHERE "
	                + UsersPersistenceContract.UserEntry.COLUMN_NAME_ENTRY_ID + " = ?";
	        Cursor cursor = db.rawQuery(sql, new String[]{userId});
	        User user = null;
	        if (cursor != null && cursor.getCount() > 0) {
	            cursor.moveToFirst();
	            String id = cursor.getString(cursor.getColumnIndex(UsersPersistenceContract.UserEntry.COLUMN_NAME_ENTRY_ID));
	            String name = cursor.getString(cursor.getColumnIndex(UsersPersistenceContract.UserEntry.COLUMN_NAME_USER_NAME));
	            String sex = cursor.getString(cursor.getColumnIndex(UsersPersistenceContract.UserEntry.COLUMN_NAME_USER_SEX));
	            user = new User(name, sex, id);
	        }
	        return user;
	    }
	
	    //查s android
	    public List<User> queryUsers() {
	        List<User> users = new ArrayList<>();
	        SQLiteDatabase db = dbHelper.getReadableDatabase();
	        String[] cs = {UsersPersistenceContract.UserEntry.COLUMN_NAME_ENTRY_ID,
	                UsersPersistenceContract.UserEntry.COLUMN_NAME_USER_NAME,
	                UsersPersistenceContract.UserEntry.COLUMN_NAME_USER_SEX
	        };
	        Cursor cursor = db.query(UsersPersistenceContract.UserEntry.TABLE_NAME, null, null, null, null, null, null);
	        if (cursor != null && cursor.getCount() > 0) {
	            for (cursor.moveToFirst(); !cursor.isAfterLast(); cursor.moveToNext()) {
	                String id = cursor.getString(cursor.getColumnIndex(UsersPersistenceContract.UserEntry.COLUMN_NAME_ENTRY_ID));
	                String name = cursor.getString(cursor.getColumnIndex(UsersPersistenceContract.UserEntry.COLUMN_NAME_USER_NAME));
	                String sex = cursor.getString(cursor.getColumnIndex(UsersPersistenceContract.UserEntry.COLUMN_NAME_USER_SEX));
	                users.add(new User(name, sex, id));
	            }
	        }
	        return users;
	    }
	
	    //查s sql
	    public List<User> queryUsersBySql() {
	        List<User> users = new ArrayList<>();
	        SQLiteDatabase db = dbHelper.getReadableDatabase();
	        Cursor cursor = db.rawQuery("SELECT * FROM " + UsersPersistenceContract.UserEntry.TABLE_NAME, null);
	        if (cursor != null && cursor.getCount() > 0) {
	            for (cursor.moveToFirst(); !cursor.isAfterLast(); cursor.moveToNext()) {
	                String id = cursor.getString(cursor.getColumnIndex(UsersPersistenceContract.UserEntry.COLUMN_NAME_ENTRY_ID));
	                String name = cursor.getString(cursor.getColumnIndex(UsersPersistenceContract.UserEntry.COLUMN_NAME_USER_NAME));
	                String sex = cursor.getString(cursor.getColumnIndex(UsersPersistenceContract.UserEntry.COLUMN_NAME_USER_SEX));
	                users.add(new User(name, sex, id));
	            }
	        }
	        return users;
	    }
	}

### 建表

    String createTableSql = "create table tablename (_id integer primary key autoincrement, key valueType)";

### 插入

sql:带占位符，不带列名的插入：

    db.execSQL("insert into tablename values(null,?,?)",new Stirng[]{title,content});

sql:不带占位符,不带列名的插入：

    String sql ="insert into tablename values (value1,value2,null,value4)";

sql:不带占位符,带列名的插入：

    String sql= "insert into tablename (key1,key2,key3) values (value1,value2,value3)";

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

### 删除

sql:

    String sql = "delete from useritem where sendUserQcCode='" + sendUserQcCode + "' and receiveUserQcCode='" + receiveUserQcCode + "'";

android:

    public int delete(java.lang.String table,
      java.lang.String whereClause,
      java.lang.String[] whereArgs)
	//Convenience method for deleting rows in the database.

### 修改

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

### 查询(query,rawQuery)

    String sql = "select keyname from tablename [where]"

	两个方法的返回值都为Cursor