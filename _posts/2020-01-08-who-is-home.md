---
title: "who is home?"
excerpt: "all my cousins connect to my internet, which gives me the right to scan the modem and see who is home right now, I guess."
---

I used to live in a not-so-nice neighborhood of Istanbul since I was born, and there would be burglary incidents every once in a while. Also, we were living in a family building where all 3 apartments of the building were occupied by us and my uncles & their families. The thing is, our building had this old rusty external door which had a broken lock; therefore, we were not able to lock the door and we were only locking our apartment doors, but at the same time we were leaving our shoes in front of our apartments, which is pretty common here with family buildings, and this old rusty door caused quite a few pairs of shoes to be stolen throughout this time. After the last incident, we have decided to put a manual sliding lock to the door which would be locked only from the inside and the door wouldn't open from outside even with a key when this lock was locked. The problem with this lock is before we could lock the door with this new manual lock, we had to ensure at least someone from each apartment was home in case someone else comes home so that they could go downstairs, open the lock, let the others in and lock it again, which meant we would leave the door unlocked and hope that whoever comes after 12 AM would lock the door as it was hard to check all the apartments to see if there is someone home just by looking at the shoes.

On a separate topic, I don't know how it happened, but at some point I found myself supporting internet connection for 10+ people from my small dumb modem which was breaking quite often under the load. I wanted to know who is connected at which time, and I also wanted to be able to block some of my young cousins from time to time who were hijacking the bandwidth for all of us by watching Minecraft videos in full HD. I had a very simple Asus modem at the time which looked like a model spaceship. This modem had a very simple interface that was capable of doing very little, but it got the job done and allowed me to see all the connected clients, as well as blocking them.

