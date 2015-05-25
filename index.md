---
title: unofficial PSR7 FAQ
layout: post
---

# Unofficial PSR7 FAQ for PHP Devs

#### What is PSR7?
Interfaces for all the things around ``HTTP Request`` and ``HTTP Response`` proposed by [PHP Framework Interop Group](http://www.php-fig.org/).

#### What interfaces?
* RequestInterface (+ ServerRequestInterface)
* ResponseInterface
* MessageInterface
* StreamInterface
* UriInterface
* UploadedFileInterface

#### Where can i find official PSR7 documentation?
* [Interfaces Definition](http://www.php-fig.org/psr/psr-7/)
* [Meta Document](http://www.php-fig.org/psr/psr-7/meta/)

#### Why bother?
HTTP is the very heart of every web project out there and nearly every php-fig member voted in favor of this PHP Standard Recommendation.

#### Still, can i ignore it?
At some point you inevitable stumble over PSR7 as nearly all major PHP projects will have some sort of PSR7 support.

#### Can PSR7 get deprecated and replaced in the near-term?
Considering the long development time of PSR7 it's very unlikely. Furthermore projects need even more time to readopt and release new versions. It's of good use to engage and learn about PSR7 now.

#### Can i use project X and not care about PSR7 details?
No, PSR7 is invasive. If you work with PSR7 compliant ``HTTP Messages`` you need to know the philosophy and ideas behind PSR7 design decisions.

#### Where can i find those?
In the [Meta Document](http://www.php-fig.org/psr/psr-7/meta/).

#### Why PSR7 is not a common denominator of existing HTTP libraries?
I don't know.

#### What functional programming concepts are used?
Changes without side-effects. Every change is done to a new object preserving state of old instance.

#### Are PSR7 objects ``immutable``?
 * ``private`` access level: no
 * ``public`` access level: yes

#### How do i change data in PSR7 objects?
With mutator methods defined on every interface.

#### What are mutator methods?
Every method changing something. e.g.:

 * ->withHost($host)
 * ->replaceFooWithBar($foo, $bar)
 * ->setName($name)

#### What mutator methods are allowed in PSR7?
Only methods without side-effects are allowed.

#### How to implement side-effect free mutators?

```
    public function withHost($host)
    {
        $new = clone $this;
        $new->host = $host;
        return $new;
    }
```

some alternatives:

```
    $new = clone $this;
    $new = new self(...)
    $new = new static(...)
    $new = new MyClass(...)
```

#### Can i omit mutator methods?
No, there is no segregation between read and write in PSR7. You have to implement them even on readonly objects. In fact readonly objects are implicitly forbidden. You could do no-op or throw exceptions but this violates [LSP](https://en.wikipedia.org/wiki/Liskov_substitution_principle).

#### What to do for changes to take effect?
Change your code flow. PSR7 is invading you. :)

Code samples can be found in [Interfaces Definition](http://www.php-fig.org/psr/psr-7/) and [Meta Document](http://www.php-fig.org/psr/psr-7/meta/).

#### Can i use PHP's pass-by-reference-semantics to propagate object changes back to caller?
No, changes will never be propagated back to caller. You need to return the newly created object.

#### How to do this in old codebase?
No. Stop. Don't. [look here](https://github.com/symfony/psr-http-message-bridge) and [here](https://github.com/Sam-Burns/psr7-symfony-httpfoundation)

#### Can i implement only one of those interfaces?
Yes. Here is dependency graph:

* MessageInterface: 11 methods
    * dependencies: StreamInterface
* RequestInterface: 6 methods
    * dependencies: MessageInterface, StreamInterface, UriInterface
* ServerRequestInterface: 13 methods
    * dependencies: MessageInterface, RequestInterface, StreamInterface, UriInterface
* ResponseInterface: 3 methods
    * dependencies: MessageInterface, StreamInterface, UriInterface
* StreamInterface: 15 methods
    * dependencies: none
* UriInterface: 16 methods
    * dependencies: none
* UploadedFileInterface: 6 methods
    * dependencies: StreamInterface



