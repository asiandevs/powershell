# Collect TCP and UDP connections
$tcpConnections = Get-NetTCPConnection
$udpConnections = Get-NetUDPEndpoint

# Function to format connection data
function Format-Connection {
    param (
        $conn,
        $protocol
    )

    [PSCustomObject]@{
        Protocol         = $protocol
        SourceHost       = $conn.RemoteAddress
        SourcePort       = $conn.RemotePort
        DestinationHost  = $conn.LocalAddress
        DestinationPort  = $conn.LocalPort
        Direction        = if ($conn.LocalAddress -eq '0.0.0.0' -or $conn.LocalAddress -eq '::') { 'Inbound' } else { 'Outbound' }
    }
}

# Process TCP connections
$tcpFormatted = $tcpConnections | ForEach-Object { Format-Connection $_ 'TCP' }

# Process UDP connections
$udpFormatted = $udpConnections | ForEach-Object { Format-Connection $_ 'UDP' }

# Combine and separate by direction
$allConnections = $tcpFormatted + $udpFormatted
$inbound = $allConnections | Where-Object { $_.Direction -eq 'Inbound' }
$outbound = $allConnections | Where-Object { $_.Direction -eq 'Outbound' }

# Output results
Write-Host "`n--- Inbound Connections ---`n"
$inbound | Format-Table -AutoSize

Write-Host "`n--- Outbound Connections ---`n"
$outbound | Format-Table -AutoSize

# $inbound | Export-Csv -Path "InboundTraffic.csv" -NoTypeInformation
# $outbound | Export-Csv -Path "OutboundTraffic.csv" -NoTypeInformation
