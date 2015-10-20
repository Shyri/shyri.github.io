---
layout: post
title: Tutorial MaterialToolbar
categories: [Android]
tags: [Material Design, library, Toolbar]
fullview: true
comments: true
name: material-toolbar-tutorial
lang: es
---

Escribo este tutorial para enseñar a usar una pequeña librería en la que he trabajado intermitentemente desde hace tiempo. En relidad surge de la necesidad de hacer una Toolbar dinámica al estilo "Material Design" para una de nuestras apps y que más tarde decidí extraer como módulo para utilizar en otras aplicaciones, llamándolo "MaterialToolbar". La librería te facilita posicionar vistas personalizadas dentro de una Toolbar de android mientras maneja la navegación por Fragments del Activity que contiene la Toolbar.

El concepto está inspirado en el patrón **MVP** (Modelo Vista Presentador). **MaterialToolbar** incluye una clase **MaterialPresenter** encargada de hacer el papel de presentador. Inicialmente necesita que una Activity se ancle al MaterialPresenter. Este Activity debe contener en su layout al menos una vista **MaterialToolbar** y otra vista que sirva para poner los Fragments (por ejemplo un FrameLayout). Una vez anclado, la navegación a través de los fragments será manejado por el MaterialPresenter.

Además, los Fragments podrán proveer de un **MaterialToolbarContent** al presenter, que será el contenido de la Toolbar que se quiere que aparezca cuando el Fragment se hace visible, y te permitirá interactuar con todas las vistas fácilmente.

![Alt text](https://raw.githubusercontent.com/Shyri/MaterialToolbar/master/images/demo-portrait.gif) ![Alt text](https://raw.githubusercontent.com/Shyri/MaterialToolbar/master/images/demo-landscape.gif)

Basta de palabras y vamos a programar!

### 0- Preparar la app###

Lo primero, abre en tu **AndroidManifest.xml** y asegurate de poner el *theme* a **Theme.AppCompat.Light.NoActionBar** o alguno que extienda de él. Esto permitirá a **MaterialToolbar** ser el Actionbar del Activity.

{% highlight xml %}
<application
        android:icon="@drawable/ic_launcher"
        android:label="@string/app_name"
        android:theme="@style/Theme.AppCompat.Light.NoActionBar" >
        <!-- ... -->
</application>
{% endhighlight %}

Añadimos **MaterialToolbar** como dependencia dentro del fichero *build.gradle* del módulo de la app.

{% highlight javascript %}
dependencies {
    compile 'es.shyri:materialtoolbar:0.1.1'
}
{% endhighlight %}

### 1- Preparar el Activity###
Esta librería está pensada para aplicaciones cuya navegación principal se basa en Fragments que se muestran dentro de un mismo Activity. Esto no tiene por qué ser así, se pueden tener muchas Activity de navegación, pero aquí explicaremos como utilizarla con un solo Activity.

Crea un nuevo Activity con el siguiente layout, esto es lo mínimo indispensable para que el **MaterialPresenter** pueda funcionar correctamente:

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

Como puedes ver, hemos incluido un **MaterialToolbar** y un **FrameLayout**. El primero será usado por el presentador para poner en ella la vista que le diga cada fragment. Y el segundo, para poner el fragment en si.

**Nota:** Si quieres que el **MaterialToolbar** tenga una animación chula cuando su contenido cambia, pon la propiedad *animateLayoutChanges* a *true* tanto en el **MaterialToolbar** como en el **FrameLayout**

Ahora tenemos que sobreescribir el método *onCreate()* del Activity:

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

En el punto **1** recuperamos el singleton **MaterialPresenter**. Como he dicho antes, el tutorial se plantea como una app de una sola Activity de navegación, así que asumimos que será suficiente con tener una sola instancia de **MaterialPresenter** para toda la app. 

En el punto **2** anclamos el activity al presenter con el método *attachActivty()*. Es necesario además decirle cuáles son los ids del **MaterialToolbar** y el **FrameLayout** dentro del Activity.

Ahora sobreescribimos el método *onDestroy()*:

{% highlight java %}
	@Override
    protected void onDestroy() {
        super.onDestroy();
        presenter.detachActivity();
    }
{% endhighlight %}

El método *detachActivity()* deshará el anclaje entrel el activity y el presentador. Esto es muy importante para evitar crashes en los cambios de configuración (por ejemplo en rotaciones) y también para evitar view leaks.

Finalmente sobreescribimos el método *onBackPressed()* para dejar que el presentador maneje el backstack de Fragments.

Opcionalmente puedes añadir un poco de código para que la flecha de "back" de la barra superior se muestre siempre que exista un Fragment "anterior" en la navegación. Esto lo puedes hacer añadiendo esta línea en el método *onCreate()*:

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

### 2- Preparar los Fragment ###

Si quieres que un Fragment provea de una vista personalizada para el Toolbar cuando éste aparezca, primero sobreescribe el método *onCreateView()*:

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

**1:** Crear  tu **MaterialToolbarContent** desde un layout xml. Atento: igual que para el resto de vistas puedes tener distintos fichero xml que permitan distinta visualización del contenido de la barra en distintas configuraciones, por ejemplo si el dispositivo está en vertical u horizontal.

**2:** Dile al **MaterialPresenter** que coja este *toolbarContent* y lo ponga dentro de la **MaterialToolbar**

**3:** Instancia cualquier vista de tu **MaterialToolbarContent** y añade cualquier listener que quieras. Puedes ver cómo hemos hecho una llamada al presenter para que al hacer click en un botón navegue hacia otro Fragment con el método *navigateTo()*.

Y... ¿De dónde sacamos la instancia del presenter? Para que este Fragment pueda darle el contenido de la barra superior al presenter, es necesario implementar el interfaz **MaterialToolbarSupplier** y sus métodos:

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

### 3- Trucos bonus: ###
Si has seguido el tutorial, en este punto tendrás una Toolbar que no tiene la típica sombra que dictan las guías de diseño de **Material Design** de Google. Para conseguirlo, solo tienes que añadir el parámetor *elevation="4dp"* en el **MaterialToolbar** del xml de layout del Activity. Esto solo funcionará para Android 21 y versiones más recientes. Para hacerlo funcionar en versiones previas es necesario añadir el parámetro *android:foreground="?android:windowContentOverlay"* al **FrameLayout** del activity principal

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

¡Eso es todo! Espero que encuentres útil esta librería y que este tutorial te haya ayudado a entenderlo un poco mejor. Si tienes opiniones, si ves algún error o simplemente quieres comentarme algo, puedes contactarme por las distintas vías que aparecen aquí.

Puedes ver el código de la librería en Github: <a href="https://github.com/Shyri/MaterialToolbar">https://github.com/Shyri/MaterialToolbar</a>

A programar!
