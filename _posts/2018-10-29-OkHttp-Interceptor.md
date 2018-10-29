---
layout: post
title:  OkHttp拦截器之责任链模式
date:   2018-10-29 11:00:00 +0800
categories: Java
tags: Java
published: true
---

* content
{:toc}

# Prologue
一般来说，对于公司的的普通员工请假流程大致都是这样的：员工在OA系统提交请假申请，系统自动转发给直属leader，leader审批通过再自动转发给部门经理审批，经理审批通过再转到HR处理。  
如果请假天数过多，可能公司章程规定还需要CEO审批，于是在部门经理和HR之间还要插入CEO审批。当然如果临时短暂请假，还可能不需要经理审批。   
这其实就是一个典型的责任链模式，该模式设计的好处是可以方便的增加或者删减审批链中的某个结点。

# OkHttp
Android开发涉及到网络访问时，大多都会使用比较流行的开源框架OkHttp，下面是一个简单的OkHttp（v3.9.1）例子：
```
OkHttpClient client = new OkHttpClient.Builder()
    .readTimeout(20_1000, TimeUnit.MILLISECONDS)
    .writeTimeout(20_1000, TimeUnit.MILLISECONDS)
    .addInterceptor(new Interceptor() {
        @Override
        public Response intercept(Chain chain) throws IOException {
            Request request = chain.request();
            // Todo：do your stuff here.
            // ···
            Response response = chain.proceed(request);
            return response;
        }
    })
    .build();

Request request = new Request.Builder()
    .url(ENDPOINT)
    .build();

Response response = client.newCall(request).execute()
```
OkHttp中的拦截器使用了责任链模式，通过`addInterceptor()`添加自定义拦截器，在拦截器实现的`intercept()`方法中通过chain获取到request，在对request进行需要处理的动作后，再通过`proceed()`方法返回response。  

下面通过分析OkHttp的源码来了解其责任链模式的实现，涉及的核心类为如下几个：
```    
Interceptor  
RealInterceptorChain  
RealCall  
OkHttpClient  
HttpLoggingInterceptor  
ConnectInterceptor  
CallServerInterceptor
```
**Note**  
严格来说，`HttpLoggingInterceptor`、`ConnectInterceptor`和`CallServerInterceptor`不算责任链模式的核心类，只是OkHttp内部实现的自定义拦截器，列举出来是方便描述如何实现自定义拦截器以及拦截器顺序。

# Interceptor
Interceptor的核心源码（方便描述，去掉了不太相关的代码，后续同）：
```
public interface Interceptor {
  Response intercept(Chain chain) throws IOException;

  interface Chain {
    Request request();

    Response proceed(Request request) throws IOException;
  }
}
```
可以看到Interceptor接口有一个`Intercept()`方法，该方法传入的参数是一个内部接口Chain（当然这个接口也可以写成一个单独的类文件），Chain接口中有两个关键方法`request()`和`proceed()`，通过前面的OkHttp小例子我们知道自定义的拦截器实现`intercept(chain)`方法获取到参数chain，再通过chain的`request()`方法获取到request，便可以对request进行操作，处理结束后通过chain的`proceed(request)`方法递归调用下一个拦截器处理，所有的拦截器都处理完成后依次返回response。  
`proceed(request)`方法是如何做到递归调用下一个拦截器的呢？这就涉及到`RealInterceptorChain类`了。

# RealInterceptorChain
```
public final class RealInterceptorChain implements Interceptor.Chain {
  private final List<Interceptor> interceptors;
  private final int index;
  private final Request request;
  // ···

  public RealInterceptorChain(List<Interceptor> interceptors, StreamAllocation streamAllocation,
      HttpCodec httpCodec, RealConnection connection, int index, Request request, Call call,
      EventListener eventListener, int connectTimeout, int readTimeout, int writeTimeout) {
    this.interceptors = interceptors;
    this.index = index;
    this.request = request;
    // ···
  }

  @Override public Response proceed(Request request) throws IOException {
    return proceed(request, streamAllocation, httpCodec, connection);
  }

  public Response proceed(Request request, StreamAllocation streamAllocation, HttpCodec httpCodec,
      RealConnection connection) throws IOException {
    if (index >= interceptors.size()) throw new AssertionError();
    // ···

    // Call the next interceptor in the chain.
    RealInterceptorChain next = new RealInterceptorChain(interceptors, streamAllocation, httpCodec,
        connection, index + 1, request, call, eventListener, connectTimeout, readTimeout,
        writeTimeout);
    Interceptor interceptor = interceptors.get(index);
    Response response = interceptor.intercept(next);

    // ···

    return response;
  }
}
```
关键方法`proceed()`，通过index自增获取下一个拦截器，同时新建了一个名为next的RealInterceptorChain链，该链除了index自增，其他属性完全没变（构造参数没变），再将新建的next链作为参数传入下一个拦截器，下一个拦截器调用`intercept(next)`方法处理，处理完成得到response并返回。  
其中下一个拦截器实现的`intercept()`方法中获取到next链，处理完成后又会调用链的`proceed()`方法，于是形成了递归调用。  
`interceptors`会原封不动地传入到每一个新建的链中，通过index的自增依次递归调用`interceptors`列表中的拦截器，直至最后一个处理完成，依次返回response。  
两个问题：
* 原始的拦截器列表`interceptors`又是从哪来的呢？
* index自增不会溢出吗？  

