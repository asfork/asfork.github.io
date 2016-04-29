---
layout: post
title: "Retrofit2 使用及新特性"
date: 2016-04-29 08:14:33 +800
categories: android
---

最近 [Retrofit2](http://square.github.io/retrofit/) 正式版发布了（当前版本号已经小幅更新到2.0.2），带来了相当多的新特性，内部也默认使用 Okhttp3 作为网络请求库。使用的便捷性比 Retrofit1 再次向前迈了一步。
下面总结了几个自己用到的新特性。

### Call
2.0 支持了在一个类型中的同步和异步。只需要使用新的 call 方法。

```java
interface GitHubService {
  @GET("/repos/{owner}/{repo}/contributors")
  Call<List<Contributor>> repoContributors(
      @Path("owner") String owner,
      @Path("repo") String repo);
}

Call<List<Contributor>> call =
    gitHubService.repoContributors("square", "retrofit");
```

这里是一个同步请求。语法和 OkHttp 基本一模一样，唯一不同是这个函数知道如何做数据的反序列化。它知道如何将 HTTP 响应转换成对象。
方便很多是不是，在很多简单场景下甚至可以不用再引入 Gson 或 Jackson 了(当然你也可以轻松的用 Gson 等序列化库替换它)。

需要注意的是，每一个 call 对象实例只能被用一次，所以说 request 和 response 都是一一对应的。
你其实可以通过 Clone 方法来创建一个一模一样的实例，这个开销是很小的。比如说：你可以在每次决定发请求前 clone 一个之前的实例。

```java
Call<List<Contributor>> call =
    gitHubService.repoContributors("square", "retrofit");

response = call.execute();

// This will throw IllegalStateException:
response = call.execute();


// Clone to get new object
Call<List<Contributor>> call2 = call.clone();
// This will not throw:
response = call2.execute();
```

### 动态 URL Parameter
在涉及到分页时，这个特性十分有用。
例如向 GitHub 发出多个请求，收到一个响应，通常这个响应大概像下面这样：

```java
interface GitHubService {
  @GET("/repos/{owner}/{repo}/contributors")
  Call<List<Contributor>> repoContributors(
      @Path("owner") String owner,
      @Path("repo") String repo);
}

Call<List<Contributor>> call =
    gitHubService.repoContributors("square", "retrofit");
Response<List<Contributor>> response = call.execute();

// HTTP/1.1 200 OK
// Link: <https://api.github.com/repositories/892275/contributors?
page=2>; rel="next", <https://api.github.com/repositories/892275/
contributors?page=3>; rel="last"
// ...
```

在 Retrofit 1.0 的时候，我们没有办法去直接执行 GitHub Server 返回在 header 里的请求地址。
由于动态 URL 地址就是用在连续请求里的。在第一个请求之后，如果返回的结果里有指明下个请求的地址的话，在之前，你可能得单独写个 interface 来处理这种情况，现在就无需那么费事了。

```java
interface GitHubService {
  @GET("/repos/{owner}/{repo}/contributors")
  Call<List<Contributor>> repoContributors(
      @Path("owner") String owner,
      @Path("repo") String repo);

  @GET
  Call<List<Contributor>> repoContributorsPaginate(
      @Url String url);
}
```

Retrofit 2.0 有了新的 标注：@Url ，允许你直接传入一个请求的 URL。
有了这个方法后，我们就可以直接把刚才取出来的下一页的地址传入，是不是一切都流畅了很多：

```java
String nextLink = nextFromGitHubLinks(links);

// https://api.github.com/repositories/892275/contributors?page=2

Call<List<Contributor>> nextCall =
    gitHubService.repoContributorsPaginate(nextLink);
```

这样的话，就能通过调用 repoContributorsPaginate 来获取第二页内容，然后通过第二页的 header 来请求第三页。

Retrofit2 是不是很强大？还有更多新特性等待发掘。推荐浏览库作者之一 Jake Wharton 的演讲记录—— [Simple HTTP with Retrofit 2](https://realm.io/news/droidcon-jake-wharton-simple-http-retrofit-2/)，里面干货众多(Android 开源大牛深度就是不一样，科科)。
另外推荐 [Realm.io](https://realm.io/news/) ，上面分享的文章常常质量都不错。

*本文部分引用 [Simple HTTP with Retrofit 2 -A talk from Droidcon NYC 2015](https://realm.io/news/droidcon-jake-wharton-simple-http-retrofit-2/)*