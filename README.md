## Sending syslog from Linux systems into Graylog

The two most popular syslog deamons (the programs that run in the background to accept and write or forward logs) are *rsyslog* and *syslog-ng*. One of these will most likely be running on your Linux distribution. (Please refer to your distribution documentation if you are unsure)

### rsyslog

Forwarding syslog messages with rsyslog is easy. The only important thing to get the most out of your logs is following
[RFC 5424](http://www.ietf.org/rfc/rfc5424.txt). The following examples configure your `rsyslog` daemon to send
RFC 5424 date to Graylog syslog inputs:

##### UDP:

    $template GRAYLOGRFC5424,"<%PRI%>%PROTOCOL-VERSION% %TIMESTAMP:::date-rfc3339% %HOSTNAME% %APP-NAME% %PROCID% %MSGID% %STRUCTURED-DATA% %msg%\n"
    *.* @graylog.example.org:514;GRAYLOGRFC5424

##### TCP:

    $template GRAYLOGRFC5424,"<%PRI%>%PROTOCOL-VERSION% %TIMESTAMP:::date-rfc3339% %HOSTNAME% %APP-NAME% %PROCID% %MSGID% %STRUCTURED-DATA% %msg%\n"
    *.* @@graylog.example.org:514;GRAYLOGRFC5424

(The difference between UDP and TCP is using `@` instead of `@@` as target descriptor.)

Alternatively, the rsyslog built-in template [RSYSLOG_SyslogProtocol23Format](http://www.rsyslog.com/doc/v5-stable/configuration/templates.html#string-based-templates>) sends log messages in the same format as above. This exists in rsyslog versions of at least 5.10 or later.

The UDP examples above becomes:

    *.* @graylog.example.org:514;RSYSLOG_SyslogProtocol23Format

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
