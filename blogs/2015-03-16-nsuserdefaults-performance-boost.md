---
title: "NSUserDefaults Performance Boost"
url: "http://engineering.flipboard.com//2015/03/nsuserdefaults-performance"
date: "Mon, 16 Mar 2015 00:00:00 +0000"
author: "https://twitter.com/timonus (Tim Johnsen)"
feed_url: "https://engineering.flipboard.com/feed.xml"
---
<p>Since iOS 8 was released we’ve noticed some sluggishness when using <a href="http://flipboard.com">Flipboard</a> in the simulator. When taking a trace with Instruments in normal use we noticed a significant amount of time was being spent in <code class="highlighter-rouge"><table class="rouge-table"><tbody><tr><td class="rouge-gutter gl"><pre class="lineno">1
</pre></td><td class="rouge-code"><pre>CFPreferences</pre></td></tr></tbody></table></code>.<!--break--></p>

<center>
    <div class="row">
        <img src="/assets/nsuserdefaults-performance/before.png" />
    </div>
</center>

<p>On Twitter, an Apple engineer acknowledged that there were some changes in iOS 8 that added something called <code class="highlighter-rouge"><table class="rouge-table"><tbody><tr><td class="rouge-gutter gl"><pre class="lineno">1
</pre></td><td class="rouge-code"><pre>cfprefsd</pre></td></tr></tbody></table></code>, and that emulating that in the simulator required synchronous reads from disk. This seemed to be the bottleneck we were encountering.</p>

<center>
	<blockquote class="twitter-tweet" lang="en"><p><a href="https://twitter.com/timonus">@timonus</a> unfortunately cfprefsd doesn’t currently exist in the simulator, and emulating its behavior requires synchronize disk IO</p>&mdash; David Smith (@Catfish_Man) <a href="https://twitter.com/Catfish_Man/status/557977673461268481">January 21, 2015</a></blockquote>
	<blockquote class="twitter-tweet" lang="en"><p><a href="https://twitter.com/timonus">@timonus</a> it’s on my list to fix, but it’s a large effort and device took priority</p>&mdash; David Smith (@Catfish_Man) <a href="https://twitter.com/Catfish_Man/status/557977746215673856">January 21, 2015</a></blockquote>
	
</center>

<p>About a month ago we were talking as a team about how slow we felt our app had become to debug in the simulator, so we decided to try to do something to speed it up. Our approach was to introduce a man-in-the-middle write-through cache to <code class="highlighter-rouge"><table class="rouge-table"><tbody><tr><td class="rouge-gutter gl"><pre class="lineno">1
</pre></td><td class="rouge-code"><pre>NSUserDefaults</pre></td></tr></tbody></table></code> in memory. We do so by swizzling out all of <code class="highlighter-rouge"><table class="rouge-table"><tbody><tr><td class="rouge-gutter gl"><pre class="lineno">1
</pre></td><td class="rouge-code"><pre>NSUserDefaults</pre></td></tr></tbody></table></code>’ setters and getters and adding a per-instance <code class="highlighter-rouge"><table class="rouge-table"><tbody><tr><td class="rouge-gutter gl"><pre class="lineno">1
</pre></td><td class="rouge-code"><pre>NSMutableDictionary</pre></td></tr></tbody></table></code> to cache values. We avoid compiling this for device builds using <code class="highlighter-rouge"><table class="rouge-table"><tbody><tr><td class="rouge-gutter gl"><pre class="lineno">1
</pre></td><td class="rouge-code"><pre>TARGET_IPHONE_SIMULATOR</pre></td></tr></tbody></table></code> because there aren’t such performance issues on device. The increase in performance is dramatic, where we were once spending 80% of our time in <code class="highlighter-rouge"><table class="rouge-table"><tbody><tr><td class="rouge-gutter gl"><pre class="lineno">1
</pre></td><td class="rouge-code"><pre>CFPreferences</pre></td></tr></tbody></table></code> we’re now spending 1%.</p>

<center>
    <div class="row">
        <img src="/assets/nsuserdefaults-performance/after.png" />
    </div>
</center>

<p>If you’re seeing performance issues related to <code class="highlighter-rouge"><table class="rouge-table"><tbody><tr><td class="rouge-gutter gl"><pre class="lineno">1
</pre></td><td class="rouge-code"><pre>NSUserDefaults</pre></td></tr></tbody></table></code> in the simulator I recommend trying this out, it’s <a href="https://github.com/Flipboard/NSUserDefaultsSimulatorPerformanceBoost">open source and available for download on GitHub</a>. To get started using it all you need to do is include it in your project.</p>

<p><em>Please note that use of this category may cause side effects when debugging extensions. This is <a href="https://github.com/Flipboard/NSUserDefaultsSimulatorPerformanceBoost/blob/master/README.md#extensions">documented</a> in the GitHub project.</em></p>
