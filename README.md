# LQtUtils
This is a module containing a few tools that I typically use in Qt apps. Using this module in your app is simple: add a submodule to your git repo and include the headers. Most of the code is in independent headers, so you don't need to build anything separately.
A couple of headers need the related source file to be compiled explicitly; in that case I typically add this project as a submodule and link to my main project file.

More articles related to these topics: https://bugfreeblog.duckdns.org/tag/lqtutils.

## Index

- [How to Include](#how-to-include)
- [Synthesize Qt properties in a short way (lqtutils_prop.h)](#synthesize-qt-props)
- [Synthesize Qt enums and quickly expose to QML (lqtutils_enum.h)](#synthesize-qt-enums)
- [Synthesize Qt settings with support for signals (lqtutils_settings.h)](#synthesize-qt-settings)
- [Cache values and init automatically](#cache-values)
- [Threading tools](#threading)
- [Auto execute actions when exiting a scope (lqtutils_autoexec.h)](#autoexec)
- [Measuring rate](#measure-rate)
- [Measuring framerate](#measure-framerate)
- [Showing Local Notifications](#local-notifications)
- [Single shot timer for QML (lqtutils_ui.h)](#singleshot-qml)
- [Getting safe areas on mobile](#safe-areas)
- [Measure Performance (lqtutils_perf.h)](#measure-performance)
- [Blocking Queue for qt (lqtutils_bqueue.h)](#blocking-queue)
- [Download a File with Progress Notifications (lqtutils_net.h)](#download-file)
- [FontAwesome in QML](#fontawesome)
- [Compute total and available RAM (lqtutils_system.h) [Linux only]](#available-ram)

<a id="how-to-include"></a>
## How to Include

To include in a qmake app:

```
include(lqtutils/lqtutils.pri)
```

To include in a cmake app in Qt6:

```
add_subdirectory(${CMAKE_CURRENT_SOURCE_DIR}/deps/lqtutils)
target_link_libraries(yourproj lqtutilsplugin)
```

NOTE: the name of the library to link is `lqtutilsplugin`, not just `lqtutils`.

To include in a cmake app in Qt5:

```
include(${CMAKE_CURRENT_SOURCE_DIR}/deps/lqtutils/CMakeLists_qt5.txt)
target_link_libraries(LQtUtilsTest PRIVATE lqtutils)
```

<a id="synthesize-qt-props"></a>
## Synthesize Qt properties in a short way (lqtutils_prop.h)
**For more info: https://bugfreeblog.duckdns.org/2020/05/synthesize-qt-properties.html.**

Contains a few useful macros to synthesize Qt props. For instance:
```c++
L_BEGIN_CLASS(Fraction)
L_RW_PROP_AS(double, numerator, 5)
L_RW_PROP_AS(double, denominator, 9)
L_END_CLASS
```
instead of:
```c++
class Fraction : public QObject
{
    Q_OBJECT
    Q_PROPERTY(double numerator READ numerator WRITE setNumerator NOTIFY numeratorChanged)
    Q_PROPERTY(double denominator READ denominator WRITE setDenominator NOTIFY denominatorChanged)
public:
    Fraction(QObject* parent = nullptr) :
        QObject(parent),
        m_numerator(5),
        m_denominator(9)
        {}

    double numerator() const {
        return m_numerator;
    }

    double denominator() const {
        return m_denominator;
    }

public slots:
    void setNumerator(double numerator) {
        if (m_numerator == numerator)
            return;
        m_numerator = numerator;
        emit numeratorChanged(numerator);
    }

    void setDenominator(double denominator) {
        if (m_denominator == denominator)
            return;
        m_denominator = denominator;
        emit denominatorChanged(denominator);
    }

signals:
    void numeratorChanged(double numerator);
    void denominatorChanged(double denominator);

private:
    double m_numerator;
    double m_denominator;
};
```
This makes the usage of Qt properties more synthetic and speeds up the development. It is very useful when many properties are needed to expose data to QML. Properties will provide a setter, a getter, notifications through their signal and support binding in QML.

You can use macros even in a larger class, with a regular syntax:
```c++
class Fraction : public QObject
{
    Q_OBJECT
    L_RW_PROP_AS(double, numerator, 5)
    L_RW_PROP_AS(double, denominator, 9)
public:
    Fraction(QObject* parent = nullptr) : QObject(parent) {}
};
```

### References

If you need to be able to modify the property itself from C++ instead of resetting it, you can use the *_REF alternatives of L_RW_PROP and L_RO_PROP. In that case, getter methods return a reference to the type in C++.

### Signals With or Without Parameters

By default, signals are generated with the value passed in the argument. If you prefer signals without params, you can define LQTUTILS_OMIT_ARG_FROM_SIGNAL **before** including the header.

### Autosetter

It is also possible to omit the name of the setter in the declaration. This results in the auto-generation of a setter with the name "set_<property name>". To obtain this behavior, simply use the variants of the above macros with the suffix "_AS". This should work for all property types: ro, rw, ref and all gadget props.

### Custom Setter

It is possible to generate a property without its setter implementation. The setter con be implemented with a slot explicitly by writing a method named "set_<propname>" returning void. An example can be found [here](lqtutils/tst_lqtutils.cpp#L52).

### Complete List of Available Macros

For QObjects:

    L_RW_PROP(type, name, setter)
    L_RW_PROP(type, name, setter, default)
    L_RW_PROP_AS(type, name)
    L_RW_PROP_AS(type, name, default)
    L_RW_PROP_CS(type, name)
    L_RW_PROP_CS(type, name, default)
    L_RW_PROP_REF(type, name, setter)
    L_RW_PROP_REF(type, name, setter, default)
    L_RW_PROP_REF_AS(type, name)
    L_RW_PROP_REF_AS(type, name, default)
    L_RW_PROP_REF_CS(type, name)
    L_RW_PROP_REF_CS(type, name, default)
    L_RO_PROP(type, name, setter)
    L_RO_PROP(type, name, setter, default)
    L_RO_PROP_AS(type, name)
    L_RO_PROP_AS(type, name, default)
    L_RO_PROP_CS(type, name)
    L_RO_PROP_CS(type, name, default)
    L_RO_PROP_REF(type, name, setter)
    L_RO_PROP_REF(type, name, setter, default)
    L_RO_PROP_REF_AS(type, name)
    L_RO_PROP_REF_AS(type, name, default)
    L_RO_PROP_REF_CS(type, name)
    L_RO_PROP_REF_CS(type, name, default)
    L_BEGIN_CLASS(name)
    L_END_CLASS

For gadgets:

    L_RW_GPROP(type, name, setter)
    L_RW_GPROP(type, name, setter, default)
    L_RW_GPROP_AS(type, name)
    L_RW_GPROP_AS(type, name, default)
    L_RW_GPROP_CS(type, name)
    L_RW_GPROP_CS(type, name, default)
    L_RO_GPROP(type, name, setter)
    L_RO_GPROP(type, name, setter, default)
    L_RO_GPROP_AS(type, name)
    L_RO_GPROP_AS(type, name, default)
    L_RO_GPROP_CS(type, name)
    L_RO_GPROP_CS(type, name, default)
    L_BEGIN_GADGET(name)
    L_END_GADGET

<a id="synthesize-qt-settings"></a>
## Synthesize Qt settings with support for signals (lqtutils_settings.h)
**For more info: https://bugfreeblog.duckdns.org/2023/01/lqtutils-settings.html.**

Contains a few tools that can be used to speed up writing simple settings to a file. Settings will still use QSettings and are therefore fully compatible. The macros are simply shortcuts to synthetise code. I only used this for creating ini files, but should work for other formats. An example:
```c++
L_DECLARE_SETTINGS(LSettingsTest, new QSettings("settings.ini", QSettings::IniFormat))
L_DEFINE_VALUE(QString, string1, QString("string1"))
L_DEFINE_VALUE(QSize, size, QSize(100, 100))
L_DEFINE_VALUE(double, temperature, -1)
L_DEFINE_VALUE(QByteArray, image, QByteArray())
L_END_CLASS

L_DECLARE_SETTINGS(LSettingsTestSec1, new QSettings("settings.ini", QSettings::IniFormat), "SECTION_1")
L_DEFINE_VALUE(QString, string2, QString("string2"))
L_END_CLASS
```
This will provide an interface to a "strong type" settings file containing a string, a QSize value, a double, a jpg image and another string, in a specific section of the ini file. Each class is reentrant like QSettings and can be instantiated in multiple threads.
Each class also provides a unique notifier: the notifier can be used to receive notifications when any thread in the code changes the settings, and can also be used in bindings in QML code. For an example, refer to LQtUtilsQuick:
```c++
Window {
    visible: true
    x: settings.appX
    y: settings.appY
    width: settings.appWidth
    height: settings.appHeight
    title: qsTr("Hello World")

    Connections {
        target: settings
        onAppWidthChanged:
            console.log("App width saved:", settings.appWidth)
        onAppHeightChanged:
            console.log("App width saved:", settings.appHeight)
    }

    Binding { target: settings; property: "appWidth"; value: width }
    Binding { target: settings; property: "appHeight"; value: height }
    Binding { target: settings; property: "appX"; value: x }
    Binding { target: settings; property: "appY"; value: y }
}
```

<a id="synthesize-qt-enums"></a>
## synthesize Qt enums and quickly expose to QML (lqtutils_enum.h)
**For more info: https://bugfreeblog.duckdns.org/2020/06/synthesizing-qt-settings.html.**

Contains a macro to define a enum and register it with the meta-object system. This enum can then be exposed to the QML. To create the enum simply do:
```c++
L_DECLARE_ENUM(MyEnum,
               Value1 = 1,
               Value2,
               Value3)
```
This enum is exposed using a namespace without subclassing QObject. Register with the QML engine with:
```c++
MyEnum::registerEnum("com.luke", 1, 0);
```

<a id="cache-values"></a>
## Cache values and init automatically
```lqt::CacheValue``` caches values of any type in a hash and calls the provided lambda if the value was never initialized. This is useful when writing settings classes and you want to read only once.

<a id="threading"></a>
## Threading tools (lqtutils_threading.h)
```lqt::RecursiveMutex``` is a simple QMutex subclass defaulting to recursive mode.

```INVOKE_AWAIT_ASYNC``` is a wrapper around QMetaObject::invokeMethod that allows to execute a slot or lambda in the thread of an obj, synchronously awaiting for the result. E.g.:
```c++
QThread* t = new QThread;
t->start();
QObject* obj = new QObject;
obj->moveToThread(t);
INVOKE_AWAIT_ASYNC(obj, [&i, currentThread, t] {
    i++;
    QCOMPARE(t, QThread::currentThread());
    QVERIFY(QThread::currentThread() != currentThread);
    QCOMPARE(i, 11);
});
QCOMPARE(i, 11);
```

```lqt_run_in_thread``` runs a lambda in a specific QThread asynchronously.

<a id="autoexec"></a>
## Auto execute actions when exiting a scope (lqtutils_autoexec.h)
A class that can be used to execute a lambda whenever the current scope ends, e.g.:
```c++
int i = 9;
{
    lqt::AutoExec autoexec([&] {
        i++;
    });
    QCOMPARE(i, 9);
}
QCOMPARE(i, 10);
```

```lqt::SharedAutoExec```: a class that can create copiable autoexec objects. Useful to implement locks with a function being executed when the lock is released.

<a id="measure-rate"></a>
## Measuring rate (lqtutils_freq.h)

```lqt::FreqMeter``` is a class that allows to measure the rate of any sampling activity. For example the refresh rate (see lqtutils_ui.h) or the rate of some kind of processing.

<a id="measure-framerate"></a>
## Measuring frame rate in QML (lqtutils_ui.h)

**For more info: https://bugfreeblog.duckdns.org/2021/09/measure-framerate-qt.html.**

```lqt::FrameRateMonitor``` is a class that can be used to measure the current frame rate of a QQuickWindow. The class monitors the frequency of frame swaps and provides a property with a value reporting the frame rate during the last second.

<a id="local-notifications"></a>
## Showing Local Notifications

The `lqt::SystemNotification` and the `lqt::AndroidSystemNotification` objects can be used to show a local notification on Linux, Android, Windows, iOS and MacOS.

Example:

```c++
#ifdef Q_OS_ANDROID
    lqt::AndroidSystemNotification notification;
    notification.set_icon(QImage(":/qt/qml/FlashbackPrism/assets/icon_96.png"));
    notification.set_activityClass(QSL("org.qtproject.qt.android.bindings.QtActivity"));
#else
    lqt::SystemNotification notification;
#endif
    notification.set_appName(qApp->applicationName());
    notification.set_title(tr("Flashbacks available"));
    notification.set_message(tr("You have %1 memories taken in %2 years for today. Have a look!").arg(photos).arg(years));
    notification.set_openApp(true);
    notification.send();
```

<a id="singleshot-qml"></a>
## Single shot timer for QML (lqtutils_ui.h)

```c++
lqt::QmlUtils::singleShot(int msec, QJSValue callback)
```

Useful to defer an action of a specified amount of milleseconds in QML, without having to create a timer. Example:

```QML
lqtUtils.singleShot(5000, () => console.log("Hello!"))
```

remember to expose lqt::QmlUtils to QML with:

```c++
engine.rootContext()->setContextProperty("lqtUtils", new lqt::QmlUtils(qApp));
```

<a id="safe-areas"></a>
## Getting safe areas on mobile (lqtutils_ui.h)

**For more info: https://bugfreeblog.duckdns.org/2023/01/qt-qml-cutouts.html.**

Both iOS and Android smartphones may include cutouts and you may want to know where it is safe to draw your QML UI. These four methods are useful to get the safe area both on iOS and Android (on Desktop the methods are still defined and simply return 0).

```c++
Q_INVOKABLE static double safeAreaBottomInset();
Q_INVOKABLE static double safeAreaTopInset();
Q_INVOKABLE static double safeAreaRightInset();
Q_INVOKABLE static double safeAreaLeftInset();
```

Note that the methods return a value in logical pixels, so device pixel ratio is already taken into account.

This is an example of a demo app where some elements extends under the cutouts, while the interactive area does not.

<img src="docs/cutouts.webp" width="20%"></img>

### How to Use

To use it, simply instantiate it when you need it like this:

```c++
QQuickView view;
LQTFrameRateMonitor* monitor = new lqt::FrameRateMonitor(&view);
```

expose it to QML if you need it:

```c++
view.engine()->rootContext()->setContextProperty("fpsmonitor", monitor);
```

and use it in QML (or in C++ if you prefer):

```js
Text { text: qsTr("fps: ") + fpsmonitor.freq }
```

If there have been no updates to the scene, Qt will not redraw it and you will end up with a measurement of ~1fps. If you want to measure how many frames *can* be drawn per second, you can enable automatic frame update triggers via the helper

```c++
lqt::enableAutoFrameUpdates(view);
```

which causes the window to request an update on every frame swap (that is on every vsync in most cases).

### Details

The frame rate is provided in a notifiable property of the ```lqt::FrameRateMonitor``` class:

```c++
L_RO_PROP_AS(int, freq, 0)
```

Note that this property is defined using the prop macros provided by this library. The property is recomputed when each frame is swapped or after a second. The overhead of the component should be minimal.

<a id="measure-performance"></a>
## Measure Performance (lqtutils_perf.h)

```measure_time``` is a shortcut to quickly measure a procedure provided in a lambda for benchmarking it. The macro ```L_MEASURE_TIME``` can be used to be able to enable/disable the measurement through ```L_ENABLE_BENCHMARKS``` at project level reducing overhead to zero.

### How to Use

This is an exmaple:

```c++
measure_time([&] {
    const uchar* rgba = ...
    for (int x = 0; x < width; x++) {
        for (int y = 0; y < height; y++) {
            [process pixel]
        }
    }
}, [] (qint64 time) {
    qWarning() << "Image processed in:" << time;
});
```

<a id="blocking-queue"></a>
## Blocking Queue for qt (lqtutils_bqueue.h)

This header contains a simple blocking queue. It allows blocking/nonblocking insertions with timeout, blocking/nonblocking removal and a safe processing of the queue. This is an example of the producer/consumer pattern:

```c++
struct LQTTestProducer : public QThread
{
    LQTTestProducer(LQTBlockingQueue<int>* queue) : QThread(), m_queue(queue) {}
    void run() override {
        static int i = 0;
        while (!isInterruptionRequested()) {
            QThread::msleep(10);
            m_queue->enqueue(i++);
            QVERIFY(m_queue->size() <= 10);
        }
    }
    void requestDispose() {
        requestInterruption();
        m_queue->requestDispose();
    }

private:
    LQTBlockingQueue<int>* m_queue;
};

struct LQTTestConsumer : public QThread
{
    LQTTestConsumer(LQTBlockingQueue<int>* queue) : QThread(), m_queue(queue) {}
    void run() override {
        static int i = 0;
        while (!isInterruptionRequested()) {
            QThread::msleep(15);
            std::optional<int> ret = m_queue->dequeue();
            if (!ret)
                return;
            QVERIFY(*ret == i++);
            QVERIFY(lqt_in_range(m_queue->size(), 0, 11));
        }
    }
    void requestDispose() {
        requestInterruption();
        m_queue->requestDispose();
    }

private:
    LQTBlockingQueue<int>* m_queue;
};

[...]

lqt::BlockingQueue<int> queue(10, QSL("display_name"));
LQTTestConsumer consumer(&queue);
LQTTestProducer producer(&queue);
consumer.start();
producer.start();

QEventLoop loop;
QTimer::singleShot(30*1000, this, [&] { loop.quit(); });
loop.exec();

producer.requestDispose();
producer.wait();
consumer.requestDispose();
consumer.wait();
```

<a id="download-file"></a>
## Download a File with Progress Notifications (lqtutils_net.h)

The class ```lqt::Downloader``` can be used to download a URL to a file in a background thread. Example:

```c++
QScopedPointer<lqt::Downloader> downloader(new lqt::Downloader(url, filePath));
downloader->download();
connect(downloader.data(), &lqt::Downloader::downloadProgress, this, [&downloader] (qint64 progress, qint64 total) {
    qDebug() << "Downloading:" << progress << "/" << total << downloader->state();
});
connect(downloader.data(), &lqt::Downloader::stateChanged, this, [&loop, &downloader] {
    if (downloader->state() == lqt::DownloaderState::S_DONE)
        // Done
});
```

## lqtutils_data.h

Hash of a file:

```c++
QByteArray lqt::hash(const QString &fileName,
                     QCryptographicHash::Algorithm algo = QCryptographicHash::Md5)
```

Writes a random file of the specified size:

```c++
bool lqt::random_file(const QString& fileName, qint64 size)
```

<a id="fontawesome"></a>
## FontAwesome in QML (lqtutils_fa.h)

**For more info: https://bugfreeblog.duckdns.org/2023/05/fontawesome-qml-lqtutils.html.**

This header includes a function to load FontAwesome fonts and QML items to render the fonts in QML. To include FontAwesome in Qt6 add the support to your cmake file when including the project:

```cmake
set(ENABLE_FONT_AWESOME true)
add_subdirectory(${CMAKE_CURRENT_SOURCE_DIR}/lqtutils)
```

then init FontAwesome in your main function:

```c++
lqt::embed_font_awesome(view.engine()->rootContext());
```

Now types and fonts are loaded. To use it in QML:

```qml
import "qrc:/lqtutils/fontawesome"

[...]

LQTFontAwesomeFreeSolid {
    width: 16
    height: 16
    iconUtf8: "\uf013"
}
```

to ensure the plugin is linked, add these two lines to your main.cpp:

```c++
#include <QtQml/QQmlExtensionPlugin>
Q_IMPORT_QML_PLUGIN(lqtutilsPlugin)
```

Note that the size is mandatory. The available types are: `LQTFontAwesomeFreeSolid`, `LQTFontAwesomeFreeRegular` and `LQTFontAwesomeBrandsRegular`. The init function also set the context properties: `fontAwesomeBrandsRegular`, `fontAwesomeFreeRegular` and `fontAwesomeFreeSolid`. These are QFont instances available in QML.

<a id="available-ram"></a>
## Compute total and available RAM (lqtutils_system.h) [Linux only]

**For more info: https://bugfreeblog.duckdns.org/2024/01/computing-total-and-free-ram-in-linux-with-qt.html.**

This header includes a function to compute the total available RAM and the RAM potentially allocatable.

```c++
std::optional<lqt::MemData> mem = lqt::read_mem_data();
if (!mem)
    return;
qInfo() << "Total:" << mem->totalMemBytes;
qInfo() << "Free :" << mem->freeMemBytes;
```