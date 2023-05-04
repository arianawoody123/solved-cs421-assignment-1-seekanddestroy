Download Link: https://assignmentchef.com/product/solved-cs421-assignment-1-seekanddestroy
<br>



<h1>    I.     Introduction</h1>

In this programming assignment, you are asked to implement a program in <strong>Java</strong>. The program must search for a file in the server, download it, and then delete it from the server. Your program must communicate with the server by using a custom application layer protocol named <em>CustomFTP</em>, which is inspired by the File Transfer Protocol (FTP). The server program written and tested in <strong>Python 3.6</strong> (to avoid decompiling to Java source code to use in the assignment) is provided for you to test your program.

The goal of the assignment is to make you familiar with the application layer and TCP sockets. You must implement your program using the <strong>Java Socket API of the JDK</strong>. If you have any doubt about what to use or not to use, please contact your teaching assistant.

When preparing your project please keep in mind that your projects will be auto-graded by a computer program. Any problems in the formatting can create problems in the grading; while you will not get a zero from your project, you may need to have an appointment with your teaching assistant for a manual grading. Errors caused by <strong>incorrectly naming</strong> the project files and folder structure will cause you to lose points.

<h1>II.    Design Specifications a. <em>CustomFTP</em> Specifications</h1>

The server program we provide, <em>CustomFTPServer</em>, uses an application layer protocol called <em>CustomFTP.</em> CustomFTP is a simplified file transfer protocol which is inspired by FTP.




<h1>i. Control Connection</h1>

All commands are sent from the client (your program) to the server through the <em>control connection. </em>After each command, the server sends a response to the client again using the same control connection. This control connection is persistent, i.e., it stays open throughout the session. The commands must be constructed as strings encoded in <strong>US-ASCII </strong>and have the following format:

&lt;CommandName&gt;&lt;Space&gt;&lt;Argument&gt;&lt;CR&gt;&lt;LF&gt;

<ul>

 <li>&lt;CommandName&gt; is the name of the command. For all possible commands and their explanations, see Table 1.</li>

 <li>&lt;Space&gt; is a single space character. Can be omitted if &lt;Argument&gt; is empty.</li>

 <li>&lt;Argument&gt; is the argument specific to the command. Can be empty if no argument is needed for the given command. See Table 1 for more information.</li>

 <li>&lt;CR&gt;&lt;LF&gt; is a carriage return character followed by a line feed character, i.e., “r
”.</li>

</ul>

<em>Table 1: List of CustomFTP commands with explanations. </em>

<table width="613">

 <tbody>

  <tr>

   <td width="129"><strong>&lt;CommandName&gt;</strong></td>

   <td width="112"><strong>&lt;Argument&gt; </strong></td>

   <td width="212"><strong>Explanation</strong></td>

   <td width="160"><strong>Example Usage </strong></td>

  </tr>

  <tr>

   <td width="129">USER</td>

   <td width="112">&lt;username&gt;</td>

   <td width="212">Send username to the server.</td>

   <td width="160">USER bilkentr
</td>

  </tr>

  <tr>

   <td width="129">PASS</td>

   <td width="112">&lt;password&gt;</td>

   <td width="212">Send password to the server.</td>

   <td width="160">PASS cs421r
</td>

  </tr>

  <tr>

   <td width="129">PORT</td>

   <td width="112">&lt;port&gt;</td>

   <td width="212">Send port number of the data connection that the client is bound to.</td>

   <td width="160">PORT 60001r
</td>

  </tr>

  <tr>

   <td width="129">NLST</td>

   <td width="112">–</td>

   <td width="212">Obtain the list of all files and directories in the current working folder of the server.</td>

   <td width="160">NLSTr
</td>

  </tr>

  <tr>

   <td width="129">CWD</td>

   <td width="112">&lt;child&gt;</td>

   <td width="212">Change the working directory to the one of the child directories of the current working directory of the server.</td>

   <td width="160">CWD imagesr
</td>

  </tr>

  <tr>

   <td width="129">CDUP</td>

   <td width="112">–</td>

   <td width="212">Go to the parent directory of the current working directory of the server.</td>

   <td width="160">CDUPr
</td>

  </tr>

  <tr>

   <td width="129">RETR</td>

   <td width="112">&lt;filename&gt;</td>

   <td width="212">Retrieve the file &lt;filename&gt; from the current working directory of the server.</td>

   <td width="160">RETRsample.txtr
