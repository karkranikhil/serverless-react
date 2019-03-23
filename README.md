# Serverless react

### What is Serverless

Few issues with the Traditional way of handling the server

* We are charged for keeping the server up even when we are not serving out any requests.
* We are responsible for uptime and maintenance of the server and all its resources.
* We are also responsible for applying the appropriate security updates to the server.
* As our usage scales we need to manage scaling up our server as well. And as a result manage scaling it down when we don’t have as much usage.

For smaller companies and individual developers this can be a lot to handle. This ends up distracting from the more important job that we have; building and maintaining the actual application. As developers we’ve been looking for a solution to these problems and this is where serverless comes in.

```
Serverless is an execution model where the cloud provider (AWS, Azure, or Google Cloud) is responsible for executing a piece of code by dynamically allocating the resources.And only charging for the amount of resources used to run the code. The code is typically run inside stateless containers that can be triggered by a variety of events including http requests, database events, queuing services, monitoring alerts, file uploads, scheduled events (cron jobs), etc. 
```

* The code that is sent to the cloud provider for execution is usually in the form of a function. Hence serverless is sometimes referred to as “Functions as a Service” or “FaaS”. 
* In the serverless world you are typically required to adopt a more microservice based architecture

### Stateless functions
functions are typically run inside secure (almost) stateless containers. This means that you won’t be able to run code in your application server that executes long after an event has completed or uses a prior execution context to serve a request. You have to effectively assume that your function is invoked anew every single time.

* <b>Cold start</b> - functions are run inside a container that is brought up on demand to respond to an event, there is some latency associated with it. This is referred to as a Cold Start. 
* <b>Warm start</b> - container might be kept around for a little while after your function has completed execution. If another event is triggered during this time it responds far more quickly and this is typically known as a Warm Start.

### What is AWS Lambda?
Its is a serverless computing service provided by AWS. 

Lambda supports the following runtimes.

* Node.js: v8.10 and v6.10
* Java 8
* Python: 3.6 and 2.7
* .NET Core: 1.0.1 and 2.0
* Go 1.x
* Ruby 2.5
* Rust

Each function runs inside a container with a 64-bit Amazon Linux AMI. And the execution environment has:

* <b>Memory: 128MB - 3008MB, in 64 MB increments</b> (As you increase the memory, the CPU is increased as well.)
* <b>Ephemeral disk space: 512MB</b>  - ephemeral disk space is available in the form of the /tmp directory. You can only use this space for temporary storage since subsequent invocations will not have access to this. 
* <b>Max execution duration: 900 seconds</b>  - The execution duration means that your Lambda function can run for a maximum of 900 seconds or 15 minutes. This means that Lambda isn’t meant for long running processes.
* <b>Compressed package size: 50MB</b>  - The package size refers to all your code necessary to run your function. This includes any dependencies (node_modules/ directory in case of Node.js) that your function might import. T
* <b>Uncompressed package size: 250MB</b> - There is a limit of 250MB on the uncompressed package and a 50MB limit once it has been compressed. 

### Lambda Function

```
exports.myHandler = function (event, context, callback){
    callback(error, result)
}
```

* myHandler - name of function
* event - contains all the information about the event that triggered this Lambda.
* context - It contains info about the runtime our Lambda function is executing in. 
* Callback - After we do all the work inside our Lambda function we call the callback function with result or error and AWS will respond to the HTTP request with it.

### Packaging functions
Lambda functions need to be packaged and sent to AWS. This is usually a process of compressing the function and all its dependencies and uploading it to a S3 bucket. And letting AWS know that you want to use this package when a specific event takes place. 

### Execution model
* The container (and the resources used by it) that runs our function is managed completely by AWS. It is brought up when an event takes place and is turned off if it is not being used. If additional requests are made while the original event is being served, a new container is brought up to serve a request. This means that if we are undergoing a usage spike, the cloud provider simply creates multiple instances of the container with our function to serve those requests.

* This has some interesting implications.

        *  Firstly, our functions are effectively stateless. 
        * Secondly, each request (or event) is served by a single instance of a Lambda function. 

* This means that you are not going to be handling concurrent requests in your code. AWS brings up a container whenever there is a new request. It does make some optimizations here. It will hang on to the container for a few minutes (5 - 15mins depending on the load) so it can respond to subsequent requests without a cold start.

### Stateless Functions & Example 
every time your Lambda function is triggered by an event it is invoked in a completely new environment. You don’t have access to the execution context of the previous event.

<b>Note ** </b> the actual Lambda function is invoked only once per container instantiation. Recall that our functions are run inside containers. So when a function is first invoked, all the code in our handler function gets executed and the handler function gets invoked. If the container is still available for subsequent requests, your function will get invoked and not the code around it.


** Example 
the createNewDbConnection method below is called once per container instantiation and not every time the Lambda function is invoked. The myHandler function on the other hand is called on every invocation.

```
var dbConnection = createNewDbConnection();

exports.myHandler = function(event, context, callback) {
  var result = dbConnection.makeQuery();
  callback(null, result);
};

```
### Pricing
Lambda functions are billed only for the time it takes to execute your function. And it is calculated from the time it begins executing till when it returns or terminates. It is rounded up to the nearest 100ms.