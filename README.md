#在Android布局中经常需要复用title布局，但是每次定义很麻烦，并且如果哪天即便只需要修改一个背景就会让你抓狂了，并且每个title只是内容不一样，一般只有左边按钮，中间文字和左边按钮，所以我们可以自定义一个TopBar，以便每次复用。

#首先自定义控件需要分析需要哪些属性，这里就简单的写了几个，有文字大小，颜色和内容等，然后仿照系统的处理方法创建一个attrs.xml文件在里面声明一下。如下：
    <declare-styleable name="topBar">
        <!--中间的标题-->
        <attr name="CenterText" format="string"></attr>
        <attr name="CenterTextColor" format="color"></attr>
        <attr name="CenterTextSize" format="dimension"></attr>
        <!--左边的Button-->
        <attr name="LeftButtonText" format="string"></attr>
        <attr name="LeftButtonTextColor" format="color"></attr>
        <attr name="LeftButtonBackground" format="color|reference"></attr>

        <!--右边的Button-->
        <attr name="RightButtonText" format="string"></attr>
        <attr name="RightButtonTextColor" format="color"></attr>
        <attr name="RightButtonBackground" format="color|reference"></attr>
    </declare-styleable>
#然后创建一个TopBar类集成系统的View/ViewGroup，这里当然选择继承RelativeLayout了，在这里面的构造方法中完成对声明的属性的获取与使用。
    public TopBar(Context context, AttributeSet attrs) {
        super(context, attrs);
        initAttrs(context, attrs);
    }

    private void initAttrs(Context mContext, AttributeSet mAttrs) {
        TypedArray ta = mContext.obtainStyledAttributes(mAttrs, R.styleable.topBar);

        //左边Button的属性
        leftButtonText = ta.getString(R.styleable.topBar_LeftButtonText);
        leftButtonTextColor = ta.getColor(R.styleable.topBar_LeftButtonTextColor, 0);
        leftDrawable = ta.getDrawable(R.styleable.topBar_LeftButtonBackground);

        //右边Button的属性
        rightButtonText = ta.getString(R.styleable.topBar_RightButtonText);
        rightButtonTextColor = ta.getColor(R.styleable.topBar_RightButtonTextColor, 0);
        rightDrawable = ta.getDrawable(R.styleable.topBar_RightButtonBackground);

        //中间标题
        centerText = ta.getString(R.styleable.topBar_CenterText);
        centerTextColor = ta.getColor(R.styleable.topBar_CenterTextColor, 0);
        centerTextSize = ta.getDimension(R.styleable.topBar_CenterTextSize, 16);

        ta.recycle();
        //创建子控件
        leftButton = new Button(mContext);
        leftButton.setText(leftButtonText);
        leftButton.setBackground(leftDrawable);
        leftButton.setTextColor(leftButtonTextColor);

        rightButton = new Button(mContext);
        rightButton.setText(rightButtonText);
        rightButton.setBackground(rightDrawable);
        rightButton.setTextColor(rightButtonTextColor);

        centerTextView = new TextView(mContext);
        centerTextView.setText(centerText);
        centerTextView.setTextColor(centerTextColor);
        centerTextView.setTextSize(centerTextSize);
        centerTextView.setGravity(Gravity.CENTER);

        //添加位置属性
        leftLayoutParams = new LayoutParams(ViewGroup.LayoutParams.WRAP_CONTENT, ViewGroup.LayoutParams.WRAP_CONTENT);
        leftLayoutParams.addRule(RelativeLayout.ALIGN_PARENT_LEFT, TRUE);
        addView(leftButton, leftLayoutParams);

        rightLayoutParams = new LayoutParams(ViewGroup.LayoutParams.WRAP_CONTENT, ViewGroup.LayoutParams.WRAP_CONTENT);
        rightLayoutParams.addRule(RelativeLayout.ALIGN_PARENT_RIGHT, TRUE);
        addView(rightButton, rightLayoutParams);


        centerLayoutParams = new LayoutParams(ViewGroup.LayoutParams.WRAP_CONTENT, ViewGroup.LayoutParams.WRAP_CONTENT);
        centerLayoutParams.addRule(RelativeLayout.CENTER_IN_PARENT, TRUE);
        addView(centerTextView, centerLayoutParams);
    }
#然后就可以在布局中使用它了，当然了不能忘记这句命名空间代码了（xmlns:topBar="http://schemas.android.com/apk/res-auto"）：
     <topbar.tomcode.com.topbar.TopBar
        android:id="@+id/topbar"
        android:layout_width="match_parent"
        android:layout_height="50dp"
        android:layout_alignParentLeft="true"
        android:layout_alignParentStart="true"
        android:layout_alignParentTop="true"
        topBar:CenterText="中间标题"
        topBar:CenterTextColor="#00ff00"
        topBar:CenterTextSize="12dp"
        topBar:LeftButtonBackground="#ffd300"
        topBar:LeftButtonText="左边"
        topBar:LeftButtonTextColor="#ff0000"
        topBar:RightButtonBackground="#ffd300"
        topBar:RightButtonText="右边"
        topBar:RightButtonTextColor="#ff0000" />
#以上就完成了最基本的功能。
#当然点击事件也是必不可少的，这里需要自己仿照系统的来定义：
##首先定义一个接口：
     //定义点击topBar这个整体控件的接口
    public interface OnClickTopBarListener {
        void leftClick();

        void rightClick();
    }
##暴露给使用者一个方法，同时需要使用者传给我一个监听者对象，以便使用这个对象去监听。
     //暴露给使用者的方法
    public void setOnClickTopBarListener(OnClickTopBarListener mOnClickTopBarListener) {
        this.mOnClickTopBarListener = mOnClickTopBarListener;
    }
##触发监听的方法，最好能判断下监听者是否为空，这里不再判断了：
    leftButton.setOnClickListener(new OnClickListener() {
            @Override
            public void onClick(View v) {
                //触发回调监听
                mOnClickTopBarListener.leftClick();
            }
        });
        rightButton.setOnClickListener(new OnClickListener() {
            @Override
            public void onClick(View v) {
                mOnClickTopBarListener.rightClick();
            }
        });
#使用点击事件，和系统的一样一样的
     TopBar topbar = (TopBar) findViewById(R.id.topbar);
        topbar.setOnClickTopBarListener(new TopBar.OnClickTopBarListener() {
            @Override
            public void leftClick() {
                Toast.makeText(MainActivity.this, "点击了左边按钮", Toast.LENGTH_SHORT).show();
            }

            @Override
            public void rightClick() {
                Toast.makeText(MainActivity.this, "点击了右边按钮", Toast.LENGTH_SHORT).show();
            }
        });
#当然还可以添加一些隐藏按钮的方法，以便在不需要的时候隐藏掉，这里不再赘述，只是起一个抛砖引玉的作用。