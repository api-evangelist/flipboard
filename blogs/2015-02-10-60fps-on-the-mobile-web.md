---
title: "60fps on the mobile web"
url: "http://engineering.flipboard.com//2015/02/mobile-web"
date: "Tue, 10 Feb 2015 00:00:00 +0000"
author: "https://twitter.com/bapjuseyo (Michael Johnston)"
feed_url: "https://engineering.flipboard.com/feed.xml"
---
<p class="deck">
Flipboard launched during the dawn of the smartphone and tablet as a mobile-first experience, allowing us to rethink content layout principles from the web for a more elegant user experience on a variety of touchscreen form factors.
</p>

<p>Now we’re coming full circle and bringing Flipboard to the web. Much of what we do at Flipboard has value independent of what device it’s consumed on: curating the best stories from all the topics, sources, and people that you care about most. Bringing our service to the web was always a logical extension.</p>

<p>As we began to tackle the project, we knew we wanted to adapt our thinking from our mobile experience to try and elevate content layout and interaction on the web. We wanted to match the polish and performance of our native apps<!--break-->, but in a way that felt true to the browser.</p>

<p>Early on, after testing numerous prototypes, we decided our web experience should scroll. Our mobile apps are known for their book-like pagination metaphor, something that feels intuitive on a touch screen, but for a variety of reasons, scrolling feels most natural on the web.</p>

<p>In order to <a href="http://www.html5rocks.com/en/tutorials/speed/scrolling/">optimize scrolling performance</a>, we knew that we needed to keep paint times below 16ms and limit reflows and repaints. This is especially important during animations. To avoid painting during animations there are two properties you can safely animate: CSS transform and opacity. But that really limits your options.</p>

<p>What if you want to animate the width of an element?</p>

<p style="text-align: center;">
  <img alt="Follow button animation" src="/assets/mobileweb/follow_btn.gif" style="border: 1px solid #ddd;" />
</p>

<p>How about a frame-by-frame scrolling animation?</p>

<p style="text-align: center;">
  <img alt="Clipping animation" src="/assets/mobileweb/topbar.gif" style="border: 1px solid #ddd;" />
</p>

<p>(Notice in the above image that the icons at the top transition from white to black. These are 2 separate elements overlaid on each other whose bounding boxes are clipped depending on the content beneath.)</p>

<p>These types of animations have always suffered from <a href="http://jankfree.org/">jank</a> on the web, particularly on mobile devices, for one simple reason:</p>

<p><strong>The DOM is too slow.</strong></p>

<p>It’s not just slow, it’s really slow. If you touch the DOM in any way during an animation you’ve already blown through your 16ms frame budget.</p>

<h2>
  <a class="anchor-link" href="#EnterCanvas" name="EnterCanvas">
    Enter &lt;canvas&gt;
  </a>
</h2>

<p>Most modern mobile devices have hardware-accelerated canvas, so why couldn’t we take advantage of this? <a href="http://chrome.angrybirds.com/">HTML5 games</a> certainly do.  But could we really develop an application user interface in canvas?</p>

<h3>
  <a class="anchor-link" href="#DrawingModes" name="DrawingModes">
    Immediate mode vs. retained mode
  </a>
</h3>

<p>Canvas is an immediate mode drawing API, meaning that the drawing surface retains no information about the objects drawn into it. This is in opposition to retained mode, which is a declarative API that maintains a hierarchy of objects drawn into it.</p>

<p>The advantage to retained mode APIs is that they are typically easier to construct complex scenes with, e.g. the DOM for your application. It often comes with a performance cost though, as additional memory is required to hold the scene and updating the scene can be slow.</p>

<p>Canvas benefits from the immediate mode approach by allowing drawing commands to be sent directly to the GPU. But using it to build user interfaces requires a higher level abstraction to be productive. For instance something as simple as drawing one element on top of another can be problematic when resources load asynchronously, such as drawing text on top of an image. In HTML this is easily achieved with the ordering of elements or z-index in CSS.</p>

<h2>
  <a class="anchor-link" href="#CanvasUI" name="CanvasUI">
    Building a UI in &lt;canvas&gt;
  </a>
