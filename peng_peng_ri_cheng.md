# 碰碰日程


![碰碰日程架构图](碰碰日程.png)


# 登录模块

当用户第一次使用App并且查看完App新特性之后，用户进入登录页面```LoginViewController```即可登录，登录方式包含以下几种方式

| 1.手机号登录------->调用的方法：```loginEvent```|
| -- |
| 2.新浪微博登录------>调用的方法：`````` |
| 3.微信登陆|



```swift
override func viewDidLoad() {
    super.viewDidLoad()

    addChildViewController()
}

///  添加子控制器
private func addChildViewController() {
    tabBar.tintColor = UIColor.orangeColor()

    let vc = HomeTableViewController()
    vc.title = "首页"
    vc.tabBarItem.image = UIImage(named: "tabbar_home")
    vc.tabBarItem.selectedImage = UIImage(named: "tabbar_home_highlighted")
    let nav = UINavigationController(rootViewController: vc)

    addChildViewController(nav)
}

```







# 注册模块

``````

