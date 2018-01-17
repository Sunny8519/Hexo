---
title: SharedPreferences源码浅析
date: 2018-01-17
categories: SharedPreferences
author: Sunny
tags:
    - SharedPreferences
cover_picture: http://upload-images.jianshu.io/upload_images/5231076-3385be91b57dabf4.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240
---
### 1.SharedPreferences介绍
SharedPreferences是Android系统用于存储如配置信息等轻量级数据的接口（注意：这里是接口哦），实际上存储数据的是一个xml文件，这些数据以键值对的形式存在，比如下面这样：
<div align = "center">
<img src="http://upload-images.jianshu.io/upload_images/5231076-6aabd932b4810c80.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240" style = "zoom:70%" />
</div>

### 2.SharedPreferences的简单使用
**第一步：创建SharedPreferences对象**
SharedPreferences的创建有四种方式，分别为：
- `Context.MODE_PRIVATE` 默认模式，表示SharedPreferences文件是私有的，只能由创建xml文件的应用程序访问（该模式在每次写数据的时候会把原来的数据覆盖）；
- `Context.MODE_APPEND ` 追加模式，该模式会检查需要创建的文件是否已经创建，如果已经创建了，写入的数据会追加到文件的末尾；
- `Context.MODE_WORLD_READABLE` 开放读模式，该模式允许其他应用程序读当前应用程序的SharedPreferences文件，存在数据安全的风险；
- `Context.MODE_WORLD_WRITEABLE` 开放写模式，该模式允许其他应用程序往当前应用程序的SharedPreferences文件中写数据，同样存在数据安全的风险。

**第二步：获取Edit对象**
**第三步：通过Edit对象存储键值对数据**
需要调用`commit()`或者`apply()`方法才能把数据成功存入文件中
**第四步：通过SharedPreferences对象获取数据**

#### SharedPreferences使用示例代码：
```java
public class SPUtils {
    private static final String SP_NAME = "hello";
    private static SPUtils mSPUtils = null;
    private SharedPreferences mSharedPreferences = null;
    private SharedPreferences.Editor mEditor = null;
    private Context mContext = null;//全局Context

    private SPUtils(Context context) {
        this.mContext = context;
        this.mSharedPreferences = this.mContext.getSharedPreferences(SP_NAME, Context.MODE_APPEND);
        this.mEditor = this.mSharedPreferences.edit();
    }

    public static SPUtils getInstance(Context context) {
        if (mSPUtils == null) {
            synchronized (SPUtils.class) {
                if (mSPUtils == null) {
                    mSPUtils = new SPUtils(context);
                }
            }
        }
        return mSPUtils;
    }

    //存入Value类型为String的数据
    public void putString(String key, String value) {
        this.mEditor.putString(key, value);
        this.mEditor.commit();
    }
    //获取Value类型为String的数据
    public String getString(String key, String defValue) {
        return this.mSharedPreferences.getString(key, defValue);
    }
}
```

### 3.SharedPreferences源码解析
首先应该从`this.mSharedPreferences = this.mContext.getSharedPreferences(SP_NAME, Context.MODE_APPEND);`背后的原理说起。
SharedPreferences对象是Context中getSharedPreferences()方法返回的，我们先找到这个方法，在IDE中经过搜索，发现Context中有两种getSharedPreferences()方法，如下：
<div align = "center">
<img src="http://upload-images.jianshu.io/upload_images/5231076-785bc1e101cfac2e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240" style = "zoom:70%" />
</div>
从图中我们很明显的看出来两个方法的第一个参数类型不同，在实际使用中，一般都是调用第一个参数为String类型的方法`getSharedPrefereces(String,int)`，我们进入这个方法看看...

<pre><code>
public abstract class Context{
    ...
    public abstract SharedPreferences getSharedPreferences(String name,@PreferencesMode int mode);
    ...
}
</code></pre>
发现这个方法是一个抽象方法，那么`getSharedPrefereces(File,int)`是不是也是抽象方法呢，进入看一眼...
<pre><code>
public abstract class Context{
    ...
    public abstract SharedPreferences getSharedPreferences(File file,int mode);
    ...
}
</code></pre>