</h2>

<p>Canvas lacks many of the abilities taken for granted in HTML + CSS.</p>

<h3>
  <a class="anchor-link" href="#CanvasText" name="CanvasText">
    Text
  </a>
</h3>

<p>There is a single API for drawing text: <code class="highlighter-rouge"><table class="rouge-table"><tbody><tr><td class="rouge-gutter gl"><pre class="lineno">1
</pre></td><td class="rouge-code"><pre>fillText(text, x, y [, maxWidth])</pre></td></tr></tbody></table></code>. This function accepts three arguments: the text string and x-y coordinates to begin drawing. But canvas can only draw a single line of text at a time. If you want text wrapping, you need to write your own function.</p>

<h3>
  <a class="anchor-link" href="#CanvasImages" name="CanvasImages">
    Images
  </a>
</h3>

<p>To draw an image into a canvas you call <code class="highlighter-rouge"><table class="rouge-table"><tbody><tr><td class="rouge-gutter gl"><pre class="lineno">1
</pre></td><td class="rouge-code"><pre>drawImage()</pre></td></tr></tbody></table></code>. This is a variadic function where the more arguments you specify the more control you have over positioning and clipping. But canvas does not care if the image has loaded or not so make sure this is called only after the image load event.</p>

<h3>
  <a class="anchor-link" href="#CanvasOverlapping" name="CanvasOverlapping">
    Overlapping elements
  </a>
</h3>

<p>In HTML and CSS it’s easy to specify that one element should be rendered on top of another by using the order of the elements in the DOM or CSS z-index. But remember, canvas is an immediate mode drawing API. When elements overlap and either one of them needs to be redrawn, both have to be redrawn in the same order (or at least the dirtied parts).</p>

<h3>
  <a class="anchor-link" href="#CanvasFonts" name="CanvasFonts">
    Custom fonts
  </a>
</h3>

<p>Need to use a custom web font? The canvas text API does not care if a font has loaded or not. You need a way to know when a font has loaded, and redraw any regions that rely on that font. Fortunately, modern browsers have a <a href="http://dev.w3.org/csswg/css-font-loading/">promise-based API</a> for doing just that. Unfortunately, iOS WebKit (iOS 8 at the time of this writing) does not support it.</p>

<h3>
  <a class="anchor-link" href="#CanvasBenefits" name="CanvasBenefits">
    Benefits of &lt;canvas&gt;
  </a>
</h3>

<p>Given all these drawbacks, one might begin to question selecting the canvas approach over DOM. In the end, our decision was made simple by one simple truth:</p>

<p><em>You cannot build a 60fps scrolling list view with DOM.</em></p>

<p>Many (including us) have tried and failed. Scrollable elements are possible in pure HTML and CSS with <code class="highlighter-rouge"><table class="rouge-table"><tbody><tr><td class="rouge-gutter gl"><pre class="lineno">1
</pre></td><td class="rouge-code"><pre>overflow: scroll</pre></td></tr></tbody></table></code> (combined with <code class="highlighter-rouge"><table class="rouge-table"><tbody><tr><td class="rouge-gutter gl"><pre class="lineno">1
</pre></td><td class="rouge-code"><pre>-webkit-overflow-scrolling: touch</pre></td></tr></tbody></table></code> on iOS) but these do not give you frame-by-frame control over the scrolling animation and mobile browsers have a difficult time with long, complex content.</p>

<p>In order to build an infinitely scrolling list with reasonably complex content, we needed the equivalent of <a href="https://developer.apple.com/library/ios/documentation/UIKit/Reference/UITableView_Class/index.html">UITableView</a> for the web.</p>

<p>In contrast to the DOM, most devices today have hardware accelerated canvas implementations which send drawing commands directly to the GPU. This means we could render elements incredibly fast; we’re talking sub-millisecond range in many cases.</p>

<p>Canvas is also a very small API when compared to HTML + CSS, reducing the surface area for bugs or inconsistencies between browsers. There’s a reason there is no <a href="http://caniuse.com/">Can I Use?</a> equivalent for canvas.</p>

