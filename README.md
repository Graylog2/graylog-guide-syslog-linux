# WARNING: THESE INSTRUCTIONS WILL SEND UNENCRYPTED LOG DATA VIA THE NETWORK!!!
# THIS IS DANGEROUS AND NOT RECOMMENDED!

## Sending syslog from Linux systems into Graylog

The two most popular syslog deamons (the programs that run in the background to accept and write or forward logs) are *rsyslog* and *syslog-ng*. One of these will most likely be running on your Linux distribution. (Please refer to your distribution documentation if you are unsure)

### rsyslog

Forwarding syslog messages with rsyslog is easy. The only important thing to get the most out of your logs is following
[RFC 5424](http://www.ietf.org/rfc/rfc5424.txt). The following examples configure your `rsyslog` daemon to send
RFC 5424 date to Graylog syslog inputs:

##### UDP:

    *.* @graylog.example.org:514;RSYSLOG_SyslogProtocol23Format

##### TCP:

    *.* @@graylog.example.org:514;RSYSLOG_SyslogProtocol23Format

(The difference between UDP and TCP is using `@` instead of `@@` as target descriptor.)

The above configuration should be placed as new file in `/etc/rsyslog.d/` and rsyslog should be restarted. In addition the port 514 on the Graylog server need to be reachable from the sending server.  

### old rsyslog
If you're using a very old version of rsyslog (versions before rsyslog 5.10) which doesn't provide the built-in [RSYSLOG_SyslogProtocol23Format](http://www.rsyslog.com/doc/v5-stable/configuration/templates.html#string-based-templates>) template, you can create a custom message template.

For **UDP** this becomes:

    $template GRAYLOGRFC5424,"<%PRI%>%PROTOCOL-VERSION% %TIMESTAMP:::date-rfc3339% %HOSTNAME% %APP-NAME% %PROCID% %MSGID% %STRUCTURED-DATA% %msg%\n"
    *.* @graylog.example.org:514;GRAYLOGRFC5424

### syslog-ng

Configuring `syslog-ng` to send syslog to Graylog is equally simple. Use the `syslog` function to send
[RFC 5424](http://www.ietf.org/rfc/rfc5424.txt) formatted syslog messages via TCP to the remote Graylog host:

    # Define TCP syslog destination.
    destination d_net {
        syslog("graylog.example.org" port(514));
    };
    # Tell syslog-ng to send data from source s_src to the newly defined syslog destination.
    log {
        source(s_src); # Defined in the default syslog-ng configuration.
        destination(d_net);
    };
