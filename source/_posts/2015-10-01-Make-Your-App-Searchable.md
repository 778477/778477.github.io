---
layout: post
title: 'Make Your App Searchable'
date: '2015-10-01'
header-img: "img/home-bg.jpg"
tags:
     - iOS
     
author: '778477'
---
![1](https://48ce6c28e7bf5f42a1b7-2712e00ea34e3076747650c92426bbb5.ssl.cf1.rackcdn.com/2015-06-17-013455.jpeg)

![2](http://searchengineland.com/figz/wp-content/seloads/2015/06/iOS-9-Apple-Search-Indexing-Methods-800x553-800x553.png)

- [官方FAQ](https://developer.apple.com/library/prerelease/ios/technotes/tn2416/_index.html#//apple_ref/doc/uid/DTS40016269-CH1-SEARCH_API_FAQ-WHICH_IS_THE_RIGHT_API_FOR_ME_TO_USE_)


**How Does Apple Rank Apps In Apple Search?**

**Positive Ranking Factors**

- **App Installation Status.** Is the app is installed on the device? (installed apps seem to get preference)
- **Personalized App Engagement.** Does the individual engage with the screen in the app? This is based on time spent with result that Apple determines from session analytics.
- **App Result Click-Through Rate.** Do users frequently click through the search result vs. picking another result or searching again?
- **Keywords/ Title.** Do keywords from the “keywords” and “title” designations in the app markup match up with the user’s query?
- **Aggregated Engagement.** How many users engage with the app screen?
- **Structured Data on Web.** Is structured data correctly implemented?
- **Canonical App IDs.** Is the same screen associated with one unique ID or URL across multiple indexing methods (NSUserActivity, CoreSpotlight, and Web Markup)?
- **Strength/Popularity of Web URL.** How popular is the website associated with the app deep links? (Presumably, this is based on Applebot’s crawl.)

**Negative Ranking Factors**

- **Low Engagement.** Do very few users engage with the app screen? (engagement determined by session analytics)
- **Over-Indexing.** Does the app have many screens in the index with low or no engagement?
- **Returns.** Do users return to search results right after looking at the app?
- **Keywords Spamming.** Are developers stuffing too many irrelevant keywords into the keyword field?
- **Interstitials.** Is something covering the content in the app or preventing users from accessing it?
- **Javascript (web only).** Is Javascript preventing Applebot from crawling your site to find new app deep links?
- **Low Star Ratings, Low Review Volume, Poor Reviews.** Apple has not explicitly called these negative ranking factors for Apple Search, but they are negative ranking factors for the App Store, so we expect Apple to treat them similarly here.

Apple recommends pursuing multiple indexing methods to optimize app visibility, but the overlapping methods will inevitably create duplication across the various indexes. For example, private content could have both a NSUserActivity and CSSearchableItem indexed, and public content could have both a NSUserActivity and a Web Markup deep link indexed.

This is obviously not ideal for controlling Applebot’s efficiency, so Apple strongly recommends associating each NSUserActivity, CSSearchableItem, and Web Markup deep link with the same uniqueIdentifier and/or URL. This is Applebot’s version of a canonical, and it’s so important to Apple that they’ve even made it a ranking factor in the Apple Search algorithm.


[CoreSpotlight Framework](https://developer.apple.com/library/prerelease/ios/documentation/CoreSpotlight/Reference/CoreSpotlight_Framework/)

[How to use the new search API](http://applidium.com/en/news/ios9_search_api/)




> 如有任何知识产权、版权问题或理论错误，还请指正
>
> 转载请注明原作者及以上信息