<h2>
  <a class="anchor-link" href="#FasterDOM" name="FasterDOM">
    A faster DOM abstraction
  </a>
</h2>

<p>As mentioned earlier, in order to be somewhat productive, we needed a higher level of abstraction than simply drawing rectangles, text and images in immediate mode. We built a very small abstraction that allows a developer to deal with a tree of nodes, rather than a strict sequence of drawing commands.</p>

<h3>
  <a class="anchor-link" href="#RenderLayer" name="RenderLayer">
    RenderLayer
  </a>
</h3>

<p>A RenderLayer is the base node by which other nodes build upon. Common properties such as top, left, width, height, backgroundColor and zIndex are expressed at this level. A RenderLayer is nothing more than a plain JavaScript object containing these properties and an array of children.</p>

<h3>
  <a class="anchor-link" href="#RenderImage" name="RenderImage">
    Image
  </a>
</h3>

<p>There are Image layers which have additional properties to specify the image URL and cropping information. You don’t have to worry about listening for the image load event, as the Image layer will do this for you and send a signal to the drawing engine that it needs to update.</p>

<h3>
  <a class="anchor-link" href="#RenderText" name="RenderText">
    Text
  </a>
</h3>

<p>Text layers have the ability to render multi-line truncated text, something which is incredibly expensive to do in DOM. Text layers also support custom font faces, and will do the work of updating when the font loads.</p>

<h3>
  <a class="anchor-link" href="#RenderComposition" name="RenderComposition">
    Composition
  </a>
</h3>

<p>These layers can be composed to build complex interfaces. Here is an example of a RenderLayer tree:</p>

<pre>
{
  frame: [0, 0, 320, 480],
  backgroundColor: '#fff',
  children: [
    {
      type: 'image',
      frame: [0, 0, 320, 200],
      imageUrl: 'http://lorempixel.com/360/420/cats/1/'
    },
    {
      type: 'text',
      frame: [10, 210, 300, 260],
      text: 'Lorem ipsum...',
      fontSize: 18,
      lineHeight: 24
    }
  ]
}
</pre>

<h3>
  <a class="anchor-link" href="#InvalidatingLayers" name="InvalidatingLayers">
    Invalidating layers
  </a>
</h3>

<p>When a layer needs to be redrawn, for instance after an image loads, it sends a signal to the drawing engine that its frame is dirty. Changes are batched using <code class="highlighter-rouge"><table class="rouge-table"><tbody><tr><td class="rouge-gutter gl"><pre class="lineno">1
</pre></td><td class="rouge-code"><pre>requestAnimationFrame</pre></td></tr></tbody></table></code> to avoid <a href="http://wilsonpage.co.uk/preventing-layout-thrashing/">layout thrashing</a> and in the next frame the canvas redraws.</p>

<h2>
  <a class="anchor-link" href="#Scrolling60fps" name="Scrolling60fps">
    Scrolling at 60fps
  </a>
</h2>

<p>Perhaps the one aspect of the web we take for granted the most is how a browser scrolls a web page. Browser vendors have gone to <a href="http://www.chromium.org/developers/design-documents/gpu-accelerated-compositing-in-chrome">great lengths</a> to improve scrolling performance.</p>

<p>It comes with a tradeoff though. In order to scroll at 60fps on mobile, browsers used to halt JavaScript execution during scrolling for fear of DOM modifications causing reflow. Recently, iOS and Android have exposed <code class="highlighter-rouge"><table class="rouge-table"><tbody><tr><td class="rouge-gutter gl"><pre class="lineno">1
</pre></td><td class="rouge-code"><pre>onscroll</pre></td></tr></tbody></table></code> events that work more like they do on desktop browsers but your mileage may vary if you are trying to keep DOM elements synchronized with the scroll position.</p>

<p>Luckily, browser vendors are aware of the problem. In particular, the Chrome team has been <a href="http://updates.html5rocks.com/2014/05/A-More-Compatible-Smoother-Touch">open</a> about its efforts to improve this situation on mobile.</p>

<p>Turning back to canvas, the short answer is you have to implement scrolling in JavaScript.</p>

