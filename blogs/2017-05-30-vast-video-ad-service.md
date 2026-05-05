---
title: "VAST Video Ad Service"
url: "http://engineering.flipboard.com//2017/05/vast-video-ad"
date: "Tue, 30 May 2017 00:00:00 +0000"
author: "https://www.linkedin.com/in/guangle-fan-13496731/ (Guangle Fan)"
feed_url: "https://engineering.flipboard.com/feed.xml"
---
<p>Videos are more ubiquitous than ever before: there are virtually no boundaries on how, when and where people can interact with video content. Flipboard as a curation platform brings multi-format content to peoples around their personal interests and passions. There has been tremendous opportunity for video ads to perform efficiently on our platform. At Flipboard, we have been building video inventory <!--break--> since 2013 in the form of transcoding, storing, and serving proprietary format video ads on an in-house platform. As we see high-quality beautiful video ads satisfy advertisers, we also face the challenge of scaling the platform to meet the increasing demands of our video ad inventory. To bring Flipboard’s video ad inventory to a larger group of advertisers, we need a programmatic selling channel that supports standard video ad formats, provides easy performance measurement and flexible video behavior control. This is where VAST can help.</p>

<div class="row" style="text-align: center;">
  <div class="col-sm-3"><img src="/assets/vastvideoad/screen_1.png" /></div>
  <div class="col-sm-3"><img src="/assets/vastvideoad/screen_2.png" /></div>
  <div class="col-sm-3"><img src="/assets/vastvideoad/vast_demo.gif" /></div>
</div>

<h2 id="what-is-vast-">What is VAST ?</h2>
<p>VAST is the Video Ad Serving Template published by <a href="https://www.iab.com/guidelines/digital-video-ad-serving-template-vast-2-0/">IAB</a>. It is a universal XML schema that gives video players information about which ad to play, how the ad should show up, how long it should last, and whether peoples are able to skip it. It’s important because it lets video players and ad servers speak the same language. Before VAST, advertisers had very little information about the publisher’s player implementation. It also allows ad platforms to easily exchange ads with each other. For example, a physical video media file can be hosted at Service A, and passed through a chain of Services B, C, D, and served to consumers on publisher platform E. Each service will be able to wrap the ad with its own event trackers for performance measurement. VAST has been the most adopted video ad standard by advertisers and publishers, and the prevailing version is VAST 2.0.</p>

<h2 id="how-do-we-serve-vast-ads-">How do we serve VAST ads ?</h2>
<p>At Flipboard, we have ad inventory across our iOS, Android, Samsung Briefing, and Web products. The way some publishers implement a VAST compatible video player on the client side is OK, but not ideal. As a comparison, a backend-centric approach can work better, where a single VAST compliance server handles all the third-party communication, while the client side has consistent behavior across multiple platforms. It’s also easy to check ad quality before serving to the client and optimize package selection logic.
Each transaction of VAST ads involves communication among multiple ad services. Each service provides a unique XML payload which carries its own event trackers including impression/error/click through, user engagement trackers, and a special HTTP URL called VAST tag pointing to its source third-party service, or its parent ad service. The pointers form a virtual network between services involved in the transaction. Crawling pointers along the chain, our ad server can eventually fetch the inline ad with media files. It also organizes all trackers by event types, assembles with media resources, adds our own event trackers, and then sends down a validated ad package to the clients.</p>

<p>Our internal inventory management tool supports two types of VAST ads. The first type connects to open market ad platforms such as Rubicon and Tremor (with more coming), where we don’t necessarily limit a particular group of buyers. Instead, inventory would be bid on and taken by the winner, although we provide various types of topic-based packages like business, technology, fashion, etc. Geo targeting is also available.</p>

<div class="row" style="text-align: center;">
    <img src="/assets/vastvideoad/third_party_ad.png" />
</div>

<div class="row" style="text-align: center;">
    <img src="/assets/vastvideoad/ad_network.png" />
</div>

<p>Another type is through direct sale channel advertisers providing us VAST tags hosted by a third-party server, e.g DCM or their own ad server, where they set up the tag once and distribute to all VAST compatible publisher platforms. This means we don’t need to transcode or host the media content, which lets us hammer out a deal in much faster way with less cost of communication and maintenance.</p>

<div class="row" style="text-align: center;">
    <img src="/assets/vastvideoad/direct_sale_ad.png" />
</div>

<p>While both types increase our inventory sell-through rate, the former is such a flexible tool that opens a new selling channel, while the latter reinforces our direct selling channel by the unique ease of setting up.</p>

<h2 id="some-challenges">Some Challenges</h2>
<p>The communication among third-party ad platforms, DSP, and PSP has a pull-based architecture. That means inventory sellers like us need to constantly send PSP requests whenever they’re available, while buyers on the other side of the platform constantly check and buy what is available at the moment. The fill rate is low based on the total number of requests, and that’s the nature of many ad platforms. At our scale, requests per second can sustain at 10K on average. Since we just started selling a few packages on a few ad platforms, the number can only go up while we expand our footprint. Handling the scale in an efficient way is critical to meet high fill rate of our inventory. On the backend, we optimize our HTTP requests with pooling connections, asynchronous non-blocking calls, properly timing out and recycling resources, etc. Thus we control the usage of CPU, network, memory, and other system resources under a reasonable level without adding more hardware.</p>

<div class="row" style="text-align: center;">
    <img src="/assets/vastvideoad/monitor.png" />
</div>

<p>Since serving VAST ads involve multiple third-party services by design, it comes with a challenge of quickly identifying and responding to ad quality issues. While an ad can be wrapped and exchange hands several times, it’s not surprising how quickly the trace of the original buyer gets lost. In our ad server, we monitor the change of ad quality by ad system, by service domain, and identify typical reasons in real time. While new causes of ad quality problems may still come up, they tend to repeatedly happen afterward. Our ad server traces all VAST tags that lead to a problematic inline ad, samples the bad instance and surfaces it up to the logging system UI. For a bad instance, we are able to get the message of the preliminary reason, and a chain of VAST tags. That is helpful to quickly respond to the issue, limiting the impact.</p>

<p>Last but not least, as with any type of ads, tracking performance is half of the work. To be VAST compliant, our mobile app calls third-party tracking URLs from different services at the same time when the event happens, as well as our in-house event trackers. At the backend, we leverage the existing data pipeline to aggregate metrics in a structural way by campaign, order, and ad. We provide insights of quartile of video play, expand, collapse, click-through rate, play duration, etc. We also provide dimensional insights about peoples’ geo location and interests.</p>

<h2 id="bottom-line">Bottom Line</h2>
<p>Although our service can handle high throughput of third-party requests in the low-fill-rate programmatic world, we can still improve it by encouraging bidding activities with better tailored ad packages, and by leveraging our ad selection model and revenue prediction model. That is the next step to do.</p>

<p>To sum up, we are excited to announce that Flipboard supports VAST ad as a new programmatic selling channel. We think it can bring us a broader group of advertisers, higher quality ads, and better user experience to our beautiful app.</p>
