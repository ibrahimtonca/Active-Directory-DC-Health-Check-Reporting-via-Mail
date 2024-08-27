#############################################################################
# Author: İbrahim Tonca
# Date: 08/08/2024
# Status: Ping Durumu,Netlogon Servisi,NTDS Servisi,DNS Servisi Durumu,Netlogon Testi,Replication Testi,Servisler Testi,Advertising Testi,FSMO Kontrol Testi
# Description: AD Sağlık Durumu
#############################################################################
###########################Define Variables##################################

$reportpath = ".\ADReport.htm"

if (-not (Test-Path $reportpath)) {
New-Item $reportpath -Type File
}

$smtphost = "smtp_sunucunuz"
$from = "gönderici_mail"
$email1 = "alici_mail"
$timeout = 60

###############################HTML Report Content############################
$report = $reportpath

Clear-Content $report
Add-Content $report "<html>"
Add-Content $report "<head>"
Add-Content $report "<meta http-equiv='Content-Type' content='text/html; charset=iso-8859-1'>"
Add-Content $report '<title>AD Status Report</title>'
Add-Content $report '<style type="text/css">'
Add-Content $report "table { border-collapse: collapse; width: 100%; }"
Add-Content $report "td, th { font-family: Calibri; font-size: 15px; border: 1px solid #999999; padding: 10px; vertical-align: middle; }"
Add-Content $report "body { margin: 5px; }"
Add-Content $report "th { background-color: Navy; color: white; }"
Add-Content $report "</style>"
Add-Content $report "</head>"
Add-Content $report "<body>"

# Get current date and time
$timestamp = Get-Date -Format "dd-MM-yyyy HH:mm:ss"
Add-Content $report "<h1 align='center'>Active Directory Sağlık Durumu Raporu</h1>"
Add-Content $report "<h3 align='center'>Test Tarihi ve Saati: $timestamp</h3>"

# Main Test Results Table
Add-Content $report "<table>"
Add-Content $report "<thead><tr>"
Add-Content $report "<th width='5%' align='center'>DC</th>"
Add-Content $report "<th width='5%' align='center'>Ping Durumu</th>"
Add-Content $report "<th width='5%' align='center'>Netlogon Servisi</th>"
Add-Content $report "<th width='5%' align='center'>NTDS Servisi</th>"
Add-Content $report "<th width='5%' align='center'>DNS Servisi Durumu</th>"
Add-Content $report "<th width='5%' align='center'>Netlogon Testi</th>"
Add-Content $report "<th width='5%' align='center'>Replication Testi</th>"
Add-Content $report "<th width='5%' align='center'>Servisler Testi</th>"
Add-Content $report "<th width='5%' align='center'>Advertising Testi</th>"
Add-Content $report "<th width='7%' align='center'>FSMO Kontrol Testi</th>"
Add-Content $report "</tr></thead><tbody>"

#####################################Get ALL DC Servers#################################
$getForest = [system.directoryservices.activedirectory.Forest]::GetCurrentForest()

$DCServers = $getForest.Domains | ForEach-Object { $_.DomainControllers } | ForEach-Object { $_.Name }

################Ping Test######