</td>

  </tr>

  <tr>

   <td width="129">DELE</td>

   <td width="112">&lt;filename&gt;</td>

   <td width="212">Delete the file &lt;filename&gt; from the current working directory of the server.</td>

   <td width="160">DELEsample.txtr
</td>

  </tr>

  <tr>

   <td width="129">QUIT</td>

   <td width="112">–</td>

   <td width="212">Tell server to end the session and shutdown.</td>

   <td width="160">QUITr
</td>

  </tr>

 </tbody>

</table>




The response messages sent by the server are also encoded in <strong>US-ASCII</strong>. Responses have the following format:

&lt;Code&gt;&lt;Message&gt;&lt;CR&gt;&lt;LF&gt;

<ul>

 <li>&lt;Code&gt; is either 200 (success) or 400 (failure), for the sake of simplicity. You should check the &lt;Message&gt; for the reason of the failure if &lt;Code&gt; is 400.</li>

 <li>&lt;Message&gt; is the response message. It is always “OK” when &lt;Code&gt; is 200.</li>

 <li>&lt;CR&gt;&lt;LF&gt; is a carriage return character followed by a line feed character, i.e., “r
”. <strong>Data Connection </strong></li>

</ul>

While commands and responses are exchanged through a persistent control connection, the data (e.g., files obtained using RETR command or list of files and directories obtained using NLST command) is sent through a non-persistent connection named <em>data connection</em>. That is, the connection is opened just before the data transmission and closed once the transmission is complete.

CustomFTP uses what is called <em>active mode </em>in FTP. In active mode, the client binds to an available port for the data connection and sends this port to the server using the PORT command, at the beginning of the session. Then, when a data transfer is about the begin, the server connects to the client at the specified port and the transmission begins.

For data transmission mode, CustomFTP uses <em>block mode</em>. In block mode, the data is sent in the form of a block with a 16-bit header in addition to the data. This header indicates the size of the data to be transmitted (see Figure 1). Byte order of the header is in big-endian format, i.e., the most significant byte is sent first, the least significant byte is sent last.

<table width="543">

 <tbody>

  <tr>

   <td width="373">


    <table width="364">

     <tbody>

      <tr>

       <td width="231">Data Length (2 Bytes)</td>

       <td width="133">Data</td>

      </tr>

     </tbody>

    </table></td>

   <td width="75"></td>

   <td width="95">


    <table width="89">

     <tbody>

      <tr>

       <td width="89"> </td>

      </tr>

     </tbody>

    </table></td>

  </tr>

 </tbody>

</table>

<em>Figure 1: Illustration of data transmission in block mode.</em>

<h1>b. <em>SeekAndDestroy</em> Design Specifications</h1>

The server program provided for you, <em>CustomFTPServer,</em> simulates a server with some randomly generated directories and files. When a client connects to the server, the current working directory is the “root” folder. The client can view the contents of the current working directory by using the NLST command. The NLST command retrieves the contents of the current working directory as a US-ASCII encoded string using the data connection in the following form:

&lt;name1&gt;:&lt;type1&gt;&lt;CR&gt;&lt;LF&gt;&lt;name2&gt;:&lt;type2&gt;&lt;CR&gt;&lt;LF&gt;…&lt;nameN&gt;:&lt;typeN&gt;

<ul>

 <li>&lt;nameX&gt; is the name of the X<sup>th</sup> directory/file in the list.</li>

 <li>&lt;typeX&gt; is the type of the X<sup>th</sup> directory/file in the list. It is d if X is a directory, f if it is a file.</li>

 <li>&lt;CR&gt;&lt;LF&gt; is a carriage return character followed by a line feed character, i.e., “r
”.</li>

</ul>

Note that there is no extra &lt;CR&gt;&lt;LF&gt; at the end. Thus, if the current directory is empty, NLST does not send anything but a 16-bit header of zeros. Some examples strings that can be retrieved after an NLST command are as follows:

cheesecake:dr
futon.txt:fr
target.jpg:fr
pocket:d research:d

bear.png:f

Note that cheesecake, pocket and research are directories, futon.txt, target.jpg and bear.png are files in the examples above.

After listing the contents, the client can descend into any subdirectory from the received list by using the CWD command with the subdirectory name as the argument. It can go back to the parent directory with CDUP command. Basically, NLST, CWD and CDUP is the way to traverse the directories in the server.

<strong> </strong>The program you are asked to implement, <em>SeekAndDestroy</em>, is expected to find a file named “<strong>target.jpg</strong>” in the server. The exact location of this file is determined <strong>randomly</strong> at each run of the server. Once the file is found, SeekAndDestroy must download it, then delete it from the server. The following list of steps explains the job expected from your program in full detail:

