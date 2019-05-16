# NotePad
期中实验NotePad
一.NoteList中显示条目增加时间显示
1.在notelist_item.xml,增加一段代码显示时间的textview
```
<TextView
        android:id="@+id/text1_time"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:textAppearance="?android:attr/textAppearanceSmall"
        android:paddingLeft="5dip"
        android:textColor="@color/colorBlue"/>
```
2.查看数据库结构，在NotePadProvider.java中,存在时间信息
```
public void onCreate(SQLiteDatabase db) {
           db.execSQL("CREATE TABLE " + NotePad.Notes.TABLE_NAME + " ("
                   + NotePad.Notes._ID + " INTEGER PRIMARY KEY,"
                   + NotePad.Notes.COLUMN_NAME_TITLE + " TEXT,"
                   + NotePad.Notes.COLUMN_NAME_NOTE + " TEXT,"
                   + NotePad.Notes.COLUMN_NAME_CREATE_DATE + " INTEGER,"
                   + NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE + " INTEGER"
                   + ");");
    }
```
3.在NoteList.java中，查看数据被填到如何被填到列表里,并增加修改时间
```
private static final String[] PROJECTION = new String[] {
        NotePad.Notes._ID, // 0
        NotePad.Notes.COLUMN_NAME_TITLE, // 1
        NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE,//在这里加入了修改时间的显示
};
```
4.在装配的时候需要装配上相应的日期,因此NoteList.java中在dataColumns,viewIDs这两个参数需要加入时间
```
String[] dataColumns = { NotePad.Notes.COLUMN_NAME_TITLE ,NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE } ;

        // The view IDs that will display the cursor columns, initialized to the TextView in
        // noteslist_item.xml
        int[] viewIDs = { android.R.id.text1,R.id.text1_time };
```
5.到此为止，我们屏幕上会显示一串数字，我们要将这串数字修改为时间格式
在创建时显示的时间是在NotePadProvider的insert函数实现的，所以我们要对它进行修改
```
 // Gets the current system time in milliseconds
        Long now = Long.valueOf(System.currentTimeMillis());
       //Date date = new Date(now);
       SimpleDateFormat sdf= new SimpleDateFormat("yyyy.MM.dd HH:mm:ss");
        Log.d(TAG, "insert:"+sdf.format(new Date(now)));

        String nowTime = sdf.format(new Date(now));
```
以及对代码做如下的修改,将now换成nowTime
```
 if (values.containsKey(NotePad.Notes.COLUMN_NAME_CREATE_DATE) == false) {
            values.put(NotePad.Notes.COLUMN_NAME_CREATE_DATE, nowTime);
        }

        // If the values map doesn't contain the modification date, sets the value to the current
        // time.
        if (values.containsKey(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE) == false) {
            values.put(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE,nowTime);
        }

```
6.在NoteEditor.java的函数updateNote函数里也要做如下修改
```
 Long now = Long.valueOf(System.currentTimeMillis());
        //Date date = new Date(now);
        SimpleDateFormat sdf= new SimpleDateFormat("yyyy.MM.dd HH:mm:ss");
        Log.d(TAG, "insert:"+sdf.format(new Date(now)));

        String nowTime = sdf.format(new Date(now));
        // Sets up a map to contain values to be updated in the provider.
        ContentValues values = new ContentValues();
        values.put(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, nowTime);

```
时间格式显示截图:
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190516173637206.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQwOTQ2MDUz,size_16,color_FFFFFF,t_70)二  添加笔记查询功能(根据标题查询)
1.在应用中增加一个搜索的入口,添加一个搜索的item，搜索图标用安卓自带的图标，设为总是显示,这时候查询按钮就会显示在
在list_options_menu.xml中增加NotesList的主界面上 
```
 <item
        android:id="@+id/menu_search"
        android:title="@string/menu_search"
        android:icon="@android:drawable/ic_search_category_default"
        android:showAsAction="always">
    </item>
```
2. 在NoteList中找到onOptionsItemSelected方法，在switch中添加搜索的case语句:
```
 //添加搜素
            case R.id.menu_search:
                Intent intent = new Intent();
                intent.setClass(NotesList.this,NoteSearch.class);
                NotesList.this.startActivity(intent);
                return true;
```
3.新建一个命名为NoteSearch的activity
```
package com.example.android.notepad;

import android.app.ListActivity;
import android.content.Intent;
import android.content.ContentUris;
import android.database.Cursor;
import android.os.Bundle;
import android.net.Uri;
import android.view.View;
import android.widget.ListView;
import android.widget.SearchView;
import android.widget.SimpleCursorAdapter;
import android.widget.SearchView.OnQueryTextListener;
import com.example.android.notepad.NoteSearch;
public class NoteSearch extends ListActivity  implements SearchView.OnQueryTextListener {
    //定义一个数组来存放数据
    private static final String[] PROJECTION=new String[]{
            NotePad.Notes._ID, // 0
            NotePad.Notes.COLUMN_NAME_TITLE, // 1
            NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE,//在这里加入了修改时间的显示
    };
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.note_search_list);
        //获取Intent
        Intent intent = getIntent();
        if (intent.getData() == null) {
            intent.setData(NotePad.Notes.CONTENT_URI);
        }
        SearchView searchView = (SearchView) findViewById(R.id.search_view);
        //为查询文本框注册监听器
        searchView.setOnQueryTextListener(NoteSearch.this);
    }
    @Override
    public boolean onQueryTextSubmit(String query) {
        return false;
    }
    @Override
    public boolean onQueryTextChange(String newText) {
        //Text发生改变时执行的内容
        //查询条件
        String selection = NotePad.Notes.COLUMN_NAME_TITLE + " Like ? ";
        //查询条件参数，配合selection参数使用,%通配多个字符
        String[] selectionArgs = { "%"+newText+"%" };
        //查询数据库中的内容,当我们使用 SQLiteDatabase.query()方法时，就会得到Cursor对象， Cursor所指向的就是每一条数据。
        Cursor cursor = managedQuery(
                //用于ContentProvider查询的URI，从这个URI获取数据
                getIntent().getData(),
                //用于标识uri中有哪些columns需要包含在返回的Cursor对象中
                PROJECTION,                       // Return the note ID and title for each note. and modifcation date
                selection,                        // 条件左边
                selectionArgs,                    // 条件右边
                NotePad.Notes.DEFAULT_SORT_ORDER  // Use the default sort order.
        );
        String[] dataColumns = { NotePad.Notes.COLUMN_NAME_TITLE ,  NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE };
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
4.在layout中新建一个布局文件note_search_list.xml
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
笔记查询功能截图:
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190516175348806.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQwOTQ2MDUz,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190516175407892.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQwOTQ2MDUz,size_16,color_FFFFFF,t_70)
三 拓展功能，文本编辑界面中可以改变改变字体大小，颜色
1.在editor_options_menu.xml中添加对应的菜单
```
<item
        android:id="@+id/font_size"
        android:title="@string/font_size">
        <!--子菜单-->
        <menu>
            <!--定义一组单选菜单项-->
            <group>
                <!--定义多个菜单项-->
                <item
                    android:id="@+id/font_10"
                    android:title="@string/font10"
                    />

                <item
                    android:id="@+id/font_16"
                    android:title="@string/font16" />
                <item
                    android:id="@+id/font_20"
                    android:title="@string/font20" />
            </group>
        </menu>
    </item>

    <item
        android:title="@string/font_color"
        android:id="@+id/font_color"
        >
        <menu>
            <!--定义一组普通菜单项-->
            <group>
                <!--定义两个菜单项-->
                <item
                    android:id="@+id/red_font"
                    android:title="@string/red_title" />
                <item
                    android:title="@string/yellow_title"
                    android:id="@+id/yellow_font"/>
            </group>
        </menu>
    </item>