<p>The first thing you need is a way to compute scrolling momentum. If you don’t want to do the math the folks at Zynga open sourced a <a href="https://github.com/zynga/scroller">pure logic scroller</a> that fits well with any layout approach.</p>

<p>The technique we use for scrolling uses a single canvas element. At each touch event, the current render tree is updated by translating each node by the current scroll offset. The entire render tree is then redrawn with the new frame coordinates.</p>

<p>This sounds like it would be incredibly slow, but there is an important optimization technique that can be used in canvas where the result of drawing operations can be cached in an off-screen canvas. The off-screen canvas can then be used to redraw that layer at a later time.</p>

<p>This technique can be used not just for image layers, but text and shapes as well. The two most expensive drawing operations are filling text and drawing images. But once these layers are drawn once, it is very fast to redraw them using an off-screen canvas.</p>

<p>In the demonstration below, each page of content is divided into 2 layers: an image layer and a text layer. The text layer contains multiple elements that are grouped together. At each frame in the scrolling animation, the 2 layers are redrawn using cached bitmaps.</p>

<p style="text-align: center;">
  <img alt="Scrolling timeline" src="/assets/mobileweb/scrolling.gif" style="border: 1px solid #ddd; width: 320px; height: auto;" />
</p>

<h3>
  <a class="anchor-link" href="#ObjectPooling" name="ObjectPooling">
    Object pooling
  </a>
</h3>

<p>During the course of scrolling through an infinite list of items, a significant number of RenderLayers must be set up and torn down. This can create a lot of garbage, which would halt the main thread when collected.</p>

<p>To avoid the amount of garbage created, RenderLayers and associated objects are aggressively pooled. This means only a relatively small number of layer objects are ever created. When a layer is no longer needed, it is released back into the pool where it can later be reused.</p>

<h3>
  <a class="anchor-link" href="#FastSnapshotting" name="FastSnapshotting">
    Fast snapshotting
  </a>
</h3>

<p>The ability to cache composite layers leads to another advantage: the ability to treat portions of rendered structures as a bitmap. Have you ever needed to take a snapshot of only part of a DOM structure? That’s incredibly fast and easy when you render that structure in canvas.</p>

<p>The UI for flipping an item into a magazine leverages this ability to perform a smooth transition from the timeline. The snapshot contains the entire item, minus the top and bottom chrome.</p>

<p style="text-align: center;">
  <img alt="Flipping an item into a magazine" src="/assets/mobileweb/flip_ui.gif" style="border: 1px solid #ddd;" />
</p>

<h2>
  <a class="anchor-link" href="#DeclarativeAPI" name="DeclarativeAPI">
    A declarative API
  </a>
</h2>

<p>We had the basic building blocks of an application now. However, imperatively constructing a tree of RenderLayers could be tedious. Wouldn’t it be nice to have a declarative API, similar to how the DOM worked?</p>

<h3>
  <a class="anchor-link" href="#React" name="React">
    React
  </a>
</h3>

<p>We are big fans of <a href="http://facebook.github.io/react/">React</a>. Its single directional data flow and declarative API have changed the way people build apps. The most compelling feature of React is the virtual DOM. The fact that it renders to HTML in a browser container is simply an implementation detail. The recent introduction of <a href="https://www.youtube.com/watch?v=KVZ-P-ZI6W4">React Native</a> proves this out.</p>

<p>What if we could bind our canvas layout engine to React components?</p>

<h2>
  <a class="anchor-link" href="#ReactCanvas" name="ReactCanvas">
    Introducing React Canvas
  </a>
</h2>

<p><a href="https://github.com/flipboard/react-canvas">React Canvas</a> adds the ability for React components to render to <code class="highlighter-rouge"><table class="rouge-table"><tbody><tr><td class="rouge-gutter gl"><pre class="lineno">1
</pre></td><td class="rouge-code"><pre>&lt;canvas&gt;</pre></td></tr></tbody></table></code> rather than DOM.</p>

<p>The first version of the canvas layout engine looked very much like imperative view code. If you’ve ever done DOM construction in JavaScript you’ve probably run across code like this:</p>

