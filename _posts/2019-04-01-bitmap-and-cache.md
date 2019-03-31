---
layout: post
title: "Android Bitmap 的加载和 Cache"
subtitle: "Bitmap and Cache in Android"
author: "beforenight"
header-img: "img/post-bg-android.jpg"
header-mask: 0.4
tags:
  - Android
  - Bitmap
  - Cache
---

> 首先抛出一个问题：在Android中如何有效地加载一个Bitmap？
> 由于Bitmap的特殊性以及Android对单个应用所施加的内存限制，比如16MB，这导致加载Bitmap的时候很容易出现内存溢出。下面这个异常信息在开发中应该时常遇到:

`
    java.lang.OutofMemoryError: bitmap size exceeds VM budget
`

### Bitmap 的高效加载

在介绍Bitmap的高效加载之前，先说一下**如何加载一个Bitmap**, Bitmap在Android 中指的是一张图片，可以是png格式也可以是jpg等其他常见的图片格式。那么如何加载一个图片呢? BitmapFactory 类提供了四类方法:decodeFile、decodeResource、 decodeStream 和decodeByteArray，分别用于支持从文件系统、资源、输入流以及字节数组中加载出一个 Bitmap对象，其中decodeFile和decodeResource又间接调用了 decodeStream方法，这四类 方法最终是在Android的底层实现的，对应着BitmapFactory类的几个native方法。

如何高效地加载Bitmap呢?其实核心思想也很简单，那就是采用BitmapFactory. Options来加载所需尺寸的图片。

这里假设通过ImageView来显示图片，很多时候ImageView 并没有图片的原始尺寸那么大，这个时候把整个图片加载进来后再设给ImageView,这显 然是没必要的，因为ImageView并没有办法显示原始的图片。通过BitmapFactory.Options 就可以按-定的采样率来加载缩小后的图片，将缩小后的图片在ImageView中显示，这样 就会降低内存占用从而在一定程度上避免OOM, 提 高 了 Bitmap加载时的性能。 BitmapFactory提供的加载图片的四类方法都支持BitmapFactory.Options参数，通过它们就 可以很方便地对一个图片进行采样缩放。

通过**BitmapFactory.Options**来缩放图片，主要是用到了它的inSampleSize参数，即**采样率**。

当inSampleSize为1时，采样后的图片大小为图片的原始大小;当inSampleSize大于1时，比如为2,那么采样后的图片其宽/高均为原图大小的1/2,而像素数为原图的1/4, 其占有的内存大小也为原图的1/4。拿一张1024M024像素的图片来说，假定采用 **ARGB8888** 格式存储，那么它占有的内存为1024x1024x4，即4MB，如果inSampleSize为2, 那么采样后的图片其内存占用只有512x512x4，即1MB。可以发现采样率inSampleSize必须是大 于1的整数图片才会有缩小的效果，并且采样率同时作用于宽/高，这将导致缩放后的图片 大小以采样率的2次方形式递减，即缩放比例为1/ (inSampleSize的2次方)，比如 inSampleSize为4，那么缩放比例就是1/16。有一种特殊情况，那就是当inSampleSize小于 1时，其作用相当于1，即无缩放效果。

通过采样率即可有效地加载图片，那么到底如何获取采样率呢?获取采样率也很简单, 遵循如下流程:
- 将BitmapFactory.Options 的 inJustDecodeBounds 参数设为 true 并加载图片。
- 从BitmapFactory.Options中取出图片的原始宽高信息，它们对应于outWidth和
outHeight 参数。
- 根据采样率的规则并结合目标View的所需大小计算出采样率inSampleSize。
- 将 BitmapFactory.Options 的 inJustDecodeBounds 参数设为 false.然后重新加载 图片。


经过上面4个步骤，加载出的图片就是最终缩放后的图片，当然也有可能不需要缩放。 这里说明一下inJustDecodeBounds参数，当此参数设为true时，BitmapFactory只会解析图 片的原始宽/高信息，并不会去真正地加载图片，所以这个操作是轻量级的。

```
public static Bitmap decodeSampledBitmapFromResource (Resources res, int reside int reqWidth, int reqHeight) {
// First decode with inJustDecodeBounds=true to check dimensions final BitmapFactory.Options options = new BitmapFactory.Options(); options•inJustDecodeBounds = true;
 BitmapFactory•decodeResource(res resld, options); #
// Calculate inSampleSize
options.inSan^leSize = calculateInSanpleSize(options, reqWidth, reqHeight);
// Decode bitmap with inSampleSize set options.inJustDecodeBounds - false;
return BitmapFactory.decodeResource(res, resld, options)/
)
public static int calculatelnSampleSize(
BitmapFactory.Options options, int reqWidth, int reqHeight) {
// Raw height and width of image final int hei
ize value that is a power of 2 and
keeps both
// height and width larger than the requested height and width. while ((halfHeight / inSampleSize) >= reqHeight
&& (halfWidth / inSampleSize) >= reqWidth) { inSampleSize ** 2;
return inSampleSize;
)
```

