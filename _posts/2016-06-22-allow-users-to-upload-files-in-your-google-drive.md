---
ID: 23
post_title: >
  Allow users to upload files in your
  Google Drive
author: Matteo Revelli
post_date: 2016-06-22 09:01:13
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2016/06/22/allow-users-to-upload-files-in-your-google-drive/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/146299380534/allow-users-to-upload-files-in-your-google-drive
tumblr_mikamayhem_id:
  - "146299380534"
---
Google Drive is a very popular tool to store and synchronize files. In fact, you can utilize this service even without a Google account. I recently discovered an additional feature that allows other users to upload files in my google drive. This is very useful because it simplifies file sharing between the visitors of a blog.

The user will see a simple web form where they can choose the file to be uploaded to a shared online storage drive. There is no need to have a google account, invitations or any form of authorization. This avoids the limitations set by the classic invitational base system; however, a serious disadvantage is that all co-workers can see and remove any files in the shared folder.<!--more-->

<img src="http://68.media.tumblr.com/e2a4010b6fcacbc220e6ccd109bc740e/tumblr_inline_o94i2dbxI51u6zrof_540.png" alt="image" />

Here the code:
<pre><code>  &lt;h3&gt;Upload a file&lt;/h3&gt;
  &lt;form class="main" id="myForm" action="action"&gt;
   &lt;div class="row"&gt;
     &lt;div class="input-field col s12"&gt;
       &lt;input id="myName" type="text" name="myName" class="validate" required="required" aria-required="true"&gt;&lt;label for="myName"&gt;Name&lt;/label&gt;
     &lt;/div&gt;
     &lt;div class="input-field col s12"&gt;
        &lt;input id="email" type="text" name="email" class="validate" required="required" aria-required="true"&gt;&lt;label for="email"&gt;e-mail address&lt;/label&gt;
     &lt;/div&gt;
     &lt;div class="file-field input-field col s12"&gt;
       &lt;div class="btn"&gt;
         &lt;span&gt;pick a file&lt;/span&gt;
         &lt;input id="files" type="file" name="myFile"&gt;&lt;/div&gt;
       &lt;div class="file-path-wrapper"&gt;
         &lt;input class="file-path validate" type="text" placeholder="Seleziona file"&gt;&lt;/div&gt;
     &lt;/div&gt;
   &lt;/div&gt;
&lt;div class="row"&gt;
      &lt;div class="col s4"&gt;
        &lt;button class="waves-effect waves-light btn submit-btn" type="submit"&gt;choose file&lt;/button&gt;
      &lt;/div&gt;
      &lt;div class="col s2"&gt;
        &lt;div class="preloader-wrapper small active"&gt;
          &lt;div class="spinner-layer spinner-green-only spinner"&gt;
            &lt;div class="circle-clipper left"&gt;
              &lt;div class="circle"&gt;&lt;/div&gt;
            &lt;/div&gt;
            &lt;div class="gap-patch"&gt;
              &lt;div class="circle"&gt;&lt;/div&gt;
            &lt;/div&gt;
            &lt;div class="circle-clipper right"&gt;
              &lt;div class="circle"&gt;&lt;/div&gt;
            &lt;/div&gt;
          &lt;/div&gt;
        &lt;/div&gt;
      &lt;/div&gt;
      &lt;div class="col s6"&gt;
        &lt;small id="progress"&gt; &lt;/small&gt;
      &lt;/div&gt;
    &lt;/div&gt;
  &lt;/form&gt;
</code></pre>
Creating this web form logic is very easy. Just go to [script.google.com](<a href="http://script.google.com">http://script.google.com</a>), a JavaScript cloud scripting language that allows you to build web applications (and to automate task across Google products and third party services). The google script will be used to upload the file inside a Drive folder.
<pre><code>function doGet() {
  try {
  return HtmlService.createHtmlOutputFromFile('form')
  .addMetaTag('viewport', 'width=device-width, initial-scale=1')
  .setTitle("File Upload")
  .setSandboxMode(HtmlService.SandboxMode.IFRAME);
  } 
  catch (f) {
    return HtmlService.createHtmlOutput(f.toString());
  }
}

function uploadFileToGoogleDrive(data, fileName, name, email)
{
  try {
    var folder = DriveApp.getFolderById(“0B54vVA4lYrvbaXhfMWxmT1BnbFk”);
    var contentType = data.substring(5,data.indexOf(’;’)),
      bytes = Utilities.base64Decode(data.substr(data.indexOf(‘base64,’)+7)),
      blob = Utilities.newBlob(bytes, contentType, fileName),
      file = folder.createFolder([name, email].join(“ ”)).createFile(blob);
    return “File caricato con successo.”;
    return “ ”+file.getUrl();
    return file.getUrl();
    } 
    catch (f) {
      return f.toString();
    }
}
</code></pre>
The function doGet gives the script permission to upload files. The function <code>uploadFileToGoogleDrive</code> establishes the maximum file size to be loaded. After that, the script will insert the files into a new google drive folder, named with your name and e-mail inserted previously through the form. The function will return the url of the file. The last step will be the deployment as a web app of our application and you can do it from the publish menu.

I tested this feature and it is really useful, because it requires little work and anyone can use it, even those who do not have a google account.