<pre>
// Create the parent layer
var root = RenderLayer.getPooled();
root.frame = [0, 0, 320, 480];

// Add an image
var image = RenderLayer.getPooled('image');
image.frame = [0, 0, 320, 200];
image.imageUrl = 'http://lorempixel.com/360/420/cats/1/';
root.addChild(image);

// Add some text
var label = RenderLayer.getPooled('text');
label.frame = [10, 210, 300, 260];
label.text = 'Lorem ipsum...';
label.fontSize = 18;
label.lineHeight = 24;
root.addChild(label);
</pre>

<p>Sure, this works but who wants to write code this way? In addition to being error-prone it’s difficult to visualize the rendered structure.</p>

<p>With React Canvas this becomes:</p>

<pre>
var MyComponent = React.createClass({
  render: function () {
    return (
      &lt;Group style={styles.group}&gt;
        &lt;Image style={styles.image} src='http://...' /&gt;
        &lt;Text style={styles.text}&gt;
          Lorem ipsum...
        &lt;/Text&gt;
      &lt;/Group&gt;
    );
  }
});

var styles = {
  group: {
    left: 0,
    top: 0,
    width: 320,
    height: 480
  },

  image: {
    left: 0,
    top: 0,
    width: 320,
    height: 200
  },

  text: {
    left: 10,
    top: 210,
    width: 300,
    height: 260,
    fontSize: 18,
    lineHeight: 24
  }
};
</pre>

<p>You may notice that everything appears to be absolutely positioned. That’s correct. Our canvas rendering engine was born out of the need to drive pixel-perfect layouts with multi-line ellipsized text. This cannot be done with conventional CSS, so an approach where everything is absolutely positioned fit well for us. However, this approach is not well-suited for all applications.</p>

<h3>
  <a class="anchor-link" href="#css-layout" name="css-layout">
    css-layout
  </a>
</h3>

<p>Facebook recently open sourced its <a href="https://github.com/facebook/css-layout">JavaScript implementation of CSS</a>. It supports a subset of CSS like margin, padding, position and most importantly flexbox.</p>

<p>Integrating css-layout into React Canvas was a matter of hours. Check out the <a href="https://github.com/flipboard/react-canvas/blob/master/examples/css-layout/app.js">example</a> to see how this changes the way components are styled.</p>

<h2>
  <a class="anchor-link" href="#DeclarativeInfiniteScrolling" name="DeclarativeInfiniteScrolling">
    Declarative infinite scrolling
  </a>
</h2>

<p>How do you create a <em>60fps</em> infinite, paginated scrolling list in React Canvas?</p>

<p>It turns out this is quite easy because of React’s diffing of the virtual DOM. In <code class="highlighter-rouge"><table class="rouge-table"><tbody><tr><td class="rouge-gutter gl"><pre class="lineno">1
</pre></td><td class="rouge-code"><pre>render()</pre></td></tr></tbody></table></code> only the currently visible elements are returned and React takes care of updating the virtual DOM tree as needed during scrolling.</p>

<pre>
var ListView = React.createClass({
  getInitialState: function () {
    return {
      scrollTop: 0
    };
  },

  render: function () {
    var items = this.getVisibleItemIndexes().map(this.renderItem);
    return (
      &lt;Group
        onTouchStart={this.handleTouchStart}
        onTouchMove={this.handleTouchMove}
        onTouchEnd={this.handleTouchEnd}
        onTouchCancel={this.handleTouchEnd}&gt;
        {items}
      &lt;/Group&gt;
    );
  },

  renderItem: function (itemIndex) {
    // Wrap each item in a &lt;Group&gt; which is translated up/down based on
    // the current scroll offset.
    var translateY = (itemIndex * itemHeight) - this.state.scrollTop;
    var style = { translateY: translateY };
    return (
      &lt;Group style={style} key={itemIndex}&gt;
        &lt;Item /&gt;
      &lt;/Group&gt;
    );
  },

  getVisibleItemIndexes: function () {
    // Compute the visible item indexes based on `this.state.scrollTop`.
  }
});
</pre>