```
菜单就可以在文本编辑界面出现
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190516190315683.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQwOTQ2MDUz,size_16,color_FFFFFF,t_70)
2.NoteEditor的onOptionsItemSelected(MenuItem item)中添加相应的case来相应事件
```
case R.id.font_10:
             mText.setTextSize(20);
             Toast toast=Toast.makeText(NoteEditor.this,"修改成功",Toast.LENGTH_SHORT);
             toast.show();
             break;
          case R.id.font_16:
             mText.setTextSize(32);
                Toast toast2=Toast.makeText(NoteEditor.this,"修改成功",Toast.LENGTH_SHORT);
                toast2.show();
                break;
            case R.id.font_20:
                mText.setTextSize(40);
                Toast toast3=Toast.makeText(NoteEditor.this,"修改成功",Toast.LENGTH_SHORT);
                toast3.show();
                break;
            case R.id.red_font:
                mText.setTextColor(Color.RED);
                Toast toast4=Toast.makeText(NoteEditor.this,"修改成功",Toast.LENGTH_SHORT);
                toast4.show();
                break;
            case R.id.yellow_font:
                mText.setTextColor(Color.YELLOW);
                Toast toast5=Toast.makeText(NoteEditor.this,"修改成功",Toast.LENGTH_SHORT);
                toast5.show();
                break;
```
3.到这里修改功能已经完成了,为了下次打开还能有保存,需要给数据库中加入相应的参数来保存字体大小和颜色 NotePadProvider.java中onCreate函数里加上
```
 + NotePad.Notes.COLUMN_NAME_TEXT_COLOR + "INTEGER"
    + NotePad.Notes.COLUMN_NAME_TEXT_SIZE +"INTEGER"
```
结果截图:
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190516191128184.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQwOTQ2MDUz,size_16,color_FFFFFF,t_70)
