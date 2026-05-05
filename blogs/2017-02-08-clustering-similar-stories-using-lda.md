---
title: "Clustering Similar Stories Using LDA"
url: "http://engineering.flipboard.com//2017/02/storyclustering"
date: "Wed, 08 Feb 2017 00:00:00 +0000"
author: "https://www.linkedin.com/in/arnab-bhadury-a6304768 (Arnab Bhadury)"
feed_url: "https://engineering.flipboard.com/feed.xml"
---
<p>There is more to a story than meets the eye, and some stories deserve to be presented from more than just one perspective. With Flipboard 4.0, we have released story roundups, a new feature that adds coverage from multiple sources to a story and provides you with a fuller picture of an event. <!--break--></p>

<p>Here’s how it looks:</p>

<div class="row" style="text-align: center;">
    <img src="/assets/storyclustering/storycluster.gif" />
</div>

<p>With our scale of millions of articles and constant stream of documents, it’s impossible to generate these roundups manually. So, we have developed a clustering algorithm that’s both fast and scalable, and in this blog post, I will explain how we create these roundups on Flipboard.</p>

<h2 id="why-is-this-difficult">Why is this difficult?</h2>

<p>Although there are many sophisticated automatic clustering algorithms, such as <a href="https://en.wikipedia.org/wiki/K-means_clustering">K-means</a> or <a href="https://en.wikipedia.org/wiki/Hierarchical_clustering">Agglomerative clustering</a>, story clustering is a non-trivial problem. Because each text document can contain any word from our vocabulary, most text document representations are extremely high-dimensional. In high-dimensional spaces, even basic clustering or similarity measures fail or are very slow.</p>

<p>Additionally, two very similar documents often have very different word usages. For example, one article may use the term <em>kitten</em> and another may use <em>feline</em>, but both articles could be referring to the same <em>cat</em>.</p>

<p>Furthermore, we don’t know the number of roundups that we expect to see beforehand. This makes it difficult for us to directly use parameteric algorithms such as K-means. Our clustering algorithm also needs to be fast and easy to update, because there is a constant stream of documents coming into our system.</p>

<h2 id="overview">Overview</h2>

<p>Since even the most basic distance measures fail in high dimensions, the first thing we do is lower the problem’s dimensionality. We represent each of our text documents as a <a href="https://en.wikipedia.org/wiki/Bag-of-words_model">bag-of-words</a>, and remove stop-words and rare words from our vocabulary. Even after an aggressive trimming, the documents are still very high-dimensional. We then we use <a href="https://www.cs.princeton.edu/~blei/papers/BleiNgJordan2003.pdf">Latent Dirichlet Allocation</a> (LDA) to further lower the documents’ dimensionality. We use LDA because this algorithm is amenable for text modeling and provides us with interpretable lower dimensional representations of documents.</p>

<p>LDA is generally used for <a href="http://psiexp.ss.uci.edu/research/papers/sciencetopics.pdf">qualitative understanding</a> of big text corpora. The <em>latent</em> topics that the model learns are highly interpretable and provide deep insights to the data. However, that’s not the only thing it can be used for; it is also a very logical algorithm to use for dimensionality reduction as it learns a mapping of sparse document-term vectors to sparse document-topic vectors in an unsupervised setting.</p>

<div class="row" style="text-align: center;">
    <img src="/assets/storyclustering/docfactors.gif" />
</div>

<p>Once the documents are represented over a tractable number of dimensions, all similarity and distance measures come into play. For our story clustering, we simply map all the documents to this conceptual level (latent topics), and look at the neighbours for each document within a certain distance. We use an approximate <a href="https://en.wikipedia.org/wiki/Nearest_neighbor_search">nearest neighbour</a> model because it only requires us to look at a small neighbourhood of documents to generate these clusters.</p>

<p>The following sections explain how we use LDA for this problem, and then introduce some tricks to optimize LDA using Alias tables and Metropolis-Hastings tests.</p>

<h2 id="high-level-overview-of-lda">High-level overview of LDA</h2>

