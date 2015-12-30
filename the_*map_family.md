# *map家族

RxJava提供了几个mapping函数：`map()`,`flatMap()`,`concatMap()`,`flatMapIterable()`以及`switchMap()`.所有这些函数都应用一个可观测序列，变换它发射的值，用一种新的形式返回他们。让我们用合适的真实世界的例子一个个的看下它们。

## Map

RxJava的`map`函数接收一个指定的`Func`对象然后将它应用到每一个由Observable发射的值上。下图展示了如何将一个乘法函数应用到每个发出的值上以此创建一个新的Observable来发射转换的数据。

![](chapter5_1.png)

考虑我们已安装的应用列表。我们怎么才能够显示同样的列表，但是所有的名字都是小写。

我们的`loadList()`函数可以改成这样：
```java
private void loadList(List<AppInfo> apps) {
    mRecyclerView.setVisibility(View.VISIBLE);
    Observable.from(apps)
        .map(new Func1<AppInfo,AppInfo>(){
            @Override
            public Appinfo call(AppInfo appInfo){
                String currentName = appInfo.getName();
                String lowerCaseName = currentName.toLowerCase();
                appInfo.setName(lowerCaseName);
                return appInfo;
            }
        })
        .subscribe(new Observable<AppInfo>() {

            @Override
            public void onCompleted() {
                mSwipeRefreshLayout.setRefreshing(false);
            }

            @Override
            public void onError(Throwable e) {
                Toast.makeText(getActivity(), "Something went wrong!", Toast.LENGTH_SHORT).show();
                mSwipeRefreshLayout.setRefreshing(false);
            }

            @Override
            public void onNext(AppInfo appInfo) {
                mAddedApps.add(appInfo); 
                mAdapter.addApplication(mAddedApps.size() - 1,appInfo);
            }
        });
}
```

正如你看到的这样，像通常一样创建我们发射的Observable，我们加一个`map`调用，我们可以创建一个简单的函数来更新`AppInfo`对象并提供一个小写的新版本的名字给观察者。

## FlatMap

在复杂的场景中，我们有一个这样的Observable：它发射一个数据序列，这些数据本身也可以发射Observable。RxJava的`flatMap()`函数提供一种铺平序列的方式，然后合并这些Observables发射的数据，最后将合并后的结果作为最终的Observable。

![](chapter5_2.png)

当我们在处理可能有大量的Observables时，重要是记住任何一个Observables发生错误的情况，`flatMap()`函数将会触发它自己的`onError()`函数并放弃整个链。

重要的一点是关于合并部分：它允许交叉。正如上图所示，这意味着`flatMap()`函数在最后的Observable中不能够保证源Observables确切的发射顺序。

## ConcatMap

RxJava的`concatMap()`函数解决了`flatMap()`的交叉问题，提供了一种能够把发射的值连在一起的铺平函数，而不是合并它们，如下图所示：

![](chapter5_3.png)

## FlatMapIterable

作为*map家族的一员，`flatMapInterable()`和`flatMap()`很像。仅有的不同是这个变体成对的打包数据，然后生成Iterable而不是原始数据和生成的Observables。

![](chapter5_4.png)

## SwitchMap

如下图所示，`switchMap()`和`flatMap()`很像，除了一点：当原始Observable发射一个新的数据（Observable）时，它将取消订阅并停止监视之前那个数据的Observable产生的Observable，只监视当前这一个。
















