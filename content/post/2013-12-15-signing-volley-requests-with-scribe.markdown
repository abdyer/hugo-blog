---
layout: post
title: "Signing Volley Requests with Scribe"
date: "2013-12-15"
comments: true
categories: [Android, Volley, OAuth]
---

When I initially needed to integrate OAuth request signing with [Volley](https://android.googlesource.com/platform/frameworks/volley) in my Android app, I used the [Signpost library](https://code.google.com/p/oauth-signpost/). It was fairly straightforward to build a URL, pass it to the Signpost OAuthConsumer, and retrieve a new string with the appropriate OAuth parameters added. Unfortunately, as you can see from the [commit history](https://github.com/mttkay/signpost/commits/master), there has been very little activity over the past year.

Increasingly, I see recommendations for [Scribe](https://github.com/fernandezpablo85/scribe-java). The project is very active and it makes [implementing OAuth request signing](https://github.com/fernandezpablo85/scribe-java/wiki/getting-started) as easy as it's likely to get.

At first glance, one big problem integrating Scribe with Volley is that the library is designed to use its own **OAuthRequest** class. Of course, this doesn't jibe with Volley since requests must extend **com.android.volley.Request&lt;T&gt;** to take advantage of the request queue sexiness. Since Java does not have multiple inheritance, my first approach involved an unholy attempt to merge the necessary Scribe request code with my Volley request subclass. While this did familiarize me a bit with the inner workings of Scribe's request signing process, I quickly realized this was a nasty hack.

In the end, I decided to let Scribe's **OAuthRequest** class handle the signing and simply transfer the resulting OAuth parameters to my Volley request.

The general process is as follows:

1. Override the **getUrl()** method in your custom request class.
2. Translate the Volley request method to the corresponding Scribe **Verb**.
3. Create an instance of scribe **OAuthRequest** with the base request URL. Then, iterate over the Volley request parameters and add them to the **OAuthRequest** object.
4. Sign the **OAuthRequest** object using an instance of the Scribe **OAuthService** with your stored token (I stash my **OAuthService** and **Token** instances in my **Application** subclass).
5. Iterate over the **OAuthRequest** object's OAuth parameters and add them to your Volley request's parameters.
6. Add the Volley request to the queue as normal.

And some code to demonstrate:

    package com.example.MyApp;

    import com.android.volley.Request;
    import com.android.volley.Response;

    import org.scribe.model.OAuthRequest;
    import org.scribe.model.Token;
    import org.scribe.model.Verb;

    import java.util.Map;

    public class VolleyOAuthRequest<T> extends Request<T> {

        private HashMap<String, String> params;
        private OAuthRequest oAuthRequest;

        public DiscogsOAuthRequest(int method, String path, Response.ErrorListener errorListener) {
            super(method, path, errorListener);
            params = new HashMap<String, String>();
        }

        public void addParameter(String key, String value) {
            params.put(key, value);
        }

        @Override
        protected Map<String, String> getParams() {
          return params;
        }

        @Override
        public String getUrl() {
            if(oAuthRequest == null) {
                buildOAuthRequest();

                for(Map.Entry<String, String> entry : oAuthRequest.getOauthParameters().entrySet()) {
                    addParameter(entry.getKey(), entry.getValue());
                }
            }
            return super.getUrl() + getParameterString();
        }

        private void buildOAuthRequest() {
            oAuthRequest = new OAuthRequest(getVerb(), super.getUrl());
            for(Map.Entry<String, String> entry : getParams().entrySet()) {
                oAuthRequest.addQuerystringParameter(entry.getKey(), entry.getValue());
            }
            Token token = MyApp.getOAuthToken();
            MyApp.getOAuthService().signRequest(token, oAuthRequest);
        }

        private Verb getVerb() {
            switch (getMethod()) {
                case Method.GET:
                    return Verb.GET;
                case Method.DELETE:
                    return Verb.DELETE;
                case Method.POST:
                    return Verb.POST;
                case Method.PUT:
                    return Verb.PUT;
                default:
                    return Verb.GET;
            }
        }

        private String getParameterString() {
            StringBuilder sb = new StringBuilder("?");
            Iterator<String> keys = params.keySet().iterator();
            while(keys.hasNext()) {
                String key = keys.next();
                sb.append(String.format("&%s=%s", key, params.get(key)));
            }
            return sb.toString();
        }
    }