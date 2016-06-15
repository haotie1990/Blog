# 访问者模式与ButterKnife

## 访问者模式

### 1. 访问者模式介绍

访问者模式是一种将数据操作与数据结构分离的设计模式。在一个软件系统中拥有一个由许多对象构成，比较稳定的对象结构，这些类都有一个`accept`方法来接收访问者对象的访问。访问者是一个接口，它拥有一个`vist`方法，这个方法会对访问到的对象结构中不同类型的元素做出不同的处理。

### 2. 访问者模式定义

封装一些作用于某种数据结构中的各元素的操作，它可以在不改变这个数据结构的前提下定义作用于这些元素的新的操作。

### 3. 访问者模式使用场景

* 对象结构比较稳定，但经常需要在此对象结构上定义新的操作
* 需要对一个对象结构中的对象进行很多不同步的并且不相关的操作，而需要避免这些操作“污染”这些对象的类，也不希望在增加新操作时修改这些类。

### 4. 访问者模式UML

![](../Image/1335165175_6219.jpg)

## ButterKnife

### 1. ButterKnife介绍

ButterKnife是通过注解处理生成样板代码来完成`Android View`的属性和方法的绑定的类库。它简化了以往Android开发中初始化`view`和设置监听的操作。

### 2. Butterknife特性

* 通过使用`@BindView`代替`findViewById`初始化`view`
* 可以同时处理列表或数组中的多个`view`
* 使用类似`@OnClick`的注解替代原有匿名内部类监听方法
* 使用类似`@BindString`的资源注解来替代原有的`getResouces()`查询资源方式

### 3. ButterKnife使用

#### 3.1 配置Android Studio工程

最新版`8.0.1`的ButterKnife引入了`apt-plugin`,在编译期间生成代码，提高了ButterKnife注解的使用效率。

首先打开工程项目的`build.gradle`，引入`apt`编译插件

```groovy
buildscript {
    repositories {
        jcenter()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:2.1.0'
        classpath 'com.neenbedankt.gradle.plugins:android-apt:1.8'
        // NOTE: Do not place your application dependencies here; they belong
        // in the individual module build.gradle files
    }
}
```

再打开`app`下的`build.gradle`，在`app`下使用`apt`插件，并设置生成索引的包名和类名

```groovy
apply plugin: 'com.neenbedankt.android-apt'

dependencies {
    compile 'com.jakewharton:butterknife:8.0.1'
    apt 'com.jakewharton:butterknife-compiler:8.0.1'
}
```

#### 3.2 View和Resouce

首先需要使用ButterKnife绑定指定的`Activity`，在`onCreate`中调用`ButterKnife.bind(Activity activity)`方法，需要注意的是，调用必须在`setContentView`后执行。初始化View只需要使用`@BindView`注解绑定指定的`View ID`即可。

```java
public class MainActivity extends AppCompatActivity {

    @BindView(R.id.et_name)
    EditText mEtName;

    @BindView(R.id.et_password)
    EditText mEtPasswd;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        ButterKnife.bind(this);
    }
}
```
对于资源可以使用：`@BindBool, @BindColor, @BindDimen, @BindDrawable, @BindInt, @BindString`获取。

```java
public class MainActivity extends AppCompatActivity {
    //--
    @BindString(R.string.str_user_name) String strName;
    @BindString(R.string.str_user_passwod) String strPasswd;

    @BindDimen(R.dimen.layout_item_height) int layoutItemHeight;

    @BindColor(R.color.colorPrimary) int mColorPrimary;
    //--
}
```

对于`Fragment`或`ViewHolder`这样的对象同样可以使用ButterKnife。但要使用方法：`bind(@NonNull Object target, @NonNull View source)`。

```java
public class UserInfoFragment extends Fragment{

    @BindView(R.id.iv_photo)
    ImageView mUserPhoto;

    @Nullable
    @Override
    public View onCreateView(LayoutInflater inflater, @Nullable ViewGroup container, @Nullable Bundle savedInstanceState) {
        View bodyView = inflater.inflate(R.layout.fragment_userinfo, container, false);
        ButterKnife.bind(this, bodyView);
        return bodyView;
    }
}
```

ButterKnife还可以同时作用在多个view上，并通过`apply`方法对多个view的数组或列表进行统一的操作。使用`apply`方法需要实现`Action`或`Setter`接口，他们可以实现一些简单的处理。

