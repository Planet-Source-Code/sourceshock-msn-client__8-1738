<div align="center">

## MSN Client


</div>

### Description

It is a basic msn script that connects directly to an msn server.

First it connects to a correct msn server

(depends on the response from the first call) and

then gets your passport and session from the password nexus from microsoft. if all went well

you should have your passport and it will retrieve your contact list (and throws you offline if msn is on :-) )
 
### More Info
 


<span>             |<span>
---                |---
**Submitted On**   |
**By**             |[sourceshock](https://github.com/Planet-Source-Code/PSCIndex/blob/master/ByAuthor/sourceshock.md)
**Level**          |Intermediate
**User Rating**    |5.0 (10 globes from 2 users)
**Compatibility**  |PHP 4\.0
**Category**       |[Miscellaneous](https://github.com/Planet-Source-Code/PSCIndex/blob/master/ByCategory/miscellaneous__8-1.md)
**World**          |[PHP](https://github.com/Planet-Source-Code/PSCIndex/blob/master/ByWorld/php.md)
**Archive File**   |[](https://github.com/Planet-Source-Code/sourceshock-msn-client__8-1738/archive/master.zip)





### Source Code

```
<?PHP
##################################################################
##																##
##    THIS IS AN UNFINISHED SCRIPT FOR LEARNING PURPOSES  ##
##        YOU MAY USE AND DISTRIBUTE IT AS LONG AS THIS	##
##    COMMENT FIELD IS IN IT OR YOU LINK TO SOURCESHOCK.COM ##
##																##
##    the main thing it does is connecting to the msn		##
##    server and then connect to the password nexus			##
##    when done all that it gets your msn list and changes	##
##    your status, the rest is up to you :)					##
##																##
##################################################################
	$startcomm = 0;
	$username = "username";
	$password = "password";
	function getData() {
		$socket = $GLOBALS["socket"];
		$buffer="";
		while (!feof($socket)) {
			$buffer .= fread($socket,1024);
			if (preg_match("/\r/",$buffer)) {
				break;
			}
		}
		echo "<font color=green><< $buffer</font><br>";
		checkData($buffer);
	}
	function getData2() {
		$socket = $GLOBALS["socket"];
		$buffer="";
		while (!feof($socket)) {
			$buffer .= fread($socket,1024);
			if (preg_match("/\r\n\r\n/",$buffer)) {
				break;
			}
		}
		//$buffer = str_replace("\n","<br>",$buffer);
		echo "<font color=green><< $buffer</font><br>";
		checkData($buffer);
	}
	function checkData($buffer) {
		if (preg_match("/lc\=(.+?)/Ui",$buffer,$matches)) {
			$GLOBALS["challenge"] = "lc=" . $matches[1];
		}
		if (preg_match("/(XFR 3 NS )([0-9\.\:]+?) (.*) ([0-9\.\:]+?)/is",$buffer,$matches)) {
			$split = explode(":",$matches[2]);
			$GLOBALS["startcomm"] = 1;
			msn_connect($split[0],$split[1]);
		}
		if (preg_match("/tpf\=([a-zA-Z0-9]+?)/Ui",$buffer,$matches)) {
			nexus_connect($matches[1]);
		}
		$split = explode("\n",$buffer);
		for ($i=0;$i<count($split);$i++) {
			$detail = explode(" ",$split[$i]);
			if ($detail[0] == "LST") {
				//echo "<div OnMouseOver=\"style.cursor='hand';showTooltip('show','$detail[1]-$detail[3]')\" OnMouseMove=\"followTooltip('show')\" OnMouseOut=\"showTooltip('hide')\">" . urldecode($detail[2]) . "</div>";
			}
		}
		//echo $buffer;
	}
	function msn_connect($server,$port) {
		if (IsSet($GLOBALS["socket"])) {
			fclose($GLOBALS["socket"]);
		}
		 $GLOBALS["socket"] = fsockopen($server,$port);
		if (!$GLOBALS["socket"]) {
			return "Could not connect";
		} else {
			$GLOBALS["startcomm"]++;
			send_command("VER " . $GLOBALS["startcomm"] . " MSNP8 CVR0",1);
			send_command("CVR " . $GLOBALS["startcomm"] . " 0x0409 win 4.10 i386 MSNMSGR 6.2 MSMSGS " . $GLOBALS["username"] . "@hotmail.com",1);
			send_command("USR " . $GLOBALS["startcomm"] . " TWN I " . $GLOBALS["username"] . "@hotmail.com",1);
		}
	}
	function send_command($command) {
		$GLOBALS["startcomm"]++;
		$socket = $GLOBALS["socket"];
		echo "<font color=blue> >> $command<br>";
		fwrite($socket,$command . "\r\n");
		getData();
	}
	function nexus_connect($tpf) {
		echo "<font color=#a5a5a5>";
		$arr[] = "GET /rdr/pprdr.asp HTTP/1.0\r\n\r\n";
		$curl = curl_init();
		curl_setopt($curl, CURLOPT_URL, "https://nexus.passport.com:443/rdr/pprdr.asp");
		curl_setopt($curl, CURLOPT_RETURNTRANSFER, 1);
		curl_setopt($curl, CURLOPT_VERBOSE, 0);
		curl_setopt($curl, CURLOPT_HEADER,1);
		curl_setopt($curl, CURLOPT_HTTPHEADER, $arr);
		curl_setopt($curl, CURLOPT_SSL_VERIFYPEER, FALSE);
		$data = curl_exec($curl);
		curl_close($curl);
		preg_match("/DALogin=(.+?),/",$data,$matches);
		$data = str_replace("\n","<br>",$data);
		echo $data;
		echo "<br><br>";
		$split = explode("/",$matches[1]);
		$headers[0] = "GET /$split[1] HTTP/1.1\r\n";
		$headers[1] = "Authorization: Passport1.4 OrgVerb=GET,OrgURL=http%3A%2F%2Fmessenger%2Emsn%2Ecom,sign-in=" . $GLOBALS["username"] . "%40hotmail.com,pwd=" . $GLOBALS["password"] . ", " . trim($GLOBALS["challenge"]) . "\r\n";
		$curl = curl_init();
		curl_setopt($curl, CURLOPT_URL, "https://" . $split[0] . ":443/". $split[1]);
		curl_setopt($curl, CURLOPT_RETURNTRANSFER, 1);
		curl_setopt($curl, CURLOPT_VERBOSE, 0);
		curl_setopt($curl,CURLOPT_FOLLOWLOCATION,1);
		curl_setopt($curl, CURLOPT_HTTPHEADER, $headers);
		curl_setopt($curl, CURLOPT_HEADER,1);
		curl_setopt($curl, CURLOPT_SSL_VERIFYPEER, FALSE);
		$data = curl_exec($curl);
		$data = str_replace("\n","<br>\n",$data);
		echo $data;
		curl_close($curl);
		echo "</font>";
		preg_match("/t=(.+?)'/",$data,$matches);
		send_command("USR " . $GLOBALS["startcomm"] . " TWN S t=" . trim($matches[1]) . "",2);
		send_command("CHG " . $GLOBALS["startcomm"] . " HDN",2);
		send_command("SYN " . $GLOBALS["startcomm"] . " 0",2);
		getData2();
		send_command("SYN " . $GLOBALS["startcomm"] . " 1 46 2",2);
		getData2();
		send_command("CHG ". $GLOBALS["startcomm"] . " BSY");
		getData();
	}
msn_connect("messenger.hotmail.com",1863);
?>
```