的确，它也是一个抽象方法，那么问题就来了，它们的具体实现是在哪里呢？既然是Context中的方法，我们还得从Context这个抽象类出发，在前面SharedPreferences的使用示例中Context使用的是全局的Context，也就是Application对应的Context,在分析该Context之前，我们先来了解一下Context的几个重要的继承关系：
<div align = "center">
<img src="http://upload-images.jianshu.io/upload_images/5231076-95ddef75448af94b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240" style = "zoom:70%" />
</div>
从图中可以看出，Context的两个直接子类是ContextWrapper和ContextImpl，从这两个类的类名大概能猜出它们的功能：ContextWrapper类主要是对Context的功能封装，ContextImpl类则是对Context的功能实现，到这里我们的思路就明朗了，既然ContextImpl是Context的实现类，那么`getSharedPreferences(File,int)`和`getSharedPreferences(String,int)`的实现应该就在ContextImpl类中，具体是不是，我们继续向下看...

Application对应的Context对象我们通常是通过`getApplicationContext()`方法获取的，我们在Application类中搜索该方法：
<div align = "center">
<img src="http://upload-images.jianshu.io/upload_images/5231076-ace86515077a8b24.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240" style = "zoom:70%" />
</div>
从搜索结果可以看出该方法指向ContextWrapper类，这说明了Application是ContextWrapper的子类，而调用Application的`getApplicationContext()`其实调用的是ContextWrapper类中的方法，我们进入ContextWrapper中看一看...
<pre><code>
public class ContextWrapper extends Context{
    Context mBase;
    ...
    protected void attachBaseContext(Context base){
        if(mBase != null){
            throw new IllegalStateException("Base context already set");
        }
        mBase  = base;
    }
    ...
    @Override
    public Context getApplicationContext(){
        return mBase.getApplicationContext();
    }
}
</code></pre>
ContextWrapper类中有一个全局Context类型的`mBase`变量，它是在`attachBaseContext(Context base)`方法中被赋予引用的，其实这里的引用指向的就是一个ContextImpl对象，在`getApplicationContext()`方法中调用了`mBase`的`getApplicationContext()`方法，也就是ContextImpl类中的`getApplicationContext()`，我们进入ContextImpl类中的`getApplicationContext()`方法...

<pre><code>
class ContextImpl extends Context{
    ...
    @Override
    public Context getApplicationContext() {
        return (mPackageInfo != null) ?
                mPackageInfo.getApplication() : mMainThread.getApplication();
    }
    ...
}
</code></pre>
不管是LoadedApk中的`getApplication()`还是ActivityThread中的`getApplication()`，最终获得的都是当前使用的Application对象，也就是说如果我们自定义了一个Application,那么`getApplicationContext()`方法返回的是当前自定义的Application对象，转了一个大圈，最后我们还是要回到Application对象上，因此，在这行代码中`this.mSharedPreferences = this.mContext.getSharedPreferences(SP_NAME, Context.MODE_APPEND);`调用的`getSharedPreferences(String,int)`其实是Application中的，我们在Application类中搜索该方法：
<div align = "center">
<img src="http://upload-images.jianshu.io/upload_images/5231076-1e3f3a0ab3236c20.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240" style = "zoom:70%" />
</div>

从搜索结果可以看出该方法是指向ContextWrapper类的，也就是说它是ContextWrapper中的方法，我们进入看一眼...
```java
public class ContextWrapper extends Context{
    Context mBase;
    ...
    protected void attachBaseContext(Context base){
        if(mBase != null){
            throw new IllegalStateException("Base context already set");
        }
        mBase  = base;
    }
    ...
    @Override
    public Context getApplicationContext(){
        return mBase.getApplicationContext();
    }

    @Override
    public SharedPreferences getSharedPreferences(String name,int mode){
        return mBase.getSharedPreferences(name,mode);
    }
}
```

在该方法内部又调用了`mBase`的`getSharedPreferences(String,int)`方法，再进入ContextImpl中的`getSharedPreferences(String,int)`方法...

