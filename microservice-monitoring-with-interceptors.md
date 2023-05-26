Once you have some microservices in the cloud, you want to have visibility into how they’re doing. Some things you want to monitor include:

- How many requests each microservice is getting
- How many requests result in an error, and what type of error they raise
- The latency on each request
- Exception logs so you can debug later

You’ll learn about a few ways of doing this in the sections below.

### Why Not Decorators

One way you could do this, and the most natural to Python developers, is to add a decorator to each microservice endpoint. However, in this case, there are several downsides to using decorators:

- Developers of new microservices have to remember to add them to each method.
- If you have a lot of monitoring, then you might end up with a stack of decorators.
- If you have a stack of decorators, then developers may stack them in the wrong order.
- You could consolidate all your monitoring into a single decorator, but then it could get messy.

This stack of decorators is what you want to avoid:
```
class RecommendationService(recommendations_pb2_grpc.RecommendationsServicer):
    @catch_and_log_exceptions
    @log_request_counts
    @log_latency
    def Recommend(self, request, context):
        ...
```
Having this stack of decorators on every method is ugly and repetitive, and it violates the DRY programming principle: don’t repeat yourself. Decorators are also a challenge to write, especially if they accept arguments.

### Interceptors
There an alternative approach to using decorators that you’ll pursue in this tutorial: gRPC has an interceptor concept that provides functionality similar to a decorator but in a cleaner way.

#### Implementing Interceptors
- https://grpc.github.io/grpc/python/grpc.html#service-side-interceptor
- https://pypi.org/project/grpc-interceptor/
- Unfortunately, the Python implementation of gRPC has a fairly complex API for interceptors. This is because it’s incredibly flexible. However, there’s a grpc-interceptor package to simplify them. For full disclosure
```
from grpc_interceptor import ServerInterceptor


class ErrorLogger(ServerInterceptor):

    def intercept(self, method, request, context, method_name):

        try:

            return method(request, context)

        except Exception as e:

            self.log_error(e)

            raise


    def log_error(self, e: Exception) -> None:

        # ...
```
This will call `log_error()` whenever an unhandled exception in your microservice is called. You could implement this by, for example, logging exceptions to Sentry so you get alerts and debugging info when they happen.

To use this interceptor, you would pass it to grpc.server() like this:
```
interceptors = [ErrorLogger()]
server = grpc.server(futures.ThreadPoolExecutor(max_workers=10),
                     interceptors=interceptors)
```
With this code, every request to and response from your Python microservice will go through your interceptor, so you can count how many requests and errors it gets.

... (continue on) resource: https://realpython.com/python-microservices-grpc/
