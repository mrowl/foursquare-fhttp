# Foursquare Finagle Http Library #

[Finagle](https://github.com/twitter/finagle) is a wonderful protocol agnostic communication library.
Building an http client using finagle is super simple.

However, building an http request and parsing the response using the netty
library in scala is a chore compared building the client.
FHttp is a scala-idiomatic request building interface similar to 
[scalaj-http](https://github.com/scalaj/scalaj-http) for finagle http clients.

Like [scalaj-http](https://github.com/scalaj/scalaj-http), it supports multipart data and oauth1.

You will probably want to override FHttpClient.service to add your own logging and tracing filters.

##[API Docs](http://foursquare.github.com/foursquare-fhttp/api/)##

## Some Simple Examples ##
    import com.foursquare.fhttp._
    import com.foursquare.fhttp.FHttpRequest._
    import com.twitter.conversions.storage._
    import com.twitter.conversions.time._
    import com.twitter.finagle.builder.ClientBuilder
    import com.twitter.finagle.http.Http

    // Create the singleton client object using a default client spec (hostConnectionLimit=1, no SSL)
    val clientDefault = new FHttpClient("test", "localhost:80").releaseOnShutdown()

    // or customize the ClientBuilder
    val client = new FHttpClient("test2", "localhost:80", 
                    ClientBuilder().codec(Http(_maxRequestSize = 1024.bytes,_maxResponseSize = 1024.bytes))
                      .hostConnectionLimit(15)
                      .tcpConnectTimeout(30.milliseconds)
                      .retries(0)).releaseOnShutdown()

    // add parameters
    val clientWParams = client("/path").params("msg"->"hello", "to"->"world").params(List("from"->"scala"))

    // or headers
    val clientWParamsWHeaders = clientWParams.headers(List("a_header"->"a_value"))

    // non-blocking POST
    val responseFut = clientWParamsWHeaders.postFuture()

    // or issue a blocking request
    clientWParamsWHeaders.getOption()



## OAuth Example ##
    import com.foursquare.fhttp._
    import com.foursquare.fhttp.FHttpRequest._

    // Create the singleton client object using a default client spec
    val client = new FHttpClient("oauth", "term.ie:80")
    val consumer = Token("key", "secret")
    
    // Get the request token
    val token = client("/oauth/example/request_token").oauth(consumer).get_!(asOAuth1Token)

    // Get the access token
    val accessToken = client("/oauth/example/access_token").oauth(consumer, token).get_!(asOAuth1Token)

    // Try some queries
    client("/oauth/example/echo_api").params("k1"->"v1", "k2"->"v2").oauth(consumer, accessToken).get_!()
    // res0: String = k1=v1&k2=v2