```java
 @Override
 public SharedPreferences getSharedPreferences(String name, int mode) {
      // At least one application in the world actually passes in a null
      // name.  This happened to work because when we generated the file name
      // we would stringify it to "null.xml".  Nice.
      if (mPackageInfo.getApplicationInfo().targetSdkVersion <
              Build.VERSION_CODES.KITKAT) {
          if (name == null) {
              name = "null";
          }
      }
      File file;
      synchronized (ContextImpl.class) {
          if (mSharedPrefsPaths == null) {
              mSharedPrefsPaths = new ArrayMap<>();//1
          }
          file = mSharedPrefsPaths.get(name);
          if (file == null) {
              file = getSharedPreferencesPath(name);//2
              mSharedPrefsPaths.put(name, file);
          }
      }
      return getSharedPreferences(file, mode);//3
  }
```
哎呀，终于开始了，哈哈哈，这个方法中我们主要关注三个地方，分别对应注释1,2,3，先来看注释1，`mSharedPrefsPaths`是一个全局的Map，它的声明如下：
```java
private ArrayMap<String,File> mSharedPrefsPaths;
```
从`getSharedPreferences(String,int)`方法中的上下逻辑可以知道该Map的key值存放的是外面传进来的SharedPreferences文件的名称，也就是SharedPreferences使用示例中的字符串"hello"，value存放的是File类型的对象，该对象是在注释2处获取的，我们进入注释2的`getSharedPreferencesPath(String)`方法...
```java
class ContextImpl extends Context{
    ...
    @Override
    public File getSharedPreferencesPath(String name){
        return makeFilename(getPreferencesDir(),name + ".xml");
    }
    ...
}
```
该方法中又调用了getPreferencesDir()方法...
```java
class ContextImpl extends Context{
    ...
    @Override
    public File getSharedPreferencesPath(String name){
        return makeFilename(getPreferencesDir(),name + ".xml");
    }

    private File getPreferencesDir(){
        synchronized(){
            if(mPreferencesDir == null){
                mPreferencesDir = new File(getDataDir(),"shared_prefs");
            }
            return ensurePrivateDirExists(mPreferencesDir);
        }
    }
    ...
}
```
从该方法中我们知道这是在创建"shared_prefs"为名的文件夹，完整路径应该是"/data/data/应用程序包名/shared_prefs/",在该文件夹下会存放名为name的xml子文件，文件夹创建好了之后，把File对象返回去，而后，继续调用makeFilename()方法，我们来看一眼这个方法...
```java
private File makeFilename(File base, String name) {
    if (name.indexOf(File.separatorChar) < 0) {
        return new File(base, name);
    }
    throw new IllegalArgumentException(
            "File " + name + " contains a path separator");
}
```
该方法内部new了一个以"/data/data/应用程序包名/shared_prefs/"为父目录，name为子文件的File对象，并把该对象返回，该对象就是注释2处获取到的File对象,我们再回到注释2处继续向下看...
```java
 @Override
 public SharedPreferences getSharedPreferences(String name, int mode) {
      // At least one application in the world actually passes in a null
      // name.  This happened to work because when we generated the file name
      // we would stringify it to "null.xml".  Nice.
      if (mPackageInfo.getApplicationInfo().targetSdkVersion <
              Build.VERSION_CODES.KITKAT) {
          if (name == null) {
              name = "null";
          }
      }
      File file;
      synchronized (ContextImpl.class) {
          if (mSharedPrefsPaths == null) {
              mSharedPrefsPaths = new ArrayMap<>();//1
          }
          file = mSharedPrefsPaths.get(name);
          if (file == null) {
              file = getSharedPreferencesPath(name);//2
              mSharedPrefsPaths.put(name, file);
          }
      }
      return getSharedPreferences(file, mode);//3
  }
```
在注释2处的代码走完之后，`mSharedPrefsPaths`会把File对象和name暂存在ContextImpl中（这里体现在ContextWrapper的mBase变量）。我们再来看注释3，该处调用的是ContextImpl类的`getSharedPreferences(File,int)`方法，我们进入该方法...
```java
@Override
public SharedPreferences getSharedPreferences(File file, int mode) {
        checkMode(mode);
        if (getApplicationInfo().targetSdkVersion >= android.os.Build.VERSION_CODES.O) {
            if (isCredentialProtectedStorage()
                    && !getSystemService(StorageManager.class).isUserKeyUnlocked(
                            UserHandle.myUserId())
                    && !isBuggy()) {
                throw new IllegalStateException("SharedPreferences in credential encrypted "
                        + "storage are not available until after user is unlocked");
            }
        }
        SharedPreferencesImpl sp;
        synchronized (ContextImpl.class) {
            final ArrayMap<File, SharedPreferencesImpl> cache = getSharedPreferencesCacheLocked();//a
            sp = cache.get(file);
            if (sp == null) {
                sp = new SharedPreferencesImpl(file, mode);//b
                cache.put(file, sp);
                return sp;//c
            }
        }
        if ((mode & Context.MODE_MULTI_PROCESS) != 0 ||
            getApplicationInfo().targetSdkVersion < android.os.Build.VERSION_CODES.HONEYCOMB) {
            // If somebody else (some other process) changed the prefs
            // file behind our back, we reload it.  This has been the
            // historical (if undocumented) behavior.
            sp.startReloadIfChangedUnexpectedly();
        }
        return sp;
}
```

