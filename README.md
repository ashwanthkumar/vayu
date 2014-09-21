# Vayu

Vayu is a (toy) Scala only RPC system. I am going to document the design I have in mind for this system. If you have any feedback, please free to create an issue and lets discuss :)

## Notes
- Use [scala pickling](https://github.com/scala/pickling) for SerDe
- Build a Netty based Server & Client module (for fast async access)
- Need to provide a Transport protocol (take insipiration from Thrift) for server / client access.

### Macros based Service definitions
May be we can use Macros for interface definitions. Something like
```
server("in.ashwanthkumar.vayu.EchoServer") {
	def echo(input: String) = input
}
```
This should be easier for Server Implementation. How can I wrap the client code? 

### Taking thrift insipiration
Take the inspiration from [Thrift IDL](https://thrift.apache.org/docs/idl) - [here](https://git-wip-us.apache.org/repos/asf?p=thrift.git;a=blob_plain;f=contrib/fb303/if/fb303.thrift;hb=HEAD), [here](https://git-wip-us.apache.org/repos/asf?p=thrift.git;a=blob_plain;f=test/ThriftTest.thrift;hb=HEAD) and [here](http://svn.apache.org/viewvc/cassandra/trunk/interface/cassandra.thrift?view=co). We can write a simple Thrift service definition parser and generate the server / client stubs. 

### Thoughts on the Transport
We might need to use a similar strategy like Thrift to use a constant int value against the service methods. In which case the messages we send / recieve will be similar to the one shown below.
```
// client usage would be something similar to
val service = new FooServiceClient("host", "port")
val result: Future[Int] = service.foo("abcd")

...

// server usage would be something similar to 
class FooService {
	....

	/*
		All service methods return a Future for a reason. With flow like constructs, 
		the services will be completely async. This should give considerable 
		performance boost.

		Take inspiration from Play - https://www.playframework.com/documentation/2.1.1/ScalaAsync
	*/
	def foo(input: String): Future[Int] = flow {
		input.length
	}

	....
}

val server = new FooService()
server.startSpin("host", "port")
```

#### Transport Messages

For the sample server / client implementations above, I can either use case classes like these for message passing.

```
// ATTEMPT - 1
// For a service method foo: (String) => (Int)

case class VayuTransportRequest(methodToInvoke: Int, argsToService: List[Any])
case class VayuTransportResponse[T](methodInvoked: Int, returnValue: T)
```

Or may be pre-create transport message for each service method

```
// ATTEMPT - 2
// For a service method foo: (String) => (Int)

case class VayuTransportRequest_foo(methodToInvoke: Int, argsToService: String)
case class VayuTransportResponse_foo(methodInvoked: Int, returnValue: Int)
```

TODO - Need to understand how Thrift protocol is implemented. Also fiddle around how Avro works.
