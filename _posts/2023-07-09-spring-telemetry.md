## Spring Library To Potentially Log and Store Every API request?

### Disclaimer

The intention of the article is to see whether a solution which remotely resolves the problem statement exists or not. This article will also highlight the places which were looked at and will finally draw a conclusion based on the short lived and poor research that was done. 

I myself am not in a position to give a definite answer to the question in title.  The agenda is to have a productive discussion and if there was a need for such a library, realize what would be the best option to go for.


### Problem Statement

#### Overview

Create a library which gets coupled with your spring application and logs every API request that is being sent to it. This library will operate as a separate Spring instance, just like Spring Config Server or Spring Eureka Server.  I am hoping to get something as simple as this:


```
@EnableTracingServer
public class MySpringApplication {
    // Spring boiler plate
}

```

Then we shall have these in our application.properties
```
tracing.server.property1 = value
tracing.server.property2 = value
tracing.server.property3 = value

```

Ideally, this is whether the developer's headache should end. (But most probably it won't)

#### Addition Features

Although the overview covers the minimum requirement to what we want. But without these addition features, our problem will perhaps be reduced to a few lines on the [API interceptors](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/servlet/HandlerInterceptor.html) or filters . It would be a pretty bad solution, but its still a solution!

Here, we will highlight some features that are required to make such a library fit for production. I will list them down according to a priority that I personally see fit. 

1. Needs to be stateless and distributed.
2. Option to exclude and include particular APIs
3. Multiple trace storage options. Primarily JDBC compatible databases, which can be easily integrated using spring libraries.
4. Give user the ability to include, exclude different API metrics from logging. For eg: user should have to ability to not store the response or request payload. Maybe someone only wants to store the API response time etc.
5. Encrypt specific API metrics. For eg: You don’t want the basic authentication headers to be stored as plain text.

#### The Possibilities!

With the above features in order, we can perhaps link this data to  [Grafana](https://grafana.com/) or [Metabase](https://www.metabase.com/) for telemetry, alarms and tracking other metrics. 

Or, if the data storage format is standardizedb like [Otel](https://opentelemetry.io/docs/), people can contribute with their own Grafana like applications for this. The possibilities are endless!

Basically, we can have our own inhouse, open-source, cloud native [cloud watch](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/WhatIsCloudWatch.html). 


## A **very** high level architecture.

![Omnia.drawio.png](/assets/images/Omnia.drawio.png)

The client will be sending API requests to your Spring application. Your spring application in turn, finishes processing the request and logs the API request using our Spring API logger. 

The metadata might contain processing time, the response status code and other performance related metrics. 

### Concerns

There are some obvious concerns to a library like this. 

1. Every API request to your application comes with an additional network call to the logger. This can perhaps be reduced with some level of queueing + batching.
2. These additional network calls can potentially become very expensive. 
3. Not taking proper care while logging may result in some massive security breaches. 

### The Silver lining

The above concerns are precisely why we need to have a granular configuration model. Users might not and perhaps should not log **every** API request that they get. But the library still should provide the ability to do so. Users can modulate their cost and performance by picking and choosing what they wish to monitor. 

### What can we use?

#### OpenTelemetry and others

There’s a [good open telemetry demo with open telemetry and Jaeger](https://www.baeldung.com/spring-boot-opentelemetry-setup) for what we want to do. This will provide a dashboard which displays relevant metrics for all your data. Howeber, I didn't see a database linkage here.

**Question**: Does OpenTelemetry get linked with a persistent database or everything is added onto memory?  
**Answer**: It seems there's [no straight forward way to do so](https://github.com/open-telemetry/opentelemetry-java-instrumentation/discussions/5573). The methods have been explained in the GitHub issue. However, you still ought to code your way out of this problem. I would also encourage you to look at [Promscale](https://github.com/timescale/promscale?ref=timescale.com#readme).  
It seems that with this, we might just get what we want. However, we still need to manage multiple dependencies here.

#### Conclusion?

All other solutions that I found are more or less dependent on OpenTelemetry and are basically an extension of Otel. Perhaps this is how it should be since Otel is supposed to be a standard. But I am still not happy with where we are currently. To get what we want, we need to integrate multiple dependencies in our application. All these individual libraries will already be a pain to setup. Then we will be managing these guys separately as well.  


Is our problem statement too specific, or do you think there's something far better out there which can give us exactly what we want with far less complexity. Meanwhile, I will try to dig more and will perhaps try to keep this updated. 