```java
static final ButterKnife.Action<View> ENABLE = new ButterKnife.Action<View>() {
        @Override
        public void apply(@NonNull View view, int index) {
            view.setEnabled(true);
        }
};

static final ButterKnife.Setter<View, Integer> VISIABLE = new ButterKnife.Setter<View, Integer>() {
        @Override
        public void set(@NonNull View view, Integer value, int index) {
            view.setVisibility(value);
        }
};

    @BindViews({R.id.et_password, R.id.bt_submit, R.id.bt_cancel})
    List<View> mViews;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        ButterKnife.bind(this);

        ButterKnife.apply(mViews, ENABLE);

        ButterKnife.apply(mViews, VISIABLE, View.VISIBLE);
    }

    //--
```

`apply`也可以改变View的属性值。

```java
ButterKnife.apply(mViews, View.ALPHA, 0.5f);
```

#### 3.3 设置监听

Butterknife同样简化了设置view的监听功能，通过`@OnClick, @OnTouch, @OnLongClick, @OnCheckedChanged, @OnEditorAction, @OnFocusChange, @OnItemClick, @OnPageChange, @OnTextChanged`可以完成以前需要设置匿名监听类才能完成的工作，而且代码简洁。

NOTE:不同的注解方式有不同的参数和返回值

```java
@OnClick(R.id.bt_submit)
public void onBtSubmit(){
    Log.i(MainActivity.class.getSimpleName(), "onBtSubmit");
}

@OnClick(R.id.bt_cancel)
public void onBtCancel(View view){
    Log.i(MainActivity.class.getSimpleName(), "onBtCancel");
}

@OnTextChanged(value = {R.id.et_name, R.id.et_password}, callback = OnTextChanged.Callback.TEXT_CHANGED)
public void onTextChange(CharSequence text, int start, int before, int count){
    Log.i(MainActivity.class.getSimpleName(), "onTextChange");
}
```

#### 3.4 设置解绑

对于`Fragment`有不同于`Activity`的生命周期，在`onCreateView`方法中初始化，但在`onDestroyView`方法中其`View`会被置`null`，此时需要解绑ButterKnife。

```java
public class UserInfoFragment extends Fragment{

    @BindView(R.id.iv_photo)
    ImageView mUserPhoto;

    private Unbinder mUnbinder;

    @Nullable
    @Override
    public View onCreateView(LayoutInflater inflater, @Nullable ViewGroup container, @Nullable Bundle savedInstanceState) {
        View bodyView = inflater.inflate(R.layout.fragment_userinfo, container, false);
        mUnbinder = ButterKnife.bind(this, bodyView);
        return bodyView;
    }

    @Override
    public void onDestroyView() {
        super.onDestroyView();
        mUnbinder.unbind();
    }
}
```

#### 3.5 安全的绑定

默认情况下，绑定view或者设置监听要求必须可以找到目标view，否则会抛出异常。可以使用`@Nullable`在view的初始化，`@Optional`在设置监听是避免抛出异常。

```java
@Nullable @BindView(R.id.et_name)
EditText mEtName;

@Nullable @BindView(R.id.et_password)
EditText mEtPasswd;

@Optional @OnClick(R.id.bt_submit)
public void onBtSubmit(){
    Log.i(MainActivity.class.getSimpleName(), "onBtSubmit");
}
```

#### 3.6 更简单的初始化View

如果觉得使用`@BindView`注解和`ButterKnife.bind()`方法有些麻烦，可以使用`findById`方法初始化指定`view, Activity, Dialog`中的view。

```java
public class UserInfoFragment extends Fragment{

    private ImageView mPhoto;

    @Nullable
    @Override
    public View onCreateView(LayoutInflater inflater, @Nullable ViewGroup container, @Nullable Bundle savedInstanceState) {
        View bodyView = inflater.inflate(R.layout.fragment_userinfo, container, false);
        mPhoto = ButterKnife.findById(bodyView, R.id.iv_photo);
        return bodyView;
    }

    @Override
    public void onDestroyView() {
        super.onDestroyView();
    }
}
```

#### 3.7 代码混淆

```groovy
#ButterKnife
-keep class butterknife.** { *; }
-dontwarn butterknife.internal.**
-keep class **$$ViewBinder { *; }

-keepclasseswithmembernames class * {
    @butterknife.* <fields>;
}

-keepclasseswithmembernames class * {
    @butterknife.* <methods>;
}
```

