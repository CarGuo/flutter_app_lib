### Flutter Lib

这是一个测试将 Flutter 工程打包成 aar 的项目，单纯支持 Android，测试混合开发结合的可行性。


flutter packages get 之后，执行 flutter clean，之后在android目录下执行 ./gradlew assembleRelease

- flutter_shared 的问题已经修复，但是 `--local-engine` 执行下依然后问题 [#21107](https://github.com/flutter/flutter/issues/21107)



### GSY新书：[《Flutter开发实战详解》](https://item.jd.com/12883054.html)上架啦：[京东](https://item.jd.com/12883054.html) / [当当](http://product.dangdang.com/28558519.html) / 电子版[京东读书](https://e.jd.com/30624414.html)和[Kindle](https://www.amazon.cn/dp/B08BHQ4TKK/ref=sr_1_5?__mk_zh_CN=亚马逊网站&keywords=flutter&qid=1593498531&s=digital-text&sr=1-5)

[![](http://img.cdn.guoshuyu.cn/WechatIMG65.jpeg)](https://item.jd.com/12883054.html)



flutter 在打包成 aar 上比 react native 简单的原因，在于它不像 react native 那样需要 link 本地的 project

- 添加完插件如果存在原生代码，会在项目里生产一个

`.flutter-plugins` 会包含有插件相关的信息

之后 `setting.gradle`

```
def flutterProjectRoot = rootProject.projectDir.parentFile.toPath()

def plugins = new Properties()
def pluginsFile = new File(flutterProjectRoot.toFile(), '.flutter-plugins')
if (pluginsFile.exists()) {
    pluginsFile.withReader('UTF-8') { reader -> plugins.load(reader) }
}
///include进去
plugins.each { name, path ->
    def pluginDirectory = flutterProjectRoot.resolve(path).resolve('android').toFile()
    include ":$name"
    project(":$name").projectDir = pluginDirectory
}

```

之后在 app 中会使用 build.gradle 

``` 
apply from: "$flutterRoot/packages/flutter_tools/gradle/flutter.gradle"
```

之后在 flutter.gradle 中 include 进来。

``` 
   File pluginsFile = new File(project.projectDir.parentFile.parentFile, '.flutter-plugins')
        Properties plugins = readPropertiesIfExist(pluginsFile)

        plugins.each { name, _ ->
            def pluginProject = project.rootProject.findProject(":$name")
            if (pluginProject != null) {
                project.dependencies {
                    if (project.getConfigurations().findByName("implementation")) {
                        implementation pluginProject
                    } else {
                        compile pluginProject
                    }
                }
		pluginProject.afterEvaluate {
                    pluginProject.android.buildTypes {
                        profile {
                            initWith debug
                        }
                    }
		}
                pluginProject.afterEvaluate this.&addFlutterJarCompileOnlyDependency
            } else {
                project.logger.error("Plugin project :$name not found. Please update settings.gradle.")
            }
        }

```



#### 所以你需要 fat-aar 来做这件事情，因为打包 aar 时本地工程是不会打包进去的，所以需要在 dependencies 中通过配置读取相关的文件配置