foreach ($DC in $DCServers) {
$Identity = $DC
Add-Content $report "<tr>"
if (Test-Connection -ComputerName $DC -Count 1 -Quiet) {
Write-Host "$DC`t $DC`t Ping Success" -ForegroundColor Green
Add-Content $report "<td bgcolor='White' align='center'><b>$Identity</b></td>"
Add-Content $report "<td bgcolor='Aquamarine' align='center'><b>Başarılı</b></td>"

##############Netlogon Service Status################
$serviceStatus = Start-Job -ScriptBlock { Get-Service -ComputerName $using:DC -Name "Netlogon" -ErrorAction SilentlyContinue }
Wait-Job $serviceStatus -Timeout $timeout
if ($serviceStatus.State -eq "Running") {
Write-Host "$DC`t Netlogon Service Timeout" -ForegroundColor Yellow
Add-Content $report "<td bgcolor='Yellow' align='center'><b>NetlogonTimeout</b></td>"
Stop-Job $serviceStatus
} else {
$serviceStatus1 = Receive-Job $serviceStatus
if ($serviceStatus1.Status -eq "Running") {
Write-Host "$DC`t $serviceStatus1.Name`t $serviceStatus1.Status" -ForegroundColor Green
Add-Content $report "<td bgcolor='Aquamarine' align='center'><b>$($serviceStatus1.Status)</b></td>"
} else {
Write-Host "$DC`t $serviceStatus1.Name`t $serviceStatus1.Status" -ForegroundColor Red
Add-Content $report "<td bgcolor='Red' align='center'><b>$($serviceStatus1.Status)</b></td>"
}
}
######################################################
##############NTDS Service Status################
$serviceStatus = Start-Job -ScriptBlock { Get-Service -ComputerName $using:DC -Name "NTDS" -ErrorAction SilentlyContinue }
Wait-Job $serviceStatus -Timeout $timeout
if ($serviceStatus.State -eq "Running") {
Write-Host "$DC`t NTDS Service Timeout" -ForegroundColor Yellow
Add-Content $report "<td bgcolor='Yellow' align='center'><b>NTDSTimeout</b></td>"
Stop-Job $serviceStatus
} else {
$serviceStatus1 = Receive-Job $serviceStatus
if ($serviceStatus1.Status -eq "Running") {
Write-Host "$DC`t $serviceStatus1.Name`t $serviceStatus1.Status" -ForegroundColor Green
Add-Content $report "<td bgcolor='Aquamarine' align='center'><b>$($serviceStatus1.Status)</b></td>"
} else {
Write-Host "$DC`t $serviceStatus1.Name`t $serviceStatus1.Status" -ForegroundColor Red
Add-Content $report "<td bgcolor='Red' align='center'><b>$($serviceStatus1.Status)</b></td>"
}
}
######################################################
##############DNS Service Status################
$serviceStatus = Start-Job -ScriptBlock { Get-Service -ComputerName $using:DC -Name "DNS" -ErrorAction SilentlyContinue }
Wait-Job $serviceStatus -Timeout $timeout
if ($serviceStatus.State -eq "Running") {
Write-Host "$DC`t DNS Server Service Timeout" -ForegroundColor Yellow
Add-Content $report "<td bgcolor='Yellow' align='center'><b>DNSTimeout</b></td>"
Stop-Job $serviceStatus
} else {
$serviceStatus1 = Receive-Job $serviceStatus
if ($serviceStatus1.Status -eq "Running") {
Write-Host "$DC`t $serviceStatus1.Name`t $serviceStatus1.Status" -ForegroundColor Green
Add-Content $report "<td bgcolor='Aquamarine' align='center'><b>$($serviceStatus1.Status)</b></td>"
} else {
Write-Host "$DC`t $serviceStatus1.Name`t $serviceStatus1.Status" -ForegroundColor Red
Add-Content $report "<td bgcolor='Red' align='center'><b>$($serviceStatus1.Status)</b></td>"
}
}
######################################################

####################Netlogons status##################
Add-Type -AssemblyName microsoft.visualbasic
$cmp = "microsoft.visualbasic.strings" -as [type]
$sysvol = Start-Job -ScriptBlock { dcdiag /test:netlogons /s:$using:DC }
Wait-Job $sysvol -Timeout $timeout
if ($sysvol.State -eq "Running") {
Write-Host "$DC`t Netlogons Test Timeout" -ForegroundColor Yellow
Add-Content $report "<td bgcolor='Yellow' align='center'><b>NetlogonsTimeout</b></td>"
Stop-Job $sysvol
} else {
$sysvol1 = Receive-Job $sysvol
if ($cmp::instr($sysvol1, "passed test NetLogons")) {
Write-Host "$DC`t Netlogons Test passed" -ForegroundColor Green
Add-Content $report "<td bgcolor='Aquamarine' align='center'><b>NetlogonsPassed</b></td>"
} else {
Write-Host "$DC`t Netlogons Test Failed" -ForegroundColor Red
Add-Content $report "<td bgcolor='Red' align='center'><b>NetlogonsFailed</b></td>"
}
}
########################################################
#######################Replication######################
$rep = Start-Job -ScriptBlock { dcdiag /test:replications /s:$using:DC }
Wait-Job $rep -Timeout $timeout
if ($rep.State -eq "Running") {
Write-Host "$DC`t Replication Test Timeout" -ForegroundColor Yellow
Add-Content $report "<td bgcolor='Yellow' align='center'><b>ReplicationTimeout</b></td>"
Stop-Job $rep
} else {
$rep1 = Receive-Job $rep
if ($cmp::instr($rep1, "passed test Replications")) {
Write-Host "$DC`t Replication Test passed" -ForegroundColor Green
Add-Content $report "<td bgcolor='Aquamarine' align='center'><b>ReplicationPassed</b></td>"
} else {
Write-Host "$DC`t Replication Test Failed" -ForegroundColor Red
Add-Content $report "<td bgcolor='Red' align='center'><b>ReplicationFailed</b></td>"
}
}
########################################################
#######################Services######################
$sysvol = Start-Job -ScriptBlock { dcdiag /test:services /s:$using:DC }
Wait-Job $sysvol -Timeout $timeout
if ($sysvol.State -eq "Running") {
Write-Host "$DC`t Services Test Timeout" -ForegroundColor Yellow
Add-Content $report "<td bgcolor='Yellow' align='center'><b>ServicesTimeout</b></td>"
Stop-Job $sysvol
} else {
$sysvol1 = Receive-Job $sysvol
if ($cmp::instr($sysvol1, "passed test Services")) {
Write-Host "$DC`t Services Test passed" -ForegroundColor Green
Add-Content $report "<td bgcolor='Aquamarine' align='center'><b>ServicesPassed</b></td>"
} else {
Write-Host "$DC`t Services Test Failed" -ForegroundColor Red
Add-Content $report "<td bgcolor='Red' align='center'><b>ServicesFailed</b></td>"
}
}
########################################################
#######################Advertising######################
$sysvol = Start-Job -ScriptBlock { dcdiag /test:advertising /s:$using:DC }
Wait-Job $sysvol -Timeout $timeout
if ($sysvol.State -eq "Running") {
Write-Host "$DC`t Advertising Test Timeout" -ForegroundColor Yellow
Add-Content $report "<td bgcolor='Yellow' align='center'><b>AdvertisingTimeout</b></td>"
Stop-Job $sysvol
} else {
$sysvol1 = Receive-Job $sysvol
if ($cmp::instr($sysvol1, "passed test Advertising")) {
Write-Host "$DC`t Advertising Test passed" -ForegroundColor Green
Add-Content $report "<td bgcolor='Aquamarine' align='center'><b>AdvertisingPassed</b></td>"
} else {
Write-Host "$DC`t Advertising Test Failed" -ForegroundColor Red
Add-Content $report "<td bgcolor='Red' align='center'><b>AdvertisingFailed</b></td>"
}
}
########################################################
#######################FSMO Check######################
$fsmo = Start-Job -ScriptBlock { dcdiag /test:fsmoCheck /s:$using:DC }
Wait-Job $fsmo -Timeout $timeout
if ($fsmo.State -eq "Running") {
Write-Host "$DC`t FSMO Test Timeout" -ForegroundColor Yellow
Add-Content $report "<td bgcolor='Yellow' align='center'><b>FSMOTimeout</b></td>"
Stop-Job $fsmo
} else {
$fsmo1 = Receive-Job $fsmo
if ($cmp::instr($fsmo1, "passed test FsmoCheck")) {
Write-Host "$DC`t FSMO Test passed" -ForegroundColor Green
Add-Content $report "<td bgcolor='Aquamarine' align='center'><b>FsmoCheckPassed</b></td>"
} else {
Write-Host "$DC`t FSMO Test Failed" -ForegroundColor Red
Add-Content $report "<td bgcolor='Red' align='center'><b>FsmoCheckFailed</b></td>"
}
}
}
else {
Write-Host "$DC`t $DC`t Ping Failure" -ForegroundColor Red
Add-Content $report "<td bgcolor='GainsBoro' align='center'><b>$Identity</b></td>"
Add-Content $report "<td bgcolor='Red' align='center'><b>Başarısız</b></td>"
Add-Content $report "<td bgcolor='Grey' align='center' colspan='8'><b>Yok</b></td>"
}
Add-Content $report "</tr>"
}

