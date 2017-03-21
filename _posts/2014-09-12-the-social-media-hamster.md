---
ID: 210
post_title: The social media hamster
author: Stefano Guglielmetti
post_date: 2014-09-12 14:04:40
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2014/09/12/the-social-media-hamster/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/97301192064/the-social-media-hamster
tumblr_mikamayhem_id:
  - "97301192064"
---
<h3>(la muerte peluda is back)</h3>

[youtube https://www.youtube.com/watch?v=4vpNNMJSEbk&w=560&h=315]
<p>Every hackathon we host at <a href="http://www.mikamai.com">Mikamai</a> usually means a lot of leftovers. From pizza crusts to burned LEDs, you can find a lot of useful and useless stuff even in the deepest and darkest corners of our <a href="https://www.google.com/maps/preview?ie=UTF-8&amp;fb=1&amp;spn=0.18,0.3&amp;cbll=45.490463999999996,9.215694&amp;layer=c&amp;panoid=GC_miZeIFDoAAAGuxVBJvw&amp;cbp=,340.72375,,0,-0.0&amp;cid=2729404206728252762&amp;q=mikamai&amp;ei=POQSVIHSIoyR7AaaiIAY&amp;ved=0CJMBEKAfMAo">huge open space</a>.</p>

<p>And in one of these dark corners I found a scared little friend, which I suddenly decided to adopt.</p>

<p><img src="http://i.imgur.com/fzLhryMl.jpg" alt="image" /></p>

<p>We became BFFs, and we do everything together, coding, drinking, pranking the teammates and cycling, but he wanted to be useful to me, so I went into my cove and start thinking&hellip; what an <a href="http://southpark.cc.com/full-episodes/s12e10-pandemic">ex cartoon eating</a> plush hamster can do for me?</p>

<p>The answer was quite obvious! He can warn me whenever someone tweets me or cites me in a tweet! Genius!</p>

<p>So, here are the ingredients</p>

<ul><li><a href="https://dev.twitter.com/streaming/overview">Twitter streaming APIs</a></li>
<li><a href="http://www.electricimp.com">Electric IMP</a> (I decided to use an Electric Imp just because I had a spare one, you can even use a <a href="https://www.spark.io/">spark core</a> or an <a href="http://arduino.cc/en/Main/ArduinoBoardYun?from=Products.ArduinoYUN">Arduino Yún</a>)</li>
<li>A servo</li>
<li>Chopsticks</li>
<li>Some screws</li>
<li>Cable ties</li>
<li>Something heavy</li>
<li>Obviously, absulutely essential, a plush hamster</li>
<li>An USB power adapter or a USB battery pack if you want your project to be portable</li>
</ul><p><img src="http://i.imgur.com/kfBoHR8l.jpg" alt="image" /></p>

<p>I also used an LED for visual debugging, I tried to put a pair of LEDs instead of my hamster&rsquo;s eyes, but it was too scary, and it remembered a lot my <a href="http://www.instructables.com/id/The-Creepy-Doll/">creepy doll</a>, so I decided not to use them</p>

<p>Then I had to make the hamster move with the servo. It&rsquo;s just a prototype, so I decided to keep it very very simple, and I secured the hamster to the servo with a chopstick, some copper wire and some screws to keep the hamster from doing a barrel roll. The surgery was not so traumatic, so my hamster is still happy. Then I fixed the servo to an heavy metal plate with a cable tie, just to provide a solid base.</p>

<p>I drilled some holes to the chopstick</p>

<p><img src="http://i.imgur.com/VhGHFRpl.jpg" alt="image" /></p>

<p>Then put some screws in it and i secured the chopstick to the servo</p>

<p><img src="http://i.imgur.com/d634xgyl.jpg" alt="image" /></p>

<p>And I used a bit of copper wire to secure the hamster to the screws</p>

<p><img src="http://i.imgur.com/sPpeJbRl.jpg" alt="image" /></p>

<p><img src="http://i.imgur.com/JNRYwDWl.jpg" alt="image" /></p>

<p>And a cable tie to fix the servo to the metal plate</p>

<p><img src="http://i.imgur.com/Il0doDrl.jpg" alt="image" /></p>

<p>Now everything&rsquo;s in place!</p>

<p><img src="http://i.imgur.com/kZsTPnHl.jpg" alt="image" /></p>

<p>The circuit is dead simple. Just connect the power pins of the servo to the GND-VIN pins of the Electric Imp, and the third pin to the Imp&rsquo;s pin number 7</p>

<p><img src="http://i.imgur.com/zQgpprzl.jpg" alt="image" /></p>

<p>The Electric Imp is one of the easiest platforms to configure, thanks to the Blink app, so you can have it online in a couple of minutes. You can even use your phone as hotspot, it works perfectly and it makes your social hamster fully portable.</p>

<p><img src="http://i.imgur.com/5jJHAw3l.jpg" alt="image" /></p>

<p>Now, the code. The Electric imp platform gives you all the instruments to interact with the whole world of APIs and webservices. Basically, the code is splitted in two: An agent, that runs on the Electric Imp Cloud servers, and a device, which is your electric Imp</p>

<p>[Copy &amp; Paste from the official <a href="https://electricimp.com/docs/api/">Electric Imp documentation</a>]</p>

<p><em>The agent object represents the imp’s agent: the server-side Squirrel, running in the Electric Imp&rsquo;s cloud servers, that deals with Internet requests and responses on behalf of the imp. The agent object is used to mediate communication between the imp and its agent.</em></p>

<p><em>The device object represents the server-based agent’s view of the imp and is used to mediate communication between the agent and the imp.</em></p>

<p>[End of copy&amp;paste]</p>

<p>So here we go with our agent code. It&rsquo;s based on the useful twitter library included in the Electric Imp webservices reference. I also added this piece of code to generate the tweet event manually</p>

<pre><code>// test function for manual hamster shaking
function requestHandler(request, response) {
  try {
    // check if the user sent led as a query parameter
    if ("tweet" in request.query) {
        device.send("tweet", null); 
    }
    // send a response back saying everything was OK.
    response.send(200, "tweet test ok");
  } catch (ex) {
    response.send(500, "Internal Server Error: " + ex);
  }
}

// register the HTTP handler
http.onrequest(requestHandler);
</code></pre>

<p>To enable the twitter stream real time parsing, I just needed to configure these twitter constants</p>

<pre><code>// Twitter Keys
const API_KEY = "";
const API_SECRET = "";
const AUTH_TOKEN = "";
const TOKEN_SECRET = "";
</code></pre>

<p>and this line</p>

<pre><code>twitter.stream("@jeko", onTweet);
</code></pre>

<p>which basically tells to the stream object to catch all the tweets containing the string &ldquo;@jeko&rdquo;</p>

<p>So, here&rsquo;s the complete agent code</p>

<pre><code>// Copyright (c) 2013 Electric Imp
// This file is licensed under the MIT License
// <a href="http://opensource.org/licenses/MIT">http://opensource.org/licenses/MIT</a>

// Twitter Keys
const API_KEY = "";
const API_SECRET = "";
const AUTH_TOKEN = "";
const TOKEN_SECRET = "";

class Twitter {
    // OAuth
    _consumerKey = null;
    _consumerSecret = null;
    _accessToken = null;
    _accessSecret = null;

    // URLs
    streamUrl = "https://stream.twitter.com/1.1/";
    tweetUrl = "https://api.twitter.com/1.1/statuses/update.json";

    // Streaming
    streamingRequest = null;
    _reconnectTimeout = null;
    _buffer = null;


    constructor (consumerKey, consumerSecret, accessToken, accessSecret) {
        this._consumerKey = consumerKey;
        this._consumerSecret = consumerSecret;
        this._accessToken = accessToken;
        this._accessSecret = accessSecret;

        this._reconnectTimeout = 60;
        this._buffer = "";
    }

    /***************************************************************************
     * function: Tweet
     *   Posts a tweet to the user's timeline
     * 
     * Params:
     *   status - the tweet
     *   cb - an optional callback
     * 
     * Return:
     *   bool indicating whether the tweet was successful(if no cb was supplied)
     *   nothing(if a callback was supplied)
     **************************************************************************/
    function tweet(status, cb = null) {
        local headers = { };

        local request = _oAuth1Request(tweetUrl, headers, { "status": status} );
        if (cb == null) {
            local response = request.sendsync();
            if (response &amp;&amp; response.statuscode != 200) {
                server.log(format("Error updating_status tweet. HTTP Status Code %i:rn%s", response.statuscode, response.body));
                return false;
            } else {
                return true;
            }
        } else {
            request.sendasync(cb);
        }
    }

    /***************************************************************************
     * function: Stream
     *   Opens a connection to twitter's streaming API
     * 
     * Params:
     *   searchTerms - what we're searching for
     *   onTweet - callback function that executes whenever there is data
     *   onError - callback function that executes whenever there is an error
     **************************************************************************/
    function stream(searchTerms, onTweet, onError = null) {
        server.log("Opening stream for: " + searchTerms);
        // Set default error handler
        if (onError == null) onError = _defaultErrorHandler.bindenv(this);

        local method = "statuses/filter.json"
        local headers = { };
        local post = { track = searchTerms };
        local request = _oAuth1Request(streamUrl + method, headers, post);


        this.streamingRequest = request.sendasync(

            function(resp) {
                // connection timeout
                server.log("Stream Closed (" + resp.statuscode + ": " + resp.body +")");
                // if we have autoreconnect set
                if (resp.statuscode == 28) {
                    stream(searchTerms, onTweet, onError);
                } else if (resp.statuscode == 420) {
                    imp.wakeup(_reconnectTimeout, function() { stream(searchTerms, onTweet, onError); }.bindenv(this));
                    _reconnectTimeout *= 2;
                }
            }.bindenv(this),

            function(body) {
                 try {
                    if (body.len() == 2) {
                        _reconnectTimeout = 60;
                        _buffer = "";
                        return;
                    }

                    local data = null;
                    try {
                        data = http.jsondecode(body);
                    } catch(ex) {
                        _buffer += body;
                        try {
                            data = http.jsondecode(_buffer);
                        } catch (ex) {
                            return;
                        }
                    }
                    if (data == null) return;

                    // if it's an error
                    if ("errors" in data) {
                        server.log("Got an error");
                        onError(data.errors);
                        return;
                    } 
                    else {
                        if (_looksLikeATweet(data)) {
                            onTweet(data);
                            return;
                        }
                    }
                } catch(ex) {
                    // if an error occured, invoke error handler
                    onError([{ message = "Squirrel Error - " + ex, code = -1 }]);
                }
            }.bindenv(this)

        );
    }

    /***** Private Function - Do Not Call *****/
    function _encode(str) {
        return http.urlencode({ s = str }).slice(2);
    }

    function _oAuth1Request(postUrl, headers, data) {
        local time = time();
        local nonce = time;

        local parm_string = http.urlencode({ oauth_consumer_key = _consumerKey });
        parm_string += "&amp;" + http.urlencode({ oauth_nonce = nonce });
        parm_string += "&amp;" + http.urlencode({ oauth_signature_method = "HMAC-SHA1" });
        parm_string += "&amp;" + http.urlencode({ oauth_timestamp = time });
        parm_string += "&amp;" + http.urlencode({ oauth_token = _accessToken });
        parm_string += "&amp;" + http.urlencode({ oauth_version = "1.0" });
        parm_string += "&amp;" + http.urlencode(data);

        local signature_string = "POST&amp;" + _encode(postUrl) + "&amp;" + _encode(parm_string);

        local key = format("%s&amp;%s", _encode(_consumerSecret), _encode(_accessSecret));
        local sha1 = _encode(http.base64encode(http.hash.hmacsha1(signature_string, key)));

        local auth_header = "oauth_consumer_key=""+_consumerKey+"", ";
        auth_header += "oauth_nonce=""+nonce+"", ";
        auth_header += "oauth_signature=""+sha1+"", ";
        auth_header += "oauth_signature_method=""+"HMAC-SHA1"+"", ";
        auth_header += "oauth_timestamp=""+time+"", ";
        auth_header += "oauth_token=""+_accessToken+"", ";
        auth_header += "oauth_version="1.0"";

        local headers = { 
            "Authorization": "OAuth " + auth_header
        };

        local url = postUrl + "?" + http.urlencode(data);
        local request = http.post(url, headers, "");
        return request;
    }

    function _looksLikeATweet(data) {
        return (
            "created_at" in data &amp;&amp;
            "id" in data &amp;&amp;
            "text" in data &amp;&amp;
            "user" in data
        );
    }

    function _defaultErrorHandler(errors) {
        foreach(error in errors) {
            server.log("ERROR " + error.code + ": " + error.message);
        }
    }

}

twitter &lt;- Twitter(API_KEY, API_SECRET, AUTH_TOKEN, TOKEN_SECRET);

function onTweet(tweetData) {
    // log the tweet, and who tweeted it (there is a LOT more info in tweetData)
    server.log(format("%s - %s", tweetData.text, tweetData.user.screen_name));
    device.send("tweet", null); 
}


// test function for manual hamster shaking
function requestHandler(request, response) {
  try {
    // check if the user sent led as a query parameter
    if ("tweet" in request.query) {
        device.send("tweet", null); 
    }
    // send a response back saying everything was OK.
    response.send(200, "tweet test ok");
  } catch (ex) {
    response.send(500, "Internal Server Error: " + ex);
  }
}

twitter.stream("yoursearchstring", onTweet);

// register the HTTP handler
http.onrequest(requestHandler);
</code></pre>

<p>Then, here&rsquo;s the device code, based on the electric Imp <a href="https://electricimp.com/docs/examples/pwm-servo/">PWM Servo example</a></p>

<pre><code>// These values may be different for your servo
const SERVO_MIN = 0.03;
const SERVO_MAX = 0.1;

// create global variable for servo and configure
servo &lt;- hardware.pin7;
servo.configure(PWM_OUT, 0.02, SERVO_MIN);

// assign pin9 to a global variable
led &lt;- hardware.pin9;
// configure LED pin for DIGITAL_OUTPUT
led.configure(DIGITAL_OUT);

// global variable to track current state of LED pin
state &lt;- 0;
// set LED pin to initial value (0 = off, 1 = on)
led.write(state);



// expects a value between 0.0 and 1.0
function SetServo(value) {
  local scaledValue = value * (SERVO_MAX-SERVO_MIN) + SERVO_MIN;
  servo.write(scaledValue);
}

// expects a value between -80.0 and 80.0
function SetServoDegrees(value) {
  local scaledValue = (value + 81) / 161.0 * (SERVO_MAX-SERVO_MIN) + SERVO_MIN;
  servo.write(scaledValue);
}

// current position (we'll flip between 0 and 1)
position &lt;- 0;

function HamsterDance() {
  SetServoDegrees(-10);
  imp.sleep(0.5);
  SetServoDegrees(0);
  imp.sleep(0.2);
  SetServoDegrees(-15);
  imp.sleep(0.2);
  SetServoDegrees(5);  
  imp.sleep(0.2);
  SetServoDegrees(-20);
  imp.sleep(0.2);
  SetServoDegrees(0);
  imp.sleep(0.2);
  SetServoDegrees(-78);
} 


function ShakeRattleAndRoll(ledState){
    server.log("Let's shake the hamster!");  
    // turn on the led for visual debugging
    led.write(1);
    // 
    HamsterDance();
    imp.sleep(2);
    // turn off the led
    led.write(0);

} 

// initialize the servo to the start position
SetServoDegrees(-78);

//shake the hamster when got a tweet message from the Agent
agent.on("tweet", ShakeRattleAndRoll);
</code></pre>

<p>And&hellip; that&rsquo;s it! Just connect the Electric Imp to a power source, configure the wifi via the Electric Imp mobile App, and start tweeting! Your hamster will be soo happy to warn you whenever a tweet is coming!!</p>

<p>Ciao and happy hacking!</p>

<p>Stefano</p>