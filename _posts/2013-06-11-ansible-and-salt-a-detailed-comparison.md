---
layout: post
title: "Ansible and Salt: A detailed comparison"
---
The short version is that [Ansible][1] and [Salt][2] are both awesome. Seriously. If you are lucky enough to work for a company that lets you use either one of them, you're going to have a great time. Having said that, there are some key differences in the way they each attempt to solve the problems that are faced by modern sysadmins and developers---and what fun would one of these comparisons be if I didn't tell you my preference at the end? But first, some backstory.

If you haven't heard of them before, Ansible and Salt are frameworks that let you [automate various system tasks][3]. The biggest advantage that they have relative to other solutions like [Chef][4] and [Puppet][5] is that they are capable of handling not only the initial setup and provisioning of a server, but also application deployment, and command execution. This means you don't need to augment them with other tools like [Capistrano][6], [Fabric][7] or [Func][8].

Both Ansible and Salt are capable of taking you from a blank server to a fully-functional application; they can maintain code updates to that application over time; they can quickly run arbitrary commands on an ad-hoc basis; and they can do all of this across hundreds or thousands of different machines. Oh, and they are both built around the concept of using the [YAML serialization format][9] to represent configuration and execute commands. This makes them far more pleasant to work with than the competition, and their concise syntax allows you to use the resulting configuration as a form of documentation that non-programmers can easily understand.

As an experiment, I decided to write a collection of Ansible Roles and Salt States to perform the same set of tasks and configure a brand new Ubuntu 12.04.2 LTS server with the following:

*   [Ruby][10] + the [Falcon patch][11] for improved performance
*   [Nginx][12] + [Passenger][13]
*   [Bundler][14]
*   A few other crucial packages like [git][15] and [NTP][16]

I also wanted Ansible and Salt to handle deploying my [simple Sinatra test application][17]. This meant that they also needed to:

*   Create a 'deploy' user that the application files would belong to
*   Reconfigure OpenSSH to only allow access via SSH keys
*   Add my SSH public key to the deploy user's authorized_keys file
*   Set up and enable an Nginx vhost
*   Create all necessary application directories
*   Use git to checkout the latest revision of the application's codebase
*   Create required symlinks
*   Use bundler to install all Gem dependencies
*   Restart Nginx and be completely ready to go

Mission accomplished. Here is an open source Ansible Playbook that achieves the above, and an open source collection of Salt States that do the same:

> [mazer-rackham][18] - Sample Ansible Playbook for Rack applications
> 
> [salt-rack][19] - Sample Salt States for Rack applications

To make comparison easy, I did my best to match the comments in the Salt States with the corresponding Ansible names. Now, let's talk about some differences.

# Speed

Salt is fast. [0mq][20] is an incredibly slick transport layer. It is very satisfying to send commands and get instantaneous feedback once you have a Salt master connected to several minions. Out of the box, Salt is much faster than Ansible because Ansible relies on SSH as its transport layer by default. However, Ansible also supports what it calls [Fireball Mode][21] which uses SSH to bootstrap an ephemeral 0mq daemon. In my testing, I couldn't see any difference whatsoever between Ansible and Salt when they were both using 0mq, though the initial Ansible Fireball bootstrap process can still take a bit because it happens over SSH.

If you have a workflow where you will routinely need to send simultaneous commands to hundreds upon hundreds of machines, and you cannot afford to wait for Ansible to set up Fireball Mode over SSH, then Salt will be a better fit. Realistically, both Ansible and Salt will probably do just fine for your needs. They are both actively being used in supercomputing clusters with thousands of nodes. Ansible's default SSH transport is also plenty fast and can easily get the job done across hundreds of servers as long as you're comfortable with rolling updates.

# Security

0mq does not natively support encryption, so Salt includes its own AES implementation that it uses to protect its payloads. Recently, [a flaw][22] was discovered in this code along with [several other remote vulnerabilities][23]. Ansible is largely immune to such issues because its default configuration uses standard SSH and does not require any daemons to be running on the remote servers aside from OpenSSH (and I don't think there is a single package in the entire world that I trust more). Regarding Fireball Mode, Ansible's 0mq AES encryption is done using [Keyczar][24] instead of a homegrown solution, is not enabled by default, and its connections are ephemeral.

All of this *drastically* reduces Ansible's attack surface, but it isn't exactly perfect when it comes to security. Ansible uses [paramiko][25] for its SSH connections by default. Paramiko is a fine library with a great track record, but Ansible ships with an overly-permissive configuration that won't warn users if a host's key changes, nor does it prompt for confirmation the first time that a key is seen. Additionally, the documentation contains some [pretty bad advice][26] to turn off StrictHostKeyChecking in order to make remote Mercurial and git checkouts a little more seamless. Fortunately, these issues are easy to work around by using Ansible's binary SSH option (which will check host keys) and by seeding servers with a proper known_hosts file.

Still, you are far more likely to be exploited due to a full-blown remote vulnerability than you are to be exploited due to a MITM attack (and the compromise of a Salt master server is equivalent to gaining full root access on all of the minions that connect to it). Despite its lax policy toward host key verification, Ansible remains the clear winner here.

**Update** - *On July 5, 2013, Ansible 1.2.1 was released. [SSH host keys are now checked][27], and it is more secure than ever before.*

# System Impact

Salt brings in a lot of dependencies. These dependencies must be installed on every machine it is used on, regardless of whether the system is a master or minion. Because of this, you will likely want to enable the Salt Stack apt repository to make installing Salt and keeping it up-to-date as simple as possible. The master and minions will all be running persistent daemons that enable Salt to perform its magic.

Ansible's dependencies are comparatively minimal and only need to be installed on the systems that will be running the ansible and ansible-playbook commands. Its only remote dependency is a Python interpreter, and that comes with almost every Linux distribution by default. Ansible doesn't leave any traces of its existence on remote systems after it finishes running a playbook, and the daemon-less approach allows it to easily run from a local git checkout.

Some people might find these distinctions important, but the differences are largely academic when you're looking at them in terms of system impact (and persistent daemons are a *feature* if you're going to be sending a lot of commands). In my experience, the Salt daemons are very lightweight and well-behaved when they are running. The distinction between daemons vs. daemon-less is important for other reasons, however.

