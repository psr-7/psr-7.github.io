---
title: unofficial PSR7 FAQ
layout: post
---

# Unofficial PSR7 FAQ for PHP Devs

## What is PSR7?
Interfaces for all the things around ``HTTP Request`` and ``HTTP Response`` proposed by [PHP Framework Interop Group](http://www.php-fig.org/).

## What interfaces?
* RequestInterface (+ ServerRequestInterface)
* ResponseInterface
* MessageInterface
* StreamInterface
* UriInterface
* UploadedFileInterface

## Where can i find official PSR7 documentation?
* [Interfaces Definition](http://www.php-fig.org/psr/psr-7/)
* [Meta Document](http://www.php-fig.org/psr/psr-7/meta/)

## Why bother?
HTTP is the very heart of every web project out there and nearly every php-fig member voted in favor of this PHP Standard Recommendation.

## Still, can i ignore it?
At some point you inevitable stumble over PSR7 as nearly all major PHP projects will have some sort of PSR7 support.

## Can PSR7 get deprecated and replaced in the near-term?
Considering the long development time of PSR7 it's very unlikely. Furthermore projects need even more time to readopt and release new versions. It's of good use to engage and learn about PSR7 now.

## Can i use project X and not care about PSR7 details?
No, PSR7 is invasive. If you work with PSR7 compliant ``HTTP Messages`` you need to know the philosophy and ideas behind PSR7 design decisions.

## Where can i find those?
In the [Meta Document](http://www.php-fig.org/psr/psr-7/meta/).

## Why PSR7 is not a common denominator of existing HTTP libraries?
I don't know.

## What functional programming concepts are used?
Changes without side-effects. Every change is done to a new object preserving state of old instance.

## Are PSR7 objects ``immutable``?
  * ``private`` access level: no
  * ``public`` access level: yes

## How do i change data in PSR7 objects?
With mutator methods defined on every interface.

## What are mutator methods?
Every method changing state. e.g.:
  * ->withHost($host)
  * ->replaceFooWithBar($foo, $bar)
  * ->setName($name)

## What mutator methods are allowed in PSR7?
Only methods without side-effects are allowed.

## How to implement side-effect free mutators?
  * $new = clone $this;
  * $new = new self(...)
  * $new = new static(...)
  * $new = new MyClass(...)

    public function withHost($host)
    {
        $new = clone $this;
        $new->host = $host;
        return $new;
    }

## Can i omit mutatot methods?
No, there is no segregation between read and write in PSR7.

