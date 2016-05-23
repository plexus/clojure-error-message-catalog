## TypeError: can't convert symbol to string

When ClojureScript prints out a value, for example a result on the REPL, instead of a proper representation you might get this

```
#object[TypeError TypeError: can't convert symbol to string]
```

This happens for example when outputting a a React virtual dom element.

### Cause: js/Symbol can't be printed

The [`Symbol`](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Symbol) primitive 
is a relatively recent addition to JavaScript, and ClojureScript is not able to serialize it correctly. When you output a `js/Symbol`, or any object containing a `js/Symbol`, you will get this error.

``` clojure-repl
user> #js {:example (js/Symbol "i-am-a-symbol")}
;;=> #object[TypeError TypeError: can't convert symbol to string]
```

This is being worked on (see [http://dev.clojure.org/jira/browse/CLJS-1628](CLJS-1628)), but because of browser 
incompatibilities there is no immediate solution forthcoming.

### Solution 

You can extend the `IPrintWithWriter` protocol to define how symbols should be rendered. This snippets works 
on current versions of browsers, but will not work when using shims, or on Safara 6.0.x

``` clojure
(extend-type js/Symbol
  IPrintWithWriter
  (-pr-writer [obj writer _opts]
    (write-all writer (.toString obj))))
```

Result:

``` clojure-repl
user> #js {:example (js/Symbol "i-am-a-symbol")}
;;=> #js {:example Symbol(i-am-a-symbol)}
```
