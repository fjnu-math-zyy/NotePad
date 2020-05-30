# NotePad
This is an AndroidStudio rebuild of google SDK sample NotePad
## Notepad主要目录结构

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
## 添加笔记查询功能
![Image text](https://raw.githubusercontent.com/fjnu-math-zyy/NotePad/master/img-folder/search_1.jpg)
![Image text](https://raw.githubusercontent.com/fjnu-math-zyy/NotePad/master/img-folder/search_2.jpg)