该方法中我们只需要关注a,b,c三处的代码，首先来看看注释a，在注释a处调用了`getSharedPreferencesCacheLocked()`，该方法返回的是一个以File对象为key，SharedPreferencesImpl对象为value的Map,我们进入`getSharedPreferencesCacheLocked()`方法...
```java
private ArrayMap<File, SharedPreferencesImpl> getSharedPreferencesCacheLocked() {
    if (sSharedPrefsCache == null) {
        sSharedPrefsCache = new ArrayMap<>();
    }
    final String packageName = getPackageName();
    ArrayMap<File, SharedPreferencesImpl> packagePrefs = sSharedPrefsCache.get(packageName);
    if (packagePrefs == null) {
        packagePrefs = new ArrayMap<>();
        sSharedPrefsCache.put(packageName, packagePrefs);
    }
    return packagePrefs;
}
```
该方法中我们发现有一个`sSharedPrefsCache`的全局静态变量，它的声明如下...
```java
class ContextImpl extends Context{
    ...
    priavate static ArrayMap<String,ArrayMap<File,SharedPreferencesImpl>> sSharedPrefsCache;
    ...
}
```
再回到刚才的注释a处继续向下看，我们发现`sSharedPrefsCache`key存放的是应用程序的包名，value存放的是以File对象为key,SharedPreferencesImpl对象为value的map。继续，在注释b处new了一个SharedPreferencesImpl对象，并把File和Mode传进入了，我们进入SharedPreferencesImpl的构造方法...
```java
final class SharedPreferencesImpl implements SharedPreferences{
     ...
     private final File mFile;
     private final File mBackupFile;
     private final int mMode;
     private Map<String, Object> mMap;     // guarded by 'this'
     private boolean mLoaded = false;      // guarded by 'this'
    
     SharedPreferencesImpl(File file, int mode) {
        //存储的文件
        mFile = file;
        //创建跟原文件名相同的备份文件
        mBackupFile = makeBackupFile(file);
        //访问模式
        mMode = mode;
        mLoaded = false;
        mMap = null;
        //从flash或者sdcard中异步加载文件数据
        startLoadFromDisk();
    }
    
    static File makeBackupFile(File prefsFile) {
        //new一个跟原文件名相同的以.bak为后缀的备份文件
        return new File(prefsFile.getPath() + ".bak");
    }
    
    private void startLoadFromDisk() {
        //对mLoaded标志位进行同步操作
        synchronized (this) {
            mLoaded = false;
        }
        //开启一个线程加载文件数据
        new Thread("SharedPreferencesImpl-load") {
            public void run() {
                loadFromDisk();
            }
        }.start();
    }
    
    private void loadFromDisk() {
        //持有SharedPreferencesImpl对象锁
        synchronized (SharedPreferencesImpl.this) {
            //如果已经加载过了，直接返回
            if (mLoaded) {
                return;
            }
            //如果存在备份文件则把原文件删除，备份文件按File文件来命名
            if (mBackupFile.exists()) {
                mFile.delete();
                mBackupFile.renameTo(mFile);
            }
        }

        // Debugging
        if (mFile.exists() && !mFile.canRead()) {
            Log.w(TAG, "Attempt to read preferences file " + mFile + " without permission");
        }

        Map map = null;
        StructStat stat = null;
        try {
            stat = Os.stat(mFile.getPath());
            //文件是可读的
            if (mFile.canRead()) {
                BufferedInputStream str = null;
                try {
                    //把文件以流的形式读出来
                    str = new BufferedInputStream(
                            new FileInputStream(mFile), 16*1024);
                    //把输出流解析成Map的格式
                    map = XmlUtils.readMapXml(str);
                } catch (XmlPullParserException | IOException e) {
                    Log.w(TAG, "getSharedPreferences", e);
                } finally {
                    IoUtils.closeQuietly(str);
                }
            }
        } catch (ErrnoException e) {
            /* ignore */
        }

        //加SharedPreferencesImpl对象锁，防止在getXXX()时出错
        synchronized (SharedPreferencesImpl.this) {
            //到这里说明文件加载完毕，mLoaded标志位置true
            mLoaded = true;
            if (map != null) {
                //把map赋值给全局变量
                mMap = map;
                //记录本次读取文件的时间戳
                mStatTimestamp = stat.st_mtime;
                //记录文件的大小
                mStatSize = stat.st_size;
            } else {
                //如果文件第一次创建没有数据，则直接new一个HashMap返回供后续putXXX()数据时存放数据
                mMap = new HashMap<>();
            }
            //唤醒其他等待的线程，这里指的是调用getXXX()的线程，因为在mLoaded为falses时，调用getXXX()的线程会进入wait()状态
            notifyAll();
        }
    }
    ...
}
```
上面我已经把关键的代码和注释贴出来了，这里不再细说...
**小结：**
> - SharedPreferences对象在第一次实例化的时候会从xml文件中读取数据并存储在Map中(也就是内存中);
- 在以后的使用中，会先根据xml文件名在ContextImpl的mSharedPrefsPaths中找到对应的File对象，然后根据包名在静态全局变量sSharedPrefsCache中找出对应的File-SharedPreferencesImpl map集合，再根据前面获得的File对象得到SharedPreferencesImpl对象。
- 从第二条总结来看，我们知道SharedPreferences一旦创建了之后会一直存在于系统中，需要使用时直接就能拿到。

