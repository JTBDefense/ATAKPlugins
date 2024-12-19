## Summary

This guide shows steps to improve TAK server performance, specifically for Raspberry Pi 5. Based on TAK server 5.3-RELEASE-4.

## Baseline - Normal start

    systemctl start takserver

Result:
> Ports 8446 and 8089 became available 
> after  **90** seconds.

## Disable PluginService

Start the TAK server without plugin support

    systemctl start takserver-noplugins
Result:
> Ports 8446 and 8089 became available
> after  **75** seconds.


## Reduce logging

### Remove the verbose class logging from takserver-api-console.log

    sed -i 's/-verbose:class//g' /opt/tak/takserver-api.sh


### Reduce the logging levels for all loggers

    sed -i 's/level="\(INFO\|WARN\)"/level="ERROR"/g' /opt/tak/extract/WEB-INF/classes/logback-spring.xml


## JVM tuning

### takserver-messaging.sh

    #!/bin/sh
    rm -rf ./tmp/
    . ./setenv.sh
    
    java -Xmx${API_MAX_HEAP}m \
      -Xms${API_MAX_HEAP}m \
      -XX:+UseG1GC \
      -XX:InitiatingHeapOccupancyPercent=75 \
      -XX:+ParallelRefProcEnabled \
      -XX:+HeapDumpOnOutOfMemoryError \
      -XX:HeapDumpPath=/opt/tak/dumps/heapdump.hprof \
      -XX:+AlwaysPreTouch \
      -XX:+UseStringDeduplication \
      -XX:+CompactStrings \
      -Dspring.profiles.active=messaging \
      -Dorg.xml.sax.parser="com.sun.org.apache.xerces.internal.parsers.SAXParser" \
      -Djavax.xml.parsers.DocumentBuilderFactory="com.sun.org.apache.xerces.internal.jaxp.DocumentBuilderFactoryImpl" \
      -Djavax.xml.parsers.SAXParserFactory="com.sun.org.apache.xerces.internal.jaxp.SAXParserFactoryImpl" \
      -classpath "extract/WEB-INF/lib/takserver-common-5.3-RELEASE-4.jar:extract/WEB-INF/lib/takserver-war-5.3-RELEASE-4.jar:extract/WEB-INF/lib/takserver-fig-core-5.3-RELEASE-4.jar:extract/WEB-INF/lib/takserver-plugins-5.3-RELEASE-4.jar:extract/WEB-INF/lib/*:extract/WEB-INF/classes/*:extract/WEB-INF/classes/."  \
      tak.server.ServerConfiguration

### takserver-api.sh 

    #!/bin/sh
    . ./setenv.sh
    
    java -Xmx${API_MAX_HEAP}m \
      -Xms${API_MAX_HEAP}m \
      -XX:+UseG1GC \
      -XX:InitiatingHeapOccupancyPercent=75 \
      -XX:+ParallelRefProcEnabled \
      -XX:+HeapDumpOnOutOfMemoryError \
      -XX:HeapDumpPath=/opt/tak/dumps/heapdump.hprof \
      -XX:+AlwaysPreTouch \
      -XX:+UseStringDeduplication \
      -XX:+CompactStrings \
      -Dspring.profiles.active=api \
      -Dkeystore.pkcs12.legacy \
      -Dorg.xml.sax.parser="com.sun.org.apache.xerces.internal.parsers.SAXParser" \
      -Djavax.xml.parsers.DocumentBuilderFactory="com.sun.org.apache.xerces.internal.jaxp.DocumentBuilderFactoryImpl" \
      -Djavax.xml.parsers.SAXParserFactory="com.sun.org.apache.xerces.internal.jaxp.SAXParserFactoryImpl" \
      -classpath "extract/WEB-INF/lib/takserver-common-5.3-RELEASE-4.jar:extract/WEB-INF/lib/takserver-war-5.3-RELEASE-4.jar:extract/WEB-INF/lib/takserver-fig-core-5.3-RELEASE-4.jar:extract/WEB-INF/lib/takserver-plugins-5.3-RELEASE-4.jar:extract/WEB-INF/lib/*:extract/WEB-INF/classes/*:extract/WEB-INF/classes/."  \
      tak.server.ServerConfiguration
    
  
### takserver-config.sh
 

     java -server \
      -XX:InitiatingHeapOccupancyPercent=75 \
      -XX:+ParallelRefProcEnabled \
      -XX:+HeapDumpOnOutOfMemoryError \
      -XX:HeapDumpPath=/opt/tak/dumps/heapdump.hprof \
      -XX:+AlwaysPreTouch \
      -XX:+UseStringDeduplication \
      -XX:+UseG1GC \
      -XX:+ScavengeBeforeFullGC \
      -XX:+DisableExplicitGC \
      -Xms${CONFIG_MAX_HEAP}m \
      -Xmx${CONFIG_MAX_HEAP}m \
      -Dspring.profiles.active=config \
      -Dkeystore.pkcs12.legacy \
      -jar takserver.war $@

Result:
> Ports 8446 and 8089 became available 
> after **70** seconds.



