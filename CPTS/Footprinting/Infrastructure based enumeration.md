## Online Presence 
One of the first sources to check the online presence of the company is SSL Certificates. You can check these certs using [crt.sh](https://crt.sh), we also prettify the output using json from the terminal like :
```shell
curl 'https://crt.sh?q=example.com&output=json' | jq
```
we can also extract the subdomains only like:
```shell
curl -s "https://crt.sh/?q=example.com&output=json" | jq . | grep name | cut -d":" -f2 | grep -v "CN=" | cut -d'"' -f2 | awk '{gsub(/\\n/,"\n");}1;' | sort -u
```
we can also filter the hostnames that belong to the target company like :
```shell
for i in $(cat subdomainlist);do host $i | grep "has address" | grep example.com | cut -d" " -f1,4;done
```
then we can extract the IPs then pass them to shodan for passive enumeration:
```shell
for i in $(cat subdomainlist);do host $i | grep "has address" | grep example.com | cut -d" " -f4 >> ip-addresses.txt;done
```
```shell
for i in $(cat ip-addresses.txt);do shodan host $i;done
```
## Cloud Resources
Alot of companies nowadays have a part of there infrastructure on the cloud services such as AWS, Azure, Cloud Storage. usually these can be accessed by a url,  we can use the above command to extract subdomains and look for ones with the criteria or use google dorking with the inurl and intext filters
- AWS ; has 's3' and a location like 'us-west'. Use inurl:amazonaws.com for dorking
- Azure ; has windows.net. Use inurl:blob.core.windows.net for dorking
There are two great resources to gather information about the cloud infrastructure of the company,  [domain.glass](https://domain.glass/) & [GrayHatWarfare](https://buckets.grayhatwarfare.com/) which can discover files on the server. Some companies use abbreviations for the company name which is a good detail that might come in handy in the future