<p>LDA is a probabilistic <a href="https://en.wikipedia.org/wiki/Generative_model">generative model</a> that extracts the thematic structure in a big document collection. The model assumes that every topic is a distribution of words in the vocabulary, and every document (described over the same vocabulary) is a distribution of a small subset of these topics. This is a statistical way to say that each topic (e.g. <em>space</em>) has some representative words (<em>star</em>, <em>planet</em>, etc.), and each document is only about a handful of these topics.</p>

<div class="row" style="text-align: center;">
    <img src="/assets/storyclustering/lda.png" />
</div>

<p>For example, let’s assume that we have a few topics as shown in the figure above. Knowing these topics, when we see a document explaining <em>Detecting and classifying pets using deep learning</em>, we can confidently say that the document is mostly about <em>Topic 2</em> and a little about <em>Topic 1</em> but not at all about <em>Topic 3</em> and <em>Topic 4</em>.</p>

<p>LDA automatically infers these topics given a large collection of documents and expresses those (and future) documents in terms of topics instead of raw terms. The key advantage of doing this is that we allow term variability as the document is represented at a higher conceptual (topic) level rather than at the raw word level. This is akin to many success stories in image classification tasks using deep learning, where the classification is done on a higher conceptual level instead of on the pixel level.</p>

<p>To infer the above <em>latent</em> topics, we do posterior inference on the model using Gibbs Sampling. What it essentially comes down to is to estimate the <em>best</em> topic for each word seen in every document. This is estimated by:</p>



<p>where  is the word seen in the th position in document ,  is the number of times the topic  has appeared in document ,  is the number of times the word  has been estimated with topic  in the whole corpus, and  is the number of times the topic  has been assigned in the corpus. It is a sensible equation which suggests that a topic is more likely to be assigned if it has been assigned to other words in the same document or when the term has been assigned to that same topic several times in the whole document corpus.</p>

<p>We calculate the above equation for each topic and define a <a href="https://en.wikipedia.org/wiki/Multinomial_distribution">multinomial distribution</a> (a weighted dice roll), and generate a random topic from that distribution. The code for LDA’s inference (CGS-LDA) looks like this:</p>



<div class="language-c highlighter-rouge"><div class="highlight"><pre class="highlight"><code><table class="rouge-table"><tbody><tr><td class="rouge-gutter gl"><pre class="lineno">1
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
</pre></td><td class="rouge-code"><pre><span class="k">for</span> <span class="p">(</span><span class="n">d</span> <span class="o">=</span> <span class="mi">0</span><span class="p">;</span> <span class="n">d</span> <span class="o">&lt;</span> <span class="n">D</span><span class="p">;</span> <span class="n">d</span><span class="o">++</span><span class="p">)</span> <span class="p">{</span>
  <span class="k">for</span> <span class="p">(</span><span class="n">n</span> <span class="o">=</span> <span class="mi">0</span><span class="p">;</span> <span class="n">n</span> <span class="o">&lt;</span> <span class="n">len</span><span class="p">(</span><span class="n">data</span><span class="p">[</span><span class="n">d</span><span class="p">]);</span> <span class="n">n</span><span class="o">++</span><span class="p">)</span> <span class="p">{</span>
    <span class="n">w</span> <span class="o">=</span> <span class="n">data</span><span class="p">[</span><span class="n">d</span><span class="p">][</span><span class="n">n</span><span class="p">];</span>
    <span class="n">k</span> <span class="o">=</span> <span class="n">Z</span><span class="p">[</span><span class="n">d</span><span class="p">][</span><span class="n">n</span><span class="p">];</span>
    <span class="n">decrement_count_matrices</span><span class="p">(</span><span class="n">d</span><span class="p">,</span> <span class="n">w</span><span class="p">,</span> <span class="n">k</span><span class="p">);</span>
    <span class="k">for</span> <span class="p">(</span><span class="n">k</span> <span class="o">=</span> <span class="mi">0</span><span class="p">;</span> <span class="n">k</span> <span class="o">&lt;</span> <span class="n">K</span><span class="p">;</span> <span class="n">k</span><span class="o">++</span><span class="p">)</span> <span class="p">{</span>
      <span class="n">p</span><span class="p">[</span><span class="n">k</span><span class="p">]</span> <span class="o">=</span> <span class="p">(</span><span class="n">CDK</span><span class="p">[</span><span class="n">d</span><span class="p">][</span><span class="n">k</span><span class="p">]</span> <span class="o">+</span> <span class="n">alpha</span><span class="p">)</span> <span class="o">*</span> <span class="p">(</span><span class="n">CWK</span><span class="p">[</span><span class="n">w</span><span class="p">][</span><span class="n">k</span><span class="p">]</span> <span class="o">+</span> <span class="n">beta</span><span class="p">)</span> <span class="o">/</span> <span class="p">(</span><span class="n">CK</span><span class="p">[</span><span class="n">k</span><span class="p">]</span> <span class="o">+</span> <span class="n">V</span> <span class="o">*</span> <span class="n">beta</span><span class="p">);</span>
    <span class="p">}</span>
    <span class="n">k</span> <span class="o">=</span> <span class="n">multinomial_distribution</span><span class="p">(</span><span class="n">p</span><span class="p">);</span>
    <span class="n">increment_count_matrices</span><span class="p">(</span><span class="n">d</span><span class="p">,</span> <span class="n">w</span><span class="p">,</span> <span class="n">k</span><span class="p">);</span>
    <span class="n">Z</span><span class="p">[</span><span class="n">d</span><span class="p">][</span><span class="n">n</span><span class="p">]</span> <span class="o">=</span> <span class="n">k</span><span class="p">;</span>
  <span class="p">}</span>
<span class="p">}</span>
</pre></td></tr></tbody></table></code></pre></div></div>

