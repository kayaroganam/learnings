# Networking Core Protocols


| Protocol | Transport Protocol | Default Port Number |
|---------|-------------------|--------------------|
| TELNET  | TCP               | 23                 |
| DNS     | UDP or TCP        | 53                 |
| HTTP    | TCP               | 80                 |
| HTTPS   | TCP               | 443                |
| FTP     | TCP               | 21                 |
| SMTP    | TCP               | 25                 |
| POP3    | TCP               | 110                |
| IMAP    | TCP               | 143                |


## **DHCP**
### Release The DHCP
```bash
sudo dhclient -r <interface>
```
### Capture DHCP Packets
```bash
sudo tshark -Y dhcp
```
### Renew IP
```bash
sudo dhclient
```

## **WEB with telnet**
input
```bash
┌─[kayarogan@parrot]─[~]
└──╼ $telnet 10.48.141.166 80 #command
Trying 10.48.141.166...
Connected to 10.48.141.166.
Escape character is '^]'.

GET /flag.html HTTP/1.1 #input
Host: anything #input
```
output
```bash
HTTP/1.1 200 OK
Server: nginx/1.18.0 (Ubuntu)
Date: Sun, 15 Feb 2026 17:20:45 GMT
Content-Type: text/html
Content-Length: 478
Last-Modified: Thu, 27 Jun 2024 07:28:15 GMT
Connection: keep-alive
ETag: "667d148f-1de"
Accept-Ranges: bytes

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    ....
<body>
    <div class="hidden-text">THM{TELNET-HTTP}</div>
</body>
</html>
```

## **FTP**
Default port: 20
* `USER` is used to input the username
* `PASS` is used to enter the password
* `RETR` (retrieve) is used to download a file from the FTP server to the client.
* `STOR` (store) is used to upload a file from the client to the FTP server.

### Retrive file from FTP server
```bash
┌─[✗]─[kayarogan@parrot]─[~]
└──╼ $ftp 10.48.141.166
Connected to 10.48.141.166.
220 (vsFTPd 3.0.5)
Name (10.48.141.166:kayarogan): anonymous # default ftp user
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
```
```bash
ftp> ls
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
-rw-r--r--    1 0        0            1480 Jun 27  2024 coffee.txt
-rw-r--r--    1 0        0              14 Jun 27  2024 flag.txt
-rw-r--r--    1 0        0            1595 Jun 27  2024 tea.txt
226 Directory send OK.
```
```bash
ftp> type ascii
200 Switching to ASCII mode.
```
```bash
ftp> get flag.txt
200 PORT command successful. Consider using PASV.
150 Opening BINARY mode data connection for flag.txt (14 bytes).
WARNING! 1 bare linefeeds received in ASCII mode
File may not have transferred correctly.
226 Transfer complete.
14 bytes received in 0.000987 seconds (13.9 kbytes/s)
```
```bash
ftp> quit
221 Goodbye.
┌─[kayarogan@parrot]─[~]
└──╼ $ls
flag.txt  Desktop  DHCP-G5000.pcap  Documents  Downloads  Exercise.pcapng Music  Pictures  Public  Templates  Videos
```
## wireshark packets
input
```bash
┌─[✗]─[kayarogan@parrot]─[~]
└──╼ $sudo tshark -Y ftp -i tun0
```
output
```bash
4 0.075988949 10.48.141.166 → 192.168.219.202 FTP 72 Response: 220 (vsFTPd 3.0.5)
6 2.221101704 192.168.219.202 → 10.48.141.166 FTP 68 Request: USER anonymous
8 2.263352844 10.48.141.166 → 192.168.219.202 FTP 86 Response: 331 Please specify the password.
10 2.653105344 192.168.219.202 → 10.48.141.166 FTP 59 Request: PASS
11 2.705344568 10.48.141.166 → 192.168.219.202 FTP 75 Response: 230 Login successful.
13 2.707475577 192.168.219.202 → 10.48.141.166 FTP 58 Request: SYST
14 2.741276638 10.48.141.166 → 192.168.219.202 FTP 71 Response: 215 UNIX Type: L8
16 4.814273637 192.168.219.202 → 10.48.141.166 FTP 81 Request: PORT 192,168,219,202,219,69
17 4.851553422 10.48.141.166 → 192.168.219.202 FTP 103 Response: 200 PORT command successful. Consider using PASV.
19 4.852949011 192.168.219.202 → 10.48.141.166 FTP 58 Request: LIST
23 4.921060344 10.48.141.166 → 192.168.219.202 FTP 91 Response: 150 Here comes the directory listing.
29 4.957834044 10.48.141.166 → 192.168.219.202 FTP 76 Response: 226 Directory send OK.
31 14.186231774 192.168.219.202 → 10.48.141.166 FTP 60 Request: TYPE A
32 14.220334754 10.48.141.166 → 192.168.219.202 FTP 82 Response: 200 Switching to ASCII mode.
34 22.277459049 192.168.219.202 → 10.48.141.166 FTP 81 Request: PORT 192,168,219,202,139,97
35 22.319729065 10.48.141.166 → 192.168.219.202 FTP 103 Response: 200 PORT command successful. Consider using PASV.
37 22.321365655 192.168.219.202 → 10.48.141.166 FTP 67 Request: RETR flag.txt
41 22.388684821 10.48.141.166 → 192.168.219.202 FTP 118 Response: 150 Opening BINARY mode data connection for flag.txt (14 bytes).
48 22.433714932 10.48.141.166 → 192.168.219.202 FTP 76 Response: 226 Transfer complete.
50 27.443061909 192.168.219.202 → 10.48.141.166 FTP 58 Request: QUIT
51 27.483837585 10.48.141.166 → 192.168.219.202 FTP 66 Response: 221 Goodbye.
```