![[Source](https://www.asus.com/media/global/products/W8ra60WPu6xP3C6T/P_setting_fff_1_90_end_500.png)](/blog/assets/images/who-is-home/modem.png){: .center-image }
<div style="text-align: center;">
<small markdown="1">The modem looked exactly like this one, but I don't remember the exact model name.</small>
</div>

## wtf are you talking about dude?
At the time, just like most (read _all_) of my ideas, I had this idea that would never turn into reality: if I could get the list of all the connected clients of my modem, associate their MAC addresses with who they actually are, and display this list downstairs next to the door using an old phone taped to the wall, I might allow everyone coming home to check this screen to see who is home, and they would be able to decide whether to lock the door. Albeit this is a bit invasive of their privacy, I was more intrigued by the technical challenges it poses, that's why I didn't give a fuck and started building it.

The first challenge was to get the list of clients, and I thought I could scan the ports in the local network to see who is connected to the network using nmap, then I'd get additional information regarding the clients somehow.

### using nmap
Python had nmap bindings through the [python-nmap](https://xael.org/pages/python-nmap-en.html) package, which was dead easy to use:
```python
nm.scan(hosts='192.168.2.0/24', arguments='-sn')
hosts_list = [(x, nm[x]['status']['state']) for x in nm.all_hosts()]
for host, status in hosts_list:
    print('{0}: {1}'.format(host, status))
```

By using this, I was hoping I'd get a list of IPs, but guess what happened? every time I'd run the script, the modem would die and I had to restart it for it to come back to life. I tried a bunch of different options, but I haven't properly used nmap before, and I had no idea what was wrong with the modem as well, that's why I had to find another solution to this.


### using the admin interface of the modem
The second idea was to scrape the admin interface of the modem somehow since the list of clients was available on the parental control section of the interface. Before that, I checked the network requests, and found that it was using HTTP basic to communicate with the backend, which was returning a full HTML response with values embedded into the script section of the HTML. When I checked the HTML body, I found that it contained four main variables that had all the required values in a weird fashion: `leases`, `arps`, `wireless` and `ipmonitor`. Each of these values were holding arrays of arrays, and they were something like this when simplified:
```js
var leases = [["android-4c28aa974083dc66", "40:D3:05:G7:BB:F9", "192.168.2.228"]];	// [hostname, MAC, ip]
var arps = [["192.168.2.22", "40:0E:85:52:D4:43"]];	// [ip, MAC]
var wireless = [["40:F3:08:F7:BB:F9", "Yes", "Yes"]];	// [MAC, associated, authorized]
var ipmonitor = [["192.168.2.12", "3C:BD:D8:D7:E8:17", "3CBDD8D7E817"]];	// [IP, MAC, DeviceName]
```

I am very ignorant when it comes to networking, therefore I don't know what is `leases` or why would these things be stored in different variables like `leases` and `ipmonitor`, each having different properties and details; however, I had the hunch that these were the things I was looking for. While playing around with it, I noticed that the `ipmonitor` variable was always the biggest array which contained all the items that were scattered among other variables, and it seemed to be the source of truth regarding the connected devices. Being the worst python programmer in the world, I decided this was a good job for python.

```python
def get_leases_arps_monitors(contents):
    contents = contents.split('\n')
    leases = []
    arps = []
    ip_monitors = []
    wireless_devices = []

    # Loop over the lines to find items.
    found_count = 0
    for line in contents:
        if 'var leases' in line and not leases:
            leases = line.split("= ", 1)[1]
            leases = leases.split(";")[0]
            leases = json.loads(leases)
            found_count += 1
        elif 'var arps' in line and not arps:
            arps = line.split("= ", 1)[1]
            arps = arps.split(";")[0]
            arps = json.loads(arps)
            found_count += 1
        elif 'var ipmonitor' in line and not ip_monitors:
            ip_monitors = line.split("= ", 1)[1]
            ip_monitors = ip_monitors.split(";")[0]
            ip_monitors = json.loads(ip_monitors)
            found_count += 1
        elif 'var wireless' in line and not wireless_devices:
            wireless_devices = line.split("= ", 1)[1]
            wireless_devices = wireless_devices.split(";")[0]
            wireless_devices = json.loads(wireless_devices)

            found_count += 1

        if found_count == 4:
            break

    return leases, arps, ip_monitors, wireless_devices
```

Once I have the values of these variables, it was easy to build a simple array of dictionaries by looping over them. I built a simple class to do all these, including fetching the HTML response, parsing it and building a list of connected clients.
```python
import base64
import json

import urllib2

class Asus(object):
    def __init__(self, ip, username, password):
        self.ip = ip
        self.username = username
        self.password = password
        self.auth_token = base64.b64encode("%s:%s" % (self.username, self.password))
        self.clients = []

    def get_raw_html(self):
        # Get the request data.
        request = urllib2.Request("http://%s/ParentalControl.asp" % self.ip,
                                  headers={'Authorization': 'Basic %s' % self.auth_token})
        return urllib2.urlopen(request).read()

    @staticmethod
    def find_item_in_list(items, index, value):
        for item in items:
            if item[index] == value:
                return item

        return []

    @staticmethod
    def get_leases_arps_monitors(contents):
        contents = contents.split('\n')
        leases = []
        arps = []
        ip_monitors = []
        wireless_devices = []

        # Loop over the lines to find items.
        found_count = 0
        for line in contents:
            if 'var leases' in line and not leases:
                leases = line.split("= ", 1)[1]
                leases = leases.split(";")[0]
                leases = json.loads(leases)
                found_count += 1
            elif 'var arps' in line and not arps:
                arps = line.split("= ", 1)[1]
                arps = arps.split(";")[0]
                arps = json.loads(arps)
                found_count += 1
            elif 'var ipmonitor' in line and not ip_monitors:
                ip_monitors = line.split("= ", 1)[1]
                ip_monitors = ip_monitors.split(";")[0]
                ip_monitors = json.loads(ip_monitors)
                found_count += 1
            elif 'var wireless' in line and not wireless_devices:
                wireless_devices = line.split("= ", 1)[1]
                wireless_devices = wireless_devices.split(";")[0]
                wireless_devices = json.loads(wireless_devices)

                found_count += 1

            if found_count == 4:
                break

        return leases, arps, ip_monitors, wireless_devices

    def get_clients(self):
        # Get the contents of the result.
        contents = self.get_raw_html()

        # Get the leases and arps from the html response.
        leases, arps, ip_monitors, wireless_devices = self.get_leases_arps_monitors(contents)

        # Loop over the array to construct a dictionary array.
        for ip_monitor in ip_monitors:
            ip = ip_monitor[0]
            mac = ip_monitor[1]
            device_name = ip_monitor[2]
            is_wireless = False

            found = self.find_item_in_list(wireless_devices, 0, mac)
            if found:
                is_wireless = True

            self.clients.append({
                'ip': ip,
                'mac': mac,
                'name': device_name,
                'is_wireless': is_wireless
            })

        return self.clients
```

At the end, I was able to pull the list of clients with two lines of code:
```python
asus = Asus('192.168.2.1', 'admin_username', 'admin_password')
clients = asus.get_clients()
```

The list of items would be something like this, ready to be consumed by the clients:
```json
[
    {
      "is_wireless": false,
      "ip": "192.168.2.12",
      "mac": "DB:45:D8:C3:38:14",
      "name": "burak's desktop computer"
    },
    {
      "is_wireless": true,
      "ip": "192.168.2.228",
      "mac": "50:33:D8:37:CC:F5",
      "name": "burak's phone"
    }
]
```

Up until this point, all was perfect.

## then what?
Since the code to pull the list of clients was ready, it was time to build a simple frontend to display these by polling the backend with `setInterval` continuously, every 5 seconds or so. At this point, all was ready to be built upon, but guess who couldn't keep his calm to not to fuck this up? 

At the back of my mind, I was still thinking about the nmap issue I had before; basically, the modem was not able to handle scans, which gave me the hunch that continuously polling its admin interface would consume its resources and saturate the connection for everyone, and I didn't want that. Did I have any data to back this claim up? No. Did I know anything about how modems work? No. Did I do any proof of concept to test whether this was actually the case? Of course not.

Having a strong faith in my gut feeling and focusing on this non-existent, unrealistic issue, I decided to replace the default firmware of the modem with [OpenWrt](https://openwrt.org/), which would be more stable, would allow me to get more metrics with less code, and simplify the job here. Did I have any data to back this claim up? No. Did I know anything about how modems firmware work? No. Did I do any proof of concept to test whether this was actually the case? Of course not. Once again, all I had was a pure gut feeling.

**At the end, I was replacing the default firmware of my router with a custom open-source firmware to lock a god-damn old door.**

<div style="text-align: center;">
-- INSERT A HUGE FACEPALM HERE --
</div>

### attempting to upgrade the router firmware
I vaguely remember how it happened, but I remember reading the documentation of the router regarding updating the firmware, and the docs had a sentence written in big bold fonts, stating that **if the process somehow fails or gets interrupted in the middle, the device will be dead**. I have read this statement and immediately marked it as irrelevant in my mind as it should be quick and easy to upgrade this small simple device. Once again, did I have any data to back this claim up? No. Did I know anything about how does the upgrade process works? No. Did I do any proof of concept to test whether this was actually the case? **OF FUCKING COURSE NOT.**

And guess what happened? **SOMETHING HAPPENED, AND THE ROUTER DECIDED TO RESTART ITSELF IN THE MIDDLE OF THE UPGRADE.** Essentially, I bricked my perfectly fine router for a project of locking a rusty old door that would never come to life even though the router worked; you can insert another huge facepalm here.

----

## I have to conclude
After all this work, all I had was a simple Python script that was supposed to work against this specific Asus router which was already dead. Eventually, after having no internet connection for around 3 hours, I bought another router and got everything working again. As a result of this, I had:
- a Python script for pulling the clients connected to my Asus router
- a dead Asus router
- a brand new router
- a demotivated CS student (*hint: this is me*)

Overall, this was a fun failed experiment that cost me a router, but it was fun, which was the main reason I started working on this. As an  additional cost, my family made fun of me a little bit once I explained why there was no internet connection for a couple of hours, but that was fine, I deserved that.

If you think of building something similar, I'd suggest replacing the lock of the door somehow so that everyone could use good old keys instead of a complicated setup like I planned; it would probably be cheaper to replace the lock than to buy a new router.