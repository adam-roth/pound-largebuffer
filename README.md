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

Pound does provide the `--with-maxbuf` configuration option, and various references mentioned that this could be used to increase the maximum URI length supported by pound.  Unfortunately, this option also controls the size of a number of other, _completely unrelated_ buffers, and **all of them are allocated on the stack!**  

So, increase the `--with-maxbuf` option to a size large enough to allow a reasonably sized image to be uploaded using a GET request, and pound is liable immediately exceed the allotted stack-size and then segfault.  That's a problem!  And since there are a massive number of buffers that are sized according to the `--with-maxbuf` option (including ones used to store a string representation of an IP address...which never needs a 4K buffer, let alone anything larger), increasing the system stack-size isn't really a solution as your memory footprint would baloon to several times what it actually required.

This project attempts to solve that problem, by providing an option that modifies just those buffers that are actually relevant to parsing an HTTP request.  It also brings in dynamic memory allocation and more intelligent scanning of the inbound request to help ensure that the maximum buffer size is only allocated if the request actually calls for it.

All because I need to send large files to a JSONP webservice through pound.  That's why this project was created.

**_Why should I use this project?_**<br />
Use this project if you've got a corner-case that requires you to submit large GET requests to a webserver that is running behind pound, or any similarly esoteric use-case that benefits from being able to give pound a very large URI or request header without having it bail out with a [code 414](http://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html).

**_Why should I NOT use this project?_**<br />
Don't use this project in memory-constrained situations.  Also don't need it if you care about supporting clients that use the 'chunked' transfer encoding.  Or if you don't require arbitrarily large URI and header sizes in your application.  

Unless you need to do something that cannot be done using the official version of pound, you should absolutely stick with the official version of pound.

**_What are your license terms?_**<br />
Pound is licensed under the GPL.  Not what I'd personally pick, but it is what it is.


### Acknowledgements

This project was supported and made possible by Inspection Toolbox.  Find out more at http://www.inspectiontoolbox.com/.