<ol>

 <li>Start the control connection by connecting to the server from the control port.</li>

 <li>Send a USER command with the username <strong>bilkent</strong>.</li>

 <li>Send a PASS command with the password <strong>cs421</strong>.</li>

 <li>Bind to an available port on your machine for the data connection. Send this port number to the server with a PORT</li>

 <li>Use a combination of NLST, CWD and CDUP commands to search for the file <strong>jpg</strong>. There is no restriction or requirement on how you search for the target file; you can implement any search technique as long as it finds the file.</li>

 <li>Download the file from the server using a RETR Note that once you send a RETR command successfully, the server will try to connect to your program from the data connection port that you are bound to. After you download the file, save it to the same directory as your program with the name <strong>received.jpg</strong>.</li>

 <li>Delete the file from the server using a DELE</li>

 <li>Send a QUIT command to end the session.</li>

</ol>

A few important tips and notes that you should keep in mind:

<ul>

 <li>Check the response message after sending each command and debug your program according to the message. The server shuts itself down if it sends a failure response.</li>

 <li>Compare received.jpg with target.jpg to see if you received the file correctly.</li>

 <li>The RETR command deletes the file from the server’s “artificial” disk, but you will still see target.jpg in the same directory as the server code (it is required for the server code to work).</li>

</ul>




<h1>c. Running the Server Program</h1>

The server program we provide is written and tested in <strong>Python 3.6</strong>. You are also provided an image file named target.jpg, which is required for the server program to work. You must put target.jpg in the <strong>same directory</strong> as CustomFTPServer.py and start the server program <strong>before</strong> running your own program using the following command:

python CustomFTPServer.py &lt;Addr&gt; &lt;ControlPort&gt;

where  “&lt; &gt;” denotes command-line arguments. These command-line arguments are:

<ul>

 <li>&lt;Addr&gt; [Required] The IP address of the server. Since you will be running both your program and the server program on your own machine you should use 0.0.1 or localhost for this argument.</li>

 <li>&lt;ControlPort&gt; [Required] The control port to which the server will bind. Your program should connect to the server from this port to send the control commands.</li>

</ul>

Example:

python CustomFTPServer.py 127.0.0.1 60000

The command above starts the server with IP 127.0.0.1, i.e., localhost, which uses port 60000 for the control commands.

<h1>d. Running <em>SeekAndDestroy</em></h1>

Your program must be <strong>a console application</strong> (no graphical user interface, GUI, is allowed) and should be named as SeekAndDestroy.java (i.e., the name of the class that includes the main method should be SeekAndDestroy). Your program should run with the command

java SeekAndDestroy &lt;Addr&gt; &lt;ControlPort&gt;

where  “&lt; &gt;” denotes command-line arguments. These arguments must be the same as the arguments for the server program, which are explained above.

Example:

java SeekAndDestroy 127.0.0.1 60000

In this example, the program connects to the server with IP 127.0.0.1, i.e., localhost, on port 60000. Please note that you must run your program <strong>after</strong> you start the server program.

<h1>III.      Example</h1>

In this part, we provide you a simple example showing a full session between SeekAndDestroy and CustomFTPServer for clarification. Figure 2 shows the organization of the files and the directories in the server in a tree structure. Table 2 shows all the interactions between SeekAndDestroy and CustomFTPServer throughout the session. Note that the server responses received from the control connection are not shown in this example. Also note that the search technique used here is just an example; you can search for the target using a combination of NLST, CWD and CDUP commands in whichever fashion you like.




<em>Figure 2: Organization of the files and the directories in the server for the given example. Directories are shown in blue, files are shown in orange. </em>

<em>Table 2: An example session. </em>

<table width="626">

 <tbody>

  <tr>

   <td width="39"><strong>Time </strong></td>

   <td width="235"><strong>Action </strong></td>

   <td width="255"><strong>Received Data </strong></td>

   <td width="97"><strong>Comments </strong></td>

  </tr>

  <tr>

   <td width="39">1</td>

   <td width="235">Connect to the server from the control port.</td>

   <td width="255"> </td>

   <td width="97"> </td>

  </tr>

  <tr>

   <td width="39">2</td>

   <td width="235">Send command: USER bilkentr
</td>

   <td width="255"> </td>

   <td width="97"> </td>

  </tr>

  <tr>

   <td width="39">3</td>

   <td width="235">Send command: PASS cs421r