## **SMTP: Sending Email using telnet**
Default port: 25
Some common SMTP commands are:
* `HELO` or `EHLO` initiates an SMTP session
* `MAIL FROM` specifies the sender’s email address
* `RCPT TO` specifies the recipient’s email address
* `DATA` indicates that the client will begin sending the content of the email message
* ` . ` is sent on a line by itself to indicate the end of the email message

```bash
┌─[kayarogan@parrot]─[~]
└──╼ $telnet 10.49.128.47 25
Trying 10.49.128.47...
Connected to 10.49.128.47.
Escape character is '^]'.
220 ip-10-49-128-47.ap-south-1.compute.internal ESMTP Exim 4.95 Ubuntu Mon, 16 Feb 2026 02:47:39 +0000
HELO client.thm #input
250 ip-10-49-128-47.ap-south-1.compute.internal Hello client.thm [192.168.219.202]
MAIL FROM: <user@client.thm> #input
250 OK
RCPT TO: <strategos@server.thm> #input
250 Accepted
DATA #input
354 Enter message, ending with "." on a line by itself
From: user@client.thm #input
To: strategos@server.thm #input
Subject: Telnet email #input

Hello. I am using telnet to send you an email! #input
. #input
250 OK id=1vroeT-0000Of-JL
QUIT #input
221 ip-10-49-128-47.ap-south-1.compute.internal closing connection
Connection closed by foreign host.
```

## **POP3: Receiving Email**
Default port: 110
Some common POP3 commands are:
* `USER` <username> identifies the user
* `PASS` <password> provides the user’s password
* `STAT` requests the number of messages and total size
* `LIST` lists all messages and their sizes
* `RETR` <message_number> retrieves the specified message
* `DELE` <message_number> marks a message for deletion
* `QUIT` ends the POP3 session applying changes, such as deletions

