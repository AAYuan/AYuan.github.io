我看的源码版本是[objc4-680源码](http://opensource.apple.com/tarballs/objc4/objc4-680.tar.gz) 

##Class

Objective-C类是由Class类型来表示的，Class是一个指向objc_class结构体的指针。

```
typedef struct objc_class *Class;
```

`objc/runtime.h` 中`objc_class`结构体如下

```c
struct objc_class {
    Class isa  OBJC_ISA_AVAILABILITY;

#if !__OBJC2__
    Class super_class                                        OBJC2_UNAVAILABLE;
    const char *name                                         OBJC2_UNAVAILABLE;
    long version                                             OBJC2_UNAVAILABLE;
    long info                                                OBJC2_UNAVAILABLE;
    long instance_size                                       OBJC2_UNAVAILABLE;
    struct objc_ivar_list *ivars                             OBJC2_UNAVAILABLE;
    struct objc_method_list **methodLists                    OBJC2_UNAVAILABLE;
    struct objc_cache *cache                                 OBJC2_UNAVAILABLE;
    struct objc_protocol_list *protocols                     OBJC2_UNAVAILABLE;
#endif

} OBJC2_UNAVAILABLE;
/* Use `Class` instead of `struct objc_class *` */

```

- isa:在Objective-C中每个类都是一个对象，这个对象的Class里面有个isa指针，指向metaClass(元类)。一个接收者接受到一个消息时，isa指针会去查找能够响应这个消息的对象。
- super_class：指向该类的父类，如果该类已经是最顶层的根类(NSObject/NSProxy),则super_class为NULL
- version：类的版本信息，默认为0，对对象的序列化非常有用。
- methodList：方法定义的链表
- cache:用于缓存最近调用的方法。一般情况下，一个对象只有一部分方法经常使用，每次都去methodLists中遍历查找效率很低，所以设置了cache。每次调用一个方法后，该方法都会被缓存到cache列表中，下次直接在缓存表中查找、如果cache中没有，再去methodLists种查找。

常见的一个id，是一个objc_object结构类型的指针，塔可以让我们实现类似于C++的泛型的一些操作。该类型的对象可以转换为任意类型的对象。

```c
typedef struct objc_object *id;
```
##metaClass

OC种所有的类也是对象，我们可以向这个对象发送消息（调用类方法）。比如：

```c
[NSArray array];
```

既然是对象，就是一个objc_object指针，它包含一个指向其类isa的一个指针。

```c
struct objc_object {
private:
    isa_t isa;
```

为了调用类方法array，这个类的isa指针指向一个包含这些类方法的objc_class结构体。
`而类对象的类则是元类(meta-class).`

每个类都有一个元类(meta-class),存储着一个对应类的所有类方法。
当我们向一个对象发送消息时，runtime会在这个对象所属的这个类的方法列表中查找方法；而向一个类发送消息时，会在这个类的meta-class的方法列表中查找。

meta-class也是一个类，所以也可以向它发送消息，为了不让这种结构无线延伸下去，Objective-C的设计者让meta-class的isa指向父类的meta-class，作为他们的附属类。所有在NSObject继承体系下的meta-class，都在NSObject的meta-class附属下，而meta-class的isa指向他自己，形成了一个闭环。
结构如下图：

![1475472-70ff45d63765663](media/14682778572700/1475472-70ff45d63765663c.png)





我看的源码版本是[objc4-680源码](http://opensource.apple.com/tarballs/objc4/objc4-680.tar.gz) 

##Class

Objective-C类是由Class类型来表示的，Class是一个指向objc_class结构体的指针。

```c
typedef struct objc_class *Class;
```

`objc/runtime.h` 中`objc_class`结构体如下

```c
struct objc_class {
    Class isa  OBJC_ISA_AVAILABILITY;

#if !__OBJC2__
    Class super_class                                        OBJC2_UNAVAILABLE;
    const char *name                                         OBJC2_UNAVAILABLE;
    long version                                             OBJC2_UNAVAILABLE;
    long info                                                OBJC2_UNAVAILABLE;
    long instance_size                                       OBJC2_UNAVAILABLE;
    struct objc_ivar_list *ivars                             OBJC2_UNAVAILABLE;
    struct objc_method_list **methodLists                    OBJC2_UNAVAILABLE;
    struct objc_cache *cache                                 OBJC2_UNAVAILABLE;
    struct objc_protocol_list *protocols                     OBJC2_UNAVAILABLE;
#endif

} OBJC2_UNAVAILABLE;
/* Use `Class` instead of `struct objc_class *` */

```

- isa:在Objective-C中每个类都是一个对象，这个对象的Class里面有个isa指针，指向metaClass(元类)。一个接收者接受到一个消息时，isa指针会去查找能够响应这个消息的对象。
- super_class：指向该类的父类，如果该类已经是最顶层的根类(NSObject/NSProxy),则super_class为NULL
- version：类的版本信息，默认为0，对对象的序列化非常有用。
- methodList：方法定义的链表
- cache:用于缓存最近调用的方法。一般情况下，一个对象只有一部分方法经常使用，每次都去methodLists中遍历查找效率很低，所以设置了cache。每次调用一个方法后，该方法都会被缓存到cache列表中，下次直接在缓存表中查找、如果cache中没有，再去methodLists种查找。

常见的一个id，是一个objc_object结构类型的指针，塔可以让我们实现类似于C++的泛型的一些操作。该类型的对象可以转换为任意类型的对象。

```c
typedef struct objc_object *id;
```
##metaClass

OC种所有的类也是对象，我们可以向这个对象发送消息（调用类方法）。比如：

```c
[NSArray array];
```

既然是对象，就是一个objc_object指针，它包含一个指向其类isa的一个指针。

```c
struct objc_object {
private:
    isa_t isa;
```

为了调用类方法array，这个类的isa指针指向一个包含这些类方法的objc_class结构体。
`而类对象的类则是元类(meta-class).`

每个类都有一个元类(meta-class),存储着一个对应类的所有类方法。
当我们向一个对象发送消息时，runtime会在这个对象所属的这个类的方法列表中查找方法；而向一个类发送消息时，会在这个类的meta-class的方法列表中查找。

meta-class也是一个类，所以也可以向它发送消息，为了不让这种结构无线延伸下去，Objective-C的设计者让meta-class的isa指向父类的meta-class，作为他们的附属类。所有在NSObject继承体系下的meta-class，都在NSObject的meta-class附属下，而meta-class的isa指向他自己，形成了一个闭环。
结构如下图：

![1475472-70ff45d63765663](http://upload-images.jianshu.io/upload_images/1475472-70ff45d63765663c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)






