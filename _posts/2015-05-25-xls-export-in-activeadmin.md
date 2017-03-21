---
ID: 119
post_title: XLS Export in ActiveAdmin
author: Diomede Tripicchio
post_date: 2015-05-25 12:29:04
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2015/05/25/xls-export-in-activeadmin/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/119844499939/xls-export-in-activeadmin
tumblr_mikamayhem_id:
  - "119844499939"
---
<p>Today I want to talk about my recent experience with ActiveAdmin and export to XLS.</p>

<p>To obtain an XLS from ActiveAdmin we currently have two simple solutions that allow us to do it in a short time: <a href="https://github.com/randym/activeadmin-axlsx">activeadmin-axlsx</a> and <a href="https://github.com/thambley/activeadmin-xls">activeadmin-xls</a>. The second takes much from the first.</p>

<p>Unfortunately for me, neither of the two was giving me the result that I wanted, therefore I decided to approach the problem from a lower level and use <a href="https://github.com/zdavatz/spreadsheet">spreadsheet</a>.</p>
<!--more-->

<p>Thanks to <strong>spreadsheet</strong> we can have complete control over the file that is generated, though at the price of having to write a little more code.</p>

<p>As the first thing we add <em>spreadsheet</em> to the <em>Gemfile</em></p>

<pre><code class="ruby">gem ‘spreadsheet’
</code></pre>

<p>Next, in order to maintain the folder structure as ordered as possible we create a folder (`app/spreadsheets&rsquo;) where we wil place all the classes that will take care of the generation of XLS files. Then we add to <em>config/application.rb</em>:</p>

<pre><code class="ruby">module TestApp
  class Application &lt; Rails::Application
    …
    config.autoload_paths += %W(#{config.root}/app/spreadsheets)
    …
  end
end
</code></pre>

<p>In this way we can have classes for creating XLS without using <em>require</em>.</p>

<p><strong>MimeType</strong></p>

<p>In order to allow our application to export xls file, we must define the format. To do this we modify the file <em>config/initializers/mime_types.rb</em> and add</p>

<pre><code class="ruby">Mime::Type.register “application/vnd.ms-excel”, :xls
</code></pre>

<p><strong>ActiveAdmin download links</strong></p>

<p>The download links for ActiveAdmin are set in <em>config/initializers/active_admin.rb</em>, in my case I did not need the pdf so I set:</p>

<pre><code class="ruby">ActiveAdmin.setup do |config|
  …
  config.download_links = [:csv, :xml, :xls]
  …
end
</code></pre>

<p>At this point we can set up our controller ActiveAdmin to respond properly to the XLS format. We will take the model <strong>User</strong> as example. We edit the file <em>app/admin/user.rb</em> adding the code needed to handle the format <strong>xls</strong></p>

<pre><code class="ruby">ActiveAdmin.register User do
  …
  controller do
    def index
      …
      index! do |format|
        format.xls {
          spreadsheet = UsersSpreadsheet.new @users
          send_data spreadsheet.generate_xls, filename: “users.xls”
        }
      end
    end
  end
  …
end
</code></pre>

<p><strong>XLS File generation</strong></p>

<p>At this point we are finally able to generate XLS files without constraints.</p>

<p>Create the file <strong>app/spreadsheets/users_spreadsheet.rb</strong></p>

<pre><code class="ruby">class UsersSpreadsheet
  attr_accessor :users
  
  def initialize users
    @users = users
  end

  def generate_xls
    book = Spreadsheet::Workbook.new
    sheet = book.create_worksheet name: “Users”

    create_body sheet

    data_to_send = StringIO.new
    book.write data_to_send
    data_to_send.string 
  end

  def create_body sheet
    # Header row with a specific format
    sheet.row(0).concat %w{Email Username Newsletter}
    sheet.row(0).default_format = Spreadsheet::Format.new weight: :bold

    row_index = 1
    users.each do |user|
      sheet.row(row_index).concat [user.email, user.username, user.newsletter_subscription]
      # sheet.row(row_index).default_format = Spreadsheet.Format.new # define a custom format
      row_index += 1
    end
  end
end
</code></pre>

<p>Note: I’ve used the row_index variable to set more easily the cell format, if needed.</p>

<p>We just have to click on the “XLS” link in the Users index.</p>

<p>At this point we have our xls, simple, but that we can manage as we want through the application&rsquo;s code. We can set the format by entire row or by single cell but we can do more complex things such as printing more rows for record, maybe for handling a relationship. Such customizations were impossible with the two gems mentioned at the beginning.</p>

<p>I leave you with the link of an <a href="https://github.com/zdavatz/spreadsheet/blob/master/GUIDE.md">initial guide</a> on what you can do with Spreadsheet, it is small but on Stack Overflow can find many examples about how to vary many cell formats and settings.</p>

<p>I hope this post has been helpful. </p>

<p>Bye and see you soon!</p>