# Maintenance

Ansible is dramatically easier to maintain. It has been less than a month and a half since I first started using Salt and during that time there have been five releases. To be fair, I would only consider one of those upgrades to be mandatory and that's the aforementioned [0.15.1 security release][23]. While it's true that you can easily use Salt to run upgrades across all of your minions, and upgrading the master is a simple apt-get command away, the fact remains that the daemons are an extra layer of moving parts that you simply don't have to deal with in Ansible.

The lack of daemons also makes Ansible easier to use on existing servers. Plus, if you want to use the latest version, you just upgrade it in one location and you're done. The Ansible CHANGELOG is also regularly refreshed so that even if you're running from the development branch (which a lot of people do) it's still easy to keep track of what is going on.

In comparison, I think that it would be wonderful if the Salt project did a better job of publishing Release Notes for their minor updates. As of now, short of digging through the git logs, there is no way to determine what happened in Salt 0.15.2 or 0.15.3. Presumably they contain bug fixes, but the sysadmin in me likes to know what I am getting into before I perform upgrades---especially upgrades to something like Salt that is running as root and that can easily become a core part of your infrastructure.

# Execution Order and Dependency Chains

Salt and Ansible take wildly different approaches to controlling execution order.

[Ordering Salt States][28] is a decidedly complicated affair because it necessitates defining tasks in terms of their requirements. You have several options when working with requisites. You can use require statements, require_in statements, or a mixture of the two. The [Salt documentation for requisites][29] describes the difference like this:

> Requisite_in statements are the opposite [of requisite statements]. Instead of saying "I depend on something", requisite_ins say "Someone depends on me".

Well, you can depend on this to be confusing. Given my options, I found it more intuitive to think about things in terms of what each individual state required and therefore eschewed the usage of requisite_in statements. I had to use a **lot** of requisite statements in order to get Salt to execute things in the correct order. My [salt-rack][30] States contain 34 requisite statements, and that excludes the initial 'require:' line that preceded them in each state. They also contain 5 include statements that are necessary in order for a state file to require states that are located elsewhere.

Perhaps the most aggravating example of requisite issues was when I was upgrading to Passenger 4 from 3.0.19. After making the simple changes to accommodate this, I ran state.highstate on a couple of new minions. It failed the first time, but would succeed if I ran it a second time. So right away I knew that I was dealing with an execution order issue that likely had something to do with a task in the ruby-falcon state file not executing when it should.

It turns out that Passenger had made a subtle change to their install script that caused it to shell out to run a rake task and shell out to run a ruby command at two separate points in the installation process. I discovered this by digging through the Passenger source around where Salt's output was indicating the install process was exiting. The fix was simply to add the final two lines to this state:

{% highlight yaml %}
{% raw %}
# Run the Nginx install script
nginx-install:
  cmd.script:
    - name: salt://nginx-passenger/install.sh
    - unless: /opt/nginx/sbin/nginx -v 2>&1 | grep 1.4.1
    - template: jinja
    - require:
      - file: nginx-source
      - cmd: passenger
      - file: /usr/local/bin/rake
      - file: /usr/local/bin/ruby
{% endraw %}
{% endhighlight %}

Unfortunately, there is no way to tell Salt to execute an entire state file completely before moving onto another. When this problem first occurred, I was briefly tempted to make the nginx-install state require every single state from the ruby-falcon state file. But this didn't exactly feel professional. If my experience is any indication, you will spend quite a bit of time in Salt being frustrated by what often feels like an opaque lottery that randomly determines execution order.

Ansible executes playbooks sequentially. There are no require statements. A task that is listed first will run first; a role that is listed second will run second. It's simple. It's satisfying. It works.

# Clarity

In general, I find Ansible's syntax more readable than Salt's---particularly when it comes to loops. I also really like the ability that Ansible gives you to describe the behavior of a task using the name parameter. I prefer this to Salt's ID declarations that I always felt the need to pair with an additional comment.