Add-Content $report "</tbody></table>"

# Table for Status Explanations
Add-Content $report "<br>"
Add-Content $report "<table>"
Add-Content $report "<thead><tr>"
Add-Content $report "<th width='10%' align='center'>Durum Mesajı</th>"
Add-Content $report "<th width='90%' align='center'>Açıklama</th>"
Add-Content $report "</tr></thead><tbody>"

# Add rows for each explanation
Add-Content $report "<tr><td bgcolor='Aquamarine' align='center'><b>Başarılı</b></td><td>Bu durum, testin veya hizmetin başarıyla geçtiğini gösterir.</td></tr>"
Add-Content $report "<tr><td bgcolor='Red' align='center'><b>Başarısız</b></td><td>Bu durum, testin veya hizmetin başarısız olduğunu gösterir.</td></tr>"
Add-Content $report "<tr><td bgcolor='Yellow' align='center'><b>Timeout</b></td><td>Bu durum, testin veya hizmetin belirtilen süre içinde tamamlanamadığını gösterir.</td></tr>"
Add-Content $report "<tr><td bgcolor='Grey' align='center'><b>Yok</b></td><td>Bu durum, hizmetin mevcut olmadığını gösterir.</td></tr>"

Add-Content $report "</tbody></table>"

# Table for Test Explanations with Two Columns
Add-Content $report "<br>"
Add-Content $report "<table>"
Add-Content $report "<thead><tr>"
Add-Content $report "<th width='15%' align='center'>Test Adı</th>"
Add-Content $report "<th width='85%' align='center'>Açıklama</th>"
Add-Content $report "</tr></thead><tbody>"
Add-Content $report "<tr><b>Ping Testi:</b></td><td>Sunucunun ağ üzerinde erişilebilirliğini ve yanıt verebilirliğini kontrol eder. Sunucuya veri paketleri gönderir ve geri dönüş alıp almadığını değerlendirir.</td></tr>"
Add-Content $report "<tr><b>Netlogon Servisi:</b></td><td>Netlogon servisinin aktif olup olmadığını kontrol eder. Bu servis, etki alanı denetleyicisinin oturum açma ve kimlik doğrulama taleplerini yönetir.</td></tr>"
Add-Content $report "<tr><b>NTDS Servisi:</b></td><td>NTDS (NT Directory Services) servisinin çalışıp çalışmadığını kontrol eder. Bu servis, Active Directory veri tabanını ve dizin hizmetlerini yönetir.</td></tr>"
Add-Content $report "<tr><b>DNS Servisi Durumu:</b></td><td>DNS (Domain Name System) servisinin çalışıp çalışmadığını kontrol eder. DNS, etki alanı adı çözümlemesi ve IP adresi ile ilişkili işlemleri yürütür.</td></tr>"
Add-Content $report "<tr><b>Netlogon Testi:</b></td><td>Netlogon güvenli kanal testini gerçekleştirir. Bu test, sunucular arasında güvenli iletişim ve kimlik doğrulamanın doğru çalışıp çalışmadığını değerlendirir.</td></tr>"
Add-Content $report "<tr><b>Replication Testi:</b></td><td>Active Directory replikasyon işlemlerinin doğru bir şekilde gerçekleşip gerçekleşmediğini kontrol eder. Replikasyon, etki alanı denetleyicileri arasındaki veri senkronizasyonunu sağlar.</td></tr>"
Add-Content $report "<tr><b>Servisler Testi:</b></td><td>Sunucu üzerindeki kritik servislerin (örneğin, Kerberos, LDAP, ve diğer etki alanı hizmetleri) durumunu kontrol eder. Bu test, servislerin düzgün çalışıp çalışmadığını belirler.</td></tr>"
Add-Content $report "<tr><b>Advertising Testi:</b></td><td>Sunucunun kendisini Active Directory hizmetleri olarak tanıtıp tanıtamadığını kontrol eder. Bu test, sunucunun diğer etki alanı denetleyicilerine doğru bilgi gönderip göndermediğini değerlendirir.</td></tr>"
Add-Content $report "<tr><b>FSMO Testi:</b></td><td>FSMO (Flexible Single Master Operations) rollerinin doğru bir şekilde çalışıp çalışmadığını kontrol eder. FSMO roller, Active Directory içindeki belirli işlemleri yönetir ve bu test, rollerin aktif olduğunu doğrular.</td></tr>"
Add-Content $report "</tbody></table>"

 

Add-Content $report "</body>"
Add-Content $report "</html>"

#########################Send Email####################################
$smtp = New-Object Net.Mail.SmtpClient($smtphost)
$msg = New-Object Net.Mail.MailMessage
$msg.From = $from
$msg.To.Add($email1)
$msg.Subject = "Active Directory Durum Raporu"
$msg.IsBodyHtml = $true
$msg.Body = Get-Content $reportpath | Out-String
$smtp.Send($msg)
# Removed the following line to prevent the report from opening automatically
# start-process $reportpath