```bash
openssl s_client -starttls pop3 -connect 10.49.128.47:110 -tls1_2
or
telnet 10.49.128.47 110
```
```
+OK [XCLIENT] Dovecot (Ubuntu) ready.
AUTH #input
+OK
PLAIN
.
USER linda #input
+OK
PASS Pa$$123 #input
+OK Logged in.
STAT #input
+OK 3 1264
LIST #input
+OK 3 messages:
1 407
2 412
3 445
.
RETR 3 #input
+OK 454 octets
Return-path: <user@client.thm>
Envelope-to: linda@server.thm
Delivery-date: Thu, 12 Sep 2024 20:12:42 +0000
Received: from [10.11.81.126] (helo=client.thm)
	by example.thm with smtp (Exim 4.95)
	(envelope-from <user@client.thm>)
	id 1soqAj-0007li-39
	for linda@server.thm;
	Thu, 12 Sep 2024 20:12:42 +0000
From: user@client.thm
To: linda@server.thm
Subject: Your Flag

Hello!
Here's your flag:
THM{TELNET_RETR_EMAIL}
Enjoy your journey!
.
QUIT #input
+OK Logging out.
Connection closed by foreign host.
```

## **IMAP: Synchronizing Emai**
Default Port: 143
The IMAP protocol commands are more complicated than the POP3 protocol commands. We list a few examples below:

* `LOGIN` <username> <password> authenticates the user
* `SELECT` <mailbox> selects the mailbox folder to work with
* `FETCH` <mail_number> <data_item_name> Example fetch 3 body[] to fetch message number 3, header and body.
* `MOVE` <sequence_set> <mailbox> moves the specified messages to another mailbox
* `COPY` <sequence_set> <data_item_name> copies the specified messages to another mailbox
* `LOGOUT` logs out

```bash
root@ip-10-49-76-10:~# telnet 10.49.130.101 143
Trying 10.49.130.101...
Connected to 10.49.130.101.
Escape character is '^]'.
* OK [CAPABILITY IMAP4rev1 SASL-IR LOGIN-REFERRALS ID ENABLE IDLE LITERAL+ STARTTLS AUTH=PLAIN] Dovecot (Ubuntu) ready.
A LOGIN linda Pa$$123 #input
A OK [CAPABILITY IMAP4rev1 SASL-IR LOGIN-REFERRALS ID ENABLE IDLE SORT SORT=DISPLAY THREAD=REFERENCES THREAD=REFS THREAD=ORDEREDSUBJECT MULTIAPPEND URL-PARTIAL CATENATE UNSELECT CHILDREN NAMESPACE UIDPLUS LIST-EXTENDED I18NLEVEL=1 CONDSTORE QRESYNC ESEARCH ESORT SEARCHRES WITHIN CONTEXT=SEARCH LIST-STATUS BINARY MOVE SNIPPET=FUZZY PREVIEW=FUZZY PREVIEW STATUS=SIZE SAVEDATE LITERAL+ NOTIFY SPECIAL-USE] Logged in
B SELECT inbox #input
* FLAGS (\Answered \Flagged \Deleted \Seen \Draft)
* OK [PERMANENTFLAGS (\Answered \Flagged \Deleted \Seen \Draft \*)] Flags permitted.
* 4 EXISTS
* 0 RECENT
* OK [UNSEEN 1] First unseen.
* OK [UIDVALIDITY 1719824692] UIDs valid
* OK [UIDNEXT 9] Predicted next UID
B OK [READ-WRITE] Select completed (0.002 + 0.000 + 0.001 secs).
C FETCH 4 body[] #input fetch 4 email body
* 4 FETCH (FLAGS (\Seen) BODY[] {454}
Return-path: <user@client.thm>
Envelope-to: linda@server.thm
Delivery-date: Thu, 12 Sep 2024 20:12:42 +0000
Received: from [10.11.81.126] (helo=client.thm)
	by example.thm with smtp (Exim 4.95)
	(envelope-from <user@client.thm>)
	id 1soqAj-0007li-39
	for linda@server.thm;
	Thu, 12 Sep 2024 20:12:42 +0000
From: user@client.thm
To: linda@server.thm
Subject: Your Flag

Hello!
Here's your flag:
THM{TELNET_RETR_EMAIL}
Enjoy your journey!
)
C OK Fetch completed (0.001 + 0.000 secs).
D LOGOUT #input
* BYE Logging out
D OK Logout completed (0.001 + 0.000 secs).
Connection closed by foreign host.

```