### 4. ButterKnife源码解析

现在使用注解的第三方Android库越来越多，按照注解`@Retention`的处理时期，一版可以分为三种：

* `@RetentionPolicy.SOURCE`：源码注解，例如`@Override`，`@SupporWarnings`。这类注解在编译成功后就不会再起作用，并且不会出现在.class文件中
* `@RetentionPolicy.CLASS`：编译时注解，默认的注解方式，会最终保留在.class文件中，但无法再运行时获取。
* `@RetentionPolicy.RUNTIME`：运行时注解，会保留在.class文件中，这类注解可以用反射API中的`getAnnotations()`获取到。

这里面，运行时注解由于要用到反射会来带性能问题而被诟病，编译注解则可以依赖APT(Annotation Processing Tools)在编译时编译器会检查`AbstractProcessor`的子类，并且调用该类的`process`函数，然后将添加了注解的所有元素都传递到`process`函数中，使得开发人员可以处理这些注解甚至生成新的代码文件，同时让编译器将生成的代码文件和原代码文件一起编译。编写注解处理器的核心是`AnnotationProcessorFactory`和`AnnotationProcessor`两个接口，后者表示注解处理器，前者则是为某些注解类型创建注解处理器的工厂。

#### 4.1 ButterKnifeProcessor

已经知道编译时注解的核心是继承`AbstractProcessor`类并覆写`process`函数来处理所有的注解。在ButterKnife中继承`AbstractProcessor`的类是`ButterKnifeProcessor`,那么我们直接来看这个类实现。

```java
@AutoService(Processor.class)
public final class ButterKnifeProcessor extends AbstractProcessor {
//--
@Override public boolean process(Set<? extends TypeElement> elements, RoundEnvironment env) {
    Map<TypeElement, BindingClass> targetClassMap = findAndParseTargets(env);

    for (Map.Entry<TypeElement, BindingClass> entry : targetClassMap.entrySet()) {
      TypeElement typeElement = entry.getKey();
      BindingClass bindingClass = entry.getValue();

      try {
        bindingClass.brewJava().writeTo(filer);
      } catch (IOException e) {
        error(typeElement, "Unable to write view binder for type %s: %s", typeElement,
            e.getMessage());
      }
    }

    return true;
  }
//--  
}
```
首先会获取一个类型元素和注解类信息的`map`，用于后面写入不同的类文件。这里需要说明对于编译器来说，代码中的元素结构是基本不变的，例如，组成代码的基本元素有包，类，函数，字段，类型参数，变量。JDK中为这些元素定义了一个基类，也就是Element类，它有如下几个子类：

