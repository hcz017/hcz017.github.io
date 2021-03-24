---
date:  2019-05-07 10:40
title:  'Camera API1 和 API2 设置获取试试预览的回调数据的格式'
---

名字有点拗口哈，本文想说的是在获得camera 实时预览的数据时，怎么设置需要的数据的格式。

# API 1

设置预览格式和预览回调：

```java
parameters.setPreviewFormat(ImageFormat.NV21);
mCamera.setParameters(parameters);
mCamera.setPreviewCallback(previewCallback);
```

默认NV21 格式，**传入参数支持NV21 和YV12**，也可以通过 getSupportedPreviewFormats 查询支持的格式。
```java
         * @param pixel_format the desired preview picture format, defined by
         *   one of the {@link android.graphics.ImageFormat} constants.  (E.g.,
         *   <var>ImageFormat.NV21</var> (default), or
         *   <var>ImageFormat.YV12</var>)
         *
         * @see android.graphics.ImageFormat
         * @see android.hardware.Camera.Parameters#getSupportedPreviewFormats
         */
        public void setPreviewFormat(int pixel_format) {
            String s = cameraFormatForPixelFormat(pixel_format);
            if (s == null) {
                throw new IllegalArgumentException(
                        "Invalid pixel_format=" + pixel_format);
            }

            set(KEY_PREVIEW_FORMAT, s);
        }
```
如果getSupportedPreviewFormats  返回  [17, 842094169]，可以查到这两种正是NV21 和YV12 格式的十进制表示：

```java
    /**
     * YCrCb format used for images, which uses the NV21 encoding format.
     *
     * <p>This is the default format
     * for {@link android.hardware.Camera} preview images, when not otherwise set with
     * {@link android.hardware.Camera.Parameters#setPreviewFormat(int)}.</p>
     *
     * <p>For the {@link android.hardware.camera2} API, the {@link #YUV_420_888} format is
     * recommended for YUV output instead.</p>
     */
    public static final int NV21 = 0x11;

    /**
    ...
     * @see android.hardware.Camera.Parameters#setPreviewCallback
     * @see android.hardware.Camera.Parameters#setPreviewFormat
     * @see android.hardware.Camera.Parameters#getSupportedPreviewFormats
     * </p>
     */
    public static final int YV12 = 0x32315659;
```

另外 pixel format 和camera format 的转换如下

```java
        private String cameraFormatForPixelFormat(int pixel_format) {
            switch(pixel_format) {
            case ImageFormat.NV16: return PIXEL_FORMAT_YUV422SP;
            case ImageFormat.NV21: return PIXEL_FORMAT_YUV420SP;
            case ImageFormat.YUY2: return PIXEL_FORMAT_YUV422I;
            case ImageFormat.YV12: return PIXEL_FORMAT_YUV420P;
            case ImageFormat.RGB_565: return PIXEL_FORMAT_RGB565;
            case ImageFormat.JPEG: return PIXEL_FORMAT_JPEG;
            default: return null;
            }
        }
```

可能有同学问了“这是设置的预览的格式啊，怎么保证回调回来的数据也是这个格式呢？”
因为 previewCallback 的作用就是  used to deliver copies of preview frames as they are displayed. 拷贝预览帧。

# API 2

预览回调是通过增加一个ImageReader 实时获取预览数据，设置的格式不区分预览和拍照，因为拍照也是通过ImageReader 的onImageAvailable 回调获得数据。

```java
previewReader = ImageReader.newInstance(previewSize.getWidth(), previewSize.getHeight
                    (), ImageFormat.YUV_420_888, 2);
```

这里点名不支持NV21
```java
     * @param width The default width in pixels of the Images that this reader
     *            will produce.
     * @param height The default height in pixels of the Images that this reader
     *            will produce.
     * @param format The format of the Image that this reader will produce. This
     *            must be one of the {@link android.graphics.ImageFormat} or
     *            {@link android.graphics.PixelFormat} constants. Note that not
     *            all formats are supported, like ImageFormat.NV21.
     * @param maxImages The maximum number of images the user will want to
     *            access simultaneously. This should be as small as possible to
     *            limit memory use. Once maxImages Images are obtained by the
     *            user, one of them has to be released before a new Image will
     *            become available for access through
     *            {@link #acquireLatestImage()} or {@link #acquireNextImage()}.
     *            Must be greater than 0.
     * @see Image
     */
    public static ImageReader newInstance(int width, int height, int format, int maxImages) {
        return new ImageReader(width, height, format, maxImages, BUFFER_USAGE_UNKNOWN);
    }
```

那API 1 中通过setPreviewFormat() 设置的ImageFormat.NV21 预览格式，在API 2 中设置成哪个格式呢？
答案是 **ImageFormat.YUV_420_888**。至于为什么，源码跳到jni 里面去了，我没跟到后面的源码。
从dumpsys media.camera 的信息中看，preview-format 的值和API 1 中设置NV21 是一样的，都是 **yuv420sp**。

```shell
$ adb shell dumpsys media.camera | grep -E "preview-format|Latest set parameters|Device"
  Device 0 is open. Client instance dump:
Latest set parameters:
preview-format: yuv420sp
preview-format-values: yuv420sp,yuv420p,nv12-venus
  Device 1 is closed, no client instance
preview-format: yuv420sp
preview-format-values: yuv420sp,yuv420p,nv12-venus
  Device 2 is open. Client instance dump:
Latest set parameters:
preview-format: yuv420p
preview-format-values: yuv420sp,yuv420p,nv12-venus
  Device 3 is closed, no client instance
```

在API 2 上可以使用下面的方法查询支持输出的格式：

```java
StreamConfigurationMap map = characteristics.get(
                CameraCharacteristics.SCALER_STREAM_CONFIGURATION_MAP);
Log.d(TAG, "get supported out put format: " + map.getOutputFormats());
```

通常来讲如果你设置一个不是getOutputFormats() 返回的格式，你是不能成功创建预览的（可以open），但不排除部分厂商在底层做了workaround（比如小米8）。