至此，整个SharedPreferences创建过程就解析完了，下面我们来看看数据是如何获取和存储的...
以`getString(String,String)`为例：
```java
final class SharedPreferencesImpl implements SharedPreferences{
     ...
     private final File mFile;
     private final File mBackupFile;
     private final int mMode;
     private Map<String, Object> mMap;     // guarded by 'this'
     private boolean mLoaded = false;      // guarded by 'this'
    
     SharedPreferencesImpl(File file, int mode) {
        //存储的文件
        mFile = file;
        //创建跟原文件名相同的备份文件
        mBackupFile = makeBackupFile(file);
        //访问模式
        mMode = mode;
        mLoaded = false;
        mMap = null;
        //从flash或者sdcard中异步加载文件数据
        startLoadFromDisk();
    }
    
    @Nullable
    public String getString(String key, @Nullable String defValue) {
        synchronized (this) {
            //如果xml文件没有加载解析完毕，调用线程一直wait
            awaitLoadedLocked();
            //从内存中获取key所对应的value值
            String v = (String)mMap.get(key);
            //如果没有则返回默认值
            return v != null ? v : defValue;
        }
    }
    
    private void awaitLoadedLocked() {
        if (!mLoaded) {
            // Raise an explicit StrictMode onReadFromDisk for this
            // thread, since the real read will be in a different
            // thread and otherwise ignored by StrictMode.
            //??
            BlockGuard.getThreadPolicy().onReadFromDisk();
        }
        //mLoaded为false一直wait
        while (!mLoaded) {
            try {
                wait();
            } catch (InterruptedException unused) {
            }
        }
    }
    ...
}
```
上面只贴出了getString()的代码，其他几种原理差不多，这里不再一一解释，到这儿，数据的获取就讲完了。下面我们再来看看putXXX()方法，存储数据，由于putXXX()方法是依靠Editor对象来操作的，我们先来看看创建Editor对象的过程...
```java
final class SharedPreferencesImpl implements SharedPreferences{
     ...
     private final File mFile;
     private final File mBackupFile;
     private final int mMode;
     private Map<String, Object> mMap;     // guarded by 'this'
     private boolean mLoaded = false;      // guarded by 'this'
    
     SharedPreferencesImpl(File file, int mode) {
        //存储的文件
        mFile = file;
        //创建跟原文件名相同的备份文件
        mBackupFile = makeBackupFile(file);
        //访问模式
        mMode = mode;
        mLoaded = false;
        mMap = null;
        //从flash或者sdcard中异步加载文件数据
        startLoadFromDisk();
    }
    
    public Editor edit() {
        // TODO: remove the need to call awaitLoadedLocked() when
        // requesting an editor.  will require some work on the
        // Editor, but then we should be able to do:
        //
        //      context.getSharedPreferences(..).edit().putString(..).apply()
        //
        // ... all without blocking.
        //这里和异步的mLoaded使用的是同一把锁，因为这里面也用到了mLoaded标志位
        synchronized (this) {
            //mLoaded为false时等待
            awaitLoadedLocked();
        }
        //new一个EditorImpl对象返回
        return new EditorImpl();
    }
    
    private void awaitLoadedLocked() {
        if (!mLoaded) {
            // Raise an explicit StrictMode onReadFromDisk for this
            // thread, since the real read will be in a different
            // thread and otherwise ignored by StrictMode.
            //??
            BlockGuard.getThreadPolicy().onReadFromDisk();
        }
        //mLoaded为false一直wait
        while (!mLoaded) {
            try {
                wait();
            } catch (InterruptedException unused) {
            }
        }
    }
    ...
}
```
在`edit()`方法中最后new了一个EditorImpl对象，进入EditorImpl类...
```java
public final class EditorImpl implements Editor {
        //创建一个key-value的集合，用来暂存putXXX()数据
        private final Map<String, Object> mModified = Maps.newHashMap();
        //是否清除SharedPreferences的标志位
        private boolean mClear = false;

        public Editor putString(String key, @Nullable String value) {
            //同步锁
            synchronized (this) {
                //将要存储的数据暂存在mModified中
                mModified.put(key, value);
                //返回当前对象，方便链式调用
                return this;
            }
        }
        
        public Editor remove(String key) {
            //同步删除key-value
            synchronized (this) {
                //这里为什么是put呢，而不是remove,原因在后面会讲解
                mModified.put(key, this);
                return this;
            }
        }

        public Editor clear() {
            //清空所有数据只需要把mClear标志位置为true
            synchronized (this) {
                mClear = true;
                return this;
            }
        }
        ...
}
```
从上面的代码中我们发现put的数据只是暂存到了`mModified`变量中，并没有我们想象中那样直接存储到文件，这样就对了，因为我们在存储数据时最后还要调用commit()或者apply()方法呢，因此我们就可以大胆的猜测数据写入文件的操作是在commit()或者apply()方法中进行的，好了，废话不多说，我们先来分析commit()方法...
```java
public final class EditorImpl implements Editor {
        //创建一个key-value的集合，用来暂存putXXX()数据
        private final Map<String, Object> mModified = Maps.newHashMap();
        //是否清除SharedPreferences的标志位
        private boolean mClear = false;
        
        //先来看看commit
        public boolean commit() {
            //把数据存储到mMap中，并在MemoryCommitResult中持有mMap的引用
            MemoryCommitResult mcr = commitToMemory();
            //把mMap中的数据存入到xml文件
            SharedPreferencesImpl.this.enqueueDiskWrite(
                mcr, null /* sync write on this thread okay */);
            try {
                mcr.writtenToDiskLatch.await();
            } catch (InterruptedException e) {
                return false;
            }
            notifyListeners(mcr);
            return mcr.writeToDiskResult;
        }
        
        // Returns true if any changes were made
        private MemoryCommitResult commitToMemory() {
            //new了一个MemoryCommitResult对象，MemoryCommitResult是SharedPreferences中的静态内部类
            MemoryCommitResult mcr = new MemoryCommitResult();
            synchronized (SharedPreferencesImpl.this) {
                // We optimistically don't make a deep copy until
                // a memory commit comes in when we're already
                // writing to disk.
                if (mDiskWritesInFlight > 0) {
                    // We can't modify our mMap as a currently
                    // in-flight write owns it.  Clone it before
                    // modifying it.
                    // noinspection unchecked
                    mMap = new HashMap<String, Object>(mMap);
                }
                //把mMap赋值到MemoryCommitResult中的要写到disk的Map
                mcr.mapToWriteToDisk = mMap;
                //增加一个未完成的写操作
                mDiskWritesInFlight++;

                //判断有没有监听
                boolean hasListeners = mListeners.size() > 0;
                if (hasListeners) {
                    mcr.keysModified = new ArrayList<String>();
                    mcr.listeners =
                            new HashSet<OnSharedPreferenceChangeListener>(mListeners.keySet());
                }

                synchronized (this) {
                    //如果调用了clear()方法，该mClear标志位就是true
                    if (mClear) {
                        if (!mMap.isEmpty()) {
                            mcr.changesMade = true;
                            //清空内存中mMap,并不会清空缓存在Editor中的数据
                            mMap.clear();
                        }
                        //mClear重置为false
                        mClear = false;
                    }

                    for (Map.Entry<String, Object> e : mModified.entrySet()) {
                        String k = e.getKey();
                        Object v = e.getValue();
                        // "this" is the magic value for a removal mutation. In addition,
                        // setting a value to "null" for a given key is specified to be
                        // equivalent to calling remove on that key.
                        //在remove()方法中put(key,this)，用于删除数据
                        if (v == this || v == null) {
                            //如果mMap中包含这个key,就会把这个key对应的key-value删除
                            if (!mMap.containsKey(k)) {
                                continue;
                            }
                            mMap.remove(k);
                        } else {
                            if (mMap.containsKey(k)) {
                                Object existingValue = mMap.get(k);
                                //如果原来mMap中有对应的key-value值不会重复添加
                                if (existingValue != null && existingValue.equals(v)) {
                                    continue;
                                }
                            }
                            mMap.put(k, v);
                        }

                        mcr.changesMade = true;
                        //如果有监听器，把变化的key放入MemoryCommitResult中的List
                        if (hasListeners) {
                            mcr.keysModified.add(k);
                        }
                    }
                    //数据向mMap中存完之后清空Editor中的mModified
                    mModified.clear();
                }
            }
            //返回MemoryCommitResult对象
            return mcr;
        }
        ...
}

    // Return value from EditorImpl#commitToMemory()
    private static class MemoryCommitResult {
        public boolean changesMade;  // any keys different?
        public List<String> keysModified;  // may be null
        public Set<OnSharedPreferenceChangeListener> listeners;  // may be null
        public Map<?, ?> mapToWriteToDisk;
        public final CountDownLatch writtenToDiskLatch = new CountDownLatch(1);
        public volatile boolean writeToDiskResult = false;

        public void setDiskWriteResult(boolean result) {
            writeToDiskResult = result;
            writtenToDiskLatch.countDown();
        }
    }
```
上面我们只分析了commit()方法中的`commitToMemory()`方法，该方法主要是把Eidtor中缓存的数据存入mMap中（也就是我们俗称的“内存”中），并且MemoryCommitResult中的mapToWriteToDisk持有该map的引用，接下来我们分析enqueueDiskWrite()方法...
```java
public final class EditorImpl implements Editor {
        //创建一个key-value的集合，用来暂存putXXX()数据
        private final Map<String, Object> mModified = Maps.newHashMap();
        //是否清除SharedPreferences的标志位
        private boolean mClear = false;
        
        //先来看看commit
        public boolean commit() {
            //把数据存储到mMap中，并在MemoryCommitResult中持有mMap的引用
            MemoryCommitResult mcr = commitToMemory();
            //把mMap中的数据存入到xml文件
            SharedPreferencesImpl.this.enqueueDiskWrite(
                mcr, null /* sync write on this thread okay */);
            try {
                mcr.writtenToDiskLatch.await();
            } catch (InterruptedException e) {
                return false;
            }
            //如果有监听器并且数据有改变则通知这些监听器
            notifyListeners(mcr);
            //写成功返回true，写失败返回false
            return mcr.writeToDiskResult;
        }
        
        private void enqueueDiskWrite(final MemoryCommitResult mcr,
                                  final Runnable postWriteRunnable) {
        final Runnable writeToDiskRunnable = new Runnable() {
                public void run() {
                    synchronized (mWritingToDiskLock) {
                        //写文件操作
                        writeToFile(mcr);
                    }
                    synchronized (SharedPreferencesImpl.this) {
                        //写完一个计数器减一
                        mDiskWritesInFlight--;
                    }
                    if (postWriteRunnable != null) {
                        postWriteRunnable.run();
                    }
                }
            };
        //判断是同步还是异步
        final boolean isFromSyncCommit = (postWriteRunnable == null);

        // Typical #commit() path with fewer allocations, doing a write on
        // the current thread.
        //commit()会走这里，因为commit是同步方法
        if (isFromSyncCommit) {
            boolean wasEmpty = false;
            synchronized (SharedPreferencesImpl.this) {
                //如果只有一个写操作
                wasEmpty = mDiskWritesInFlight == 1;
            }
            if (wasEmpty) {
                //一个写操作直接在当前线程中执行写文件操作，不用另起线程，写完返回
                writeToDiskRunnable.run();
                return;
            }
        }
        //如果是apply()会在线程池中执行写文件操作
        QueuedWork.singleThreadExecutor().execute(writeToDiskRunnable);
     }
     
     // Note: must hold mWritingToDiskLock
     //真正的写文件操作
    private void writeToFile(MemoryCommitResult mcr) {
        // Rename the current file so it may be used as a backup during the next read
        if (mFile.exists()) {
            if (!mcr.changesMade) {
                // If the file already exists, but no changes were
                // made to the underlying map, it's wasteful to
                // re-write the file.  Return as if we wrote it
                // out.
                //如果文件存在并且没有改变则直接返回并标记为写成功
                mcr.setDiskWriteResult(true);
                return;
            }
            if (!mBackupFile.exists()) {
                //如果要写入的文件已经存在，并且备份文件不存在时把当前文件备份一份，因为本次写操作如果失败会导致数据紊乱，下次实例化load数据时从备份文件中恢复
                if (!mFile.renameTo(mBackupFile)) {
                    Log.e(TAG, "Couldn't rename file " + mFile
                          + " to backup file " + mBackupFile);
                    //重命名失败直接返回，并且标记写失败
                    mcr.setDiskWriteResult(false);
                    return;
                }
            } else {
                //备份文件如果存在把原文件删掉，重新写新的
                mFile.delete();
            }
        }

        // Attempt to write the file, delete the backup and return true as atomically as
        // possible.  If any exception occurs, delete the new file; next time we will restore
        // from the backup.
        try {
            FileOutputStream str = createFileOutputStream(mFile);
            if (str == null) {
                //文件输出流创建失败直接返回，并标记写文件失败
                mcr.setDiskWriteResult(false);
                return;
            }
            //把mMap中的数据写入mFile文件中
            XmlUtils.writeMapXml(mcr.mapToWriteToDisk, str);
            //同步到磁盘文件中
            FileUtils.sync(str);
            str.close();
            //设置文件访问权限
            ContextImpl.setFilePermissionsFromMode(mFile.getPath(), mMode, 0);
            try {
                final StructStat stat = Os.stat(mFile.getPath());
                synchronized (this) {
                    //更新时间戳和文件大小
                    mStatTimestamp = stat.st_mtime;
                    mStatSize = stat.st_size;
                }
            } catch (ErrnoException e) {
                // Do nothing
            }
            // Writing was successful, delete the backup file if there is one.
            //写入成功则把备份文件删除
            mBackupFile.delete();
            //设置标志位为true，表示写入成功
            mcr.setDiskWriteResult(true);
            //返回
            return;
        } catch (XmlPullParserException e) {
            Log.w(TAG, "writeToFile: Got exception:", e);
        } catch (IOException e) {
            Log.w(TAG, "writeToFile: Got exception:", e);
        }
        // Clean up an unsuccessfully written file
        if (mFile.exists()) {
            if (!mFile.delete()) {
                Log.e(TAG, "Couldn't clean up partially-written file " + mFile);
            }
        }
        //上面如果没有走成功则说明写失败了，置标志位为false
        mcr.setDiskWriteResult(false);
    }
        ...
}
```
看完上面一坨代码和注释之后，我们发现原来真正写文件操作是在这里。至此，commit()方法分析完了。
**小结：**
- commit()方法是先把Editor中缓存的数据写进“内存”中，并让MemoryCommitResult的mapToWriteToDisk持有mMap的引用，然后再对mMap进行写文件操作，实际引用的是mapToWriteToDisk，再然后就是通知监听数据变化的实例。