* [`PackageElement`](http://grepcode.com/file/repository.grepcode.com/java/root/jdk/openjdk/8u40-b25/javax/lang/model/element/PackageElement.java#PackageElement)：包元素，包含了某个包下的信息
* [`TypeElement`](http://grepcode.com/file/repository.grepcode.com/java/root/jdk/openjdk/8u40-b25/javax/lang/model/element/TypeElement.java#TypeElement)：类型元素，包含了一个类或接口定义的字段和函数
* [`ExecutableElement`](http://grepcode.com/file/repository.grepcode.com/java/root/jdk/openjdk/8u40-b25/javax/lang/model/element/ExecutableElement.java#ExecutableElement)：可执行元素，代表了函数类型的元素
* [`VariableElement`](http://grepcode.com/file/repository.grepcode.com/java/root/jdk/openjdk/8u40-b25/javax/lang/model/element/VariableElement.java#VariableElement)：变量元素
* [`TypeParameterElement`](http://grepcode.com/file/repository.grepcode.com/java/root/jdk/openjdk/8u40-b25/javax/lang/model/element/TypeParameterElement.java#TypeParameterElement)：类型参数元素

与`TypeElement`成对的`BindingClass`类则是封装了使用ButterKnife注解的类的信息，包括字段和方法。后面的`bindingClass.brewJava().writeTo(filer);`就是使用其中包含的信息生成新的类文件。

**findAndParseTargets**函数即使解析所有的注解的方法。

```java
private Map<TypeElement, BindingClass> findAndParseTargets(RoundEnvironment env) {
    Map<TypeElement, BindingClass> targetClassMap = new LinkedHashMap<>();
    Set<TypeElement> erasedTargetNames = new LinkedHashSet<>();

    // Process each @BindArray element.
    // Process each @BindBitmap element.
    // Process each @BindBool element.
    // Process each @BindColor element.
    // Process each @BindDimen element.
    // Process each @BindDrawable element.
    // Process each @BindInt element.
    // Process each @BindString element.

    // Process each @BindView element.
    // 获取所有使用@BindView注解的元素
    for (Element element : env.getElementsAnnotatedWith(BindView.class)) {
      // 判断元素的信息都是正确的    
      if (!SuperficialValidation.validateElement(element)) continue;
      try {
        parseBindView(element, targetClassMap, erasedTargetNames);
      } catch (Exception e) {
        logParsingError(element, BindView.class, e);
      }
    }

    // Process each @BindViews element.
    // Process each annotation that corresponds to a listener.

    // Try to find a parent binder for each.
    for (Map.Entry<TypeElement, BindingClass> entry : targetClassMap.entrySet()) {
      TypeElement parentType = findParentType(entry.getKey(), erasedTargetNames);
      if (parentType != null) {
        BindingClass bindingClass = entry.getValue();
        BindingClass parentBindingClass = targetClassMap.get(parentType);
        bindingClass.setParent(parentBindingClass);
      }
    }

    return targetClassMap;
}
```

我们在这里只先看对`@BindView`注解的解析，解析处理的关键在函数`parseBindView`中。

```java
private void parseBindView(Element element, Map<TypeElement, BindingClass> targetClassMap,
      Set<TypeElement> erasedTargetNames) {
    TypeElement enclosingElement = (TypeElement) element.getEnclosingElement();

    // Start by verifying common generated code restrictions.
    // 验证元素的修饰符，不可以是`private`或`static`
    // 验证元素不可以是非类对象
    // 验证元素内部类不可以私有
    // 验证元素不可是`android`或`java`包下的类，即ButterKnife不可以在Android Framework和JDK中使用
    boolean hasError = isInaccessibleViaGeneratedCode(BindView.class, "fields", element)
        || isBindingInWrongPackage(BindView.class, element);

    // Verify that the target type extends from View.
    // 判断元素是否继承自View
    TypeMirror elementType = element.asType();
    if (elementType.getKind() == TypeKind.TYPEVAR) {
      TypeVariable typeVariable = (TypeVariable) elementType;
      elementType = typeVariable.getUpperBound();
    }
    if (!isSubtypeOfType(elementType, VIEW_TYPE) && !isInterface(elementType)) {
      error(element, "@%s fields must extend from View or be an interface. (%s.%s)",
          BindView.class.getSimpleName(), enclosingElement.getQualifiedName(),
          element.getSimpleName());
      hasError = true;
    }

    if (hasError) {
      return;
    }

    // Assemble information on the field.
    // 获得注解标注的值，在这里就是View ID
    int id = element.getAnnotation(BindView.class).value();


    BindingClass bindingClass = targetClassMap.get(enclosingElement);
    if (bindingClass != null) {
      ViewBindings viewBindings = bindingClass.getViewBinding(id);
      // 判断被注解的元素是否已经被绑定过，并放入到BindingClass中
      // 如果被注解元素已经处理过，则会报错
      if (viewBindings != null && viewBindings.getFieldBinding() != null) {
        FieldViewBinding existingBinding = viewBindings.getFieldBinding();
        error(element, "Attempt to use @%s for an already bound ID %d on '%s'. (%s.%s)",
            BindView.class.getSimpleName(), id, existingBinding.getName(),
            enclosingElement.getQualifiedName(), element.getSimpleName());
        return;
      }
    } else {
      // 根据元素信息创建一个BindingClass实例
      bindingClass = getOrCreateTargetClass(targetClassMap, enclosingElement);
    }

    String name = element.getSimpleName().toString();
    TypeName type = TypeName.get(elementType);
    boolean required = isFieldRequired(element);

    // 创建一个被注解View的FieldViewBinding实例，并放入其所在类的BindingClass对象中
    FieldViewBinding binding = new FieldViewBinding(name, type, required);
    bindingClass.addField(id, binding);

    // Add the type-erased version to the valid binding targets set.
    erasedTargetNames.add(enclosingElement);
  }
```
这里面关键是创建`BindingClass`类和`FieldViewBinding`类，并将`FieldViewBinding`类的实例放入`BindingClass`类对象中去。`FieldViewBinding`类用于绑定变量View(与之对应的还有Drawable、Bitmap等)，它记录了绑定元素的变量名、类名。在`BindingClass`中会有一个对应的`View ID`与`FieldViewBinding`绑定，因此如果发现`BindingClass`类对象中已经存在View ID对应的`FieldViewBinding`就会报错。

在`parseBindView`中还有一个重要的方法是`getOrCreateTargetClass`，其方法即解析元素获取到类名，包名，然后构建一个`BindingClass`对象。其中的`BINDING_CLASS_SUFFIX`是一个`$$ViewBinder`的字符串，ButterKnife生成的辅助类都是以此为后缀。

```java
private BindingClass getOrCreateTargetClass(Map<TypeElement, BindingClass> targetClassMap,
      TypeElement enclosingElement) {
    BindingClass bindingClass = targetClassMap.get(enclosingElement);
    if (bindingClass == null) {
      TypeName targetType = TypeName.get(enclosingElement.asType());
      if (targetType instanceof ParameterizedTypeName) {
        targetType = ((ParameterizedTypeName) targetType).rawType;
      }

      String packageName = getPackageName(enclosingElement);
      ClassName classFqcn = ClassName.get(packageName,
          getClassName(enclosingElement, packageName) + BINDING_CLASS_SUFFIX);

      boolean isFinal = enclosingElement.getModifiers().contains(Modifier.FINAL);

      bindingClass = new BindingClass(targetType, classFqcn, isFinal);
      targetClassMap.put(enclosingElement, bindingClass);
    }
    return bindingClass;
  }
```

现在可以回到`process`方法，解析所有的注解后，生成一个`TypeElement`和`BindingClass`的键值对map,其中`TypeElement`是对使用注解的类的类类型元素的信息，`BindingClass`则是应用ButterKnife注解的类的封装。接下来就是调用`bindingClass.brewJava.write(filer)`生成相应的辅助类。在这里生成辅助类的使用的是`javapoet`，square公司开发的生成.java文件的工具。那么进入`brewJava`方法中。

```java
JavaFile brewJava() {
    TypeSpec.Builder result = TypeSpec.classBuilder(className)
        .addModifiers(PUBLIC)
        .addTypeVariable(TypeVariableName.get("T", ClassName.bestGuess(targetClass)));
    if (isFinal) {
      result.addModifiers(Modifier.FINAL);
    }

    if (hasParentBinding()) {
      result.superclass(ParameterizedTypeName.get(ClassName.bestGuess(parentBinding.classFqcn),
          TypeVariableName.get("T")));
    } else {
      result.addSuperinterface(ParameterizedTypeName.get(VIEW_BINDER, TypeVariableName.get("T")));
    }

    result.addMethod(createBindMethod());

    if (hasUnbinder() && hasViewBindings()) {
      // Create unbinding class.
      result.addType(createUnbinderClass());

      if (!isFinal) {
        // Now we need to provide child classes to access and override unbinder implementations.
        createUnbinderCreateUnbinderMethod(result);
      }
    }

    return JavaFile.builder(classPackage, result.build())
        .addFileComment("Generated code from Butter Knife. Do not modify!")
        .build();
  }
```
在这里函数中首先会生成一个`public class {CLASS}$$ViewBinder<T extend {CLASS}> implements ViewBinder<T>`的类结构，其中`{CLASS}`代表了ButterKnife注解所在的类名。同时它还会在其中添加一个实现方法`bind(finder, target, source)`，方法`bind`在函数`createBindMethod`中创建。

```java
private MethodSpec createBindMethod() {
    // 设置函数头
    MethodSpec.Builder result = MethodSpec.methodBuilder("bind")
        .addAnnotation(Override.class)
        .addModifiers(PUBLIC)
        .returns(UNBINDER)
        .addParameter(FINDER, "finder", FINAL)
        .addParameter(TypeVariableName.get("T"), "target", FINAL)
        .addParameter(Object.class, "source");

    // 绑定资源
    // 调用父类的bind

    // 设置初始化view
    if (!viewIdMap.isEmpty() || !collectionBindings.isEmpty()) {
      // Local variable in which all views will be temporarily stored.
      result.addStatement("$T view", VIEW);

      // Loop over each view bindings and emit it.
      for (ViewBindings bindings : viewIdMap.values()) {
        addViewBindings(result, bindings);
      }

      // Loop over each collection binding and emit it.
      for (Map.Entry<FieldCollectionViewBinding, int[]> entry : collectionBindings.entrySet()) {
        emitCollectionBinding(result, entry.getKey(), entry.getValue());
      }
    }

    // 绑定资源
    //---

    return result.build();
  }
```

进入`addViewBindings`函数.首先如果该`ViewBinding`需要绑定就用`findRequiredView`去初始化view，这个方法后面就看到。

```java
private void addViewBindings(MethodSpec.Builder result, ViewBindings bindings) {
    List<ViewBinding> requiredViewBindings = bindings.getRequiredBindings();
    if (requiredViewBindings.isEmpty()) {
      result.addStatement("view = finder.findOptionalView(source, $L, null)", bindings.getId());
    } else {
      if (bindings.getId() == NO_ID) {
        result.addStatement("view = target");
      } else {
        result.addStatement("view = finder.findRequiredView(source, $L, $S)", bindings.getId(),
            asHumanDescription(requiredViewBindings));
      }
    }

    addFieldBindings(result, bindings);
    addMethodBindings(result, bindings);
 }
```

在`brewJava`方法中完成一个辅助类的构建后，就可以调用`writeTo`方法生成一个.java文件。至此一个解析`@BindView`来生成辅助类的流程完成，现在来看看生成的辅助类。

```java
public class MainActivity$$ViewBinder<T extends MainActivity> implements ViewBinder<T> {
  @Override
  @SuppressWarnings("ResourceType")
  public Unbinder bind(final Finder finder, final T target, Object source) {
    InnerUnbinder unbinder = createUnbinder(target);
    View view;
    // 查找初始化view
    view = finder.findRequiredView(source, 2131492945, "method 'onTextChange'");
    target.mEtName = finder.castView(view, 2131492945, "field 'mEtName'");
    unbinder.view2131492945 = view;
    ((TextView) view).addTextChangedListener(new TextWatcher() {
      @Override
      public void onTextChanged(CharSequence p0, int p1, int p2, int p3) {
        target.onTextChange(p0, p1, p2, p3);
      }

      @Override
      public void beforeTextChanged(CharSequence p0, int p1, int p2, int p3) {
      }

      @Override
      public void afterTextChanged(Editable p0) {
      }
    });

    // 初始化list<view>
    target.mViews = Utils.listOf(
        finder.<View>findRequiredView(source, 2131492947, "field 'mViews'"),
        finder.<View>findRequiredView(source, 2131492948, "field 'mViews'"),
        finder.<View>findRequiredView(source, 2131492949, "field 'mViews'"));

    // 初始化资源
    Context context = finder.getContext(source);
    Resources res = context.getResources();
    Resources.Theme theme = context.getTheme();
    target.mColorPrimary = Utils.getColor(res, theme, 2131427347);
    target.layoutItemHeight = res.getDimensionPixelSize(2131230794);
    target.strName = res.getString(2131099669);
    target.strPasswd = res.getString(2131099670);
    return unbinder;
  }

  // 设置Unbinder
}
```
在生成的辅助类中，通过如下来初始view。

```java
view = finder.findRequiredView(source, 2131492945, "method 'onTextChange'");
target.mEtName = finder.castView(view, 2131492945, "field 'mEtName'");
```

这里看到`bind`方法是不是很熟悉，和我们在`Activity`的`onCreate`函数中的`ButterKnife.bind()`长得一样，那么我们进入ButterKnife的`bind`方法。

```java
public static Unbinder bind(@NonNull Activity target) {
    return bind(target, target, Finder.ACTIVITY);
 }

 static Unbinder bind(@NonNull Object target, @NonNull Object source, @NonNull Finder finder) {
    Class<?> targetClass = target.getClass();
    try {
      if (debug) Log.d(TAG, "Looking up view binder for " + targetClass.getName());
      ViewBinder<Object> viewBinder = findViewBinderForClass(targetClass);
      return viewBinder.bind(finder, target, source);
    } catch (Exception e) {
      throw new RuntimeException("Unable to bind views for " + targetClass.getName(), e);
    }
 }
```

这里通过`findViewBinderForClass`找到`viewBinder`，那么我们再进入`findViewBinderForClass`中。

```java
private static ViewBinder<Object> findViewBinderForClass(Class<?> cls)
      throws IllegalAccessException, InstantiationException {
    // 从缓存中获取viewBinder
    ViewBinder<Object> viewBinder = BINDERS.get(cls);
    if (viewBinder != null) {
      if (debug) Log.d(TAG, "HIT: Cached in view binder map.");
      return viewBinder;
    }

    // 如果类在android开头的包下，则报错
    String clsName = cls.getName();
    if (clsName.startsWith("android.") || clsName.startsWith("java.")) {
      if (debug) Log.d(TAG, "MISS: Reached framework class. Abandoning search.");
      return NOP_VIEW_BINDER;
    }

    // 构建一个类
    try {
      Class<?> viewBindingClass = Class.forName(clsName + "$$ViewBinder");
      //noinspection unchecked
      viewBinder = (ViewBinder<Object>) viewBindingClass.newInstance();
      if (debug) Log.d(TAG, "HIT: Loaded view binder class.");
    } catch (ClassNotFoundException e) {
      if (debug) Log.d(TAG, "Not found. Trying superclass " + cls.getSuperclass().getName());
      viewBinder = findViewBinderForClass(cls.getSuperclass());
    }
    BINDERS.put(cls, viewBinder);
    return viewBinder;
  }
```
到这里我们又见到似曾相识的东西`$$ViewBinder`这个就是辅助类的后缀，那么就是在这里构建了一个`MainActivity$$ViewBinder`实例。

现在回到`bind(target, source, finder)`方法中，当获得`viewBinder`实例后，调用其`bind`方法，也就是`MainActivity$$ViewBinder`的`bind`方法,那么此时我们就可以知道传入的`finder`为`Finder.ACTIVITY`。`Finder`是一个枚举类，我们看其中的`ACTIVITY`枚举：

```java
ACTIVITY {
    @Override protected View findView(Object source, int id) {
      return ((Activity) source).findViewById(id);
    }

    @Override public Context getContext(Object source) {
      return (Activity) source;
    }
  },
```

我们再看在`MainActivity$$ViewBinder`中的初始化view的`findRequiredView`方法：

```java
public <T> T findRequiredView(Object source, int id, String who) {
    T view = findOptionalView(source, id, who);
    if (view == null) {
      String name = getResourceEntryName(source, id);
      throw new IllegalStateException("Required view '"
          + name
          + "' with ID "
          + id
          + " for "
          + who
          + " was not found. If this view is optional add '@Nullable' (fields) or '@Optional'"
          + " (methods) annotation.");
    }
    return view;
  }

  public <T> T findOptionalView(Object source, int id, String who) {
    View view = findView(source, id);
    return castView(view, id, who);
  }
```

到现在我们看到了最终通过`findView`抽象方法调到枚举`ACTIVITY`中的`findView`方法，到这里我们看到更加熟悉的`findViewById`Android提供的初始化一个view的方法。到此为止我们知道了通过`ButterKnife.bind(Activity)`和`@BindView`注解初始化的过程。其他的注解处理的方式也是一样的。

# 参考

* [Android源码设计模式解析与实战](http://product.dangdang.com/23802445.html)
* [绝对不容错过，ButterKnife使用详谈](http://www.jianshu.com/p/b6fe647e368b)
* [Butter Knife 源码解析](http://mp.weixin.qq.com/s?__biz=MzA4MjU5NTY0NA==&mid=404147665&idx=1&sn=a16153b2a658db64ab80926cd3b76447&scene=2&srcid=0316ZcLDaO7NOqcReomltlmA&from=timeline&isappinstalled=0#wechat_redirect)
* [公共技术点之 Java 注解 Annotation](http://a.codekk.com/detail/Android/Trinea/%E5%85%AC%E5%85%B1%E6%8A%80%E6%9C%AF%E7%82%B9%E4%B9%8B%20Java%20%E6%B3%A8%E8%A7%A3%20Annotation)
* [Butter Knife](http://jakewharton.github.io/butterknife/)
* [ButterKnife使用详解](http://blog.csdn.net/itjianghuxiaoxiong/article/details/50177549)
* [Android 浅析 ButterKnife (二) 源码解析](http://www.jianshu.com/p/a5ea2ea75bb7)
