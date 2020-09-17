# java.util.Array
• static Object get(Object array,int index)
• static xxx get/xx(Object array,int index)
( xxx 是 boolean、byte、 char、 double、 float、 int、 long、 short 之中的一种基本类型。）
这些方法将返回存储在给定位置上的给定数组的内容。
• static void set(Object array,int index,Object newValue)
•static setXxx(Object array,int index,xxx newValue)
( xxx 是 boolean、 byte、char、double、float、 int、 long、 short 之中的一种基本类型。）
这些方法将一个新值存储到给定位置上的给定数组中。
•static int getLength(Object array)
返回数组的长度。
• static Object newInstance(Class componentType,int length)
•static Object newInstance(Class componentType,int[]lengths)
返回一个具有给定类型、 给定维数的新数组。
# java.util.Objects
• static boolean equals(Object a, Object b)
如果 a 和 b 都为 null， 返回 true ; 如果只有其中之一为 null， 则返回 false ; 否 则 返 回a.equals(b)
•static int hash(Object... objects)
返回一个散列码，由提供的所有对象的散列码组合而得到。
•static int hashCode(Object a )
如果 a 为 null 返回 0， 否则返回 a.hashCode() 
# java.util.Object
•int hashCode( )
返回对象的散列码。散列码可以是任意的整数， 包括正数或负数。两个相等的对象要求返回相等的散列码。
•Class getClass( )
返回包含对象信息的类对象。
•boolean equals(Object otherObject )
比较两个对象是否相等， 如果两个对象指向同一块存储区域， 方法返回 true ; 否则方法返回 false。在自定义的类中， 应该覆盖这个方法。
•String toString( )
返冋描述该对象值的字符串。在自定义的类中， 应该覆盖这个方法。
# java.lang.Class
•String getName( )
返回这个类的名字。
•Class getSuperclass( )
以 Class 对象的形式返回这个类的超类信息。
• Field[] getFields()
• Filed[] getDeclaredFields()
getFields 方法将返回一个包含 Field 对象的数组， 这些对象记录了这个类或其超类的公有域。getDeclaredField 方法也将返回包含 Field 对象的数组， 这些对象记录了这个类的全部域。如果类中没有域， 或者 Class 对象描述的是基本类型或数组类型， 这些方法将返回一个长度为 0 的数组。
• Method[] getMethods()
• Method[] getDeclareMethods()
返回包含 Method 对象的数组：getMethods 将返回所有的公有方法， 包括从超类继承来的公有方法；getDeclaredMethods 返回这个类或接口的全部方法， 但不包括由超类继承了的方法。
• Constructor[] getConstructors()
• Constructor[] getDeclaredConstructors()
返回包含 Constructor 对象的数组， 其中包含了 Class 对象所描述的类的所有公有构造器（getConstructors) 或所有构造（getDeclaredConstructors）。

• static Class forName(String className)
返回描述类名为 className 的 Class 对象。
•Object newlnstance()
返回这个类的一个新实例。
•Field getField(String name )
•Field[] getField()
返回指定名称的公有域， 或包含所有域的数组
•Field getDeclaredField(String name )
•Field[] getDeclaredFields( )
返回类中声明的给定名称的域， 或者包含声明的全部域的数组。
•Class< ? > getComponentType()
确定数组对应的类型，如果不是数组返回null。

# java.lang.Class< T >
• T newInstance()
返回无参数构造器构造器构造的一个新实例，返回类型与Class< T >描述的类相同，这样就免除了类型转换
• T cast(Object obj)
如果obj为null或有可能转换成类型T，则返回obj；否则抛出BadCastException异常
•T[] getEnumConstants()
如果T是枚举类型，则返回所有值组成的数组，否则返回null。
•Class< ? super T > getSuperclass()
返回每个类的超类。如果T不是一个类或者Object类，则返回null。
•Constructor< T > getConstructor(Class... parameterTypes)
•Constructor< T > getDeclaredConstructor(Class... parameterTypes)
获得公有的构造器，或带有给定参数类型的构造器。
•TypeVariable[] getTypeParameters()
如果这个类型被声明为泛型类型，则获得泛型类型变量，否则获得一个长度为 0 的数组。
•Type getGenericSuperclass( ) 
获得被声明为这一类型的超类的泛型类型；如果这个类型是 Object 或不是一个类类型( class type), 则返回 null。
•Type[ ] getGenericInterfaces( ) 5.0
获得被声明为这个类型的接口的泛型类型（以声明的次序，) 否则， 如果这个类型没有实现接口，返回长度为 0 的数组。

