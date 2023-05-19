---
title: 如何展示quartz所有可执行job
date: 2021-11-09 17:29:25
tags:
- quartz
- java
categories:
- backend
---

最近在写一个定时任务系统，其实是相当简单，也就是用了quartz，但是写的时候遇到了一个小问题，前端创建scheduler的时候，总不能传待执行task的类名吧，这也太尬了。于是就又有了一个小小的需求：要展示所有可执行task的名称

在我的代码里，我的所有可执行task都提供了一个接口，在一个quartz模块里有一个package里边写了很多个类就专门来调task接口，比如：

<!--more-->


```java

/job/job001.java

public class job001 implements Job {

    @Override
    public void execute(JobExecutionContext jobExecutionContext) throws JobExecutionException {
        log.info("do job001");
    }
}

```

于是就在job这个package下所有的类就全部都是可供执行的job。那么这个任务就等价于：如何展示这个package下所有class的类名和名称

其实最简单的方法：就是搞上一个数据库，把数据都存进去，可是这样未免过于不优雅，每次新增任务的时候还要更新数据库里边的数据。于是我就想：能不能直接操作文件？

于是搜了一番果然找到有大佬早早就做过类似的工作：[获取java接口的所有实现类](https://www.cnblogs.com/wangzhen-fly/p/11002814.html),大佬做的工作也非常详细，直接提供类方法可以查看jar包和file里的所有类名，一番删除修改之后就直接可以获取到这个package下所有的类名了，可以说是非常方便

那么还有一个问题：如何给这个类名对应一个便于前端理解的名字呢？我就给每个task类都加了一个`getName()`的方法，使用反射来调用这个方法，就可以获取到类所对应的名称了。一般而言，这里的方法都是有其固定意义的，想要写重复还是挺难的把！再说就算写重复也还是可以校验的。于是我就写了一个接口：

```java
public interface SchJob extends Job, Serializable {
    String getName();
}
```

所有的job类都要继承这个SchJob，那么写一个Job类的话就变成了这样子：

```java
public class Test1 implements SchJob {

    @Override
    public String getName() {
        return "测试方法1";
    }

    @Override
    public void execute(JobExecutionContext jobExecutionContext) throws JobExecutionException {
        doSomeThing();
    }
}

```
然后要写一个配置类来方便的建立所有job class name和name的对应关系：

```java
@Component
public class InitTaskName {

    public static Map<String, String> NameMapClass = new HashMap<>();
    public static Map<String, String> ClassMapName = new HashMap<>();
    public static List<String> NameList = new ArrayList<>();

    @PostConstruct
    public void init() {
        NameMapClass.clear();
        ClassMapName.clear();
        NameList.clear();
        List<String> res = ClazzUtil.getClassName("com.test.proj1");
        res.forEach(r -> {
            try {
                Class classzz = Class.forName(r);
                Constructor constructor = classzz.getConstructor();
                Object o = constructor.newInstance();
                Method method = classzz.getMethod("getName");
                NameMapClass.put((String) method.invoke(o), r);
                ClassMapName.put(r, (String) method.invoke(o));
                NameList.add((String) method.invoke(o));
            } catch (ClassNotFoundException | InstantiationException | InvocationTargetException | NoSuchMethodException | IllegalAccessException e) {
                e.printStackTrace();
            }
        });
        if (NameList.size() != ClassMapName.size() && NameList.size() != NameMapClass.size()) {
            throw new ServiceException("duplication job class name!");
        }
    }
}
```
这样的话在其他类里边就可以很方便的获取所有可执行job的class name和name的对应关系了！

最后贴一下大神写的如何获取java接口的所有实现类，我删除了一些用不到的类：

```java

/**
 * 获取接口的所有实现类 理论上也可以用来获取类的所有子类
 * 查询路径有限制，只局限于接口所在模块下，比如pandora-gateway,而非整个pandora（会递归搜索该文件夹下所以的实现类）
 * 路径中不可含中文，否则会异常。若要支持中文路径，需对该模块代码中url.getPath() 返回值进行urldecode.
 * Created by wangzhen3 on 2017/6/23.
 */
public class ClazzUtil {
    private static final Logger LOG = LoggerFactory.getLogger(ClazzUtil.class);

    /**
     * 获取某包下所有类
     *
     * @param packageName 包名
     * @return 类的完整名称
     */
    public static List<String> getClassName(String packageName) {

        List<String> fileNames = null;
        ClassLoader loader = Thread.currentThread().getContextClassLoader();
        String packagePath = packageName.replace(".", "/");
        URL url = loader.getResource(packagePath);
        System.out.println("now url is " + url);
        if (url != null) {
            String type = url.getProtocol();
            LOG.debug("file type : " + type);
            if (type.equals("file")) {
                String fileSearchPath = url.getPath();
                LOG.debug("fileSearchPath: " + fileSearchPath);
                fileSearchPath = fileSearchPath.substring(0, fileSearchPath.indexOf("/classes"));
                LOG.debug("fileSearchPath: " + fileSearchPath);
                fileNames = getClassNameByFile(fileSearchPath);
            }  else if (type.equals("jar")) {
                try{
                    JarURLConnection jarURLConnection = (JarURLConnection)url.openConnection();
                    JarFile jarFile = jarURLConnection.getJarFile();
                    fileNames = getClassNameByJar(jarFile,packagePath);
                }catch (java.io.IOException e){
                    throw new RuntimeException("open Package URL failed："+e.getMessage());
                }

            }else {
                throw new RuntimeException("file system not support! cannot load MsgProcessor！");
            }
        }
        return fileNames;
    }

    /**
     * 从项目文件获取某包下所有类
     *
     * @param filePath 文件路径
     * @return 类的完整名称
     */
    private static List<String> getClassNameByFile(String filePath) {
        List<String> myClassName = new ArrayList<String>();
        File file = new File(filePath);
        File[] childFiles = file.listFiles();
        for (File childFile : childFiles) {
            if (childFile.isDirectory()) {
                myClassName.addAll(getClassNameByFile(childFile.getPath()));
            } else {
                String childFilePath = childFile.getPath();
                if (childFilePath.endsWith(".class")) {
                    childFilePath = childFilePath.substring(childFilePath.indexOf("\\classes") + 9, childFilePath.lastIndexOf("."));
                    childFilePath = childFilePath.replace("\\", ".");
                    myClassName.add(childFilePath);
                }
            }
        }

        return myClassName;
    }


    private static List<String> getClassNameByJar(JarFile jarFile ,String packagePath) {
        List<String> myClassName = new ArrayList<String>();
        try {
            Enumeration<JarEntry> entrys = jarFile.entries();
            while (entrys.hasMoreElements()) {
                JarEntry jarEntry = entrys.nextElement();
                String entryName = jarEntry.getName();
                //LOG.info("entrys jarfile:"+entryName);
                if (entryName.endsWith(".class")) {
                    entryName = entryName.replace("/", ".").substring(0, entryName.lastIndexOf("."));
                    myClassName.add(entryName);
                    //LOG.debug("Find Class :"+entryName);
                }
            }
        } catch (Exception e) {
            LOG.error("发生异常:"+e.getMessage());
            throw new RuntimeException("发生异常:"+e.getMessage());
        }
        return myClassName;
    }
}
```