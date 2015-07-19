#Rajvi Trivedi 
#roll no.: 131039
#Computr Networing
# Assignment1


# simulate thenew LAN
set ns [new Simulator]

# Defining the three colors for the three different data flows
$ns color 1 Blue
$ns color 2 Red
$ns color 3 green

#Using below statements can open the trace and win files
set tracefile1 [open out.tr w]
set winfile [open winfile w]
$ns trace-all $tracefile1

#Using below statement open the namfile
set namfile [open out.nam w]
$ns namtrace-all $namfile

# Definig the finish procedure
proc finish {} \
{
global ns tracefile1 namfile
$ns flush-trace
close $tracefile1
close $namfile
exec nam out.nam &
exit 0
}

# Here eleven nodes are created
set n0 [$ns node]
set n1 [$ns node]
set n2 [$ns node]
set n3 [$ns node]
set n4 [$ns node]
set n5 [$ns node]
set n6 [$ns node]
set n7 [$ns node]
set n8 [$ns node]
set n9 [$ns node]
set n10 [$ns node]

# Declare two different color red and green for node n1 and n8 respectively
$n1 color Red
$n1 shape box

$n8 color green
$n8 shape box

# Create duplex links between the nodes 
$ns duplex-link $n0 $n3 2Mb 10ms DropTail
$ns duplex-link $n3 $n2 2Mb 10ms DropTail
$ns duplex-link $n0 $n1 2Mb 10ms DropTail
$ns duplex-link $n1 $n2 2Mb 10ms DropTail
$ns duplex-link $n4 $n5 2Mb 10ms DropTail
$ns duplex-link $n5 $n6 2Mb 10ms DropTail
$ns duplex-link $n4 $n6 2Mb 10ms DropTail
$ns duplex-link $n6 $n7 2Mb 10ms DropTail
$ns duplex-link $n6 $n8 2Mb 10ms DropTail
$ns duplex-link $n9 $n10 2Mb 10ms DropTail

# Set the bus topology (new LAN) at nodes n2,n4 and n9
set lan [$ns newLan "$n2 $n4 $n9" 0.5Mb 50ms LL Queue/DropTail MAC/Csma/Cd Channel]

# This give the position of the node
$ns duplex-link-op $n0 $n3 orient right
$ns duplex-link-op $n3 $n2 orient down
$ns duplex-link-op $n0 $n1 orient down
$ns duplex-link-op $n1 $n2 orient right
$ns duplex-link-op $n4 $n5 orient right-up
$ns duplex-link-op $n5 $n6 orient down
$ns duplex-link-op $n4 $n6 orient right-down
$ns duplex-link-op $n6 $n7 orient left-down
$ns duplex-link-op $n6 $n8 orient right-down
$ns duplex-link-op $n9 $n10 orient right

# Packet sending through FTP connection from n2 to n7
set tcp [new Agent/TCP/Newreno]
$ns attach-agent $n2 $tcp
set sink [new Agent/TCPSink/DelAck]
$ns attach-agent $n7 $sink
$ns connect $tcp $sink
$tcp set fid_ 1
$tcp set packet_size_ 552

# Set the FTP over tcp connection
set ftp [new Application/FTP]
$ftp attach-agent $tcp

# Packet sending through UDP connection from n1 to n10
set udp1 [new Agent/UDP]
$ns attach-agent $n1 $udp1
set null1 [new Agent/Null]
$ns attach-agent $n10 $null1
$ns connect $udp1 $null1
$udp1 set fid_ 2

# Packet sending through UDP connection from n8 to n0
set udp2 [new Agent/UDP]
$ns attach-agent $n8 $udp2
set null2 [new Agent/Null]
$ns attach-agent $n0 $null2
$ns connect $udp2 $null2
$udp2 set fid_ 3

# Set the CBR connection over first UDP connection 
set cbr1 [new Application/Traffic/CBR]
$cbr1 attach-agent $udp1
$cbr1 set type_ CBR
$cbr1 set packet_size_ 1000
$cbr1 set rate_ 0.01Mb
$cbr1 set random_ false

# Set the CBR connection over second UDP connection
set cbr2 [new Application/Traffic/CBR]
$cbr2 attach-agent $udp2
$cbr2 set type_ CBR
$cbr2 set packet_size_ 1000
$cbr2 set rate_ 0.02Mb
$cbr2 set random_ false

# Schedule all the events
$ns at 0.1 "$cbr1 start"
$ns at 0.1 "$cbr2 start"
$ns at 1.0 "$ftp start"
$ns at 1.0 "$ftp start"
$ns at 124.0 "$ftp stop"
$ns at 124.0 "$ftp stop"
$ns at 125.5 "$cbr1 stop"
$ns at 125.5 "$cbr2 stop"

proc plotWindow {tcpSource file} {
global ns
set time 0.1
set now [$ns now]
set cwnd [$tcpSource set cwnd_]
puts $file "$now $cwnd"
$ns at [expr $now+$time ] "plotWindow $tcpSource $file"
}
$ns at 0.1 "plotWindow $tcp $winfile"
$ns at 125.0 "finish"
$ns run











