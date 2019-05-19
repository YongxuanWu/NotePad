实验内容
==
    基本要求：
      1.NoteList中显示条目增加时间戳显示
      2.添加笔记本查询功能（根据标题查询）
    附加功能：
      1.UI美化
      2.导出笔记
      3.笔记内容排序
关键代码
==
时间戳显示
--
NotePadProvider.java
  创建表代码
```
       public void onCreate(SQLiteDatabase db) {
           db.execSQL("CREATE TABLE " + NotePad.Notes.TABLE_NAME + " ("
                   + NotePad.Notes._ID + " INTEGER PRIMARY KEY,"
                   + NotePad.Notes.COLUMN_NAME_TITLE + " TEXT,"
                   + NotePad.Notes.COLUMN_NAME_NOTE + " TEXT,"
                   + NotePad.Notes.COLUMN_NAME_CREATE_DATE + " INTEGER,"
                   + NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE + " INTEGER,"
                   + NotePad.Notes.COLUMN_NAME_TEXT_COLOR + " INTEGER," 
                   + NotePad.Notes.COLUMN_NAME_TEXT_SIZE + " INTEGER" 
                   + ");");
       }
 ```
   按格式显示时间
 ```
        Long now = Long.valueOf(System.currentTimeMillis());
        Date date = new Date(now);
        SimpleDateFormat format = new SimpleDateFormat("yyyy/MM/dd  HH:mm");
        String dateTime = format.format(date);

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
 noteslist_item.xml  
   增加显示时间的TextView
  ```
    <TextView
        android:id="@+id/text2"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:textAppearance="?android:attr/textAppearanceSmall"
        android:textColor="@color/grey"
        android:paddingTop="2dp"
        android:paddingLeft="20dp"
        android:paddingBottom="4dp"
        android:textSize="12sp"
        />
```
NoteList.java
  增加修改笔记的时间
  ```
  private static final String[] PROJECTION = new String[] {
            NotePad.Notes._ID, // 0
            NotePad.Notes.COLUMN_NAME_TITLE, // 1
            NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, // 显示修改时间

    };
  ```
  修改装配部分
  ```
  String[] dataColumns = {
                NotePad.Notes.COLUMN_NAME_TITLE ,
                NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE
        } ;
    
  int[] viewIDs = { android.R.id.text1 ,R.id.text2};
  ```
查询笔记
--
list_options_menu.xml
  增加一个查询选项
```
<item
        android:id="@+id/menu_search"
        android:icon="@drawable/search"
        android:title="@string/menu_search"
        android:showAsAction="always"
        />
```
NoteList.java
   在onOptionsItemSelected中添加
```
        case R.id.menu_search:
                startActivity(new Intent(Intent.ACTION_SEARCH,getIntent().getData()));
                return true;
 ```
AndroidManifest.xml
   增加
```
        <activity
            android:name=".NoteSearch"
            android:label="NoteSearch"
            android:theme="@android:style/Theme.Holo.Light">
            <intent-filter>
                <action android:name="android.intent.action.NoteSearch" />
                <action android:name="android.intent.action.SEARCH" />
                <action android:name="android.intent.action.SEARCH_LONG_PRESS" />

                <category android:name="android.intent.category.DEFAULT" />

                <data android:mimeType="vnd.android.cursor.dir/vnd.google.note" />
                <!-- 1.vnd.android.cursor.dir代表返回结果为多列数据 -->
                <!-- 2.vnd.android.cursor.item 代表返回结果为单列数据 -->
            </intent-filter>
        </activity>
