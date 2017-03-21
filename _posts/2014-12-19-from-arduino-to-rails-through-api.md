---
ID: 170
post_title: From arduino to rails through api
author: Emanuele Serafini
post_date: 2014-12-19 09:35:49
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2014/12/19/from-arduino-to-rails-through-api/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/105596485149/from-arduino-to-rails-through-api
tumblr_mikamayhem_id:
  - "105596485149"
---
<p>Some years ago I used pachube.com / cosm.com to save a stream of data generated from a sensor attached to an Arduino board. Unfortunately  this service is no more available, but you can build your own data stream repository with rails and heroku.</p>
<p>In this example we read temperatures from an Arduino sensor and then we save data via API using a rails app.</p>
<p>There are several guides to build REST APIs using rails. This tutorial integrates some best practices but you need to focus your work on a smaller subset, depending on your need. Rails and JSON give you the perfect combination for a fast and comprehensive developement.</p>
<h4>Rails api controllers</h4>
<pre><code># /app/api/temperatures_controller.rb

module Api
  class TemperaturesController &lt; ActionController::Base
    protect_from_forgery
    skip_before_action :verify_authenticity_token, if: :json_request?

    def index
      temperatures = Temperature.all
      if temperatures.any?
        render json: temperatures
      else
        render json: {}, status: :not_found
      end
    end

    def show
      temperature = Temperature.find(params[:id]) rescue nil
      if temperature
        render json: temperature
      else
        render json: {}, status: :not_found
      end
    end

    def create
      temperature = Temperature.new(temperature_params)
      if temperature.save
        render json: temperature, location: api_temperature_path(temperature.id), status: :created
      else
        render json: { errors: temperature.errors }, status: :unprocessable_entity
      end
    end

    ...
    ...

    protected

    def json_request?
      request.format.json?
    end


    private

    def temperature_params
      params.require(:temperature).permit(:value)
    end
  end
end
</code></pre>
<h4>Test locally</h4>
<p>During your development you can test your API with curl, here are some examples for saving and retrieving JSON data feeds:</p>
<pre><code># create
curl -v loalhost:3000/api/temperatures -X POST 
     -H "Accept: application/json" 
     -H "Content-Type: application/json" 
     -d '{"temperature": {"value": 20}}'

# index
curl -v loalhost:3000/api/temperatures

# show
curl -v loalhost:3000/api/temperatures/1
</code></pre>
<p>Your goal now is to create a POST request from Arduino just like the one generated with curl.</p>
<h4>Arduino post request</h4>
<p>To send data to a rails app you need an Anduino Ethernet Shield + Arduino or an Arduino Yun. In this example we are using an Anduino Ethernet Shield and we run on it this code:</p>
<pre><code>#include &lt;SPI.h&gt;
#include &lt;Ethernet.h&gt;

// assign a MAC address for the ethernet controller.
// Newer Ethernet shields have a MAC address printed on a sticker on the shield
// fill in your address here:
byte mac[] = {0xDE, 0xAD, 0xBE, 0xEF, 0xFE, 0xED};

// fill in an available IP address on your network here,
// for manual configuration:
IPAddress ip(192,168,1,10);
// initialize the library instance:
EthernetClient client;

char server[] = "tantiauguriatutti.herokuapp.com";   

unsigned long lastConnectionTime = 0;       
boolean lastConnected = false;              
const unsigned long postingInterval = 60000;

float temperature;  
int reading;  
int lm35Pin = 5;
float referenceVoltage;

void setup() {
  // Open serial communications and wait for port to open:
  Serial.begin(9600);

  analogReference(INTERNAL);
  referenceVoltage = 1.1;

  // start the Ethernet connection:
  if (Ethernet.begin(mac) == 0) {
    Serial.println("Failed to configure Ethernet using DHCP");
    // DHCP failed, so use a fixed IP address:
    Ethernet.begin(mac, ip);
  }
}

void loop() {
  // if there's incoming data from the net connection.
  // send it out the serial port.  This is for debugging
  // purposes only:
  if (client.available()) {
    char c = client.read();
    Serial.print(c);
  }

  // if there's no net connection, but there was one last time
  // through the loop, then stop the client:
  if (!client.connected() &amp;&amp; lastConnected) {
    Serial.println();
    Serial.println("disconnecting.");
    client.stop();
  }

  // if you're not connected, and ten seconds have passed since
  // your last connection, then connect again and send data:
  if(!client.connected() &amp;&amp; (millis() - lastConnectionTime &gt; postingInterval)) {
    reading = analogRead(lm35Pin);
    temperature = (referenceVoltage * reading) / 1023;
    sendData(temperature);
  }
  // store the state of the connection for next time through
  // the loop:
  lastConnected = client.connected();
}

// this method makes a HTTP connection to the server:
void sendData(int thisData) {
  // if there's a successful connection:
  String JsonData = "{"temperature": {"value": ";
  JsonData = JsonData + thisData;
  JsonData = JsonData + "}}";
  if (client.connect(server, 80)) {
    Serial.println("connecting...");
    // send the HTTP PUT request:
    client.println("POST /api/temperatures HTTP/1.1");
    client.println("Host: tantiauguriatutti.herokuapp.com");
    client.println("User-Agent: Arduino/1.0");
    client.println("Accept: application/json");
    client.print("Content-Length: ");
    client.println(JsonData.length());

    // last pieces of the HTTP PUT request:
    client.println("Content-Type: application/json");
    client.println("Connection: close");
    client.println();
    client.println(JsonData);
  } 
  else {
    // if you couldn't make a connection:
    Serial.println("connection failed");
    Serial.println();
    Serial.println("disconnecting.");
    client.stop();
  }
   // note the time that the connection was made or attempted:
  lastConnectionTime = millis();
}
</code></pre>
<p>The example above shows how to manage ethernet configuration, transform reading from analog input sensor and how to send JSON data to our Rails app.</p>
<p>Keep in mind it is possible to use another board to collect and send data, for example <a href="https://www.spark.io/">Spark Core</a> and <a href="https://electricimp.com/">Electric Imp</a> are perfect beacause already integrated with WiFi module.</p>