# RealCall
```
final class RealCall implements Call {
    final OkHttpClient client;
    // ···

    @Override public Response execute() throws IOException {
        // ···
        Response result = getResponseWithInterceptorChain();
        // ···
    }

    Response getResponseWithInterceptorChain() throws IOException {
    // Build a full stack of interceptors.
    List<Interceptor> interceptors = new ArrayList<>();
    interceptors.addAll(client.interceptors());
    interceptors.add(retryAndFollowUpInterceptor);
    interceptors.add(new BridgeInterceptor(client.cookieJar()));
    interceptors.add(new CacheInterceptor(client.internalCache()));
    interceptors.add(new ConnectInterceptor(client));
    if (!forWebSocket) {
        interceptors.addAll(client.networkInterceptors());
    }
    interceptors.add(new CallServerInterceptor(forWebSocket));

    Interceptor.Chain chain = new RealInterceptorChain(interceptors, null, null, null, 0,
        originalRequest, this, eventListener, client.connectTimeoutMillis(),
        client.readTimeoutMillis(), client.writeTimeoutMillis());

    return chain.proceed(originalRequest);
    }
}
```
可以看到RealCall在执行`execute()`方法的时候会调用拦截器链，在这里先添加所有OkHttClient的拦截器，然后依次添加OkHttp内部默认实现的拦截器。  

# OkHttpClient
```
public class OkHttpClient implements Cloneable, Call.Factory, WebSocket.Factory {
    final List<Interceptor> interceptors;

    OkHttpClient(Builder builder) {
        this.interceptors = Util.immutableList(builder.interceptors);
        // ···
    }

    public List<Interceptor> interceptors() {
        return interceptors;
    }

    public static final class Builder {
        final List<Interceptor> interceptors = new ArrayList<>();
        // ···

        public Builder addInterceptor(Interceptor interceptor) {
            if (interceptor == null) throw new IllegalArgumentException("interceptor == null");
            interceptors.add(interceptor);
            return this;
        }

        public OkHttpClient build() {
            return new OkHttpClient(this);
        }
    }
}
```
OkHttpClient使用了构造者模式，存在一个内部类Builder，Builder的`addInterceptor()`方法把库使用者自定义的拦截器给添加进来。  
在Builder的`build()`方法中构造OkHttpClient，并把interceptors传递给OkHttpClient。

# CallServerInterceptor
回到`RealInterceptorChain`末尾的第二个问题：index自增不会溢出吗？  
不会，因为OkHttp默认实现了一个拦截器CallServerInterceptor 并添加到列表最后，在该拦截器里不再调用proceed方法。