```
NoteSearch.java
  ```
    private static final String[] PROJECTION = new String[] {
            NotePad.Notes._ID, // 0
            NotePad.Notes.COLUMN_NAME_TITLE, // 1
            //扩展 显示时间 颜色
            NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, // 2
            //NotePad.Notes.COLUMN_NAME_BACK_COLOR
    };
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.note_search);
        Intent intent = getIntent();
        if (intent.getData() == null) {
            intent.setData(NotePad.Notes.CONTENT_URI);
        }
        getListView().setOnCreateContextMenuListener(this);

        SearchView searchview = (SearchView)findViewById(R.id.search_view);
        searchview.setIconifiedByDefault(false); //显示搜索的天幕，默认只有一个放大镜图标
        searchview.setSubmitButtonEnabled(true); //显示搜索按钮
        searchview.setBackgroundColor(getResources().getColor(R.color.thistle)); //设置背景颜色
        searchview.setOnQueryTextListener(this);
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
        String[] dataColumns = { NotePad.Notes.COLUMN_NAME_TITLE ,  NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE };
        int[] viewIDs = { android.R.id.text1 , R.id.text2 };
        SimpleCursorAdapter adapter = new SimpleCursorAdapter(
                this,                   //context:上下文
                R.layout.noteslist_item,         //layout:布局文件，至少有int[]的所有视图
                cursor,                          //cursor：游标
                dataColumns,                     //from：绑定到视图的数据
                viewIDs                          //to:用来展示from数组中数据的视图
                //flags：用来确定适配器行为的标志，Android3.0之后淘汰
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
  ```
note_search.xml
```
    <SearchView
        android:id="@+id/search_view"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:queryHint="请输入搜索内容...">
    </SearchView>

    <ListView
        android:id="@android:id/list"
        android:layout_width="match_parent"
        android:layout_height="wrap_content">
    </ListView>
```
  导出笔记
  --
  editor_options_menu.xml
    增加导出笔记选项
  ```
  <item android:id="@+id/menu_output"
        android:title="@string/menu_output" />
  ```
  NoteEditor.java
    onOptionsItemSelected()中增加
  ```
  case R.id.menu_output:
        outputNote();
        break;
  ```
  NoteEdit.java
  ```
  private final void outputNote() {
        Intent intent = new Intent(null,mUri);
        intent.setClass(NoteEditor.this,OutputText.class);
        NoteEditor.this.startActivity(intent);
    }
  ```
  OutputText.java
  ```
  private void write()
    {
        try
        {
            // 如果手机插入了SD卡，而且应用程序具有访问SD的权限
            if (Environment.getExternalStorageState().equals(
                    Environment.MEDIA_MOUNTED)) {
                // 获取SD卡的目录
                File sdCardDir = Environment.getExternalStorageDirectory();
                //创建文件目录
                File targetFile = new File(sdCardDir.getCanonicalPath() + "/" + mName.getText() + ".txt");
                //写文件
                PrintWriter ps = new PrintWriter(new OutputStreamWriter(new FileOutputStream(targetFile), "UTF-8"));
                ps.println(TITLE);
                ps.println(NOTE);
                ps.println("创建时间：" + CREATE_DATE);
                ps.println("最后一次修改时间：" + MODIFICATION_DATE);
                ps.close();
                Toast.makeText(this, "保存成功,保存位置：" + sdCardDir.getCanonicalPath() + "/" + mName.getText() + ".txt", Toast.LENGTH_LONG).show();
            }
        }
        catch (Exception e)
        {
            e.printStackTrace();
        }
    }
  ```
在AndroidManifest.xml中为导出笔记功能增加权限
output_text.xml
```
<EditText android:id="@+id/output_name"
        android:maxLines="1"
        android:layout_marginTop="2dp"
        android:layout_marginBottom="15dp"
        android:layout_width="wrap_content"
        android:ems="25"
        android:layout_height="wrap_content"
        android:autoText="true"
        android:capitalize="sentences"
        android:scrollHorizontally="true" />
    <Button android:id="@+id/output_ok"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="right"
        android:text="@string/output_ok"
        android:onClick="OutputOk" />
```
字体大小、颜色
--
  NoteList.java
  ```
     public boolean onOptionsItemSelected(MenuItem item) {
        switch (item.getItemId()) {
        ...
          case R.id.menu_search:
                startActivity(new Intent(Intent.ACTION_SEARCH,getIntent().getData()));
                return true;
          //创建时间排序
            case R.id.menu_sort1:
                cursor = managedQuery(
                        getIntent().getData(),
                        PROJECTION,
                        null,
                        null,
                        NotePad.Notes._ID
                );
                adapter = new SimpleCursorAdapter(
                        this,
                        R.layout.noteslist_item,
                        cursor,
                        dataColumns,
                        viewIDs
                );
                setListAdapter(adapter);
                return true;
            //修改时间排序
            case R.id.menu_sort2:
                cursor = managedQuery(
                        getIntent().getData(),
                        PROJECTION,
                        null,
                        null,
                        NotePad.Notes.DEFAULT_SORT_ORDER
                );
                adapter = new SimpleCursorAdapter(
                        this,
                        R.layout.noteslist_item,
                        cursor,
                        dataColumns,
                        viewIDs
                );
                setListAdapter(adapter);
                return true;
        default:
            return super.onOptionsItemSelected(item);
        }
    }
  ```
结果截图
===
![image](https://github.com/YongxuanWu/NotePad/blob/master/app/src/main/res/pictures1/Screenshot_20190519_193531.jpg)<br>
更改字体大小、颜色
--
![image](https://github.com/YongxuanWu/NotePad/blob/master/app/src/main/res/pictures1/Screenshot_20190519_193104.jpg)
![image](https://github.com/YongxuanWu/NotePad/blob/master/app/src/main/res/pictures1/Screenshot_20190519_193152.jpg)<br>
编辑标题
--
![image](https://github.com/YongxuanWu/NotePad/blob/master/app/src/main/res/pictures1/Screenshot_20190519_193212.jpg)
![image](https://github.com/YongxuanWu/NotePad/blob/master/app/src/main/res/pictures1/Screenshot_20190519_193233.jpg)
![image](https://github.com/YongxuanWu/NotePad/blob/master/app/src/main/res/pictures1/Screenshot_20190519_193255.jpg)<br>
查询笔记
--
![image](https://github.com/YongxuanWu/NotePad/blob/master/app/src/main/res/pictures1/Screenshot_20190519_193310.jpg)<br>
笔记排序
--
![image](https://github.com/YongxuanWu/NotePad/blob/master/app/src/main/res/pictures1/Screenshot_20190519_193330.jpg)
![image](https://github.com/YongxuanWu/NotePad/blob/master/app/src/main/res/pictures1/Screenshot_20190519_193350.jpg)
![image](https://github.com/YongxuanWu/NotePad/blob/master/app/src/main/res/pictures1/Screenshot_20190519_193410.jpg)<br>
导出笔记
--
![image](https://github.com/YongxuanWu/NotePad/blob/master/app/src/main/res/pictures1/Screenshot_20190519_193427.jpg)
![image](https://github.com/YongxuanWu/NotePad/blob/master/app/src/main/res/pictures1/Screenshot_20190519_193442.jpg)
![image](https://github.com/YongxuanWu/NotePad/blob/master/app/src/main/res/pictures1/Screenshot_20190519_193501.jpg)