</td>

   <td width="255"> </td>

   <td width="97"> </td>

  </tr>

  <tr>

   <td width="39">4</td>

   <td width="235">Start listening for incoming data connection requests at an available port. Let that port number be 53462 in this example.</td>

   <td width="255"> </td>

   <td width="97"> </td>

  </tr>

  <tr>

   <td width="39">5</td>

   <td width="235">Send command: PORT 53462r
</td>

   <td width="255"> </td>

   <td width="97"> </td>

  </tr>

 </tbody>

</table>




<table width="626">

 <tbody>

  <tr>

   <td width="39">6</td>

   <td width="235">Send command: NLSTr
</td>

   <td width="255">cheesecake:dr
copper:dr
futon.jpg:f</td>

   <td width="97">Directories:cheesecake, copperFile:futon.jpgSince target is not found, we should descend into the other directories and search there.</td>

  </tr>

  <tr>

   <td width="39">7</td>

   <td width="235">Send command: CWD cheesecaker
</td>

   <td width="255"> </td>

   <td width="97">Current working directory is switched from root to cheesecake.</td>

  </tr>

  <tr>

   <td width="39">8</td>

   <td width="235">Send command: NLSTr
</td>

   <td width="255">gasket.png:fr
hundred:dr
milk shake:d</td>

   <td width="97">Still no target.We should look somewhere else.</td>

  </tr>

  <tr>

   <td width="39">9</td>

   <td width="235">Send command: CWD hundredr
</td>

   <td width="255"> </td>

   <td width="97">Current working directory: hundred.</td>

  </tr>

  <tr>

   <td width="39">10</td>

   <td width="235">Send command: NLSTr
</td>

   <td width="255">&lt;Nothing but a 16-bit header of 0’s received&gt;</td>

   <td width="97">Directory is empty.</td>

  </tr>

  <tr>

   <td width="39">11</td>

   <td width="235">Send command: CDUPr
</td>

   <td width="255"> </td>

   <td width="97">We should go back to the parent directory.Current workingdirectory: cheesecake.</td>

  </tr>

  <tr>

   <td width="39">12</td>

   <td width="235">Send command: CWD milkshaker
</td>

   <td width="255"> </td>

   <td width="97">Current working directory: milkshake.</td>

  </tr>

  <tr>

   <td width="39">13</td>

   <td width="235">Send command: NLSTr
</td>

   <td width="255">woodwind.html:f</td>

   <td width="97">Target not found. No subdirectories exist either.</td>

  </tr>

  <tr>

   <td width="39">14</td>

   <td width="235">Send command: CDUPr
</td>

   <td width="255"> </td>

   <td width="97">Go up. Current working directory: cheesecake.</td>

  </tr>

  <tr>

   <td width="39">15</td>

   <td width="235">Send command: CDUPr
</td>

   <td width="255"> </td>

   <td width="97">We searched for all thesubdirectories but did not find the target. We should go up. Current working directory: root.</td>

  </tr>

  <tr>

   <td width="39">16</td>

   <td width="235">Send command: CWD copperr
</td>

   <td width="255"> </td>

   <td width="97">Current working directory: copper.</td>

  </tr>

  <tr>

   <td width="39">17</td>

   <td width="235">Send command: NLSTr
</td>

   <td width="255">shaw:d</td>

   <td width="97"> </td>

  </tr>

  <tr>

   <td width="39">18</td>

   <td width="235">Send command: CWD shawr
</td>

   <td width="255"> </td>

   <td width="97">Current working directory: shaw.</td>

  </tr>

  <tr>

   <td width="39">19</td>

   <td width="235">Send command: NLSTr
</td>

   <td width="255">target.jpg:fr
spaghetti:d</td>

   <td width="97">Target located.</td>

  </tr>

  <tr>

   <td width="39">20</td>

   <td width="235">Send command: RETR target.jpgr
</td>

   <td width="255">&lt;contents of the file target.jpg&gt;</td>

   <td width="97">Target file downloaded.</td>

  </tr>

  <tr>

   <td width="39">21</td>

   <td width="235">Save the received file to the disk with name “received.jpg”.</td>

   <td width="255"> </td>

   <td width="97"> </td>

  </tr>

  <tr>

   <td width="39">22</td>

   <td width="235">Send command: DELE target.jpgr
</td>

   <td width="255"> </td>

   <td width="97">Delete the file from the server.</td>

  </tr>

  <tr>

   <td width="39">23</td>

   <td width="235">Send command: QUIT r
</td>

   <td width="255"> </td>

   <td width="97">End the session.</td>

  </tr>

 </tbody>

</table>