<p>This computation can be very expensive if we try to capture more than a thousand topics, because the algorithmic complexity becomes  per iteration. What we would ideally like is to get rid of  loop for each word in a document.</p>

<h2 id="optimizing-lda-using-alias-tables-and-metropolis-hastings-tests">Optimizing LDA using Alias Tables and Metropolis-Hastings Tests</h2>

<p>Fortunately, there has been a lot of new research (<a href="https://arxiv.org/pdf/1412.1576v1">LightLDA</a>, <a href="http://www.sravi.org/pubs/fastlda-kdd2014.pdf">AliasLDA</a>) to speed up the sampling process and reduce the computational complexity to . The key question here is: Is it really possible to generate a single sample from a weighted multinomial distribution in under  time?</p>

<p>The answer, not surprisingly, is “no” because generating a  dimensional multinomial probability array  takes at least  time because we need to know the weight for each index. But once this array is created, generating a sample is simply a matter of generating a random number from  and checking which index of the array the number falls in. And if we needed more samples from the same distribution, all future samples would only require  time.</p>

<div class="row" style="text-align: center;">
    <img src="/assets/storyclustering/prob_vector.gif" />
</div>

<p>Instead of taking a single sample each time from the distribution, if we take  samples each time the table is generated, then the amortized sampling complexity would be . But this method has huge memory implications because the sizes of these arrays are dependent on the sum of the weights, and in LDA’s case, they can get extremely massive due to dependencies on count matrices (,  and ).</p>

<h3 id="alias-sampling">Alias Sampling</h3>

