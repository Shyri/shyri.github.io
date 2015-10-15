---
layout: post
title: MaterialToolbar Tutorial
categories: [Android]
tags: [Material Design, library, Toolbar]
fullview: true
comments: true
---

This tutorial describes how to use a library I have been working on for several months. Actually it all started as a couple of classes to set custom views inside a toolbar in a "Material Design" way, and later I decided make a module from it and publish it as a library. MaterialToolbar makes it easy to place custom views inside your Android Toolbar while it handles navigation through Fragments.

The whole concept is inspired on the so called **MVP** pattern. **MaterialToolbar** library provides a **MaterialPresenter**. Initially an Activity must be attached to the MaterialPresenter. This Activity must have a MaterialToolbar view and Fragment container (i.e. FrameLayout) in its layout. Once attached, navigation through Fragments will be handled by the MaterialPresenter. 

Additionally your Fragments could provide a **MaterialToolbarContent** to the presenter. This is the view which the **MaterialPresenter** will place inside the **MaterialToolbar** when this Fragment becomes visible. From this Fragment you can interact with your toolbar components freely.

![Alt text](https://raw.githubusercontent.com/Shyri/MaterialToolbar/master/images/demo-portrait.gif) ![Alt text](https://raw.githubusercontent.com/Shyri/MaterialToolbar/master/images/demo-landscape.gif)

So let's stop talking and start coding!

### 0- Prepare your App###

First open your **AndroidManifest.xml** file and make sure your application has property *theme* set to **Theme.AppCompat.Light.NoActionBar** or any other theme that extends it. This will allow you to use the **MaterialToolbar** as the Activity ActionBar.

{% highlight xml %}
<application
        android:icon="@drawable/ic_launcher"
        android:label="@string/app_name"
        android:theme="@style/Theme.AppCompat.Light.NoActionBar" >
        <!-- ... -->
</application>
{% endhighlight %}

Then add **MaterialToolbar** as a dependency in your *build.gradle* file:

{% highlight javascript %}
dependencies {
    compile 'es.shyri:materialtoolbar:0.1.1'
}
{% endhighlight %}

### 1- Prepare your activity###
Create a new Activity with the following layout, this is a minimum for the **MaterialPresenter** in order to work correctly:

{% highlight xml %}
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    tools:context=".MainActivity">

    <es.shyri.materialtoolbar.MaterialToolbar
        android:id="@+id/main_toolbar"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:animateLayoutChanges="true"
        android:background="?attr/colorPrimary"
        android:minHeight="?attr/actionBarSize" />

    <FrameLayout
        android:id="@+id/fragmentContainer"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:animateLayoutChanges="true" />

</LinearLayout>
{% endhighlight %}

As you can see, there is a **MaterialToolbar** and a **FrameLayout**. The first will be used by the presenter to set the content view you want to show. The second will also be used by the presenter which will place your Fragments there.

**Note:** If you want the **MaterialToolbar** to have smooth transitions set the property *animateLayoutChanges* to true in both the Toolbar and the FrameLayout.

Now override your Activity's onCreate() method:

{% highlight java %}
	@Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        // 1
        presenter = MaterialPresenter.getInstance();

        // 2
        presenter.attachActivity(this, R.id.main_toolbar, R.id.fragmentContainer);  
        
        // 3
        if (getFragmentManager().findFragmentById(R.id.fragmentContainer) == null) {
            presenter.navigateTo(new MyFirstFragment());
        }
    }
{% endhighlight %}

In **1** we get the **MaterialPresenter** singleton. In **2** you attach this activity to the presenter and you must provide the **MaterialToolbar**'s' and **FrameLayout**'s' ids. In **3** you are telling the presenter to navigate to **MyFirstFragment** when the app starts.

Now override the onDestroy() method:
{% highlight java %}
	@Override
    protected void onDestroy() {
        super.onDestroy();
        presenter.detachActivity();
    }
{% endhighlight %}

