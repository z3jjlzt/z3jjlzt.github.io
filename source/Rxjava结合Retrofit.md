---
title:  Rxjava结合Retrofit
date: 2016-04-10 15:33:47
tags: android
---

### android中Rxjava结合Retrofit使用简介。
<!-- more -->
### 一. 添加依赖
```
    compile 'com.squareup.retrofit2:retrofit:2.0.0-beta4'
    compile 'io.reactivex:rxjava:1.1.1'

    compile 'com.squareup.retrofit2:converter-gson:2.0.0-beta4'
    //Gson解析器

    compile 'io.reactivex:rxandroid:1.1.0'
    //rxjava中用到的AndroidSchedulers.mainThread()

    compile 'com.squareup.retrofit2:adapter-rxjava:2.0.0-beta4'
    这里使用的是retrofit2.0,默认为okhttp3.0
```
### 二. 定义请求接口，转换HTTPAPI为Java接口
```java
public interface IgankApi {
    @GET("a.json")
    Call<List<GirlEntity>> getGirl();

    @GET("data/%E7%A6%8F%E5%88%A9/{count}/{page}")
    Call<GirlJsonData> getGirl(@Path("count") int count, @Path("page") int page);

//与rxjava结合api
    @GET("data/%E7%A6%8F%E5%88%A9/{count}/{page}")
   Observable<GirlJsonData> getG(@Path("count") int count, @Path("page") int page);

}
```
### 三. 接着使用类Retrofit生成 接口的实现，使用了动态代理。
```java
   public static IgankApi getIgankApi() {

        if (igankApi == null) {
            synchronized (IgankApi.class) {
                if (igankApi == null) {
                    Retrofit retrofit = new Retrofit.Builder().baseUrl("http://gank.io/api/")
                            .addConverterFactory(gsonConverterFactory)
                            .client(okHttpClient)
                            .addCallAdapterFactory(rxJavaCallAdapterFactory)
                            .build();
                    igankApi = retrofit.create(IgankApi.class);
                }
            }
        }
        return igankApi;
    }
```
### 四. 调用接口。
```java
  private void getImg() {
        NetUtils.getIgankApi().getG(10, 2).flatMap(new Func1<GirlJsonData, Observable<List<GirlEntity>>>() {
            @Override
            public Observable<List<GirlEntity>> call(GirlJsonData girlJsonData) {
                return Observable.just(girlJsonData.getResults());
            }
        })
                .subscribeOn(Schedulers.io())
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(new Subscriber<List<GirlEntity>>() {
                    @Override
                    public void onCompleted() {

                    }

                    @Override
                    public void onError(Throwable e) {

                    }

                    @Override
                    public void onNext(List<GirlEntity> girlEntities) {
                        girlEntityList.addAll(girlEntities);
                        loge(girlEntityList.get(1).getUrl() + "");
                        myRecycleViewAdapter.notifyDataSetChanged();
                    }
                });

    }
```
### 五. 参考资料
[tough1985的作品](http://gank.io/post/56e80c2c677659311bed9841)
[完整项目](https://github.com/z3jjlzt/Girl)