# java.lang.reflect.Field
# java.lang.reflect.Method
# java.Iang.reflect.Constructor
• Object newlnstance(Object[] args) (Constructor)
构造一个这个构造器所属类的新实例。
参数：args 这是提供给构造器的参数
•Class getDeclaringClass( )
返回一个用于描述类中定义的构造器、 方法或域的 Class 对象。
•Class[] getExceptionTypes ( ) ( 在 Constructor 和 Method 类 中）
返回一个用于描述方法抛出的异常类型的 Class 对象数组。
•int getModifiers( )
返回一个用于描述构造器、 方法或域的修饰符的整型数值。使用 Modifier 类中的这个方法可以分析这个返回值。
•String getName( )
返冋一个用于描述构造器、 方法或域名的字符串。
•Class[] getParameterTypes ( ) ( 在 Constructor 和 Method 类 中）
返回一个用于描述参数类型的 Class 对象数组。
•Class getReturnType( ) ( 在 Method 类 中）
返回一个用于描述返H类型的 Class 对象。
•Object get(Object obj) ( 在 Field类 中）
返回 obj 对象中用 Field 对象表示的域值。
•void set(Object obj ,Object newValue )( 在 Field类 中）
用一个新值设置 Obj 对象中 Field 对象表示的域。

•public Object invoke(Object implicitParameter,Object[] explicitParamenters)( 在 Method 类 中）
调用这个对象所描述的方法， 传递给定参数，并返回方法的返回值。对于静态方法，把 null 作为隐式参数传递。在使用包装器传递基本类型的值时， 基本类型的返回值必须是未包装的。

•TypeVariableC ] getTypeParameters( ) ( 在 Method 类 中）
如果这个方法被声明为泛型方法， 则获得泛型类型变量，否则返回长度为 0 的数组。
•Type getGenericReturnType( ) ( 在 Method 类 中）
获得这个方法被声明的泛型返回类型。
•Type[ ] getGenericParameterTypes( )( 在 Method 类 中）
获得这个方法被声明的泛型参数类型。 如果这个方法没有参数，返回长度为 0 的数组。

# java.lang.reflect.Constructor< T >
• T newInstance(Object... parameters)
返回用指定参数构造的新实例。

# java.lang.reflect.Modifier
•static String toString(int modifiers )
返回对应 modifiers 中位设置的修饰符的字符串表7K。
•static boolean isAbstract(int modifiers )
•static boolean isFinal (int modifiers )
•static boolean islnterface(int modifiers )
•static boolean isNative(int modifiers )
•static boolean isPrivate(int modifiers )
•static boolean isProtected(int modifiers )
•static boolean isPublic(int modifiers )
•static boolean isStatic(int modifiers )
•static boolean isStrict(int modifiers )
•static boolean isSynchronized(int modifiers )
•static boolean isVolati1e(int modifiers )
这些方法将检测方法名中对应的修饰符在 modffiers 值中的位。


# java.Iang.reflect.AccessibleObject
• void setAccessible(boolean flag)
为反射对象设置可访问标志。flag 为 true 表明屏蔽 Java 语言的访问检查，使得对象的私有属性也可以被査询和设置。
•boolean isAccessible( )
返回反射对象的可访问标志的值。
• static void setAccessible(AccessibleObject[] array,boolean flag)
是一种设置对象数组可访问标志的快捷方法。

# java.Iang.reflect.InvocationHandler
• Object invoke(Object proxy, Method method, 0bject[] args)
定义了代理对象调用方法时希望执行的动作。

# java.Iang.reflect.Proxy
•static Class< ? > getProxyClass(ClassLoader loader, Class< ? >... interfaces)
返回实现指定接口的代理类。
•static Object newProxyInstance(ClassLoader loader, Class< ? >[] interfaces, InvocationHandler handler)
构造实现指定接口的代理类的一个新实例。
所有方法会调用给定处理器对象的 invoke 方法。
•static boolean isProxyClass(Class< ? > cl)
如果 cl 是一个代理类则返回 true。

# 基础数据类型
•static int hashCode((int11ong|short|byte|double|f1oat|char|boolean) value)
返回给定值的散列码。

# java.Iang.ClassLoader
• void setDefaultAssertionStatus( boolean b ) 
对于通过类加载器加载的所有类来说， 如果没有显式地说明类或包的断言状态， 就启用或禁用断言。
• void setCIassAssertionStatus(String className , boolean b ) 
对于给定的类和它的内部类，启用或禁用断言。
• void setPackageAssertionStatus( String packageName , boolean b ) 
对于给定包和其子包中的所有类，启用或禁用断言。
• void clearAssertionStatus( ) 
移去所有类和包的显式断言状态设置， 并禁用所有通过这个类加载器加载的类的断言。

# java.lang.reflect.TypeVariable
String getName()
获得类型变量的名字
Type[] getBounds()
获得类型变量的子类限定，否则，如果该变量无限定，则返回长度为0的数组

# java.lang.reflect.WildcardType
Type[] getUpperBounds()
获得这个类型变量的子类限定，否则，如果没有子类限定，则会返回长度为0的数组
Type[] getLowerBounds()
获得这个类型变量的超类限定，否则，如果没有超类限定，则会返回长度为0的数组

# java.lang.reflect.ParameterizedType
Type getRawType()
获得这个参数化类型的原始类型
Type[] geyActualTypeArguments()
获得这个参数化类型声明时所使用的类型参数
Type getOwnerType()
如果是内部类型，则返回其外部类型，如果是一个顶级类型，则返回null。

# java.lang.reflect.GenericArrayType
Type getGenericComponentType()
获得声明该数组类型的泛型组件类型