好了，我们再来看看apply()方法...
```java
      public void apply() {
            //和commit()一样的操作，先把Editor中缓存的数据提交到mMap(内存)中
            final MemoryCommitResult mcr = commitToMemory();
            final Runnable awaitCommit = new Runnable() {
                    public void run() {
                        try {
                            //等待写文件结束
                            mcr.writtenToDiskLatch.await();
                        } catch (InterruptedException ignored) {
                        }
                    }
                };

            QueuedWork.add(awaitCommit);

            Runnable postWriteRunnable = new Runnable() {
                    public void run() {
                        awaitCommit.run();
                        QueuedWork.remove(awaitCommit);
                    }
                };

            //这里的postWriteRunnable不为null，所以会在另一个线程中进行写文件操作
            SharedPreferencesImpl.this.enqueueDiskWrite(mcr, postWriteRunnable);

            // Okay to notify the listeners before it's hit disk
            // because the listeners should always get the same
            // SharedPreferences instance back, which has the
            // changes reflected in memory.
            //如果数据有变化并且写文件成功，通知监听者
            notifyListeners(mcr);
        }
```
感觉和commit()差不多嘛，关键的还是这句`SharedPreferencesImpl.this.enqueueDiskWrite(mcr, postWriteRunnable);`,postWriteRunnable不为空，进入enqueueDiskWrite()方法后就会走线程池另起线程执行写文件操作，因而，commit和apply的区别就在于同步和异步。

