---
layout: post
title:  "nth-child is weird"
date:   2019-01-01 21:50:00 -0600
---

CSS's nth-child looks straight forward but there is a trick I wish I had known when I was learning CSS.

If you have not used nth-child before, I highly recommend [CSS Tricks' Article](https://css-tricks.com/almanac/selectors/n/nth-child/) on the topic. Make sure you try out the testers included in the article.

The issue with learning this property is that we are presented with a list of similar items.

```html
<style>
  /* Select the third element in the list */
  .my-list li:nth-child(3) {
    color: #68AF15;
  }
</style>

<ul class="my-list">
  <li>item 1</li>
  <li>item 2</li>
  <li>item 3</li>
  <li>item 4</li>
  <li>item 5</li>
</ul>
```

<style>
  .my-list li:nth-child(3) {
    color: #68AF15;
  }
</style>

<ul class="my-list">
  <li>item 1</li>
  <li>item 2</li>
  <li>item 3</li>
  <li>item 4</li>
  <li>item 5</li>
</ul>

I find that layouts are not always that straight forward or you may not have control over the markup. Consider this example

```html
<style>
  /* Select the fifth post from the list */
  .feed .post:nth-child(5) {
    color: #68AF15;
  }
</style>

<ul class="feed">
  <li class="post">post 1</li>
  <li class="post">post 2</li>
  <li class="post">post 3</li>
  <li class="post">post 4</li>
  <li class="ad">Some Ad</li>
  <li class="post">post 5</li>
</ul>
```

<style>
  .feed .post:nth-child(5) {
    color: #68AF15;
  }
</style>

<ul class="feed">
  <li class="post">post 1</li>
  <li class="post">post 2</li>
  <li class="post">post 3</li>
  <li class="post">post 4</li>
  <li class="ad">Some Ad</li>
  <li class="post">post 5</li>
</ul>

Hmm... why is post 5 not being selected, it's the 5th element with the class `post` in the list?

Here is the trick to nth-child selectors, it's an IF statement! in the code above I am declaring select the 5th element in this list of sibling elements *if* it has the class `post`. Thinking this way has reduced confusion around this interesting selector.

This can be used to create interesting solutions. Say we only want the ad to be highlighted if it's in the 5th position.

```html
<style>
    /* Select an Ad if it's the fifth item in the list */
  .feed.with-ad .ad:nth-child(5) {
    color: #68AF15;
  }
</style>

<ul class="feed with-ad">
  <li class="post">post 1</li>
  <li class="post">post 2</li>
  <li class="post">post 3</li>
  <li class="post">post 4</li>
  <li class="ad">Look at me</li>
  <li class="post">post 5</li>
</ul>
```

<style>
  .feed.with-ad .ad:nth-child(5) {
    color: #68AF15;
  }
</style>

<ul class="feed with-ad">
  <li class="post">post 1</li>
  <li class="post">post 2</li>
  <li class="post">post 3</li>
  <li class="post">post 4</li>
  <li class="ad">Look at me</li>
  <li class="post">post 5</li>
</ul>

This was a game changer for me, hopefully you find it helpful.
