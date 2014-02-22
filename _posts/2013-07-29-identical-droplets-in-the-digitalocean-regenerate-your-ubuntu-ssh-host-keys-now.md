---
layout: post
title: "Identical Droplets in the DigitalOcean: Regenerate your Ubuntu SSH Host Keys now!"
---
When I started working on my [Ansible and Salt comparison][1], I knew that I was going to be cycling through tons of virtual machines. Micro instances at AWS are only good at making you feel sad, so I decided to give [DigitalOcean][2] a try. I had heard great things about their performance and price, and after using them for the past month and a half it's very easy for me to understand their meteoric rise in popularity. But in the midst of testing everything, I noticed something strange. I was rapidly provisioning and destroying droplets, and most of the time the new instances were coming back on the same set of IP addresses. I soon realized that I was not being prompted to accept new host keys, nor was I ever being warned that a server's identification had changed. So I started up five brand new Ubuntu 12.04.2 64-bit instances. I put two in San Francisco, two in New York, and one in Amsterdam just for good measure:

![Host key flaw 1](/assets/2013-07-29-identical-droplets-in-the-digitalocean-regenerate-your-ubuntu-ssh-host-keys-now/ssh-host-key-flaw-1.png)

Once they were up and running, I used \`ssh-keyscan\` to check their host keys. This was the result:

![Host key flaw 2](/assets/2013-07-29-identical-droplets-in-the-digitalocean-regenerate-your-ubuntu-ssh-host-keys-now/ssh-host-key-flaw-2.png)

All of the servers had identical fingerprints. The fingerprints from another set of servers on a separate DigitalOcean account were also the same.

# Why this is a big deal

SSH host keys protect users from [man-in-the-middle attacks][5]. The idea is to make it impossible for an adversary to secretly impersonate your server or intercept your traffic because they won't be able to match your server's unique cryptographic fingerprint. These fingerprints get cached in your known_hosts file, and if one of them changes unexpectedly then your SSH client can alert you with its infamous capitalized warning that "IT IS POSSIBLE THAT SOMEONE IS DOING SOMETHING NASTY!" This allows you to tell the duplicitous barista that their cunning plan to infiltrate your machines has failed, and proudly walk away from the sketchy coffee shop in disgust. In the absence of these warnings, the protection provided by SSH is severely compromised. The OpenSSH [man pages][6] state that the known_hosts file is used to "prevent server spoofing or man-in-the-middle attacks, which could otherwise be used to circumvent the encryption." The fact that these droplets were truly identical, including their SSH host keys, meant that an attacker could easily spoof another user's DigitalOcean server without raising any red flags. Thankfully, DigitalOcean has fixed the issue and new droplets are now being created with unique host keys. If you are a DigitalOcean customer and you haven't done so already, now would be a good time to regenerate your host keys.

# How to generate new host keys on an existing server

First, remove your old host keys:

    sudo rm -rf /etc/ssh/ssh_host_*
    

Next, generate new host keys using ssh-keygen:

    sudo ssh-keygen -A
    

If you are using Debian or Ubuntu, you can also accomplish the same thing by reconfiguring the openssh-server package. Make sure you remove the old keys first:

    sudo dpkg-reconfigure openssh-server
    

After you have run those commands, simply restart the SSH daemon so it starts up with the new keys in place:

    sudo service ssh restart
    

Finally, you'll need to remove the server's old keys from your local known_hosts file, otherwise you will see a warning telling you that the key has changed:

    ssh-keygen -f "/home/[your-username-here]/.ssh/known_hosts" -R [your-server's-ip-here]
    

Now you're ready to connect to the server, and accept its new (unique!) host keys.

# Timeline

**June 13, 2013** * Email is sent to the DigitalOcean security team * The issue is acknowledged * Some server images are updated with fixes in place

**June 14, 2013** * Testing reveals some cases where droplets are still being provisioned with identical keys * Follow-up email is sent

**June 20, 2013** * Further fixes are pushed, and DigitalOcean begins the process of working with existing customers to update their keys

**July 2, 2013** * I confirm that the previously-affected images are all correctly generating new keys

**July 26, 2013** * DigitalOcean publishes their [advisory][7]

**July 29, 2013** * I publish this post

# Final thoughts

I haven't had time to thoroughly explore the issue, but I suspect that this problem might affect other VPS providers as well. It would also be easy to fall into the same trap if you are using disk images to rapidly provision new hardware. Take some time to make sure that all of your servers have unique fingerprints. It's a critical part of ensuring that your SSH traffic remains secure. Thanks to the

[DigitalOcean Security Team][8] for their quick and responsible handling of this issue. They were very pleasant to work with throughout the entire process and I remain a happy customer.

 [1]: http://missingm.co/2013/06/ansible-and-salt-a-detailed-comparison/ "Ansible and Salt: A detailed comparison"
 [2]: https://www.digitalocean.com/ "DigitalOcean"
 [3]: http://missingm.co/wp-content/uploads/2013/07/ssh-host-key-flaw-1.png
 [4]: http://missingm.co/wp-content/uploads/2013/07/ssh-host-key-flaw-2.png
 [5]: http://en.wikipedia.org/wiki/Man_in_the_middle_attack "Wikipedia: Man-in-the-middle attack"
 [6]: http://openssh.com/manual.html "OpenSSH Manual Pages"
 [7]: https://www.digitalocean.com/blog_posts/avoid-duplicate-ssh-host-keys "Avoid Duplicate SSH Host Keys"
 [8]: https://www.digitalocean.com/security
