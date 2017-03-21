---
ID: 258
post_title: (Physically) Printing with Ruby!
author: Andrew Coates
post_date: 2014-04-16 14:31:00
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2014/04/16/physically-printing-with-ruby/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/82894788234/physically-printing-with-ruby
tumblr_mikamayhem_id:
  - "82894788234"
---
Recently, we were asked to develop an application that would silently print to a physical printer (in this case, a Zebra ZR203) on a Windows computer. The aim of the program was to use it to print coupons in conjunction with a touch screen computer.
<br /><br />
One way we considered doing this was by using the <a href="https://en.wikipedia.org/wiki/Zebra_%28programming_language%29">Zebra Programming Language</a> (ZPL) to create the label, connect the Zebra printer to the LAN, and sending it to TCP port 9100 on the printer. However, this method could get very messy when deploying the finished product to dozens of locations. We needed the setup to be as trouble-free as possible.
<br /><br />
I found <a href="http://www.foxitsoftware.com/Secure_PDF_Reader/">Foxit</a>, a (Windows only) PDF reader, which conveniently allows for command line printing. Together with the great PDF generator gem, <a href="https://github.com/prawnpdf/prawn">Prawn</a>, I generated a PDF from the coupon PNG image:

<pre>
Prawn::Document.generate("coupon.pdf", :page_size =&gt; [225, 1132], :margin =&gt; [0,0,0,0]) do |pdf|
    pdf.image("coupon.png", :fit =&gt; [225, 1132])
end
</pre>

 and executed a system command using Ruby&rsquo;s &ldquo;system&rdquo; command:

<pre>
	system(""C:\Program Files (x86)\Foxit Software\Foxit Reader\Foxit Reader.exe" /t "coupon.pdf"")
</pre>

Of course this command depends on the install location of Foxit, but Program Files (x86) is the default location. This command will print to the current default printer, but you can specifiy a printer by appending its name to the end of the command like so:

<pre>
   system(""C:\Program Files (x86)\Foxit Software\Foxit Reader\Foxit Reader.exe" /t "coupon.pdf" "#{printerName}" ")
</pre>

Finally, we had this small application running locally with Sinatra, and had the frontend perform Ajax GET requests to the backend:

<pre>
$.get('http://localhost:4567/print', (data) -&gt;)
</pre>

And that&rsquo;s it!