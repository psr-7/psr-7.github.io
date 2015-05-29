---
title: PSR7
layout: default
---

# Unofficial PSR7 FAQ

#### What is PSR7?
Attempt to reinvent and harmonize ``Request`` and ``Response`` proposed by [PHP Framework Interop Group](http://www.php-fig.org/).

#### What is PSR7 not?
PSR7 is not a common denominator of existing HTTP libraries.

#### Where can I find official PSR7 documentation?
* [Interfaces Definition](http://www.php-fig.org/psr/psr-7/)
* [Meta Document](http://www.php-fig.org/psr/psr-7/meta/)

#### Why bother?
HTTP is the very heart of every web project out there and nearly every php-fig member voted in favor of this PHP Standard Recommendation.

#### Still, can I ignore it?
At some point you inevitable stumble over PSR7 as nearly all major PHP projects will have some sort of PSR7 support.

#### Can PSR7 get deprecated and replaced in the near-term?
Considering the long development time of PSR7 it's very unlikely. Furthermore projects need even more time to readopt and release new versions. It's of good use to engage and learn about PSR7 now.

#### Can I use project X and not care about PSR7 details?
No, PSR7 is invasive. If you work with PSR7 compliant ``HTTP Messages`` you need to know the philosophy and ideas behind PSR7 design decisions.

#### Where can I find those?
In the [Meta Document](http://www.php-fig.org/psr/psr-7/meta/).


---------------------------------------------------------------------------------


### Functional Programming

#### What functional programming concepts are used?
Changes without side-effects. Every change is done to a new object preserving state of old instance.

#### How do I change data in PSR7 objects?
With mutator methods defined on every interface.

#### What are mutator methods?
Every method changing something. e.g.:

		->withHost($host)
		->replaceFooWithBar($foo, $bar)
		->setName($name)

#### Can I omit mutator methods?
No, there is no segregation between read and write in PSR7. You have to implement them even on readonly objects. In fact readonly objects are implicitly forbidden. You could do no-op or throw exceptions but this violates [LSP](https://en.wikipedia.org/wiki/Liskov_substitution_principle).

#### What mutator methods are defined by PSR7?
Only methods without side-effects are defined by PSR7.

		public function withHeader($name, $value);
		public function withoutHeader($name);

#### Can I implement my own mutator methods?
Yes.


#### How to implement side-effect free mutators?
Create new instance and reassign data from old object. Cloning old object is easy and efficient way to do so:

		public function withHost($host)
		{
			$new = clone $this;
			$new->host = $host;
			return $new;
		}

some alternatives:

		$new = clone $this;
		$new = new self(...)
		$new = new static(...)
		$new = new MyClass(...)

#### Can I use PHP's pass-by-reference-semantics to propagate object changes back to caller?
No, changes may not affect instance in caller. You should ``return`` the newly created object.
You can implement additional mutators non-violating interface contract. e.g. ->setHost($host);

#### What to do for changes to take effect?

Change your code flow. PSR7 is invading you. :)

		function changeFoo($request, $response) {
			$request = $request->withUri(new Uri('https://example.org/'));
			$response = $response->withStatus(200, 'OK');
			
			// return changes to outer scope
			return array($request, $response);
		}
		list ($req2, $res2) = changeFoo($req, $res);
		// $req, $res are unchanged
		// $req2, $res2 are fooChanged



#### How to implement PSR7 support in existing codebase?
Transform your objects on project boundaries to some PSR7 implementation and vice versa. Bridge/Wrapper/Proxy/you-name-it [look here](https://github.com/symfony/psr-http-message-bridge) and [here](https://github.com/Sam-Burns/psr7-symfony-httpfoundation)




---------------------------------------------------------------------------------

### Middleware

#### What is Middleware?
Buzzword for "passing messages around" layers in contrast to "calling methods".
Actually nor [MOM](https://en.wikipedia.org/wiki/Message-oriented_middleware) nor any other type of middleware (application, communication, content) is specific or in any way related to PSR7.
See [Middleware 1](https://en.wikipedia.org/wiki/Middleware)
[Middleware 2](https://en.wikipedia.org/wiki/Middleware_(distributed_applications))
and [EDA](https://en.wikipedia.org/wiki/Event-driven_architecture).

Connecting and passing data between PSR7 aware projects is called interoperability.


#### What message types are defined by PSR7?
PSR7 specify generic "Request", "Response", "Message" and "ServerRequest".



---------------------------------------------------------------------------------

### Value Objects

#### What are Value Objects?
Objects without explicit identity. e.g.: 20 ikea coffee cups

#### Is this my coffee or yours?
Yes.

#### Does PSR7 define identity?
Yes, "Uniform Resource Identifier" (URI) is defined in UriInterface and used (only) in RequestInterface.

#### Why only PSR7 Requests have an identity?
I don't know.

#### Can URI be used to identify Responses and other message types?
Yes, always implement ->getURI() on all message types. URIs are solid base to identify, validate and compare http resources.
All of HTTP is unthinkable without URIs. ;)




---------------------------------------------------------------------------------

### URIs

#### What can I expect from instance of UriInterface?
not NULL, empty string

#### Can I expect any info to be available in instance of UriInterface?
No. There is no guarantee any data is available.

#### Default Scheme?
not NULL, empty string

#### Default port?
NULL (not integer!)

#### What is the default path?
empty string (because of bugs in arbitrary frontcontrollers losing context)

#### How to achieve a solid Uri implementation?
Fill in sane defaults for missing values in constructor.

		default scheme: http
		default host: 0.0.0.0
		default port: 80
		default path: /
		empty string as default for all other parts

minimum valid URI:

		"http://0.0.0.0/" === (new Uri())->__toString()

#### Can and should request-targets be derived from data in UriInterface?
Yes. PSR7 does not define any methods to do so, however. Implement those methods in Uri definition:

		public function originForm(); // absolute-path [ "?" query ]
		public function absoluteForm(); // alias for ->__toString()
		public function authorityForm(); // alias for ->getAuthority()
		public function asteriskForm(); // always: *

usage:

		$request = $request->withRequestTarget($uri->originForm())

For implementation details see: [5.3.  Request Target](http://tools.ietf.org/html/rfc7230#section-5.3)


#### Valid values for Host?
		127.0.0.1
		[::1]
		localhost
		a
		foo.bar




---------------------------------------------------------------------------------

### Streams

#### What to expect from getBody() on messages without body content (HEAD, 204)?
no-op instance of StreamInterface (php://memory)





---------------------------------------------------------------------------------

### Interfaces

#### What interfaces are defined by PSR7?
* RequestInterface (+ ServerRequestInterface)
* ResponseInterface
* MessageInterface
* StreamInterface
* UriInterface
* UploadedFileInterface

#### Can I implement only one of those interfaces?
Yes. Here is dependency graph:


		UriInterface
				methods:	16
				dependencies:	-
		StreamInterface
				methods:	15
				dependencies:	-
		MessageInterface
				methods:	11
				dependencies:	StreamInterface
		ResponseInterface
				methods:	3
				dependencies:	MessageInterface, StreamInterface, UriInterface
		RequestInterface
				methods:	6
				dependencies:	MessageInterface, StreamInterface, UriInterface
		ServerRequestInterface
				methods:	13
				dependencies:	MessageInterface, RequestInterface, StreamInterface, UriInterface
		UploadedFileInterface
				methods:	6
				dependencies:	StreamInterface


#### method list

* interface MessageInterface

		* public function getProtocolVersion();
		* public function withProtocolVersion($version);
		* public function getHeaders();
		* public function hasHeader($name);
		* public function getHeader($name);
		* public function getHeaderLine($name);
		* public function withHeader($name, $value);
		* public function withAddedHeader($name, $value);
		* public function withoutHeader($name);
		* public function withBody(StreamInterface $body);
		* public function getBody();


* interface RequestInterface extends MessageInterface

		* public function getRequestTarget();
		* public function withRequestTarget($requestTarget);
		* public function getMethod();
		* public function withMethod($method);
		* public function getUri();
		* public function withUri(UriInterface $uri, $preserveHost = false);


* interface ServerRequestInterface extends RequestInterface

		* public function getServerParams();
		* public function getCookieParams();
		* public function withCookieParams(array $cookies);
		* public function getQueryParams();
		* public function withQueryParams(array $query);
		* public function getUploadedFiles();
		* public function withUploadedFiles(array $uploadedFiles);
		* public function getParsedBody();
		* public function withParsedBody($data);
		* public function getAttributes();
		* public function getAttribute($name, $default = null);
		* public function withAttribute($name, $value);
		* public function withoutAttribute($name);


* interface ResponseInterface extends MessageInterface

		* public function getStatusCode();
		* public function withStatus($code, $reasonPhrase = '');
		* public function getReasonPhrase();


* interface StreamInterface

		* public function __toString();
		* public function close();
		* public function detach();
		* public function getSize();
		* public function tell();
		* public function eof();
		* public function isSeekable();
		* public function seek($offset, $whence = SEEK_SET);
		* public function rewind();
		* public function isWritable();
		* public function write($string);
		* public function isReadable();
		* public function read($length);
		* public function getContents();
		* public function getMetadata($key = null);


* interface UriInterface

		* public function getScheme();
		* public function getAuthority();
		* public function getUserInfo();
		* public function getHost();
		* public function getPort();
		* public function getPath();
		* public function getQuery();
		* public function getFragment();
		* public function withScheme($scheme);
		* public function withUserInfo($user, $password = null);
		* public function withHost($host);
		* public function withPort($port);
		* public function withPath($path);
		* public function withQuery($query);
		* public function withFragment($fragment);
		* public function __toString();


* interface UploadedFileInterface

		* public function getStream();
		* public function moveTo($targetPath);
		* public function getSize();
		* public function getError();
		* public function getClientFilename();
		* public function getClientMediaType();

