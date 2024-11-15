
Navigation组件适用于模块内和跨模块的路由切换，通过组件级路由能力实现更加自然流畅的转场体验，并提供多种标题栏样式来呈现更好的标题和内容联动效果。一次开发，多端部署场景下，Navigation组件能够自动适配窗口显示大小，在窗口较大的场景下自动切换分栏展示效果。


## 根页面设置


我们在Entry的入口处Index.ets使用Navigation当作根页面，这里会面临一个问题，怎么从启动页跳转到首页，并关闭启动页，使用首页一直留在页面栈中，不允许销毁，在前面的文章《鸿蒙Navigation处理启动页跳转到首页问题》中有介绍处理方法，在此不展开。在使用Navigation时，由于默认带有TitleBar和Toolbar，样式不好自定义，我们直接隐藏TitleBar和Toolbar，实际需要的话，可以自己实现。由于默认是Auto模式，以便于适配大屏设备，若我们在大屏设备上不使用分栏效果，可以强制设置单页面模式。



```
@Entry
@ComponentV2
struct Index {
  nav: NavPathStack = new NavPathStack()

  build() {
    Navigation(this.nav)
    .mode(NavigationMode.Stack)
    .hideToolBar(true)
    .hideTitleBar(true)
    .height('100%')
    .width('100%')
  }
}

```

## 系统路由表配置


接下来我们需要配置系统路由表，在resources/base/profile目录下新建一个json文件，例如router\_map.json，并在里面配置相关的路由页面，例如我们配置一个弹窗页面和一个登录页面。



```
{
  "routerMap": [
    {
      "name": "dialog",
      "pageSourceFile": "src/main/ets/pages/dialog/DialogPage.ets",
      "buildFunction": "dialogBuilder"
    },
    {
      "name": "login",
      "pageSourceFile": "src/main/ets/pages/login/LoginPage.ets",
      "buildFunction": "loginBuilder"
    }
  ]
}

```

在routerMap里面配置具体的页面，name为页面名称，使用push打开新页面时，传的name名称就是这里定义的。pageSourceFile为页面源文件，buildFunction为页面入口builder，通过源文件找到这个入口builder。在module.json5文件中有一个routerMap字段，值为我们前面定义的router\_map.json


## 实现子页面


路由表字义好了后，我们需要实现具体的页面，这里分别实现一个弹窗页面和标准页面。弹窗示例如下，页面模式需要设置为NavDestinationMode.DIALOG



```
@Builder
function dialogBuilder() {
  DialogPage()
}
@ComponentV2
export struct DialogPage {
  build() {
    NavDestination() {
      Stack() {
        Column() {
          Text('弹窗标题')
          Text('弹窗内容')

          Row() {
            Text('取消').layoutWeight(1)
            Text('确定').layoutWeight(1)
          }
        }
        .width('80%')
        .borderRadius(16)
        .backgroundColor($r('app.color.start_window_background'))
      }.width('100%').height('100%')
    }.hideTitleBar(true).mode(NavDestinationMode.DIALOG)
  }
}

```

标准登录页面如下，默认为标准模式，mode可以省略不设置



```
@Builder
function loginBuilder() {
  LoginPage()
}
@ComponentV2
export struct LoginPage {
  build() {
    NavDestination() {
      Column() {
        Text('账号')
        Text('密码')
        Text('登录')
      }
    }
    .width('100%')
    .height('100%')
    .hideTitleBar(true)
  }
}

```

## 页面跳转操作


### 打开新页面


使用NavPathStack的pushPath或pushPathByName可以打开一个新页面，例如打开登录页面则是navPathStack.pushPathByName('login', undefined)，显示一个弹窗则是navPathStack.pushPathByName('dialog', undefined)。可以发现使用Navigation来实现弹窗还是非常简单的，而且可以全局显示，不依赖于具体页面以及Context，而且弹窗还有显示隐藏等回调。


### 返回页面


使用NavPathStack的pop方法关闭当前页面，回到上一个页面，我们还可以使用popToName返回到指定的页面，也可以使用popToIndex返回到第几个页面，甚至还可以使用clear方法直接回到首页，使用起来还是非常方便的。


 本博客参考[wgetCloud机场](https://longdu.org)。转载请注明出处！
