---
title: "Introducing PSync"
url: "http://engineering.flipboard.com//2015/09/psync"
date: "Tue, 08 Sep 2015 00:00:00 +0000"
author: "https://github.com/hzsweers (Zac Sweers)"
feed_url: "https://engineering.flipboard.com/feed.xml"
---
<p>Here on the Android team at Flipboard, we have a lot of settings for users to adjust their experience. If you throw in internal settings, we have about 100 total preferences to manage. This is <em>a lot</em> of boilerplate to maintain, because preferences in Android have no built-in synchronization (unlike Resources). Our <code class="highlighter-rouge"><table class="rouge-table"><tbody><tr><td class="rouge-gutter gl"><pre class="lineno">1
</pre></td><td class="rouge-code"><pre>PreferenceFragment</pre></td></tr></tbody></table></code> class has a couple hundred lines of boilerplate fields at the top where we keep in-code mirrors to these preference values. This design is tedious, brittle, and requires a lot of overhead to keep in sync with XML. We developed a Gradle plugin called PSync to solve this problem.
<!--break--></p>

<p>PSync is an Android-specific Gradle plugin that generates Java representations of XML preferences. These Java classes can then be used directly in your code. The generated code is very much inspired by how <code class="highlighter-rouge"><table class="rouge-table"><tbody><tr><td class="rouge-gutter gl"><pre class="lineno">1
</pre></td><td class="rouge-code"><pre>R.java</pre></td></tr></tbody></table></code> works for resources, and should feel familiar to developers. It’s easy to use, has some simple configurations for fine-tuning your generated code, and is ready to drop into both library and application projects.</p>

<p>A full overview can be found on the <a href="https://github.com/Flipboard/psync">GitHub README</a>, but here’s a quick preview:</p>

<p>Say you have a preference:</p>

<div class="highlighter-rouge"><div class="highlight"><pre class="highlight"><code><table class="rouge-table"><tbody><tr><td class="rouge-gutter gl"><pre class="lineno">1
2
3
4
</pre></td><td class="rouge-code"><pre>&lt;CheckboxPreference
    android:key="show_images"
    android:defaultValue="true"
    /&gt;
</pre></td></tr></tbody></table></code></pre></div></div>

<p>You can reference it like this:</p>

<div class="highlighter-rouge"><div class="highlight"><pre class="highlight"><code><table class="rouge-table"><tbody><tr><td class="rouge-gutter gl"><pre class="lineno">1
2
3
4
5
6
7
8
9
</pre></td><td class="rouge-code"><pre>String theKey = P.showImages.key;
boolean current = P.showImages.get();
P.showImages.put(false).apply();

// If you use Rx-Preferences
P.showImages.rx().asObservable().omgDoRxStuff!

// If you need it
boolean theDefault = P.showImages.defaultValue();
</pre></td></tr></tbody></table></code></pre></div></div>

<p>The generated Java code looks like so:</p>

<div class="highlighter-rouge"><div class="highlight"><pre class="highlight"><code><table class="rouge-table"><tbody><tr><td class="rouge-gutter gl"><pre class="lineno">1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
</pre></td><td class="rouge-code"><pre>public final class P {
    public static final class showImages {
        public static final String key = "show_images";

        public static final boolean defaultValue() {
            return true;
        }

        public static final boolean get() {
            return PREFERENCES.getBoolean(key, defaultValue());
        }

        public static final SharedPreferences.Editor put(final boolean val) {
            return PREFERENCES.edit().putBoolean(key, val);
        }

        public static final Preference&lt;Boolean&gt; rx() {
            return RX_SHARED_PREFERENCES.getBoolean(key);
        }
    }
}
</pre></td></tr></tbody></table></code></pre></div></div>

<p>It’s robust, fully tested, and should be ready for use with standard XML preferences. The process of writing tests for a plugin like PSync is worth its own blog post or talk, so look forward to that coming up. Future plans for this project include adding support for mixing in your own non-XML preferences, and supporting the full range of the SharedPreferences API.</p>

<p>We hope you find it useful, and welcome contributions and feedback on the repo: <a href="https://github.com/Flipboard/psync">https://github.com/Flipboard/psync</a></p>

<p>If you like open source and working on cool projects, we are looking for talented Android engineers to join our team here in Palo Alto, California! <a href="http://hire.jobvite.com/m?31pnnhwW">Apply here</a></p>
