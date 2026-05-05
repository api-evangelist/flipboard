---
title: "Presenting BottomSheet"
url: "http://engineering.flipboard.com//2015/06/bottomsheet"
date: "Thu, 04 Jun 2015 00:00:00 +0000"
author: "https://github.com/emilsjolander (Emil Sjölander)"
feed_url: "https://engineering.flipboard.com/feed.xml"
---
<p>We are happy to introduce <a href="http://github.com/flipboard/bottomsheet">BottomSheet</a> a new Open Source Android UI Component!</p>

<p>At Flipboard, we love building visually stunning and highly interactive UIs. When building these UIs, we tend to build them as fairly stand alone components. This makes it very easy for our developers to implement a similar interaction model and aesthetic across the whole product while working in parallel. BottomSheet is a UI component we developed to facilitate a new interaction model for saving an article to one of your magazines (otherwise known as “The Flip UI”).
<!--break--></p>

<p>The result of our efforts is a design you can now see in the current version of Flipboard.</p>

<center>
    <div class="row">
        <img src="/assets/bottomsheet/flipui.gif" />
    </div>
</center>

<p>BottomSheet fits perfectly with Google’s Material Design aesthetic as well, matching their Bottom Sheet <a href="http://www.google.com/design/spec/components/bottom-sheets.html">specification</a>. We think this component is great for displaying many types of views, from a simple intent picker to something as complex as the example shown from Flipboard. In its simplest form, BottomSheet can be used to show any view in a sort of modal UI that’s presented from the bottom of the screen can be interactively dismissed. At Flipboard, we think it’s very important that things feel responsive, and having interactive animations is an important part of this.</p>

<p>Apart from providing a component for presenting views in a BottomSheet, we have also have a module for common views developers might like to present in a BottomSheet. The first such component we’re releasing is the <code class="highlighter-rouge"><table class="rouge-table"><tbody><tr><td class="rouge-gutter gl"><pre class="lineno">1
</pre></td><td class="rouge-code"><pre>IntentPickerSheetView</pre></td></tr></tbody></table></code>. This sheet view is initialized with an intent and will show a grid of activities that can handle the intent. It works very similarly to Android’s built in IntentChooser, and looks very similar to the picker used in Lollipop and above. There are a couple of ways in which our implementation improves upon the system intent chooser:</p>
<ul>
  <li>It can be interactively dismissed, making it feel much more responsive.</li>
  <li>It adds an easy-to-use API for filtering and sorting the activities shown. Ever wanted to filter out the Bluetooth activity when adding sharing functionality to your app? With the <code class="highlighter-rouge"><table class="rouge-table"><tbody><tr><td class="rouge-gutter gl"><pre class="lineno">1
</pre></td><td class="rouge-code"><pre>IntentPickerSheetView</pre></td></tr></tbody></table></code>, that’s simple!</li>
</ul>

<center>
    <div class="row">
        <img src="/assets/bottomsheet/intentpicker.gif" />
    </div>
</center>

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
22
</pre></td><td class="rouge-code"><pre>IntentPickerSheetView intentPickerSheet = new IntentPickerSheetView(MainActivity.this, shareIntent, "Share with...", new IntentPickerSheetView.OnIntentPickedListener() {
    @Override
    public void onIntentPicked(Intent intent) {
        bottomSheet.dismissSheet();
        startActivity(intent);
    }
});
// Filter out built in sharing options such as bluetooth and beam.
intentPickerSheet.setFilter(new IntentPickerSheetView.Filter() {
    @Override
    public boolean include(IntentPickerSheetView.ActvityInfo info) {
        return !info.componentName.getPackageName().startsWith("com.android");
    }
});
// Sort activities in reverse order for no good reason
intentPickerSheet.setSortMethod(new Comparator&lt;IntentPickerSheetView.ActvityInfo&gt;() {
    @Override
    public int compare(IntentPickerSheetView.ActvityInfo lhs, IntentPickerSheetView.ActvityInfo rhs) {
        return rhs.label.compareTo(lhs.label);
    }
});
bottomSheet.showWithSheetView(intentPickerSheet);
</pre></td></tr></tbody></table></code></pre></div></div>

<p>We would love for you to join us on <a href="http://github.com/flipboard/bottomsheet">GitHub</a> to collaborate in adding many more of these common components!</p>

<p>And before I send you off to scour the interwebs for cat GIFs to display in your new BottomSheets, I want to take this opportunity to advertise that we are looking for talented Android engineers to join our team here in Palo Alto, California. <a href="http://hire.jobvite.com/m?31pnnhwW">Apply here</a></p>
