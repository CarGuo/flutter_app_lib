### Flutter Lib

这是一个测试将 Flutter 工程打包成 aar 的项目，单纯支持 Android，测试混合开发结合的可行性。


flutter packages get 之后，执行 flutter clean，之后在android目录下执行 ./gradlew assembleRelease

- flutter_shared 的问题已经修复，但是 `--local-engine` 执行下依然后问题 [#21107](https://github.com/flutter/flutter/issues/21107)