### 总结
- SharedPreferences对象在第一次实例化的时候会从xml文件中读取数据并存储在Map中(也就是内存中);
- 在以后的使用中，会先根据xml文件名在ContextImpl的mSharedPrefsPaths中找到对应的File对象，然后根据包名在静态全局变量sSharedPrefsCache中找出对应的File-SharedPreferencesImpl map集合，再根据前面获得的File对象得到SharedPreferencesImpl对象，不会重复创建;
- getXXX()操作是从mMap中(内存中)拿数据;
- putXXX()只是把数据缓存在Editor的mModified中，clear()操作也只是改变了一个标志位，真正把数据提交到内存中(mMap)和写文件的是commit或者apply;
- commit()有三级锁，分别为SharedPreferences对象锁，Editor对象锁，Object对象锁，因此效率相对来说比较低，我们在使用时应该集中put操作，最后commit。

### 参考
[Android源码分析之SharedPreferences](http://blog.csdn.net/ljz2009y/article/details/36436887 "Android源码分析之SharedPreferences")
[Android应用Preference相关及源码浅析(SharePreferences篇)](http://blog.csdn.net/yanbober/article/details/47866369 "Android应用Preference相关及源码浅析(SharePreferences篇)")
[工匠若水博客](http://blog.csdn.net/yanbober/article/list/2 "工匠若水博客")