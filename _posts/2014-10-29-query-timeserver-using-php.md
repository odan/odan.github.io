---
title: Query Timeserver using PHP
layout: post
comments: true
published: false
description: 
keywords: 
---

NTP Timeservers provide a high-precision time stamp e.g. from an atomic clock, GPS or radio clock. 

Time servers are mainly used to synchronize hosts. 

With this function it is possible to query a timeserver via NTP:

```php
function query_time_server($timeserver, $port = 37)
{
    $fp = fsockopen($timeserver, $port, $errno, $errstr, 3);
    
    if (!$fp) {
        throw new \RuntimeException('NTP connection error: ' . $errstr, $errno); 
    }
    
    //fputs($fp, "\n");
    $timeValue = fread($fp, 49);
    fclose($fp);
    
    return $timeValue;
}
```

### Usage

```php
// Physikalisch-Technische Bundesanstalt
$timeserver = "ntp1.ptb.de";
//$timeserver = "ntp2.ptb.de"; 
//$timeserver = "ptbtime1.ptb.de"; 
//$timeserver = "ptbtime2.ptb.de";

// uni-erlangen  http://www.rrze.uni-erlangen.de/infrastruktur/spezial-geraete/zeitserver.shtml
//$timeserver = "ntp0.fau.de"; // uni-erlangen - GPS über externe Externer Link: Meinberg GPS167 Uhr
//$timeserver = "ntp1.fau.de"; // uni-erlangen - DCF77 über externe Externer Link: Meinberg PZF510 Uhr
//$timeserver = "ntp2.fau.de"; // uni-erlangen - GPS über interne Empfängerkarte
//$timeserver = "ntp3.fau.de"; // uni-erlangen - DCF77 über interne Empfängerkarte

echo "Query time server: $timeserver\n";
$timercvd = query_time_server($timeserver, 37);

// if no error from query_time_server
$timevalue = bin2hex($timercvd);
$timevalue = abs(hexdec('7fffffff') - hexdec($timevalue) - hexdec('7fffffff'));
$tmestamp = $timevalue - 2208988800; # convert to UNIX epoch time stamp
$date = date("Y-m-d (D) H:i:s", $tmestamp /* - date("Z",$tmestamp) */); /* incl time zone offset */
$day = (date("z", $tmestamp) + 1);

echo "Time check from time server ", $timeserver, " : [", $timevalue, "]";
echo " (seconds since 1900-01-01 00:00.00).\n";
echo "The current date and universal time is ", $date, " UTC. ";
echo "It is day ", $day, " of this year.\n";
echo "The unix epoch time stamp is $tmestamp.\n";
```