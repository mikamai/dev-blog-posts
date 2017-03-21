---
ID: 193
post_title: Mp3 addiction
author: Emanuele Serafini
post_date: 2014-10-27 14:32:39
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2014/10/27/mp3-addiction/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/101088267469/mp3-addiction
tumblr_mikamayhem_id:
  - "101088267469"
---
<p>I’ve recently developed a tool that helps in keeping your music library clean and organized.</p>
<p><img alt="image" src="http://68.media.tumblr.com/2e41abab03df6066cc9012337394b50d/tumblr_inline_ne3pm2IDBs1qfdn20.jpg" /></p>
<p>It allows you to rename and reorganize all your music library in batch using your mp3 tags.</p>
<p>Of course the tool works best only if your mp3 files have tags complete with <span>all the details such as artist, album, year, track number… </span></p>
<p>The script is based on the <a href="https://github.com/moumar/ruby-mp3info">mp3info</a> gem; this gem can read information and manipulate mp3 tags.</p>
<p>My tool examines all mp3 files contained into <code>base_source_dir</code> folder, it moves them into the <code>base_target_dir</code> folder organizing them following the user defined structure according to <code>dir</code> method and rename files according to <code>filename</code> method. <a href="https://gist.github.com/emaserafini/64ae2c24a5f1c77246cd">Here&rsquo;s the code</a>:</p>
<pre><code>#!/usr/bin/env ruby

require 'rubygems'
require 'mp3info'
require 'titleize'
require 'fileutils'
require 'ruby-progressbar'


def base_source_dir
  '/Volumes/Shared/Music/Tmp'
end

def base_target_dir
  '/Volumes/Shared/Music/Temporary'
end

def invalid_character
  Regexp.new('[|/\\&lt;&gt;:"?*;,^]')
end

class Mp3
  attr_accessor :source_path, :artist, :album, :title, :year, :track_n, :disc_n

  def initialize(path)
    @source_path = path
    get_metadata
  end

  def self.all
    Dir.glob("#{base_source_dir}/**/*.mp3").map do |path|
      new(path)
    end
  end

  def filename
    "#{escape(artist)} - [#{year}] #{escape(album)} - #{leading_0(disc_n)}.#{leading_0(track_n)} - #{escape(title)}.mp3"
  end

  def dir
    "#{escape(artist)}/[#{year}] #{escape(album)}"
  end

  def target_dir
    "#{base_target_dir}/#{dir}"
  end

  def target_path
    "#{target_dir}/#{filename}"
  end

  private

  def get_metadata
    Mp3Info.open(source_path) do |mp3info|
      @artist  = mp3info.tag.artist
      @album   = mp3info.tag.album.titleize
      @title   = mp3info.tag.title.titleize
      @year    = mp3info.tag.year || mp3info.tag1.year || mp3info.tag2.TDRC
      @track_n = mp3info.tag.tracknum || 1
      @disc_n  = mp3info.tag2.disc_number || 1
    end
  end

  def escape(tag)
    tag.gsub(invalid_character, '_')
  end

  def leading_0(tag)
    sprintf '%02d', tag
  end
end

class Mp3Renamer
  attr_accessor :source_path, :target_path, :target_dir

  def initialize(mp3)
    @source_path = mp3.source_path
    @target_dir = mp3.target_dir
    @target_path = mp3.target_path
  end

  def perform
    prepare_folder
    FileUtils.mv(source_path, target_path)
  end

  private

  def prepare_folder
    FileUtils.mkdir_p target_dir
  end
end

mp3s = Mp3.all
bar = ProgressBar.create total: mp3s.count

FileUtils.mkdir_p base_target_dir
mp3s.each do |mp3|
  Mp3Renamer.new(mp3).perform
  bar.increment
end
puts "#{mp3s.count} mp3 processed!"
</code></pre>
<p>Here’s my organization method, enjoy your listening!</p>
<p><img alt="image" src="http://68.media.tumblr.com/5130249b7035184dd8befa4f32e26ff4/tumblr_inline_ne3pmnPBtt1qfdn20.jpg" /></p>