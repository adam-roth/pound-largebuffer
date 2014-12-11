### pound-largebuffer

A more flexible load-balancer for those complicated modern corner-cases. 


### About

This project modifies the pound reverse-proxy and load-balancer (v2.6) to support configuration of arbitrarily large maximum request/header sizes.  This can be useful if you need to support some esoteric use-cases, such as submitting encoded file data to a JSONP endpoint using a GET request (because POST requests are not usable with JSONP). 

### Building

This project configures and builds the same as pound.  Thus a good resource to consult is the [official reference documentation](http://www.apsis.ch/pound/).

In addition to the standard options when configuring your build, this project adds an additional `--with-maxrequest=nnn` configuration option that can be used to specifiy the maximum allowed request/URI size.  For example, to allow URI's of up to 12MiB in length, do:

`./configure --with-maxrequest=12582912`

Note that this option operates independently of pound's `--with-maxbuf=nnn` option, which controls the maximum length of an individual request header (excluding the URI), as well as the size used when allocating a number of completely unrelated internal buffers.   Thus if you want to allow large requests and also large header sizes, you should increase the value of both parameters.

The default `--with-maxrequest` and `--with-maxbuf` values are both 4096 bytes when not explicitly specified.


### Usage

For detailed setup and usage instructions, please consult the [official reference documentation](http://www.apsis.ch/pound/).
    

### Limitations

There are a number of important limitations that should be considered when using this code in its current state:

1.  Only the HTTP methods described in [RFC2616](http://www.w3.org/Protocols/rfc2616/rfc2616-sec9.html) are supported/tested.  Other methods may fail, particularly if they rely upon content sent in the request body.

2.  The 'chunked' transfer encoding is not currently supported.  This is a fairly substantial omission, as support for this encoding is a mandatory part of the HTTP 1.1 spec.  However, my primary use-case does not require 'chunked' transfers, although support for this will likely be restored as soon as possible.

3.  If using the pre-existing `--with-maxbuf` configuration option to increase the maximum allowed header size, be aware that this project does not modify how the associated buffers are handled.  The main consequence of this is that setting a `--with-maxbuf` value that is too large can and will cause pound to immediately segfault.  You should keep the value of this parameter as low as possible, or otherwise increase the stack-size settings on your machine to better support larger values

4.  If you use the `--with-maxrequest` configuration option to increase the maximum allowed request/URI size, pound's memory usage will naturally increase somewhat (though not as much as you might think, as in most places the need to preallocate a maximally-sized buffer is deferred until it's known that the request will actually require that much memory).  Please ensure that your server has sufficient RAM to accommodate the increased memory load, keeping in mind that pound is multithreaded and may be processing a large number of requests concurrently.


### FAQ

**_Why create this modification to pound?_**<br />
Because I have an application that provides a JSONP webservice API, and a requirement to support the upload of relatively large files (images mainly) as part of an API call.  HTML5's [File API](http://www.html5rocks.com/en/tutorials/file/dndfiles/) is sufficient to get most of the way there.  Or all the way there if my server nodes weren't running behind pound.  

But I am running pound, and with JSONP the only HTTP method you can use to submit a request is GET.  This requires that the image data be shipped as a URL parameter (base64-encoded), and naturally results in some very long request URI's.  Much longer than pound's default URI length of 4096 bytes.  

Pound does provide the `--wi`

**_Why should I use this library?_**<br />
Use this library if you've got a Core Data app and better things to do with your time than worry about whether or not you are accessing your data "correctly".  This code will give you a data layer that just works, freeing you up to focus on things that actually matter, like building your app.

**_Why should I NOT use this library?_**<br />
Don't use this library if your app is performance-limited by its data layer.  Such cases are probably rare with iOS apps, but you should be aware that synchronizing every access to the data layer is relatively costly and could conceivably degrade performance in a data-intensive app.  

You may also want to avoid this code if your app has a particularly large or complex data model.  I'm not saying that it won't work for you (and I can't even point to any specific reason why it _might_ not work for you), but so far it has only been tested and validated in simpler apps with not more than moderately complex data models.  If you need to be absolutely sure that things are going to work, stick to the old-fashioned way.

Also, if you like setting up `NSOperationQueue`'s and `dispatch_async()` calls are the only sunshine in your day then you probably don't want to use this code.

**_What are your license terms?_**<br />
License terms?  Really???  Look, this is public code in a public repository.  I put it here knowing that.  It would be absurd for me to assert that I have any right to set conditions on how people may use this code after I have knowingly made it publicly available to anyone who might stumble across it.  I know some other people like to do that, but let's try not to stoop to their level, shall we?

So my license terms are simple.  Use this code if you want, otherwise don't.  That's it.  


### Miscellaneous

Another excellent project that makes Core Data _much_ easier to work with can be found here:

https://github.com/halostatue/coredata-easyfetch
