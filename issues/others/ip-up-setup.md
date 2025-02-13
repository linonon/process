# Setup ip-up script in macOS

- issue: [https://github.com/adrienverge/openfortivpn/wiki#method-one---short-version---pass-routes-by-pppd-ipparam](https://github.com/adrienverge/openfortivpn/wiki#method-one---short-version---pass-routes-by-pppd-ipparam)
- path: /etc/ppp/ip-up
- don't forget: `chmod a+x /etc/ppp/ip-up`

```sh
#!/bin/sh -e

# Define a function to check if an input is a valid IP address
is_ip() {
    echo $1 | grep -Eq "^([0-9]{1,3}\.){3}[0-9]{1,3}(/([0-9]|[12][0-9]|3[0-2]))?$"
}

# Define a function to resolve URL and add all resolved IPs to routes
function add_route_from_url() {
    local item=$1
    local resolved
	local itemfix
	resolved=$(/sbin/ping -c1 -W1 $item 2>/dev/null | awk -F'[()]' '/PING/{print $2}')
	echo "it's ip: $resolved" >> $logfile
    if [ -z "$resolved" ]; then
		echo "can't resolved $item" > $logfile
    fi
    for item in $resolved; do
        if is_ip $item; then
			if echo $item | grep -q '/'; then
				itemfix=$item
            else
				itemfix=$item/32
            fi
			echo "format ip: $itemfix" >> $logfile
            /sbin/route add $itemfix -interface $2 >> $logfile 2>&1
        else
			echo "not ip, skip, item:$item, 2:$2" >> $logfile
            # add_route_from_url $item $2
        fi
    done
}

now=`date +%Y-%m-%d_%Hh%Mm%Ss`
logfile=/var/log/ip-up.log

echo "" >> $logfile
echo "$0 called at $now with following params: ${6}" >> $logfile
echo "The VPN interface (e.g. ppp0): $1" >> $logfile
echo "Unknown, was 0, in my case: $2" >> $logfile
echo "IP of the VPN server: $3" >> $logfile
echo "VPN gateway address: $4" >> $logfile
echo "Regular (non-vpn) gateway for your lan connections: $5" >> $logfile

for item in ${6}
do
	echo "\r\nitem is: $item" >> $logfile
    if is_ip $item; then
        /sbin/route add $item -interface $1 >> $logfile 2>&1
    else
        add_route_from_url $item $1
    fi
done

echo "./ip-up DONE" >> $logfile

```
