## Android全局异常处理

在项目中，我们常常会遇到Crash的现象，也就是程序崩溃的时候，此时应用程序会表现出闪退或已停止的现象，可能会给用户带来很不好的体验，这需要我们重新定义程序崩溃时发出一些友情提示。

这里使用Thread.UncaughtExceptionHandler接口来实现这一问。

### 自定义CashHandler

```java
**
 * 自定义的 异常处理类 , 实现了 UncaughtExceptionHandler接口
 */
public class CashHandler implements Thread.UncaughtExceptionHandler {
    private static final String TAG = "CashHandler";
    //文件夹目录
    private static final String PATH = Environment.getExternalStorageDirectory().getPath() + "/crash_log/";
    //文件名
    private static final String FILE_NAME = "crash";
    //文件名后缀
    private static final String FILE_NAME_SUFFIX = ".trace";
    private static CashHandler sCashHandler = new CashHandler();
    private  Context mContext;
    
    private CrashHandler(){  
          
    }
    
    public static CashHandler getInstance(){
        return  sCashHandler;
    }

    public  void init(Context context){
        Thread.setDefaultUncaughtExceptionHandler(sCashHandler);
        mContext = context;
    }
    
    /**
     * 当有异常产生时执行该方法
     * @param thread 异常产生的线程
     * @param ex 异常信息
     */
    @Override
    public void uncaughtException(Thread t, Throwable e) {
        //使用Toast来显示异常信息
        new Thread() {
            @Override
            public void run() {
                Looper.prepare();//准备发消息的MessageQueue
                Toast.makeText(mContext, "很抱歉,程序出现异常,即将退出.", Toast.LENGTH_LONG).show();
                Looper.loop();
            }
        }.start();
        //导出异常信息到SD卡
        dumpExceptionToSDCard(e);
        //上传异常信息到服务器
        uploadExceptionToServer(e);
        // 隔两秒直接退出程序，不会出现重启或弹窗。
        SystemClock.sleep(2000);
        android.os.Process.killProcess(android.os.Process.myPid());
        System.exit(1);
    }
    /**
     * 导出异常信息到SD卡
     * @param ex
     */
    private void dumpExceptionToSDCard(Throwable ex) {
        if (!Environment.getExternalStorageState().equals(Environment.MEDIA_MOUNTED)) {
            return;
        }
        //创建文件夹
        File dir = new File(PATH);
        if (!dir.exists()) {
            dir.mkdirs();
        }
        //获取当前时间
        long current = System.currentTimeMillis();
        String time = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss").format(new Date(current));
        //以当前时间创建log文件
        File file = new File(PATH + FILE_NAME + time + FILE_NAME_SUFFIX);
        if (!file.exists()){
            file.exists();
        }
        try {
            //输出流操作
            PrintWriter pw = new PrintWriter(new BufferedWriter(new FileWriter(file)));
            //导出手机信息和异常信息
            PackageManager pm = mContext.getPackageManager();
            PackageInfo pi = pm.getPackageInfo(mContext.getPackageName(), PackageManager.GET_ACTIVITIES);
            pw.println("发生异常时间：" + time);
            pw.println("应用版本：" + pi.versionName);
            pw.println("应用版本号：" + pi.versionCode);
            pw.println("android版本号：" + Build.VERSION.RELEASE);
            pw.println("android版本号API：" + Build.VERSION.SDK_INT);
            pw.println("手机制造商:" + Build.MANUFACTURER);
            pw.println("手机型号：" + Build.MODEL);
            ex.printStackTrace(pw);
            //关闭输出流
            pw.close();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
    /**
     * 上传异常信息到服务器
     * @param ex
     */
    private void uploadExceptionToServer(Throwable ex) {

    }
```
在Application中注册未捕获异常处理类,并在AndroidManifest.xml中注册

```java
    public class CashApplication extends Application {
        @Override
        public void onCreate() {
            super.onCreate();
            CashHandler.getInstance().init(getApplicationContext());
        }
    }
    
    <application
        android:name=".CashApplication"
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/AppTheme">
        <activity android:name=".MainActivity">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />

                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
    </application>

```

测试异常代码

```ava
    StringBuilder strError = null ;
    System.out.println(strError.toString());
```

在测试异常代码前需要进行权限申请，否则会报错:

```java
    System.err: java.io.FileNotFoundException:/storage/emulated/0/crash_log/crash2018-09-15 15:37:07.trace (Permission denied)
```


需要申请的权限：

```java
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
    <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
```

按照上面的代码来测试这个异常捕获，运行程序，在文件夹中找到我们创建的日志文件：

![1537026216(1).jpg](https://upload-images.jianshu.io/upload_images/4594291-c2cf314a06d884b6.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

查看对应的信息：
![1537026357(1).jpg](https://upload-images.jianshu.io/upload_images/4594291-48d51727db1d66c4.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

从异常日志文件中，我们可以明显的定位到错误代码行数，然后进行修正。

除了自定义异常捕获外，还可以借助第三方SDK:如腾讯 Bugly，友盟

总结及注意事项：
 -  如果代码被proguad混淆后，输出的crash log，出错的行号看不出来（Unknown Source）
   
 > 需要在代码混淆的时候，加上在混淆文件里面记得加上这句：

     -keepattributes SourceFile,LineNumberTable # keep住源文件以及行号
 - java层crash log收集
   - 自己收集，可以实现系统的UncaughtExceptionHandler类，并重写uncaughtException方法，将crash log写到手机本地，在适时的时候，上传到我们的服务器，优点是，所需类和方法少，可以自定义log输出，缺点是没有第三方平台强大，只能收集crash log,统计信息等需要自己去做；

   - 借助第三方平台，如友盟、腾讯 Bugly等，这些平台不仅能收集crash log,还添加了丰富的统计信息，可以很方便看出各版本、各平台等出现的crash,缺点是，导入第三库，代码量肯定比自己重写的类大，无法自定义应用崩溃时弹框；


### 参考文章

[Android平台的崩溃捕获机制及实现](http://www.uml.org.cn/mobiledev/2016011510.asp)

[https://www.jianshu.com/p/16b0b2fd8c0e](https://www.jianshu.com/p/16b0b2fd8c0e)

[http://stackoverflow.com/questions/1083154/how-can-i-catch-sigsegv-segmentation-fault-and-get-a-stack-trace-under-jni-on](http://stackoverflow.com/questions/1083154/how-can-i-catch-sigsegv-segmentation-fault-and-get-a-stack-trace-under-jni-on)

[https://www.jianshu.com/p/16b0b2fd8c0e](https://www.jianshu.com/p/16b0b2fd8c0e)


