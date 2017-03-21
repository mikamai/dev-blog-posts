---
ID: 60
post_title: 'Apple Music for iOS developers (hint: it doesn’t work)'
author: Lorenzo Bulfone
post_date: 2015-12-03 16:56:07
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2015/12/03/apple-music-for-ios-developers-hint-it-doesnt/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/134469497084/apple-music-for-ios-developers-hint-it-doesnt
tumblr_mikamayhem_id:
  - "134469497084"
---
<p>Back in June Apple released its own music streaming service, <strong>Apple Music</strong>, and to this date there is no API that allows developers to easily take advantage of the user&rsquo;s Apple Music library. Sure, <strong>MPMediaPickerController</strong> and <strong>MPMusicPlayerController</strong> work great with very few lines of code, but the player doesn&rsquo;t work in the background and that&rsquo;s a very big limitation.
AVFoundation&rsquo;s <strong>AVPlayer</strong> and <strong>AVAudioPlayer</strong> are two very powerful options when it comes to audio and video playback and they work in the background, but they both have a problem: they can&rsquo;t play directly the <strong>MPMediaItems</strong> that the MPMediaPickerController returns.
Before Apple Music (and iTunes Match), every MPMediaItem used to represent a single song that was <strong>physically</strong> stored on the user&rsquo;s device, so the quickest option was to access its <strong>MPMediaItemPropertyAssetURL</strong> and use it for playback. Today, there is a range of different situations and the AssetURL isn&rsquo;t always available.</p>

<table><thead><tr><th align="center">  Song origin/location </th>
<th align="center">  AssetURL </th>
</tr></thead><tbody><tr><td align="center"> <strong>iTunes (Win/Mac OSX transfer)</strong>  </td>
<td align="center"> <strong>YES</strong>  </td>
</tr><tr><td align="center"> <strong>iTunes (downloaded via iOS app)</strong>  </td>
<td align="center"> <strong>YES</strong>  </td>
</tr><tr><td align="center"> <strong>iTunes Match (stored on device)</strong> </td>
<td align="center"> <strong>YES</strong> </td>
</tr><tr><td align="center"> iTunes Match (stored on iCloud) </td>
<td align="center">  NO </td>
</tr><tr><td align="center"> Apple Music (stored on device)  </td>
<td align="center">  NO </td>
</tr><tr><td align="center"> Apple Music (stored on iCloud)  </td>
<td align="center">  NO </td>
</tr></tbody></table><p>As expected, an iTunes Match or Apple Music song that is only in the cloud doesn&rsquo;t have a local AssetURL, but Apple Music songs aren&rsquo;t available even if the user decided to save them for offline use. Since there is no support for Apple Music, if we want to let the user pick a song to be played with AVAudioPlayer, we need to be able to hide the ones that don&rsquo;t have a valid AssetURL. By default MPMediaPickerController shows every song in the library (including cloud items), so the only option is to create our own song picker using <strong>MPMediaQuery</strong>. This class doesn&rsquo;t allow to filter directly the songs based on their AssetURL, but once we have the complete list we can cycle through the array and remove the ones who don&rsquo;t have a valid URL:</p>

<pre><code>// Get all the songs from the library
MPMediaQuery *query = [MPMediaQuery songsQuery];

// Filter out cloud items
[query addFilterPredicate:[MPMediaPropertyPredicate predicateWithValue:[NSNumber numberWithBool:NO] forProperty:MPMediaItemPropertyIsCloudItem]];

NSArray *items = [query items];
NSMutableArray *itemsWithAssetURL = [NSMutableArray array];

// For every song stored locally
for (MPMediaItem *item in items)
{
    // If the song has an assetURL
    if ([item valueForProperty:MPMediaItemPropertyAssetURL]) {
        // Add it to the array of valid songs
        [itemsWithAssetURL addObject:item];
    }
}
</code></pre>

<p>This does the trick, but if the user&rsquo;s library contains thousands of songs (as it does most of the times) it&rsquo;s probably best to perform this away from the main thread and/or to limit the number of items to check in the first place.</p>