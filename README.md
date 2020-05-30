# NotePad
This is an AndroidStudio rebuild of google SDK sample NotePad
## Notepad主要目录结构
![Image text](https://raw.githubusercontent.com/fjnu-math-zyy/NotePad/master/img-folder/menu.jpg)
## NoteList中显示条目增加时间戳显示
**因为NotePad自带有显示时间功能，所以只需要稍加修改就能实现**
* 1.首先找到NotePadProvider这个类,在类中定位到insert这个方法
```
@Override
public Uri insert(Uri uri, ContentValues initialValues) {
      ...
    }
```
* 2.可以看到这里insert方法中使用了System.currentTimeMillis()方法将获取到的时间保存到now中
```
// Gets the current system time in milliseconds
Long now = Long.valueOf(System.currentTimeMillis());
```
* 3.但是这样获取到的只是1970年到现在的秒数，所以我们要对其进行格式转换处理转换为我们看得懂的时间格式
```
 // Gets the current system time in milliseconds
Long now = Long.valueOf(System.currentTimeMillis());
SimpleDateFormat format = new SimpleDateFormat("yyyy.MM.dd HH:mm:ss");
String dateTime = format.format(now);
```
* 4.别忘了将下面的所有使用到now的地方改为dataTime
```
 // Gets the current system time in milliseconds
Long now = Long.valueOf(System.currentTimeMillis());

SimpleDateFormat format = new SimpleDateFormat("yyyy.MM.dd HH:mm:ss");
String dateTime = format.format(now);
// If the values map doesn't contain the creation date, sets the value to the current time.
if (values.containsKey(NotePad.Notes.COLUMN_NAME_CREATE_DATE) == false) {
    values.put(NotePad.Notes.COLUMN_NAME_CREATE_DATE, dateTime);
}

// If the values map doesn't contain the modification date, sets the value to the current
// time.
if (values.containsKey(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE) == false) {
    values.put(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, dateTime);
}
```
* 5.因为notelslist_item中只带有一个显示文本的TextView,所以要添加一个显示时间的TextView
```
<LinearLayout  xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/layout"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">
    <TextView xmlns:android="http://schemas.android.com/apk/res/android"
        android:id="@android:id/text1"
        android:layout_width="match_parent"
        android:layout_height="?android:attr/listPreferredItemHeight"
        android:textAppearance="?android:attr/textAppearanceLarge"
        android:gravity="center_vertical"
        android:paddingLeft="5dip"
        android:textColor="@color/black"
        android:singleLine="true"
        />
    <TextView
        android:id="@+id/text1_time"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:textAppearance="?android:attr/textAppearanceSmall"
        android:paddingLeft="5dp"
        android:textColor="@color/black"/>
</LinearLayout>
```
* 6.再到NoteList类中添加一些参数即可

**在PROJECTION中添加NotePad.Notes.COLUMN_NAME_CREATE_DATE**
```
private static final String[] PROJECTION = new String[] {
            NotePad.Notes._ID, // 0
            NotePad.Notes.COLUMN_NAME_TITLE, // 1
            NotePad.Notes.COLUMN_NAME_CREATE_DATE
    };
```
**在dataColums中添加NotePad.Notes.COLUMN_NAME_CREATE_DATE**
```
// The names of the cursor columns to display in the view, initialized to the title column
String[] dataColumns = { NotePad.Notes.COLUMN_NAME_TITLE,NotePad.Notes.COLUMN_NAME_CREATE_DATE} ;
```
**在viewIDs添加我们刚刚创建的TextView的ID,R.id.text1_time**
```
// The view IDs that will display the cursor columns, initialized to the TextView in
// noteslist_item.xml
int[] viewIDs = { android.R.id.text1, R.id.text1_time };
```
* 7.现在可以显示创建时间了

![Image text](https://raw.githubusercontent.com/fjnu-math-zyy/NotePad/master/img-folder/time.jpg)
* 8.虽然时间可以正常显示，但是对笔记做出修改时，时间无法正常更新，依然显示的是创建时的时间，所以我们如果想要修改笔记后时间也同步更新的话，只需要做如下更改即可

**在NotePadProvider这个类中定位到update这个方法,并且加入以下内容即可**
```
 // Gets the current system time in milliseconds
Long now = Long.valueOf(System.currentTimeMillis());

SimpleDateFormat format = new SimpleDateFormat("yyyy.MM.dd HH:mm:ss");
String dateTime = format.format(now);
 // If the values map doesn't contain the creation date, sets the value to the current time.
if (values.containsKey(NotePad.Notes.COLUMN_NAME_CREATE_DATE) == false) {
    values.put(NotePad.Notes.COLUMN_NAME_CREATE_DATE, dateTime);
}

// If the values map doesn't contain the modification date, sets the value to the current
// time.
if (values.containsKey(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE) == false) {
    values.put(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, dateTime);
}
```
**加入后完整代码如下**
```
@Override
    public int update(Uri uri, ContentValues values, String where, String[] whereArgs) {

        // Opens the database object in "write" mode.
        SQLiteDatabase db = mOpenHelper.getWritableDatabase();
        int count;
        String finalWhere;

        // Gets the current system time in milliseconds
        Long now = Long.valueOf(System.currentTimeMillis());

        SimpleDateFormat format = new SimpleDateFormat("yyyy.MM.dd HH:mm:ss");
        String dateTime = format.format(now);
        // If the values map doesn't contain the creation date, sets the value to the current time.
        if (values.containsKey(NotePad.Notes.COLUMN_NAME_CREATE_DATE) == false) {
            values.put(NotePad.Notes.COLUMN_NAME_CREATE_DATE, dateTime);
        }

        // If the values map doesn't contain the modification date, sets the value to the current
        // time.
        if (values.containsKey(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE) == false) {
            values.put(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, dateTime);
        }

        // Does the update based on the incoming URI pattern
        switch (sUriMatcher.match(uri)) {

            // If the incoming URI matches the general notes pattern, does the update based on
            // the incoming data.
            case NOTES:

                // Does the update and returns the number of rows updated.
                count = db.update(
                    NotePad.Notes.TABLE_NAME, // The database table name.
                    values,                   // A map of column names and new values to use.
                    where,                    // The where clause column names.
                    whereArgs                 // The where clause column values to select on.
                );
                break;

            // If the incoming URI matches a single note ID, does the update based on the incoming
            // data, but modifies the where clause to restrict it to the particular note ID.
            case NOTE_ID:
                // From the incoming URI, get the note ID
                String noteId = uri.getPathSegments().get(NotePad.Notes.NOTE_ID_PATH_POSITION);

                /*
                 * Starts creating the final WHERE clause by restricting it to the incoming
                 * note ID.
                 */
                finalWhere =
                        NotePad.Notes._ID +                              // The ID column name
                        " = " +                                          // test for equality
                        uri.getPathSegments().                           // the incoming note ID
                            get(NotePad.Notes.NOTE_ID_PATH_POSITION)
                ;

                // If there were additional selection criteria, append them to the final WHERE
                // clause
                if (where !=null) {
                    finalWhere = finalWhere + " AND " + where;
                }


                // Does the update and returns the number of rows updated.
                count = db.update(
                    NotePad.Notes.TABLE_NAME, // The database table name.
                    values,                   // A map of column names and new values to use.
                    finalWhere,               // The final WHERE clause to use
                                              // placeholders for whereArgs
                    whereArgs                 // The where clause column values to select on, or
                                              // null if the values are in the where argument.
                );
                break;
            // If the incoming pattern is invalid, throws an exception.
            default:
                throw new IllegalArgumentException("Unknown URI " + uri);
        }
```
* 9.这样在修改时，时间也会同步更新了

* **选择第一条笔记**

![Image text](https://raw.githubusercontent.com/fjnu-math-zyy/NotePad/master/img-folder/change_time.jpg)

* **随便添加一条信息，点击保存**

![Image text](https://raw.githubusercontent.com/fjnu-math-zyy/NotePad/master/img-folder/change_time2.jpg)

* **时间同步修改了**

![Image text](https://raw.githubusercontent.com/fjnu-math-zyy/NotePad/master/img-folder/change_time3.jpg)
## 添加笔记查询功能
* 1.要添加笔记查询功能，就要在应用中增加一个搜索的入口。找到菜单的xml文件，list_options_menu.xml，添加一个搜索的item，搜索图标用安卓自带的图标，设为总是显示：
```
<?xml version="1.0" encoding="utf-8"?>
<menu xmlns:android="http://schemas.android.com/apk/res/android">
    <!--  This is our one standard application action (creating a new note). -->
    <item android:id="@+id/menu_add"
          android:icon="@drawable/ic_menu_compose"
          android:title="@string/menu_add"
          android:alphabeticShortcut='a'
          android:showAsAction="always" />
    <!--  If there is currently data in the clipboard, this adds a PASTE menu item to the menu
          so that the user can paste in the data.. -->
    <item android:id="@+id/menu_paste"
          android:icon="@drawable/ic_menu_compose"
          android:title="@string/menu_paste"
          android:alphabeticShortcut='p' />
    <item
        android:id="@+id/menu_search"
        android:title="@string/menu_search"
        android:icon="@android:drawable/ic_search_category_default"
        android:showAsAction="always">

    </item>
</menu>
```
* 2.在NoteList中找到onOptionsItemSelected方法，在switch中添加搜索的case语句:
```
case R.id.menu_search:
                Intent intent = new Intent();
                intent.setClass(NotesList.this,NoteSearch.class);
                NotesList.this.startActivity(intent);
                return true;
```
* 3.在case语句中写跳转activity代码之前要先写好搜索的activity，新建一个名为NoteSearch的activity。由于搜索出来的也是笔记列表，所以可以模仿NoteList的activity继承ListActivity。在安卓中有个用于搜索控件：SearchView，可以把SearchView跟ListView相结合，动态地显示搜索结果。先布局搜索页面，在layout中新建布局文件note_search_list.xml：
```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical" android:layout_width="match_parent"
    android:layout_height="match_parent">
    <SearchView
        android:id="@+id/search_view"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:iconifiedByDefault="false"
        android:queryHint="输入搜索内容..."
        android:layout_alignParentTop="true">
    </SearchView>
    <ListView
        android:id="@android:id/list"
        android:layout_width="match_parent"
        android:layout_height="wrap_content">
    </ListView>
</LinearLayout>
```
* 4.在上面的ListView的id命名方式与往常“@+id/”的方式有些不同，之前用“@+id/”方式尝试过，但是运行会出错，可能是由于是继承ListAcitivity的缘故。
要动态地显示搜索结果，就要对SearchView文本变化设置监听，NoteSearch除了要继承ListView外还要实现SearchView.OnQueryTextListener接口：
```
public class NoteSearch extends ListActivity  implements SearchView.OnQueryTextListener {
    private static final String[] PROJECTION = new String[] {
            NotePad.Notes._ID, // 0
            NotePad.Notes.COLUMN_NAME_TITLE, // 1
            //扩展 显示时间 颜色
            NotePad.Notes.COLUMN_NAME_CREATE_DATE, // 2

    };
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.note_search_list);
        Intent intent = getIntent();
        if (intent.getData() == null) {
            intent.setData(NotePad.Notes.CONTENT_URI);
        }
        SearchView searchview = (SearchView)findViewById(R.id.search_view);
        //为查询文本框注册监听器
        searchview.setOnQueryTextListener(NoteSearch.this);
    }
    @Override
    public boolean onQueryTextSubmit(String query) {
        return false;
    }
    @Override
    public boolean onQueryTextChange(String newText) {
        String selection = NotePad.Notes.COLUMN_NAME_TITLE + " Like ? ";
        String[] selectionArgs = { "%"+newText+"%" };
        Cursor cursor = managedQuery(
                getIntent().getData(),            // Use the default content URI for the provider.
                PROJECTION,                       // Return the note ID and title for each note. and modifcation date
                selection,                        // 条件左边
                selectionArgs,                    // 条件右边
                NotePad.Notes.DEFAULT_SORT_ORDER  // Use the default sort order.
        );
        String[] dataColumns = { NotePad.Notes.COLUMN_NAME_TITLE ,  NotePad.Notes.COLUMN_NAME_CREATE_DATE };
        int[] viewIDs = { android.R.id.text1 , R.id.text1_time };
        SimpleCursorAdapter adapter = new SimpleCursorAdapter(
                this,
                R.layout.noteslist_item,
                cursor,
                dataColumns,
                viewIDs
        );
        setListAdapter(adapter);
        return true;
    }
    @Override
    protected void onListItemClick(ListView l, View v, int position, long id) {
        // Constructs a new URI from the incoming URI and the row ID
        Uri uri = ContentUris.withAppendedId(getIntent().getData(), id);
        // Gets the action from the incoming Intent
        String action = getIntent().getAction();
        // Handles requests for note data
        if (Intent.ACTION_PICK.equals(action) || Intent.ACTION_GET_CONTENT.equals(action)) {
            // Sets the result to return to the component that called this Activity. The
            // result contains the new URI
            setResult(RESULT_OK, new Intent().setData(uri));
        } else {
            // Sends out an Intent to start an Activity that can handle ACTION_EDIT. The
            // Intent's data is the note ID URI. The effect is to call NoteEdit.
            startActivity(new Intent(Intent.ACTION_EDIT, uri));
        }
    }
}
```
* 5.动态搜索的实现最主要的部分在onQueryTextChange方法中，在使用这个方法，要先为SearchView注册监听：
```
SearchView searchview = (SearchView)findViewById(R.id.search_view);
        //为查询文本框注册监听器
        searchview.setOnQueryTextListener(NoteSearch.this);
```
* 6.而onQueryTextChange方法作用是，当SearchView中文本发生变化时，执行其中代码，搜索还有一个重要的部分就是要做到模糊匹配而不是严格匹配，可以使用数据库查询语句中的LIKE和%结合来实现，newText为输入搜索的内容：
```
String[] selectionArgs = { "%"+newText+"%" };
```
* 7.最后要在AndroidManifest.xml注册NoteSearch：
```
<activity android:name=".NoteSearch"
                android:label="NotSearch"
            </activity>
```
* 8.这样就可以实现动态搜索功能:

![Image text](https://raw.githubusercontent.com/fjnu-math-zyy/NotePad/master/img-folder/search_1.jpg)
![Image text](https://raw.githubusercontent.com/fjnu-math-zyy/NotePad/master/img-folder/search_2.jpg)