<p>Walker’s <a href="https://en.wikipedia.org/wiki/Alias_method">Alias method</a> is an effective way to compactly store these probability vectors (Space complexity: ), 
while keeping the sampling complexity at .  Instead of defining a long row vector, Alias sampling defines a completely filled 2-dimensional
table (dim:  from which its easy to sample from in constant time.</p>

<p>These tables are generated by first multiplying each element by , followed by a Robin Hood
algorithm and maintaining two lists: <em>rich</em> and <em>poor</em>. <em>rich</em> contains all the elements that have
a greater weight than , and the <em>poor</em> stores the rest. This is followed by a simple
iteration of putting the poor elements in the table cell first, and then filling up the height if
needed by “stealing” from the rich:</p>

<div id="aliastable"></div>

<div style="text-align: center;"><div style="display: inline-block;">
<button class="btn btn-default" id="randomize_aliastable" type="button">
  <span class="glyphicon glyphicon-refresh"></span> Randomize Probabilities
</button>
</div></div>

<p>Each column has a height of 1.0 with a maximum of two different array indices. For sample generation, a random number is generated between  to pick the column, and then we generate a random floating point between  and see which region the decimal number falls in.</p>

<h3 id="metropolis-hastings-algorithm--test">Metropolis Hastings Algorithm / Test</h3>

<p>Coming back to our LDA case, every time we need to sample a topic for a word in a document, we could generate  samples. However, that would be wrong because the update equation is dependent on the count matrices, and stale samples don’t represent the <strong>true</strong> probability distribution of the inference.</p>

<p>Here’s where <a href="https://en.wikipedia.org/wiki/Metropolis%E2%80%93Hastings_algorithm">Metropolis-Hastings</a> (MH) algorithm comes into play. MH algorithm is a <a href="https://en.wikipedia.org/wiki/Markov_chain_Monte_Carlo">Markov-Chain Monte Carlo</a> (MCMC) method to move around the probability space with the intention of converging to an objective. Given an objective function , a proposal function  and a proposed position in the probability space, MH algorithm acts as a guide telling the algorithm whether it is a good or bad idea to move from the current position  to the proposed point . The acceptance of a new proposed position is computed by:</p>



<p>If the algorithm thinks that it’s a great idea to move, MH will always accept the proposal.</p>

<p>The following toy example shows how we can approximate the area of a random function with Metropolis-Hastings sampling. In this example, we try to approximate the shape of a complicated non-linear function and use a standard Gaussian distribution () multiplied with a step-size to generate proposals.</p>



<div id="mh_vis" style="text-align: center;"></div>
<div id="mh_vis_text" style="text-align: center;"></div>
<div id="mh_vis_stepsize" style="text-align: center;"></div>
<div id="mh_vis_slider" style="text-align: center;">
<input id="mh_vis_s" list="stepsizes" max="0.9" min="0.1" step="0.1" style="text-align: center;" type="range" value="0.3" />
<datalist id="stepsizes">
  <option>0.1</option>
  <option>0.3</option>
  <option>0.5</option>
  <option>0.7</option>
  <option>0.9</option>
</datalist>
</div>






<p>From the above example, it can be seen that proposal functions control the convergence speed of the algorithm. In the ideal scenario, we would like a high-acceptance rate and the ability to move quickly around the space. For the toy example above, step-sizes of  achieve the best results.</p>

<h3 id="back-to-lda">Back to LDA</h3>

<p>We would like to have high proposal acceptance rates, good space coverage, and simple proposal generation complexity. LightLDA authors suggest using the expressions within the LDA’s update equation as proposal functions. These expressions match the function in certain regions. This has two further advantages: we don’t need to compute anything extra and simply use the statistics (count matrices) that we would have collected anyway, and we don’t actually need to create alias tables for one of the proposals.</p>



<p> acts as a proxy alias table for the doc-proposal because it stores the number of times each topic has appeared in the document , and if we generate a random number between , we get a sample from the doc-proposal. For the word-proposals, we generate alias tables for each word and use the afforementioned Alias Sampling trick.</p>

<p>We cycle between these two proposals, and do the MH test on the fly, and accept/reject the proposals. We recompute an alias table for a word every time we have used  proposals. The acceptance probabilities for doc-proposal () and word-proposal () given a proposal topic  and the current topic  can be calculated by using the LDA’s update equation as the objective function, and the doc and word proposals as the proposal functions.</p>





<p>Using Alias tables and MH tests, the algorithm looks like the following:</p>

<div class="language-cpp highlighter-rouge"><div class="highlight"><pre class="highlight"><code><table class="rouge-table"><tbody><tr><td class="rouge-gutter gl"><pre class="lineno">1
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
23
24
25
</pre></td><td class="rouge-code"><pre><span class="k">for</span> <span class="p">(</span><span class="n">d</span> <span class="o">=</span> <span class="mi">0</span><span class="p">;</span> <span class="n">d</span> <span class="o">&lt;</span> <span class="n">D</span><span class="p">;</span> <span class="n">d</span><span class="o">++</span><span class="p">)</span> <span class="p">{</span>
  <span class="k">for</span> <span class="p">(</span><span class="n">n</span> <span class="o">=</span> <span class="mi">0</span><span class="p">;</span> <span class="n">n</span> <span class="o">&lt;</span> <span class="n">len</span><span class="p">(</span><span class="n">data</span><span class="p">[</span><span class="n">d</span><span class="p">]);</span> <span class="n">n</span><span class="o">++</span><span class="p">)</span> <span class="p">{</span>
    <span class="n">proposal</span> <span class="o">=</span> <span class="n">coinflip</span><span class="p">();</span>
    <span class="n">w</span> <span class="o">=</span> <span class="n">data</span><span class="p">[</span><span class="n">d</span><span class="p">][</span><span class="n">n</span><span class="p">];</span>
    <span class="n">k</span> <span class="o">=</span> <span class="n">Z</span><span class="p">[</span><span class="n">d</span><span class="p">][</span><span class="n">n</span><span class="p">];</span>
    <span class="n">decrement_count_matrices</span><span class="p">(</span><span class="n">d</span><span class="p">,</span> <span class="n">w</span><span class="p">,</span> <span class="n">k</span><span class="p">);</span>
    <span class="k">if</span> <span class="p">(</span><span class="n">proposal</span> <span class="o">==</span> <span class="mi">0</span><span class="p">)</span> <span class="p">{</span>
      <span class="c1">// doc-proposal
</span>      <span class="n">index</span> <span class="o">=</span> <span class="n">randomInt</span><span class="p">(</span><span class="mi">0</span><span class="p">,</span> <span class="n">len</span><span class="p">(</span><span class="n">data</span><span class="p">[</span><span class="n">d</span><span class="p">]));</span>
      <span class="n">p</span> <span class="o">=</span> <span class="n">Z</span><span class="p">[</span><span class="n">d</span><span class="p">][</span><span class="n">index</span><span class="p">];</span>
      <span class="n">mh_acceptance</span> <span class="o">=</span> <span class="n">compute_doc_acceptance</span><span class="p">(</span><span class="n">k</span><span class="p">,</span> <span class="n">p</span><span class="p">);</span>
    <span class="p">}</span> <span class="k">else</span> <span class="p">{</span>
      <span class="c1">// term-proposal
</span>      <span class="n">p</span> <span class="o">=</span> <span class="n">alias_sample</span><span class="p">(</span><span class="n">w</span><span class="p">);</span>
      <span class="n">mh_acceptance</span> <span class="o">=</span> <span class="n">compute_term_acceptance</span><span class="p">(</span><span class="n">w</span><span class="p">,</span> <span class="n">p</span><span class="p">);</span>
    <span class="p">}</span>
    <span class="c1">// MH-test
</span>    <span class="n">mh_sample</span> <span class="o">=</span> <span class="n">randomFloat</span><span class="p">(</span><span class="mi">0</span><span class="p">,</span> <span class="mi">1</span><span class="p">);</span>
    <span class="k">if</span> <span class="p">(</span><span class="n">mh_sample</span> <span class="o">&lt;</span> <span class="n">mh_acceptance</span><span class="p">)</span> <span class="p">{</span>
      <span class="n">increment_count_matrices</span><span class="p">(</span><span class="n">d</span><span class="p">,</span> <span class="n">w</span><span class="p">,</span> <span class="n">k</span><span class="p">);</span>  <span class="c1">// reject proposal, revert to k
</span>    <span class="p">}</span> <span class="k">else</span> <span class="p">{</span>
      <span class="n">increment_count_matrices</span><span class="p">(</span><span class="n">d</span><span class="p">,</span> <span class="n">w</span><span class="p">,</span> <span class="n">p</span><span class="p">);</span>  <span class="c1">// accept proposal
</span>    <span class="p">}</span>	
  <span class="p">}</span>
<span class="p">}</span>
</pre></td></tr></tbody></table></code></pre></div></div>

<p>By doing this, the algorithmic complexity of LDA comes down to an amortized  which allows us to process documents an order of magnitude faster. The following table and the graph compare the runtime (in seconds) of LDA of 60,000 documents after 100 iterations (convergence) on a single process.</p>

<center>
<table align="center">
  <col width="160" />
  <col width="90" />
  <col width="90" />
  <col width="90" />
  <col width="90" />
  <col width="90" />
    <tr>
      <th>    Num. of Topics    </th>
      <th> 100 </th>
      <th> 200 </th>
      <th> 300 </th>
      <th> 500 </th>
      <th> 1000 </th>
    </tr>
    <tr>
      <td> CGS LDA </td>
      <td> 3427.99 </td>
      <td> 7605.02 </td>
      <td> 12190.54 </td>
      <td> 25274.20 </td>
      <td> 57492.22 </td>
    </tr>
    <tr>
      <td> MH/Alias LDA </td>
      <td> 601.06 </td>
      <td> 616.07 </td>
      <td> 620.82 </td>
      <td> 646.38 </td>
      <td> 685.19 </td>
    </tr>
  </table>
</center>

<div class="row" style="text-align: center;">
    <img src="/assets/storyclustering/comparison.png" />
</div>

<h2 id="clustering">Clustering</h2>

<p>LDA provides us with a sparse and robust representation of texts that reduces term variability in much lower dimensions. In 1000 or lower dimensions, most simple algorithms work really well. When each document is represented in this space, we do a fast <a href="http://en.wikipedia.org/wiki/Nearest_neighbor_search#Approximate_nearest_neighbor">Approximate Nearest Neighbour search</a>, and cluster all documents that are within a certain distance from each other.</p>

<p>Using a distance based metric has an added advantage of being able to capture near and exact duplicates. The documents that are mapped too close to each other (purple circle) are considered to be the same story. The documents that are a certain radius away from the exact duplicates (pink circle) make up the roundups for each story.</p>

<div id="cluster_vis" style="text-align: center;"></div>


<p>Removing exact duplicates helps in capturing different views on an event, and here is an example of one of our story-clusters where the roundups capture differing perspectives on the same news.</p>

<div class="row" style="text-align: center;">
    <img src="/assets/storyclustering/griezmann.gif" />
</div>

<h2 id="conclusion">Conclusion</h2>

<p>Story roundups directly help us in diversifying our users’ feeds while also providing users with multiple perspectives on important stories. We had a lot of fun implementing this cool new feature, not least because we came across several new tricks that can be applied in multiple domains.</p>

<p>Alias Sampling and MH algorithm have been around for a long time but they are only now being used in tandem to optimize posterior and predictive inference problems.</p>

<h3 id="key-takeaways">Key Takeaways</h3>

<ul>
  <li>LDA is awesome not just in context of qualitative topic analysis but also in terms of dimensionality reduction.</li>
  <li>Simple algorithms work really well in low dimensions; (almost) everything fails in very high dimensions.</li>
  <li><em>Bayesian Machine Learning doesn’t have to be slow or expensive.</em></li>
  <li>If you made it this far, I hope you learned a new way to optimize some posterior inference problems - Alias Tables and MH tests are an amazing combination.</li>
</ul>

<p><em>Extra special thanks to <a href="http://benfrederickson.com">Ben Frederickson</a> for suggestions, edits and
the Alias Table visualization. Thanks to <a href="http://www.linkedin.com/in/sizzler">Dale Cieslak</a>, <a href="https://www.linkedin.com/in/mikecora">Mike Vlad Cora</a> and <a href="https://twitter.com/miaq">Mia Quagliarello</a> for proofreading.</em></p>

<p>Enjoyed this post? <a href="https://about.flipboard.com/careers/">We’re hiring!</a>
</p>
