---
layout: post
title: Synchronizing Email with Ruby
categories: [ruby]
---

Recently, I was in the midst of a Windows 7 installation when my company decided to migrate my email to a new mail server. As we in the IT world are aware, migrations rarely go as planned. It seemed like as good a time as any for me to start a project I have long considered: migrating all of my email to Gmail.

I guess this is technically something I'm not supposed to do. Then again, it is no less secure than downloading and sending email using my local laptop and a standard email client (provided the passwords/accounts are properly encrypted). Either way, I love Gmail for personal email and there is no way my entire work organization is going to switch to Gmail, so I decided to set up my own little synchronization process.

Here is what I did:

1.) [Enable IMAP on my Gmail account][1]. My work email is already IMAP, so this let me drag and drop folders from one mail account to another using Thunderbird. Once all my folders were migrated, I only had to worry about new email in my inbox.

2.) Set up a synchronization process from my work email to my Gmail account. Transferring mail itself is pretty simple. There is an [RFC that defines what mail messages look like][2], so they are the same data from one mail account to the other. The trick is moving them automatically.

Since I already have a Linux host that runs full time (this site), it seemed like my most sane option would be to write something that I could schedule using [cron][3]. Since I am a member of the [cargo cult][4], I thought I could pretty easily find something on Google written in Java.

After some searching, though, it seemed like the best and simplest examples were all written in Ruby. Unfortunately, none of them did exactly what I wanted so I figured I would have to write a bit of code. Before I began this endeavor, I knew nothing about Ruby (yes, I am way behind), but it seemed like a good time to learn.

I started off with some setup: 

[http://wiki.dreamhost.com/Ruby][5]

[http://wiki.dreamhost.com/index.php/Ruby\_on\_Rails][6]

Next I went with a few blogs/docs and some source from the beginnings of [larch][7]:

[http://ruby-doc.org/stdlib/libdoc/net/imap/rdoc/classes/Net/IMAP.html][8]

[http://wonko.com/post/ruby\_script\_to\_sync\_email\_from\_any\_imap\_server\_to\_gmail][9]

[http://codeclimber.blogspot.com/2008/06/using-ruby-for-imap-with-gmail.html][10]

While I like larch, it doesn't delete from my inbox, has way more features than I need, and it is more object oriented (and thus harder to understand) than I would like. Since I am a Ruby novice, I wanted something simple that I could make sure was working. Here is what I came up with:

    #! ~/run/bin/ruby
    require 'net/imap'
    
    puts "Synchronizing mailboxes..."
    
    # create destination imap
    dest = Net::IMAP.new("imap.gmail.com",993,true)
    dest.login("my.account@gmail.com", "password")
    
    # create source imap
    source = Net::IMAP.new("imap.work.com",993,true)
    source.login("my.account@work.com", "password")
    
    puts "Logins complete. Checking source mailbox for mail."
    
    source.select('INBOX')
    source.search(["NOT", "DELETED"]).each do |message_id|
            puts "Found message: #{message_id}"
            msg = source.fetch(message_id, ['RFC822', 'FLAGS',
                      'INTERNALDATE'])[0]
    
            puts "Transferring message with id: #{message_id}"
            dest.append('INBOX', msg.attr['RFC822'], msg.attr['FLAGS'],
                msg.attr['INTERNALDATE'])
    
            puts "Deleting message from source inbox."
            source.store(message_id, "+FLAGS", [:Deleted])
    end
    puts "Transfer complete. Logging out."
    
    source.logout()
    source.disconnect()
    
    dest.logout()
    dest.disconnect()
    
    puts "done."

Simple, huh? I set this up to run as a cron job every ten minutes and that's all it took.

 [1]: http://mail.google.com/support/bin/answer.py?hl=en&amp;answer=75725
 [2]: http://www.faqs.org/rfcs/rfc822.html
 [3]: http://en.wikipedia.org/wiki/Cron
 [4]: http://en.wikipedia.org/wiki/Cargo_cult_programming
 [5]: http://wiki.dreamhost.com/Ruby "http://wiki.dreamhost.com/Ruby"
 [6]: http://wiki.dreamhost.com/index.php/Ruby_on_Rails "http://wiki.dreamhost.com/index.php/Ruby_on_Rails"
 [7]: http://github.com/rgrove/larch
 [8]: http://ruby-doc.org/stdlib/libdoc/net/imap/rdoc/classes/Net/IMAP.html "http://ruby-doc.org/stdlib/libdoc/net/imap/rdoc/classes/Net/IMAP.html"
 [9]: http://wonko.com/post/ruby_script_to_sync_email_from_any_imap_server_to_gmail "http://wonko.com/post/ruby_script_to_sync_email_from_any_imap_server_to_gmail"
 [10]: http://codeclimber.blogspot.com/2008/06/using-ruby-for-imap-with-gmail.html "http://codeclimber.blogspot.com/2008/06/using-ruby-for-imap-with-gmail.html"  