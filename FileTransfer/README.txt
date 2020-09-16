This is a group project done in groups of 2. The application is written in c++
and uses UDP socket.
The source code is unavailable as this is an assignment done for school.

Instructions:
       "server_config.txt", IP address of server
       to your respective TCP server's IP address. You may also choose to change
       the packet loss percentage in the last line in the configuration text
       file. It is currently defaulted to 1%.
    
Instructions for running another client executable:
    1. Ensure that "SharedFiles" and "server_config.txt" is in the same 
       relative directory as the "client.exe".
    
Executable:
    Executables, client.exe and server.exe will be found in 
    "/x64/Release" when built in Release configuration.
    
Protocol for server and clients:
    The server contains a single TCP listening socket, constantly accepting 
    connecting client sockets, performing handshaking on the main thread. It 
    then has 6 worker threads that will consume connected TCP sockets and 
    services them entirely on the worker thread itself.
    
    Each client will have a single worker thread, the worker thread constantly
    reads user input, then encrypting the message and sending the packet to the 
    TCP server. The client's main thread will then constantly be receiving 
    packets from the server's TCP socket, decrypting the packet and determining 
    what actions to take from the message given.
    
    In general, most client commands require it to query information from the
    server, then according to the information received from the TCP server, take
    action.
    
    Each client will have a receiving UDP socket and a sending UDP socket. The
    UDP is using a pipeline scheme of Go-Back-N to control the sliding window.
    
    Each client will rely on "server_config.txt" to configure its settings on
    start up. The configuration layout is similar to the specifications, but 
    with an additional line at the end, stating the packet loss percentage.
    The packet loss percentage is capped within 0 to 100.
    
    The configuration layout (per line) is as follows,
        1) Server IP
        2) Server Port
        3) Client UDP Port
        4) Directory name to store received files
        5) Directory name to store files to be shared
        6) Percentage of packet loss

Workload split for pair:
    Ang Wei Hao:
        Rubrics 1, 2, 3, 4, 6, 8.
    
    Seah Zu Xiang:
        Rubrics 5, 7, 9, 10, 13
        
Pipeline Scheme for Reliable Data Transfer:
  When a file is requested, the TCP server will ensure the two clients are not
  allowed to transfer files between anyone else. 
  
  The client who will be sending the file will be notified by the server of the 
  receiving client's ip and port. The sender will first perform a handshake by 
  sending the first sequence number (which always starts at 0) and the total 
  length of the file. If the sender does not get acknowledged after 1 second, 
  it will resend the packet.
  
  The receiver upon receiving the handshaking packet will initialized the 
  expected sequence number and the total file length. After that, it will
  acknowledge the packet by sending the expected sequence number back
  and write open the file in the ReceivedFiles directory to write into.
  
  The sender will then proceed to read the files and save them into packets.
  The sender has a sliding window to push the new packets in. It will create 
  new packets if the sliding window size is smaller than the max count we set.
  We set the max size to 5 because we do not want to resend too many packets and
  flood the receiver's buffer. The sender will record the time any packet is 
  last sent or received. This timer is used for timeout, in case all our packets
  or receiver's ack all got loss. We set the timeout to a fixed 0.1 second 
  because we had the best performance from this and we had trouble with 
  estimated RTT. If the sender timeout, it will resend all the packets inside 
  the sliding window. If 4 duplicate acknowledge is received we also resend all 
  the packets as we determine that most likely the first packet in the sliding 
  window was lost. The sender has a thread to handle the acknowledge received. 
  It will check the acknowledge with the packets in the sliding window. If the 
  acknowledge matches one of the packet, it will remove it and all the packets
  sent before it as the receiver will not acknowledge it if it have not 
  acknowledged the packets sent before it.
  
  The receiver will accept all the files received in the correct order. It 
  discards any packet with a different sequence number from the one its 
  expecting. It also sends an acknowledge of the expected sequence number if it 
  received the wrong order packet. If it received the correct packet, it will 
  add the number of bytes received to the expected sequence number and send the 
  acknowledge with the new expected sequence number. The receiver will then
  calculate the total number of bytes it received for the file. If it is equal 
  to the file size given by the sender at the start, it will exit and send a 
  packet of size 0 to the sender and determine the transfer is complete.

  If the sender receives a packet of size 0, it will determine that the 
  receiver has received all the bytes for the file and will inform the sender
  to stop sending as the receiver is done, which will then inform the TCP server
  the transfer is complete.
  
  If the server crashes, the clients will continue sending the file until the 
  transfer is done. The threads handling it will update the client's atomic
  variable that no file transfer is going after it's doneso the client can exit
  gracefully.
  
  If the receiver crashes or exits before a download is done, the sender will
  receive a SOCKET_ERROR, which will tell the sender to stop sending. Because
  the receiver is in charge of notifying the server the file transfer is 
  complete, now that receiver is dead, the server will know that the file 
  transfer is incomplete. When the receiver connects back with the TCP server, 
  the TCP server will notify the two client the file was not finished sending. 
  The sender will than send a handshake to the server with the filesize. The 
  receiver will open the file to see how much it saved before the crash and set 
  it to its expected sequence number. The receiver will also open the file for 
  appending instead of overwriting it. It will send the new expected sequence 
  number to the sender which the sender will then skip all the bytes already 
  read. The file transfer will then continue like normal.
  
  If the sender crashes however, the receiver will not actually know. The 
  revceiver has a timer of its own which will resend its acknowlege after 
  5 seconds of not receiving any packets. If the sender crashed, this will 
  unblock the receiver's recvfrom by socket error. The receiver will 
  reinitialize it's udp socket and will not inform the server the file transfer
  finished. When the sender connects to the server, the server will then do the
  same thing described above when the receiver crashes.
  
  This will work if both receiver and sender crashes as the TCP server will
  wait for both of them to be connected to it before informing them about 
  continuing the file transfer.
    
    
Additional details:
    1) If the client detects that the IP or Port number of TCP server in config
       is invalid, it will exit the application.
    
    2) A total maximum of 6 clients connected to the TCP server is allowed.
       
    3) Typing "exit" will exit the client executable and stop the sending or 
       receiving of a file until the same exitted client is booted up again.
    
    4) The folders "ReceivedFiles" and "SharedFiles" are assumed to be relative
       to the client.exe. If there is no "SharedFiles" folder, the client
       executable will exit on start up. There is no need to create 
       "ReceivedFiles" folder as the client will create it if needed.
       
    5) In the folder "SharedFiles", it is assumed that there are only files
       inside the folder, not folders.
       
    6) The "server_config.txt" file is assumed to be relative to the client.exe.
       If the configuration file is not found, the client executable will exit
       on start up.
       