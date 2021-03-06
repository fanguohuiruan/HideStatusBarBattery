# HideStatusBarBattery

###但是如何修改状态栏中电池图标呢？ 
- 注意点 Application -> statusBar -> 电池视图（UIStatusBarBatteryItemView）

##方法1   

  - 通过遍历子视图的方式，查找到电池视图，（默认知道电池视图对应的view名字是 ‘UIStatusBarBatteryItemView’）
  - 难点一 UIApplication 不是一个UIView， 所以要通过valueForKey方式查找到statusBar, 由此引发另一个难点
  - 难点二 怎么知道statusBar 对应的key， 可以通过class_copyIvarList 和 ivar_getName， String.init(utf8String: name!) 找到statusBar 对应的key，也就是 '_statusBar'
  - 难点三 通过遍历查找（递归方法找到）如下, 直接移除即刻, 方法要写在viewDidLayoutSubviews里面
##代码如下
   ```
         func getAllSubViews(view: UIView) {
             if view.subviews.count == 0 {
                return
             }
             for sub in view.subviews {
                 //print(sub)
                 if sub.isKind(of: NSClassFromString("UIStatusBarBatteryItemView")!) {
                    sub.removeFromSuperView()
                 }
                 getAllSubViews(view: sub)
             }
         }

 ```
##方法2

   - 一路下来通过key value 分别查找到对子视图的key然后通过valueforkey方式来找到对应的view，
  - 分别对于应的key是 Application -> _statusBar -> _foregroundView
  -  到foregroundView之后查找UIStatusBarBatteryItemView对应的key时发现查找不到，是一个隐式变量，这个时候最后一步还是要遍历子视图才行
##代码如下
```
        for sub in foregroundView.subviews {

            if sub.isKind(of: NSClassFromString("UIStatusBarBatteryItemView")!){
                print(sub)
                sub.removeFromSuperview()
            }
        }
   
```
##所有代码 

## method1() 对应方法一
```
 func method1(){

        let statusBar = UIApplication.shared.value(forKey: "_statusBar") as! UIView
        let foregroundView = statusBar.value(forKey: "_foregroundView") as! UIView
        for sub in foregroundView.subviews {

            if sub.isKind(of: NSClassFromString("UIStatusBarBatteryItemView")!){
                sub.removeFromSuperview()
            }
        }

    }
 ```
   ## method2() 对应方法二
```
    func method2() {
        //查找statusBar的子视图，并隐藏电池视图
        func hideBatteryViewFor(view: UIView) {
            if view.subviews.count == 0 {
                return
            }
            for sub in view.subviews {
                //print(sub)
                if sub.isKind(of: NSClassFromString("UIStatusBarBatteryItemView")!) {
                    sub.removeFromSuperview()
                }
                hideBatteryViewFor(view: sub)
            }
        }

        let statusBar = UIApplication.shared.value(forKey: "_statusBar") as! UIView
        hideBatteryViewFor(view: statusBar)

    }
```
    #方便方法 , 获取变量和属性
```
    func getIvarFrom(other: AnyClass){

        var count: UInt32 = 0
        let ivarList = class_copyIvarList(other, &count)
        for i in 0..<count{
            let name = ivar_getName(ivarList![Int(i)])
            print(NSStringFromClass(other) + String.init(utf8String: name!)!)
        }

    }
    //获取父视图
    func getSuperView(view: UIView) {

        if view.superview == UIApplication.shared || view.superview == nil {
            return
        }
        print("superviEW")
        print(view)
        getSuperView(view: view.superview!)
    }
```

[github代码地址](https://github.com/fanguohuiruan/HideStatusBarBattery)