<p>To hook up the scrolling, we use the <a href="https://github.com/zynga/scroller">Scroller</a> library to <code class="highlighter-rouge"><table class="rouge-table"><tbody><tr><td class="rouge-gutter gl"><pre class="lineno">1
</pre></td><td class="rouge-code"><pre>setState()</pre></td></tr></tbody></table></code> on our ListView component.</p>

<pre>
...

// Create the Scroller instance on mount.
componentDidMount: function () {
  this.scroller = new Scroller(this.handleScroll);
},

// This is called by the Scroller at each scroll event.
handleScroll: function (left, top) {
  this.setState({ scrollTop: top });
},

handleTouchStart: function (e) {
  this.scroller.doTouchStart(e.touches, e.timeStamp);
},

handleTouchMove: function (e) {
  e.preventDefault();
  this.scroller.doTouchMove(e.touches, e.timeStamp, e.scale);
},

handleTouchEnd: function (e) {
  this.scroller.doTouchEnd(e.timeStamp);
}

...
</pre>

<p>Though this is a simplified version it showcases some of React’s best qualities. Touch events are declaratively bound in render(). Each touchmove event is forwarded to the Scroller which computes the current scroll top offset. Each scroll event emitted from the Scroller updates the state of the ListView component, which renders only the currently visible items on screen. All of this happens in under 16ms because <a href="http://calendar.perfplanet.com/2013/diff/">React’s diffing algorithm is very fast</a>.</p>

<p>See the <a href="https://github.com/flipboard/react-canvas/blob/master/lib/ListView.js">ListView source code</a> for the complete implementation.</p>

<h2>
  <a class="anchor-link" href="#PracticalApplications" name="PracticalApplications">
    Practical applications
  </a>
</h2>

<p>React Canvas is not meant to completely replace the DOM. We utilize it in performance-critical rendering paths in our mobile web app, primarily the scrolling timeline view.</p>

<p>Where rendering performance is not a concern, DOM may be a better approach. In fact, it’s the only approach for certain elements such as input fields and audio/video.</p>

<p>In a sense, Flipboard for mobile web is a hybrid application. Rather than blending native and web technologies, it’s all web content. It mixes DOM-based UI with canvas rendering where appropriate.</p>

<h2>
  <a class="anchor-link" href="#Accessibility" name="Accessibility">
    A word on accessibility
  </a>
</h2>

<p>This area needs further exploration. Using fallback content (the canvas DOM sub-tree) should allow screen readers such as VoiceOver to interact with the content. We’ve seen mixed results with the devices we’ve tested. Additionally there is a standard for <a href="http://www.w3.org/TR/2010/WD-2dcontext-20100304/#dom-context-2d-drawfocusring">focus management</a> that is not supported by browsers yet.</p>

<p>One approach that was raised by <a href="http://vimeo.com/3195079">Bespin</a> in 2009 is to keep a <a href="http://robertnyman.com/2009/04/03/mozilla-labs-online-code-editor-bespin/#comment-560310">parallel DOM</a> in sync with the elements rendered in canvas. We are continuing to investigate the right approach to accessibility.</p>

<h2>
  <a class="anchor-link" href="#Conclusion" name="Conclusion">
    Conclusion
  </a>
</h2>

<p>In the pursuit of <em>60fps</em> we sometimes resort to extreme measures. Flipboard for mobile web is a case study in pushing the browser to its limits. While this approach may not be suitable for all applications, for us it’s enabled a level of interaction and performance that rivals native apps. We hope that by releasing the work we’ve done with <a href="https://github.com/flipboard/react-canvas">React Canvas</a> that other compelling use cases might emerge.</p>

<p>Head on over to <a href="https://flipboard.com/">flipboard.com</a> on your phone to see what we’ve built, or if you don’t have a Flipboard account, check out a <a href="https://flipboard.com/@flipboard/flipboard-picks-8a1uu7ngz">couple</a> of <a href="https://flipboard.com/@flipboard/ten-for-today-k6ln1khuz">magazines</a> to get a taste of Flipboard on the web. Let us know what you think.</p>