The detachActivity() method will detach this activity. This is very important in order to avoid crashes by configuration changes (i.e. device rotation) and view leaks.

Finally override the *onBackPressed()* method to let the presenter handle the *Fragment* backstack.

Optionally you can automate the display the back arrow in the toolbar making your Activity implement **FragmentManager.OnBackStackChangedListener** by adding this line to the Activity's onCreate() method:

{% highlight java %}
	@Override
    protected void onCreate(Bundle savedInstanceState) {
    	//[...]
    	getFragmentManager().addOnBackStackChangedListener(this);
    	//[...]
    }
    //[...]

    @Override
    public void onBackStackChanged() {
        getSupportActionBar().setDisplayHomeAsUpEnabled(getFragmentManager().getBackStackEntryCount() > 1);
    }
{% endhighlight %}

### 2- Prepare your Fragments ###

Override the Fragment's onCreateView() method:

{% highlight java %}
	@Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
    	// [...] infalte your fragment view
    	
    	// 1
    	toolbarContent = new MaterialToolbarContent(getActivity(), R.layout.my_awesome_toolbar);

    	// 2
    	presenter.getToolbar().setContent(toolbarContent);

    	// 3
    	Button myAwesomeButton = toolbarContent.findViewById(R.id.myAwesomeButton);
		myAwesomeButton.findViewById(R.id.myAwesomeButton)
			.setOnClickListener(new View.OnClickListener() {
            	@Override
            	public void onClick(View v) {
                	presenter.navigateTo(new OtherFragment());
            	}
        });

    	// [...]
    	return view;
    }
{% endhighlight %}	

**1:** Create your **MaterialToolbarContent** from an xml layout. Note you can write several layout files targeting different configurations, for example one for portrait and another for landscape mode.

**2:** Tell the Presenter to take this *toolbarContent* and place it inside the **MaterialToolbar*.

**3:** Instantiate any view your **MaterialToolbarContainer** may have and set any listener you want.
Note that once you have the presenter instance, you can call *navigateTo()* method whenever you want in oder to navigate to other Fragment

So... where do you get the presenter instance from? Don't worry, presenter will arrive through an interface method, but in exchange you must implement another method to give him the **MaterialToolbarContent** just in case he needs it again later.

{% highlight java %}
	@Override
    public void setPresenter(MaterialPresenter presenter) {
        this.presenter = presenter;
    }

    @Override
    public MaterialToolbarContent getToolbarContent() {
        return toolbarContent;
    }
{% endhighlight %}


Only those Fragments which have custom view layout inside the **MaterialToolbar** need to implement **MaterialToolbarSupplier** interface.

### 3- Bonus tips: ###
At this point you'll have a toolbar without the shadow specified by **Material Design** guidelines. To achieve this just edit the Activity's layout file adding an elevation of 4dp to the **MaterialToolbar**. This will only work for Android 21 and further. In order to give shadow for earlier versions add the property *android:foreground="?android:windowContentOverlay"* to the Fragment container layout:

{% highlight xml %}
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    xmlns:es.shyri.materialtoolbar.sample="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    tools:context=".MainActivity">

    <es.shyri.materialtoolbar.MaterialToolbar
        android:id="@+id/main_toolbar"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:animateLayoutChanges="true"
        android:background="?attr/colorPrimary"
        android:minHeight="?attr/actionBarSize"
        es.shyri.materialtoolbar.sample:theme="@style/ThemeOverlay.AppCompat.ActionBar"
        android:elevation="4dp"/>

    <FrameLayout
        android:id="@+id/fragmentContainer"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:animateLayoutChanges="true"
        android:foreground="?android:windowContentOverlay"/>

</LinearLayout>
{% endhighlight %}


And that's pretty much all. Hope you've found it useful. Feel free to give your feedback and contribute to the project on Github: https://github.com/Shyri/MaterialToolbar , where you'll find a sample app using the library.

Enjoy!