## Android中的缓存策略

> 缓存策略在Android中有着广泛的使用场景，尤其在图片加载这个场景下，缓存策略 就变得更为重要。考虑一种场景:有一批网络图片，需要下载后在用户界面上予以显示， 这个场景在PC环境下是很简单的，直接把所有的图片下载到本地再显示即可，但是放到 移动设备上就不一样了。不管是Android还是iOS设备，流量对于用户来说都是一种宝贵的资源，由于流量是收费的，所以在应用开发中并不能过多地消耗用户的流量，否则这个应用肯定不能被用户所接受。
如何避免过多的流量消耗呢? 缓存。
当程序第一次从网 络加载图片后，就将其缓存到存储设备上，这样下次使用这张图片就不用再从网络上获取了，这样就为用户节省了流量。很多时候为了提高应用的用户体验，往往还会把图片在内 存中再缓存一份，这样当应用打算从网络上请求这张图片时，程序会首先从内存中去获取， 如果内存中没有那就从存储设备中去获取，如果存储设备中也没有，那就从网络上下载这张图片。因为从内存中加载图片比从存储设备中加载图片要快，所以这样既提高了程序的 效率又为用户节约了不必要的流量开销。上述的缓存策略不仅仅适用于图片，也适用于其他文件类型。

说到缓存策略，其实并没有统的标准。一般来说，缓存策略主要包含缓存的添加、 获取和删除这三类操作。常用的一种缓存算法是LRU (Least Recently Used), LRU是近期最少使用算法， 它的核心思想是当缓存满时，会优先淘汰那些近期最少使用的缓存对象。采用LRU算法的缓存有两种:LruCache和DiskLruCache, LruCache用于实现内存缓存，而DiskLruCache 则充当了存储设备缓存，通过这二者的完美结合，就可以很方便地实现一个具有很高实用 价值的ImageLoader。

## LruCache
LruCache是Android 3.1所提供的一个缓存类，为了能够兼容Android 2.2版本，在使用LruCache时建议采用 support-v4兼容包中提供的LruCache,而不要直接使用Android 3.1提供的LruCache。

LruCache是一个泛型类，它内部采用一个LinkedHashMap以强引用的方式存储外界的缓存对象，其提供了 get和put方法来完成缓存的获取和添加操作，当缓存满时， LruCache 会移除较早使用的缓存对象，然后再添加新的缓存对象。这里读者要明白强引用、软引用 和弱引用的区别，如下所示。

- 1.强引用:直接的对象引用;
- 2.软引用:当一个对象只有软引用存在时，系统内存不足时此对象会被gc回收:
- 3.弱引用:当一个对象只有弱引用存在时，此对象会随时被gc回收。

另外LruCache是线程安全的，下面是LruCache的定义:
```
public class LruCache<K, V> {
private final LinkedHashMap<K ,V> map;
···
}
```

LruCache的典型的初始化过程:
```
    int maxMemory = (int) (Runtime•getRuntime(}•maxMemory<) / 1024); 
    int cacheSize = maxMemory / 8;
    mMemoryCache = new LruCache<String, Bitmap>(cacheSize) {
    @Override
    protected int sizeOf(String key. Bitmap bitmap) {
    return bitmap.getRowBytes() * bitmap.getHeight() / 1024;
    }
    };
```

在上面的代码中，只需要提供缓存的总容量大小并重写sizeOf方法即可。sizeOf方法 的作用是计算缓存对象的大小，这里大小的单位需要和总容景的单位一致。对于上面的示 例代码来说，总容量的大小为当前进程的可用内存的1/8,单位为KB,而sizeOf方法则完 成了 Bitmap对象的大小计算。很明显，之所以除以1024也是为了将其单位转换为KB。一 些特殊情况下，还需要重写LruCache的entry Removed方法，LruCache移除旧缓存时会调用entryRemoved方法，因此可以在entryRemoved中完成一些资源回收工作(如果需要的 话)。
除了 LruCache的创建以外，还有缓存的获取和添加，这也很简单，从LruCache中获 取一个缓存对象，如下所示，
```
mMemoryCache.get(key)
```
向LruCache中添加一个缓存对象，如下所示:
```
mMemoryCache.put(key, bitmap)
```
LruCache还支持删除操作，通过remove方法即可删除一个指定的缓存对象。可以看 到LruCache的实现以及使用都非常简单，虽然简单，但是仍然不影响它具有强大的功能， 从Android 3.1开始，LruCache就已经是Android源码的一部分了。

## DiskLruCache
DiskLruCache用于实现存储设备缓存，即磁盘缓存，它通过将缓存对象写入文件系统 从而实现缓存的效果。DiskLruCache得到了 Android官方文档的推荐，但它不属于Android SDK的一部分，它的源码可以从如下网址得到:

[DiskLruCache 源码 (需要科学上网)](https://android.googlesource.com/platform/libcore/+/android-4.1.l_rl/luni/src/main/java/libcore/io/DiskLruCache.java)

 需要注意的是，从上述网址获取的DiskLruCache的源码并不能直接在Android中使用，需要稍微修改编译错误。下面分别从DiskLruCache的创建、缓存査找和缓存添加这三个方 面来介绍DiskLruCache的使用方式。

### DiskLruCache的创建

 DiskLruCache并不能通过构造方法来创建.它提供了 open方法用于创建自身，如下所示。
```
public static DiskLruCache open(File directory, int appVersion, int valueCount, long maxSize)
```
open方法有四个参数，其中第一个参数表示磁盘缓存在文件系统中的存储路径。缓存 路径可以选择SD卡上的缓存目录，具体是指/sdcard/Android/data/package_name/cache目录， 其中package_name表示当前应用的包名，当应用被卸载后，此目录会一并被删除。当然也可以选择SD卡上的其他指定目录，还可以选择data下的当前应用的目录，具体可根据需 要灵活设定。这里给出一个建议:如果应用卸载后就希望删除缓存文件，那么就选择SD 卡上的缓存目录，如果希望保留缓存数据那就应该选择SD卡上的其他特定目录。
第二个参数表示应用的版本号，一般设为1即可。当版本号发生改变时DiskLruCache 会清空之前所有的缓存文件，而这个特性在实际开发中作用并不大，很多情况下即使应用 的版本号发生了改变缓存文件却仍然是有效的，因此这个参数设为1比较好。
第三个参数表示单个节点所对应的数据的个数,一般设为1即可。
第四个参数表示缓 存的总大小，比如50MB,当缓存大小超出这个设定值后，DiskLruCache会清除一些缓存 从而保证总大小不大于这个设定值。

### DiskLruCache的缓存添加

DiskLruCache的缓存添加的操作是通过Editor完成的，Editor表示一个缓存对象的编辑对象。
这里仍然以图片缓存举例，首先需要获取图片url所对应的key，然后根据key就可以通过edit()来获取Editor对象，如果这个缓存正在被编辑，那么edit()会返回mill,即 DiskLruCache不允许同时编辑一个缓存对象。之所以要把uri转换成key,是因为图片的url 中很可能有特殊字符，这将影响url在Android中直接使用，一般采用url的md5值作为key, 如下所示。
```
private String hashKeyFormUrl(String url) { String cacheKey;
try {
final MessageDigest mDigest = MessageDigest.getInstance(MMD5M); mDigest.update(url.getBytes());
cacheKey = bytesToHexString(mDigest.digest());
} catch (NoSuchAlgorithmException e) { cacheKey - String.valueOf(url.hashCode());
}
return cacheKey;
private String bytesToHexString(byte[】bytes) { StringBuilder sb = new StringBuilder(); for (int i = 0; i < bytes.length; i十+) {
String hex = Integer.toHexString(OxFF & bytes[i])/ if (hex.length() == 1) {
sb.append(10•); }
sb.append(hex);
}
return sb.toString();
}
```
将图片的url转成key以后，就可以获取Editor对象了。对于这个key来说，如果当前不存在其他Editor对象，那么edit()就会返回一个新的Editor对象，通过它就可以得到一个文件输出流。

### DiskLmCache的缓存査找

和缓存的添加过程类似，缓存査找过程也需要将url转换为key,然后通过DiskLruCache 的get方法得到一个Snapshot对象,接着再通过Snapshot对象即可得到缓存的文件输入流，有了文件输出流，自然就可以得到Bitmap对象了。为了避免加载图片过程中导致的 **OOM** 问题，一般不建议直接加载原始图片。在第12.1节中已经介绍了通过BitmapFactory.Options 对象来加载一张缩放后的图片，但是那种方法对FilelnputStream的缩放存在问题，原因是 FilelnputStream是一种有序的文件流，而两次decodeStream调用影响了文件流的位置属性， 导致了第二次decodeStream时得到的是mill。为了解决这个问题，可以通过文件流来得到 它所对应的文件描述符，然后再通过BitmapFactory.decodeFileDescriptor方法来加载一张缩放后的图片。除此之外，DiskLruCache还提供了 remove、delete等方法用于磁盘缓存的删除操作。


## 参考

[1. DiskLruCache 修改后的代码](https://github.com/singwhatiwanna/android-art-res/blob/1e9cc66fe8a886f58b8c88f2f1604ca2828a611b/Chapter_12/src/com/ryg/chapter_12/loader/DiskLruCache.java)

[2. 使用缓存策略实现的 ImageLoader](https://github.com/singwhatiwanna/android-art-res/blob/1e9cc66fe8a886f58b8c88f2f1604ca2828a611b/Chapter_12/src/com/ryg/chapter_12/loader/ImageLoader.java)

[3. 《Android开发艺术探索》- 任玉刚·著](https://item.jd.com/11760209.html)
