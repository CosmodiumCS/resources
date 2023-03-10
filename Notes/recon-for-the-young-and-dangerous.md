# Finding your op's internet facing infrastructure 101 (or how I learned to stop worrying and learn to love nmap)

## Crimeware - CosmodiumCS

Q. So why do this?
A. Sometimes your op's web server may be well defended and fully patched, and you don't want to waste too much time doing web app pentesting if you don't need to. Remember find their weakest facing internet servers and punch a hole in them.

Q. Makes sense so how do I be great?
A. How's your basic networking knowledge?

Q. Trash, help me plz?
A. Networking 101, always remember you have two types of IPs, internal and external. If you use ipconfig/ifconfig then the ip you see listed on your terminal is your internal ip. If you go to a website like [this](https://whatismyipaddress.com/) you will see your external IP address, this happens because of a protocol called Network Address Translation (NAT)

Q. Okay but why do I need to know this?
A. Because the same applies to your opposition, to find their servers facing the internet you need to identify EXTERNAL IPS not internal ones. For example internal ips fall into the following ranges: 10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16 anytime you see an IP that falls into these ranges you're looking for an internet facing device then you're searching in the all the wrong places. If you ever have an IP you're unsure about use your handy dandy subnet [calculator](https://www.calculator.net/ip-subnet-calculator.html) to see what range the IP falls in.

Q. That kinda makes sense but how do I apply this?
A. Now that you know all of your opposition's internet facing devices will each have an external IP address, you just need to get an idea of what IP ranges your ops have purchased. Yes thats right, external IP addresses are purchased by companies (or borrowed from your ISP in the case of you as an individual). If you know what ranges a company has purchased then you can use nmap to scan the entire range of your op's IPs and find everything internet facing. Hopefully during the scan you find a piece of crap server that hasn't been upgraded since 1976.

Q. That sounds cool but how would I find the ranges that my ops have purchased?
A. It's not always easy, it will require research on your part but its not impossible, step 1 should always be DNS queries, use the dig command in linux to try to find an A DNS record. For example:

```
dig google.com

; <<>> DiG 9.16.1-Ubuntu <<>> google.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 7933
;; flags: qr rd ad; QUERY: 1, ANSWER: 6, AUTHORITY: 0, ADDITIONAL: 0
;; WARNING: recursion requested but not available

;; QUESTION SECTION:
;google.com.                    IN      A

;; ANSWER SECTION:
google.com.             0       IN      A       64.233.185.100
google.com.             0       IN      A       64.233.185.113
google.com.             0       IN      A       64.233.185.139
google.com.             0       IN      A       64.233.185.102
google.com.             0       IN      A       64.233.185.101
google.com.             0       IN      A       64.233.185.138

;; Query time: 59 msec
;; SERVER: 172.22.192.1#53(172.22.192.1)
;; WHEN: Thu Mar 09 21:53:54 EST 2023
;; MSG SIZE  rcvd: 134
```
In the above snippet you can see that the Domain name google.com has 6 authoritative IP addresses so those would all be internet facing servers you could target.

Q. That seems really easy, is it always going to be that easy?
A. Of course not skiddo, sometimes when you dig a domain name you may get a CNAME record back as a response. Not to mention that most organizations have multiple subdomains, some of these subdomains may be in completely different IP ranges.

Q. So how do I find these other subdomains?
A. There are a couple of different ways but for today I'll teach you two of my favorite, one is to identify subdomains using certificate chains, and another is brute force enumeration. When websites utilize certificate chains to encrypt http traffic into https, they expose public information, you can search this information using sites like [this](https://crt.sh/). In the search bar you can provide a top level domain and use a wildcard to get all the results of registered subdomains using certificate chains. I know it sounds complex but trust me its not, in the website try putting in a random domain like google.com and add the wildcard symbol % to get all the subdomains for google using certificate chains.
```
In the website https://crt.sh/ type the following into the search bar:

%.google.com
```
Each one of the subdomain results you get back is a result you can run the dig command on to build up a list of other IPs to target.
The other way is bruteforcing subdomains using a wordlist and a tool like [gobuster](https://github.com/OJ/gobuster). Since there's a readme for it and it's a well documented tool I'll leave you to explore its usage.

Q. Whoa this is way too much, isn't there an easier way to do this?
A. Yeah theres another shortcut I have for you but, I pointed out those other ways first because you may find devices that are still connected to your op's network that has been abandoned or isn't part of their standard update cycle. Do you know about Autonomous System Numbers (ASNs)?

Q. I don't know, maybe, could you clarify?
A. Here is where we make the leap from cybersec to network engineering and I'll warn you now you have to be careful with this and really know what you're doing so you don't accidently target devices that are out of scope. Typically when an organization purchases its IP ranges from an ISP they also get an Autonomous System Number. This ASN is used by a networking protocol called Border Gateway Protocol (BGP) to advertise routes over the internet. Basically its a way for a router to tell other routers "hey guys if you want to reach any of my private networks I'll share them with you if we're neighbors, you can also tell me about the paths to private networks you know about as well".  For this reason, the routers participating in BGP need a way to uniquely identify each other, hence they have unique ASNs (please keep in mind I'm greatly simplifying here).

Q. Wow I just got like 5% smarter but took a lot of hit point damage just now, now can you please get to your damn point already?
A. The thing is ASN numbers have to be publicly registered meaning you can just search the ASN numbers for specific organizations and domains and get some results.

Q. Is it really that easy?
A. It depends, sometimes searching the ASN and BGP information of an organization may reveal to you the public ip ranges that your opposition has, or sometimes it may reveal to you the public IP ranges that your ops are LEASING from some ISP. At the very least in both cases you will know the ip range that all your ops internet facing infrastructure must live in, this can allow you to fine tune your nmap scanning. But again be careful you are hitting your actual ops and not other organizations who are also leasing public ips from the same ISP your op uses.

Q. Okay so how would I do the search?
A. Easy just google "ASN insert_opps_top_level_dns_name_here.com". I'll show you an example using google:

```
Go to google.com
Type in "asn number google.com"
First result was: https://www.ipqualityscore.com/asn-details/AS15169/google-llc

AS Number is 15169

IP subnets listed were:

8.8.4.0/24
8.8.8.0/24
8.34.208.0/21
8.34.216.0/21
8.35.192.0/21
8.35.200.0/21
23.236.48.0/20
23.251.128.0/19
34.64.0.0/11
34.64.0.0/14
```
As you can see above Google owns 10 blocks of IPs (thats really, really expensive) if you visit the site in the example you can even see that two of the blocks are based in China. Using this information you could scan each of these subnets and map all of the internet facing infrasturcture of Google. Well, to be more accurate everything that you found under the "google.com" domain.

```
nmap 8.8.4.0/24 -sn

Starting Nmap 7.80 ( https://nmap.org ) at 2023-03-09 22:31 EST
Nmap scan report for dns.google (8.8.4.4)
Host is up (0.049s latency).
Nmap done: 256 IP addresses (1 host up) scanned in 26.37 seconds
```
We can see in the above example that in the 8.8.4.0/24 subnet only one host is up the 8.8.4.4.
Then we can scan for what services are running on that host:
```
nmap -sV 8.8.4.4

Starting Nmap 7.80 ( https://nmap.org ) at 2023-03-09 22:38 EST
Nmap scan report for dns.google (8.8.4.4)
Host is up (0.046s latency).
Not shown: 998 filtered ports
PORT    STATE SERVICE    VERSION
53/tcp  open  tcpwrapped
443/tcp open  ssl/https  HTTP server (unknown
```

Q. Wow so I can do that for each of those subnets and find my all my ops infrastructure?
A. Yeah maybe, like I said be careful, you may be looking at subnets owned by the ISP of a particular op, not necessarily the op. Its going to take trial and error, start with DNS enumeration then, start digging deeper into which subnets are associated with said domains.

Q. Thanks Unc!
A. Don't thank me yet, next time we'll talk about more advanced nmap scanning, and network stealth techniques.