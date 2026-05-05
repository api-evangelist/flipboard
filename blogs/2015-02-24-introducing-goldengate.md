---
title: "Introducing GoldenGate"
url: "http://engineering.flipboard.com//2015/02/golden-gate"
date: "Tue, 24 Feb 2015 00:00:00 +0000"
author: "https://twitter.com/emilsjolander (Emil Sjölander)"
feed_url: "https://engineering.flipboard.com/feed.xml"
---
<p>You might not know it, but both Flipboard for iOS and Flipboard for Android make heavy use of web views. We use web views so we can ensure consistent designs for our partners’ articles across all platforms. Communication between native code and the JavaScript code running in a web view is something that is both tedious to implement as well as very bug prone as it’s mostly just string concatenation. Today we are releasing a library to make this task easier when developing Android applications which use web views.<!--break--></p>

<p><a href="http://www.github.com/flipboard/goldengate" title="GoldenGate source on GitHub">GoldenGate</a> is a annotation processing library which generates java wrappers around your JavaScript code. An annotation processing library is a piece of code which runs when you compile your code and has the ability to generate new java classes. This means that GoldenGate can ensure at compile time that you are sending the correct types into the JavaScript functions you define in your bridge. If you’re interested in knowing more about how to write your own, <a href="http://hannesdorfmann.com/annotation-processing/annotationprocessing101" title="Annotation processing blog post">this blog post</a> is a good intro.</p>

<p>Let’s get into a quick example to really show the power of the library! First of all a quick example of how to currently call a JavaScript function in a WebView.</p>

<div class="highlighter-rouge"><div class="highlight"><pre class="highlight"><code><table class="rouge-table"><tbody><tr><td class="rouge-gutter gl"><pre class="lineno">1
</pre></td><td class="rouge-code"><pre>webview.loadUrl("javascript:alert(" + myString + ");");
</pre></td></tr></tbody></table></code></pre></div></div>

<p>You should clearly see that a lot can go wrong here which the compiler won’t catch. For example you could misspell <code class="highlighter-rouge"><table class="rouge-table"><tbody><tr><td class="rouge-gutter gl"><pre class="lineno">1
</pre></td><td class="rouge-code"><pre>alert</pre></td></tr></tbody></table></code> or maybe <code class="highlighter-rouge"><table class="rouge-table"><tbody><tr><td class="rouge-gutter gl"><pre class="lineno">1
</pre></td><td class="rouge-code"><pre>myString</pre></td></tr></tbody></table></code> isn’t a string at all.</p>

<p>GoldenGate allows for the compiler to have your back. The same code as above written with GoldenGate looks like this.</p>

<div class="highlighter-rouge"><div class="highlight"><pre class="highlight"><code><table class="rouge-table"><tbody><tr><td class="rouge-gutter gl"><pre class="lineno">1
2
3
4
5
6
7
</pre></td><td class="rouge-code"><pre>@Bridge
class JavaScript {
	void alert(String message);
}

JavaScriptBridge bridge = new JavaScript(webview);
bridge.alert(myString);
</pre></td></tr></tbody></table></code></pre></div></div>

<p>We started by defining an interface which describes the JavaScript methods we want to call from java. This interface is annotated with <code class="highlighter-rouge"><table class="rouge-table"><tbody><tr><td class="rouge-gutter gl"><pre class="lineno">1
</pre></td><td class="rouge-code"><pre>@Bridge</pre></td></tr></tbody></table></code> which is where the magic happens. This will generate a class named <code class="highlighter-rouge"><table class="rouge-table"><tbody><tr><td class="rouge-gutter gl"><pre class="lineno">1
</pre></td><td class="rouge-code"><pre>JavaScriptBridge</pre></td></tr></tbody></table></code> in this case which implements all the methods defined by the interface.</p>

<p>There are a couple more options for more advanced usage that we won’t cover in this blog post but they are well documented over on <a href="http://www.github.com/flipboard/goldengate" title="GoldenGate source on GitHub">Github</a>.</p>

<p>At Flipboard we love the type safety GoldenGate gives us and we think it can help more developers using both native and web technology in their apps. Head over to the <a href="http://www.github.com/flipboard/goldengate" title="GoldenGate source on GitHub">Github repository</a> to check out the code. Pull requests are very welcome!</p>