```
public final class CallServerInterceptor implements Interceptor {
  // ···

  @Override public Response intercept(Chain chain) throws IOException {
      RealInterceptorChain realChain = (RealInterceptorChain) chain;
      HttpCodec httpCodec = realChain.httpStream();
      StreamAllocation streamAllocation = realChain.streamAllocation();
      RealConnection connection = (RealConnection) realChain.connection();
      Request request = realChain.request();

      long sentRequestMillis = System.currentTimeMillis();

      realChain.eventListener().requestHeadersStart(realChain.call());
      httpCodec.writeRequestHeaders(request);
      realChain.eventListener().requestHeadersEnd(realChain.call(), request);

      Response.Builder responseBuilder = null;
      if (HttpMethod.permitsRequestBody(request.method()) && request.body() != null) {
        // If there's a "Expect: 100-continue" header on the request, wait for a "HTTP/1.1 100
        // Continue" response before transmitting the request body. If we don't get that, return
        // what we did get (such as a 4xx response) without ever transmitting the request body.
        if ("100-continue".equalsIgnoreCase(request.header("Expect"))) {
          httpCodec.flushRequest();
          realChain.eventListener().responseHeadersStart(realChain.call());
          responseBuilder = httpCodec.readResponseHeaders(true);
        }

        if (responseBuilder == null) {
          // Write the request body if the "Expect: 100-continue" expectation was met.
          realChain.eventListener().requestBodyStart(realChain.call());
          long contentLength = request.body().contentLength();
          CountingSink requestBodyOut =
              new CountingSink(httpCodec.createRequestBody(request, contentLength));
          BufferedSink bufferedRequestBody = Okio.buffer(requestBodyOut);

          request.body().writeTo(bufferedRequestBody);
          bufferedRequestBody.close();
          realChain.eventListener()
              .requestBodyEnd(realChain.call(), requestBodyOut.successfulCount);
        } else if (!connection.isMultiplexed()) {
          // If the "Expect: 100-continue" expectation wasn't met, prevent the HTTP/1 connection
          // from being reused. Otherwise we're still obligated to transmit the request body to
          // leave the connection in a consistent state.
          streamAllocation.noNewStreams();
        }
      }

      httpCodec.finishRequest();

      if (responseBuilder == null) {
        realChain.eventListener().responseHeadersStart(realChain.call());
        responseBuilder = httpCodec.readResponseHeaders(false);
      }

      Response response = responseBuilder
          .request(request)
          .handshake(streamAllocation.connection().handshake())
          .sentRequestAtMillis(sentRequestMillis)
          .receivedResponseAtMillis(System.currentTimeMillis())
          .build();

      realChain.eventListener()
          .responseHeadersEnd(realChain.call(), response);

      int code = response.code();
      if (forWebSocket && code == 101) {
        // Connection is upgrading, but we need to ensure interceptors see a non-null response body.
        response = response.newBuilder()
            .body(Util.EMPTY_RESPONSE)
            .build();
      } else {
        response = response.newBuilder()
            .body(httpCodec.openResponseBody(response))
            .build();
      }

      if ("close".equalsIgnoreCase(response.request().header("Connection"))
          || "close".equalsIgnoreCase(response.header("Connection"))) {
        streamAllocation.noNewStreams();
      }

      if ((code == 204 || code == 205) && response.body().contentLength() > 0) {
        throw new ProtocolException(
            "HTTP " + code + " had non-zero Content-Length: " + response.body().contentLength());
      }

      return response;
    }

    // ···
}
```
可以看到在实现的`intercept()`方法中始终没有调用`chain.proceed()`方法。作为列表的最后一个拦截器，结束递归调用。

# HttpLoggingInterceptor
HttpLoggingInterceptor是OkHttp内部实现的一个log分级的拦截器，实现`intercept()`方法，通过chain获取request，处理完成后再调用`chain.proceed(request)`方法。

```
public final class HttpLoggingInterceptor implements Interceptor {

    @Override public Response intercept(Chain chain) throws IOException {
        Request request = chain.request();
        // ···
        return chain.proceed(request);
   }

}
```

# ConnectInterceptor
ConnectInterceptor同样是OkHttp内部实现的一个拦截器，与HttpLoggingInterceptor不同的是，chain的递归调用方法：
```
public final class ConnectInterceptor implements Interceptor {
  public final OkHttpClient client;

  public ConnectInterceptor(OkHttpClient client) {
    this.client = client;
  }

  @Override public Response intercept(Chain chain) throws IOException {
    RealInterceptorChain realChain = (RealInterceptorChain) chain;
    Request request = realChain.request();

    StreamAllocation streamAllocation = realChain.streamAllocation();
    // We need the network to satisfy this request. Possibly for validating a conditional GET.
    boolean doExtensiveHealthChecks = !request.method().equals("GET");
    HttpCodec httpCodec = streamAllocation.newStream(client, chain, doExtensiveHealthChecks);
    RealConnection connection = streamAllocation.connection();

    return realChain.proceed(request, streamAllocation, httpCodec, connection);
  }
}
```
ConnectInterceptor实现的`intercept()`方法中，首先是把chain强转为RealInterceptorChain，通过RealInterceptorChain的`proceed()`方法来实现递归调用，与HttpLoggingInterceptor的差别在于`proceed()`方法的参数，ConnectInterceptor这样做的可以改变后续递归链的属性（前文描述过，在RealInterceptorChain中通过传入的参数构建新的链）。  
自定义拦截器可以根据自己需求来决定使用何种调用方法。