For example, here's how I set up log rotation for Nginx and Passenger using Salt:

{% highlight yaml %}
{% raw %}
# Set up log rotation for Nginx and Passenger
{% for rotate_target in 'nginx', 'passenger' %}
/etc/logrotate.d/{{ rotate_target }}:
  file.managed:
    - source: salt://nginx-passenger/{{ rotate_target }}-logrotate
    - require:
      - cmd: nginx-install
      - cmd: passenger
{% endfor %}
{% endraw %}
{% endhighlight %}

And here's the same thing in Ansible:

{% highlight yaml %}
{% raw %}
- name: Set up log rotation for Nginx and Passenger
  copy: src={{ item }}-logrotate
        dest=/etc/logrotate.d/{{ item }}
  with_items:
    - nginx
    - passenger
{% endraw %}
{% endhighlight %}

The [recommended directory layout][31] of Ansible also feels more organized to me. When you refer to a file in Salt you must use an absolute path (you can see this in the example above where the Salt version explicitly references the 'nginx-passenger' location). Some people might (justifiably) prefer Salt's decision here because there's less magic happening behind the scenes, but I think it's extremely powerful to be able to move an Ansible role to a different directory and still have it function properly.

I also find Ansible's concept of [Notify actions and Handlers][32] to be far more elegant than Salt's [watch][33] and [module.wait][34] methods. I mean, look at how lovely this git checkout is in Ansible:

{% highlight yaml %}
{% raw %}
- name: Check out the latest revision of the codebase and notify the proper handlers if there have been updates
  git: repo=https://github.com/jlund/imgur-display.git
       dest={{ imgur_display_location }}/current
  sudo: yes
  sudo_user: deploy
  notify:
    - Symlink the log directory to the shared location
    - Install the bundle
    - Restart the imgur-display application
{% endraw %}
{% endhighlight %}

You know exactly which handlers are going to be notified if the codebase changes, and you can read in plain English what those handlers will do because they have been given descriptive names.

# Conclusion

If you've made it this far, it's probably no surprise to hear that I prefer Ansible at this time. I really meant what I said at the beginning though. Salt and Ansible are both pretty great and it amounts to an embarrassment of riches that I was even able to do a writeup like this. We are fortunate to live in a world where they both exist. Aside from Ansible, I would still rather use Salt than anything else. I plan on trying to stay current with both of them. Things are moving fast and I am excited to see where both projects are in a year from now.

I encourage you to check out my [Ansible Playbook][18] and my [Salt States][19] and see which you prefer. You really can't go wrong either way.

[1]: http://www.ansibleworks.com/ "AnsibleWorks"
[2]: http://saltstack.com/ "Salt Stack"
[3]: http://en.wikipedia.org/wiki/Comparison_of_open-source_configuration_management_software "Comparison of open-source configuration management software"
[4]: http://www.opscode.com/chef/ "Chef"
[5]: https://puppetlabs.com/ "Puppet"
[6]: https://github.com/capistrano/capistrano "Capistrano"
[7]: http://docs.fabfile.org/en/1.6/ "Fabric"
[8]: https://fedorahosted.org/func/ "Func"
[9]: http://en.wikipedia.org/wiki/YAML "YAML"
[10]: http://www.ruby-lang.org/en/
[11]: https://gist.github.com/funny-falcon/4755042
[12]: http://nginx.com/index.html
[13]: https://www.phusionpassenger.com/
[14]: http://gembundler.com/
[15]: http://git-scm.com/
[16]: http://www.ntp.org/
[17]: https://github.com/jlund/imgur-display
[18]: https://github.com/jlund/mazer-rackham
[19]: https://github.com/jlund/salt-rack
[20]: http://www.zeromq.org/ "0mq"
[21]: http://www.ansibleworks.com/docs/playbooks2.html#fireball-mode "Ansible Fireball Mode documentation"
[22]: https://github.com/saltstack/salt/commit/5dd304276ba5745ec21fc1e6686a0b28da29e6fc
[23]: http://docs.saltstack.com/topics/releases/0.15.1.html
[24]: http://www.keyczar.org/ "Keyczar"
[25]: http://www.lag.net/paramiko/
[26]: http://www.ansibleworks.com/docs/modules.html#hg
[27]: http://www.ansibleworks.com/docs/gettingstarted.html#a-note-about-host-key-checking
[28]: http://docs.saltstack.com/ref/states/ordering.html
[29]: http://docs.saltstack.com/ref/states/requisites.html
[30]: https://github.com/jlund/salt-rack "salt-rack"
[31]: http://docs.ansible.com/playbooks_best_practices.html#directory-layout
[32]: http://www.ansibleworks.com/docs/playbooks.html#running-operations-on-change
[33]: http://docs.saltstack.com/ref/states/ordering.html#watch-and-the-mod-watch-function
[34]: http://docs.saltstack.com/ref/states/all/salt.states.module.html#salt.states.module.wait
