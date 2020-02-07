# BaseLibrary
我自己的工具类合集

maven{
https://raw.githubusercontent.com/freedomangelly/BaseLibrary/master
}

dependencies {
    implementation 'com.liuy.maven.baselibrary:baselibrary:0.0.0.0'
}
倒入位置可以参考我的简书https://www.jianshu.com/p/1d42f9db2a8d

V.0.0.0.0
1.异常机制
ExceptionCrashHandle 全局异常类 当应用发生异常会在此handle种处理
使用方法，在applicaton的onCreate中调用
ExceptionCrashHandle.getInstance().init(this);
也可以在其他线程中
this.setUncaughtExceptionHandler(ExceptionCrashHandle.getInstance());

2.IOC机制
可以向butternikfe一样使用，不过是利用反射机制
包含方法
2.1.初始化组件
@ViewById(R.id.recycler_view)
private WrapRecyclerView mRecyclerView;
2.2.点击事件
    @OnClick(R.id.home_rb)
    private void homeRbClick(View view) {
        if (mHomeFragment == null) {
            mHomeFragment = new HomeFragment();
        }
        mFragmentHelper.switchFragment(mHomeFragment);
    }
2.3.检测网络
@CheckNet(R.id.recycler_view)
再上面click事件同时加上注释，会为点击事件添加网络检测
2.4.权限检测
@PermissionSuccess(requestCode = CALL_PHONE_REQUEST_CODE)//权限申请成功返回
@PermissionFail(requestCode = CALL_PHONE_REQUEST_CODE)//权限申请失败返回

3.权限检测
PermissionHelper 权限工具类
PermissionHelper.with(Activity)
                    .requestCode(请求码)
                    .requestermission(请求权限).request();
或者
PermissionHelper.requestPermission(Activity,请求码，请求权限[])

    在Activity中的onRequestPermissionsResult里调用 PermissionHelper.requestPermissionsResult方法
        @Override
    public void onRequestPermissionsResult(int requestCode, @NonNull String[] permissions, @NonNull int[] grantResults) {
        super.onRequestPermissionsResult(requestCode, permissions, grantResults);

        PermissionHelper.requestPermissionsResult(this,requestCode,permissions,grantResults);
    }
    
   之后会返回2.4中对应的注释方法。
   
   4.网络的使用
   此框架中使用了OkHttp与retrofit（我们有测试）作为引擎，也可以自己定义网络引擎,包含了get,post,delete,update,download的请求方式
   4.1HttpUtils为网络引擎的使用类，在Application中调用HttpUtils.init(new OkHttpEngine(new AuthorizationInterceptor()));初始化网络引擎    AuthorizationInterceptor为自定义拦截器，如果不需要可以设置null
   
   4.2EngineCallBack默认的网络回调接口
        //请求前传递的参数
        public void onPreExecute(Context context, Map params) {}
        //请求失败返回结果
        public void onError(Object e) {}
        //请求成功返回结果，code为返回的请求码，result为服务器返回的数据
        public void onSuccess(int code,String result) {}
        
   4.3 DownloadListener
      DownloadListener为下载用的回调函数方，包含方法如下
      void onPreExecute(Context context, Map<String, Object> params);
    /**
     *  开始下载
     */
    void start(long max);
    /**
     *  正在下载
     */
    void loading(int progress);
    /**
     *  下载完成
     */
    void complete(String path);
    /**
     *  下载过程中失败
     */
    void loadfail(Object e);
        
   4.4.请求的使用
   HttpUtils.with(上下文).url(请求连接).请求方式.execute(new EngineCallBack());
   请求方式：get() post() delete() update() download()
   
   4.5下载的使用
    HttpUtils.with(上下文).url(请求连接).download().downPath().execute(new DownloadListener())
    
  5.数据库的使用
  使用数据库需要一个数据库的对象类，数据库会通过反射，动态获取这个类与数据库中的字段相对应
  数据库类的初始化
  IDaoSoupport<CacheData> dataDaoSupport = DaoSupportFactory.getFactory().getDao(数据库对象类);
  
  IDaoSoupport中方法的定义
      public long insert(T t);
    //批量插入
    public void insert(List<T> data);
    //获取专门查询的支持类
    public QuerySupport<T> querySupport();

    int delete(String whereClause,String[] whereArgs);

    int update(T obj, String whereClause, String... whereArgs);
    数据库使用栗子
        public static String getCacheResultJson(String finalUrl) {
        final IDaoSoupport<CacheData> dataDaoSupport = DaoSupportFactory.getFactory().getDao(CacheData.class);
        List<CacheData> cacheDatas =  dataDaoSupport.querySupport().selection("mUrlKey=?").selectionArgs(OkHttpEngine.MD5Utils.stringToMD5(finalUrl)).query();
        if (cacheDatas.size() != 0) {
            CacheData cacheData = cacheDatas.get(0);
            String request = cacheData.getmResultJson();
            return request;
        }
        return null;
    }
    
6.万能自定义Dialog的使用
使用自定义布局文件，方便快速的开发
使用例子
AlertDialog quickDialog = new AlertDialog.Builder(UserVCardActivity.this)
                            .setContentView(R.layout.dialog_title)//布局文件
                            .setText(R.id.tv_dialog_title_title, getString(R.string.str_message))//标题
                            .setText(R.id.tv_dialog_title_single, getString(R.string.str_openCard_success))//显示内容
                            .setViewVisable(R.id.ll_mix_dialog_bottom, View.GONE)//底部布局因此
                            .setViewVisable(R.id.ll_mix_dialog_bottom_one, View.VISIBLE)
                            .setOnClickListener(R.id.btn_dialog_title_center, new View.OnClickListener() {//监听事件
                                @Override
                                public void onClick(View v) {
                                    quickDialog.dismiss();
                                    finish();
                                }
                            })
                            .fullWidth()
                            .show();
                    quickDialog.setCanceledOnTouchOutside(false);
                    quickDialog.setCancelable(false);
                    
