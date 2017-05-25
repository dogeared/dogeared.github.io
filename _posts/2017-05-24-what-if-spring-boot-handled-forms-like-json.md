---
layout: post
title:  "What if Spring Boot Handled Forms Like JSON?"
date:   2017-05-24 09:52:41 -0400
categories: [programming, java, spring boot]
tags: [programming, java, spring boot]
---

What follows is an unexpected journey to some of the inner workings of Spring
Boot. In particular, how it handles materializing POJOs from an incoming HTTP
request. It all started with an innocent look at the Slack API...

All of the code mentioned in this post can be found in the
[slack-slash-command-example](https://github.com/dogeared/slack-slash-command-example)
github repo.

I use [Slack](https://slack.com/). A lot. I'm currently in 12 slack orgs.
One of them is even a paid org! I thought I'd play around with the Slack API
and I started with
[Slack Slash commands](https://api.slack.com/slash-commands), as it seemed like
the easiest point of entry.

I also live and breathe [Spring Boot](https://projects.spring.io/spring-boot/).
It's so easy to write APIs with Spring Boot, that this seemed like the most
natural place to start. Here's the simplest one-file Spring Boot API app:

{% highlight java linenos %}
@SpringBootApplication
@RestController
@RequestMapping("/api/v1")
public class SlackApplication {

    public static void main(String[] args) {
        SpringApplication.run(SlackApplication.class, args);
    }

    @RequestMapping("/slack")
    public @ResponseBody Map<String, Object> slack(@RequestBody Map<String, Object> slackSlashCommand) {

        return slackSlashCommand;
    }
}
{% endhighlight %}

Thanks to the magic of Spring Boot and its creation of a fully executable jar,
I can easily fire up this example:

```
target/slack-slash-command-example-0.0.1-SNAPSHOT.jar
```

On Windows, you may have to run:

```
java -jar target/slack-slash-command-example-0.0.1-SNAPSHOT.jar
```

I can hit my API like so
(I am using [HTTPie](https://httpie.org/), a modern curl replacement):

{% highlight bash %}
http POST localhost:8080/api/v1/slack \
  token=token team_id=team_id team_domain=team_domain channel_id=channel_id \
  channel_name=channel_name user_id=user_id user_name=user_name \
  command=command text=text response_url=response_url

HTTP/1.1 200
Content-Type: application/json;charset=UTF-8
Date: Wed, 24 May 2017 14:39:48 GMT
Transfer-Encoding: chunked

{
    "channel_id": "channel_id",
    "channel_name": "channel_name",
    "command": "command",
    "response_url": "response_url",
    "team_domain": "team_domain",
    "team_id": "team_id",
    "text": "text",
    "token": "token",
    "user_id": "user_id",
    "user_name": "user_name"
}
{% endhighlight %}

NOTE: Contrary to the way the above command looks, the data is sent over as
JSON with a Content-Type header of `application/json`. Thank you, HTTPie.

Well, that was easy!

**EXCEPT, this doesn't work with the Slack Slash Command API at all.**

Looking more closely at the [API](https://api.slack.com/slash-commands), I see
that Slack does the POST with an old-school `application/x-www-form-urlencoded`
content type.

"OK", I thought, "Spring Boot is pretty smart, let me try sending over the
params as form input rather than JSON."

I can hit my API again, with `application/x-www-form-urlencoded` by simply
adding <span style="white-space:nowrap;">`--form`</span> to the command line:

{% highlight bash %}
http --form POST localhost:8080/api/v1/slack \
  token=token team_id=team_id team_domain=team_domain channel_id=channel_id \
  channel_name=channel_name user_id=user_id user_name=user_name \
  command=command text=text response_url=response_url

HTTP/1.1 415
Content-Type: application/json;charset=UTF-8
Date: Wed, 24 May 2017 14:55:34 GMT
Transfer-Encoding: chunked

{
  "error": "Unsupported Media Type",
  "exception": "org.springframework.web.HttpMediaTypeNotSupportedException",
  "message": "Content type 'application/x-www-form-urlencoded;charset=utf-8' not supported",
  "path": "/api/v1/slack",
  "status": 415,
  "timestamp": 1495637734535
}
{% endhighlight %}

Hrmph. I've gotten so used to modern APIs using JSON (ahem, Slack - what gives?)
that I wasn't sure how to handle regular old form input in an API. To the
Google!

## ModelAttributes, WebDataBinders and HttpMessageConverters - Oh My!

Turns out, Spring Boot creates Form beans with the [WebDataBinder](http://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/web/bind/WebDataBinder.html).
Non-Form beans are created using an
[HttpMessageConverter](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/http/converter/HttpMessageConverter.html). 
Spring Boot exposes the `MappingJackson2HttpMessageConverter` which handles the
JSON mapping magic.

First thing I wanted to do was to stop using `Map<String, Object>` datatypes and
to make a proper model object. Using `Map<String, Object>` is a handy trick
when you're interacting with an external API and you're not sure exactly what
you are going to get from it. So, I created the `FormSlackSlashCommand` model:

{% highlight java linenos %}
public class FormSlackSlashCommand {

    private String token;
    private String command;
    private String text;

    @JsonProperty("team_id")
    private String teamId;

    @JsonProperty("team_domain")
    private String teamDomain;

    @JsonProperty("channel_id")
    private String channelId;

    @JsonProperty("channel_name")
    private String channelName;

    @JsonProperty("user_id")
    private String userId;

    @JsonProperty("user_name")
    private String userName;

    @JsonProperty("response_url")
    private String responseUrl;

    ...
    // setters and getters
}
{% endhighlight %}

Hold on, there! If this POJO is used for Form handling, why all the
`@JsonProperty` annotations? Well, I want to be able to return this POJO as
JSON, with properties that conform to the Slack API. These annotations don't
get in the way of our Form processing in any way. We'll see how they come into
play shortly.

Here's an updated Controller:

{% highlight java linenos %}
@RestController
@RequestMapping("/api/v1")
public class SlackController {

    private static final Logger log = LoggerFactory.getLogger(SlackController.class);

    @RequestMapping(
        value = "/slack", method = RequestMethod.POST,
        consumes = MediaType.APPLICATION_FORM_URLENCODED_VALUE, produces = MediaType.APPLICATION_JSON_VALUE
    )
    public @ResponseBody FormSlackSlashCommand slack(@ModelAttribute FormSlackSlashCommand slackSlashCommand) {
        log.info("slackSlashCommand: {}", slackSlashCommand);

        return slackSlashCommand;
    }
}
{% endhighlight %}

Notice the `consumes` and `produces` attributes of the `@RequestMapping`
annotation. This controller method will take HTTP Form input and respond with
JSON.

The `@ModelAttribute` annotation ensures that the `WebDataBinder` is engaged to
materialize the `FormSlackSlashCommand` object. Looks like we're done! I'll just
hit the endpoint again to make sure:

{% highlight bash %}
http -f POST localhost:8080/api/v1/slack \
  token=token team_id=team_id team_domain=team_domain channel_id=channel_id \
  channel_name=channel_name user_id=user_id user_name=user_name \
  command=command text=text response_url=response_url

HTTP/1.1 200
Content-Type: application/json;charset=UTF-8
Date: Thu, 25 May 2017 03:55:49 GMT
Transfer-Encoding: chunked

{
    "channel_id": null,
    "channel_name": null,
    "command": "command",
    "response_url": null,
    "team_domain": null,
    "team_id": null,
    "text": "text",
    "token": "token",
    "user_id": null,
    "user_name": null
}
{% endhighlight %}

What the what?!? Well, Spring Boot currently just doesn't handle Form
submissions as flexibly as it does JSON. It automatically handles the "simple"
attributes, like `command`, `text`, and `token` using the corresponding setters
in the `FormSlackSlashCommand`. But, there is no corresponding setter to deal
with attributes like: `channel_id`.

Here's a two-year-old Jira issue that proposes adding an annotation for
handling Form input in a similar way that the Jackson mapper handles JSON
input: [https://jira.spring.io/browse/SPR-13433](https://jira.spring.io/browse/SPR-13433)
(I humbly request your votes on this issue!)

Following this discovery, I came up with three approaches to address
handling Form input more flexibly with what's currently on the truck for Spring
Boot. Many thanks to the `r/java` community for the feedback in the
[comments](https://www.reddit.com/r/java/comments/6coo2x/what_if_spring_handled_form_posts_automatically/) of my post there.

Below follows an explanation of these approaches, from my least favorite to most
favorite (and, more importantly, from worst to best approach).

## First Approach: Nasty Java, Automatic Marshaling

For, my first swipe at a solution, I wanted to have the least amount of
additional configuration code. Rather than deal with custom converters, I
wanted to hook into the existing `WebDataBinder`. To do this, I needed to
break some Java syntax conventions. In order to keep the primary code "clean",
I created a superclass for the sole purpose of properly materializing our
Object in the controller.

I give you `AbstractFormSlackSlashCommand`:

{% highlight java linenos %}
// workaround for customize x-www-form-urlencoded
public abstract class AbstractFormSlackSlashCommand {

    public void setTeam_id(String teamId) {
        setTeamId(teamId);
    }

    public void setTeam_domain(String teamDomain) {
        setTeamDomain(teamDomain);
    }

    public void setChannel_id(String channelId) {
        setChannelId(channelId);
    }

    public void setChannel_name(String channelName) {
        setChannelName(channelName);
    }

    public void setUser_id(String userId) {
        setUserId(userId);
    }

    public void setUser_name(String userName) {
        setUserName(userName);
    }

    public void setResponse_url(String responseUrl) {
        setResponseUrl(responseUrl);
    }

    abstract void setTeamId(String teamId);
    abstract void setTeamDomain(String teamDomain);
    abstract void setChannelId(String channelId);
    abstract void setChannelName(String channelName);
    abstract void setUserId(String userId);
    abstract void setUserName(String userName);
    abstract void setResponseUrl(String responseUrl);
}
{% endhighlight %}

The `FormSlackSlashCommand` class is the same as it was, except that now it
extends `AbstractFormSlackSlashCommand`:

{% highlight java linenos %}
public class FormSlackSlashCommand extends AbstractFormSlackSlashCommand {
  ...
}
{% endhighlight %}

Now, the `WebDataBinder` can handle incoming form parameters such as `team_id`
thanks to the `setTeam_id` method.

Aside from a lot of boilerplate code, `AbstractFormSlackSlashCommand` has the
drawback of violating Java method naming conventions.

On to the next approach!

## Second Approach: Custom HandlerMethodArgumentResolver

[u/sazzer](https://www.reddit.com/user/sazzer) posted the excellent suggestion
of creating an `HandleMethodArgumentResolver` to materialize the Object in the
controller. I hadn't heard or read about a `HandleMethodArgumentResolver`
prior to this and it turned out to be a solid approach.

Here's the updated controller:

{% highlight java linenos %}
public @ResponseBody SlackSlashCommand slack(SlackSlashCommand slackSlashCommand) {
    log.info("slackSlashCommand: {}", slackSlashCommand);

    return slackSlashCommand;
}
{% endhighlight %}

Notice that there's no `@ModelAttribute` annotation. That's because we don't
want the `WebDataBinder` to materialize the `SlackSlashCommand`. We'll have
a custom `HandlerMethodArgumentResolver` do that for us.

Note: In order to exercise all of the approaches in the
[slack-slash-command-example](https://github.com/dogeared/slack-slash-command-example),
the `SlackSlashCommand` class is basically the same as the
`FormSlackSlashCommand` class, except that it does not extend
`AbstractFormSlackSlashCommand`.

Here's the `HandleMethodArgumentResolver`:

{% highlight java linenos %}
public class SlackSlashCommandMethodArgumentResolver implements HandlerMethodArgumentResolver {

    @Override
    public boolean supportsParameter(MethodParameter methodParameter) {
        return methodParameter.getParameterType().equals(SlackSlashCommand.class);
    }

    @Override
    public Object resolveArgument(
        MethodParameter methodParameter, ModelAndViewContainer modelAndViewContainer,
        NativeWebRequest nativeWebRequest, WebDataBinderFactory webDataBinderFactory
    ) throws Exception {

        SlackSlashCommand ret = new SlackSlashCommand();

        ret.setChannelId(nativeWebRequest.getParameter("channel_id"));
        ret.setChannelName(nativeWebRequest.getParameter("channel_name"));
        ret.setCommand(nativeWebRequest.getParameter("command"));
        ret.setResponseUrl(nativeWebRequest.getParameter("response_url"));
        ret.setTeamDomain(nativeWebRequest.getParameter("team_domain"));
        ret.setTeamId(nativeWebRequest.getParameter("team_id"));
        ret.setText(nativeWebRequest.getParameter("text"));
        ret.setToken(nativeWebRequest.getParameter("token"));
        ret.setUserId(nativeWebRequest.getParameter("user_id"));
        ret.setUserName(nativeWebRequest.getParameter("user_name"));

        return ret;
    }

    private boolean isNotSet(String value) {
        return value == null;
    }
}
{% endhighlight %}

The `supportsParameter` method determines whether or not the resolver will be
used to return a particular Object, in this case, a `SlackSlashCommand`.

The `resolveArgument` method uses the `NativeWebRequest` to obtain all of the
form parameters and create a `SlackSlashCommand` object.

The last piece of the puzzle is to ensure that Spring Boot uses this resolver.
That's handled in a configuration:

{% highlight java linenos %}
@Configuration
public class SlackSlashCommandMethodArgumentResolverConfig extends WebMvcConfigurerAdapter {

    @Override
    public void addArgumentResolvers(List<HandlerMethodArgumentResolver> argumentResolvers) {
        argumentResolvers.add(new SlackSlashCommandMethodArgumentResolver());
    }
}
{% endhighlight %}

This is a solid approach. However, there's a lot of manual labor involved in
materializing the `SlackSlashCommand` Object. Also, it's a little more idiomatic
for a Spring Boot controller to have the `@ModelAttribute` or `@RequestBody`
annotation on an Object that's part of the request.

[u/sazzer](https://www.reddit.com/user/sazzer) also suggested using reflection
to easily convert the form parameters into setter methods on the object.

That led to my last and favorite approach.

## Third Approach: Custom HttpMessageConverter

This approach has the least amount of boilerplate code and reuses existing
components available to Spring Boot.

As I said toward the beginning of the post, there are a bunch of built in
`HttpMessageConverter` classes. The most common one is the
`MappingJackson2HttpMessageConverter`, which handles JSON.

As you would expect of Spring Boot, it's easy to create a custom
`HttpMessageConverter`.

{% highlight java linenos %}
public class SlackSlashCommandConverter extends AbstractHttpMessageConverter<SlackSlashCommand> {

    // no need to reinvent the wheel for parsing the query string
    private static final FormHttpMessageConverter formHttpMessageConverter = new FormHttpMessageConverter();
    private static final ObjectMapper mapper = new ObjectMapper();

    @Override
    protected boolean supports(Class<?> clazz) {
        return (SlackSlashCommand.class == clazz);
    }

    @Override
    protected SlackSlashCommand readInternal(Class<? extends SlackSlashCommand> clazz, HttpInputMessage inputMessage) throws IOException, HttpMessageNotReadableException {
        Map<String, String> vals = formHttpMessageConverter.read(null, inputMessage).toSingleValueMap();

        return mapper.convertValue(vals, SlackSlashCommand.class);
    }

    @Override
    protected void writeInternal(SlackSlashCommand slackSlashCommand, HttpOutputMessage outputMessage) throws IOException, HttpMessageNotWritableException {

    }
}
{% endhighlight %}

The `readInternal` method takes an `HttpInputMessage` as one if its parameters.
This is essentially a raw `InputStream` of the incoming form parameters.

At first, I thought I would have to manually read the stream and parse the
query string. Gross! Then, I remembered that the existing
`FormHttpMessageConverter` takes an incoming form and resolves it to a
`MultiValueMap`.

Based on the Slack Slash Command API, I also knew that all I needed was a single
value map. Line 14 of the above code takes care of that.

At this point, I could manually create the `SlackSlashCommand` object, like I
did in the `SlackSlashCommandMethodArgumentResolver`. However, the available
Jackson `ObjectMapper` is all ready to take a `Map` and materialize it to an
Object for us. As a bonus, it will make use of the `@JsonProperty` annotations
found in the `SlackSlashCommand` class. Whoa - that's a lot of power in that
one line on line 16!

Just two more bits tie this approach into a neat bow.

First, the controller method now needs to use the `@RequestBody` annotation so
that the collection of `HttpMessageConverter`s can be inspected and used.

{% highlight java linenos %}
public @ResponseBody SlackSlashCommand slack(@RequestBody SlackSlashCommand slackSlashCommand) {
    log.info("slackSlashCommand: {}", slackSlashCommand);

    return slackSlashCommand;
}
{% endhighlight %}

Now, that looks like a proper Spring boot controller method!

We do need to register the custom `HttpMessageConverter` with Spring Boot. This
is accomplished with a configuration as before:

{% highlight java linenos %}
@Configuration
public class SlackSlashCommandConverterConfig extends WebMvcConfigurerAdapter {

    @Override
    public void configureMessageConverters(List<HttpMessageConverter<?>> converters) {
        SlackSlashCommandConverter slackSlashCommandConverter = new SlackSlashCommandConverter();
        MediaType mediaType = new MediaType("application","x-www-form-urlencoded", Charset.forName("UTF-8"));
        slackSlashCommandConverter.setSupportedMediaTypes(Arrays.asList(mediaType));
        converters.add(slackSlashCommandConverter);
        super.configureMessageConverters(converters);
    }
}
{% endhighlight %}

This instantiates our `SlackSlashCommandConverter`, ensures that it supports
`x-www-form-urlencoded` requests and adds it to the list of
`HttpMessageConverter`s.

The only drawback of this approach is that the `@RequestBody` annotation is
typically used for non-form based requests. However, I think that the
`consumes = MediaType.APPLICATION_FORM_URLENCODED_VALUE` attribute makes it
clear what's going on.

## The Best Approach of All

I hope you've enjoyed our trek down workaround lane.

If [https://jira.spring.io/browse/SPR-13433](https://jira.spring.io/browse/SPR-13433)
is implemented (Did I mention about voting on it?), then none of the above
approaches would be necessary. You'd simply annotate your POJO in a similar way
to `@JsonProperty` to indicate how to map incoming form params to its fields.

In your controller, you would simply use the `@ModelAttribute` annotation and
you'd be done.

That would truly make Form handling as civilized as JSON handling already is in
Spring Boot.

All of the code mentioned in this post can be found in the
[slack-slash-command-example](https://github.com/dogeared/slack-slash-command-example)
github repo.
