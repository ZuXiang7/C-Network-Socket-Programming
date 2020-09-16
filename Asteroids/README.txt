This is a group project done in groups of 4. The application is written in c++
and uses UDP socket. 
The source code is unavailable as this is an assignment done for school.

Important things to note:
    You can find the config file inside the "resources" folder. Make sure 
    to change it to your server's IP address. The server is also hardcoded
    to use port 9000 so you do not need to state it in the config.
    
    For single-player mode, only the client.exe is required along with the 
    config.
    
    For multi-player mode, 4 client.exe with unique UDP ports must connect to a 
    server.exe in order for the game to start. If there are not enough players,
    it will just be a black screen for the player.
    
    To accomplish this, each client must have client.exe and a "resources" 
    folder (by default, found in "client\" folder), to connect to an existing 
    server, in order to start the game.
    
    All behaviour expected of client and server are the same as the 
    specifications given, and have been clarified with grader. More info
    on the protocol will be mentioned below.
    
    Change server_config.txt in "resources/" according to the following format:
    1. Server IP Address (e.g. 192.168.1.130)
    2. Client UDP Port (Any unique port number, e.g. 22. Note that each client
       should have a different UDP Port number.)
    3. Singleplayer/Multiplayer (0 - Multiplayer, 1 - Singleplayer, e.g. 0)
    
    Make sure the server is running before any client tries to connect.
    
Build instructions:
    1. There is a client.exe and server.exe. 1 machine will run
       the server.exe, and 4 other machines can use client.exe.
       
    2. As mentioned, a "client.exe" requires "server_config.txt" to run 
       successfully. Each player will run a copy of "client.exe" with the 
       config to connect to the server. The config will be found in the folder 
       called "resources", it must be added in the same relative directory as 
       the  "client.exe".
       
    3. To sum up, for multiplayer mode, there should be 1 server.exe running, 
       along with 4 client.exe, each with their own server_config.txt stating 
       the server's IP, server's port (9000), their own unique UDP port and 
       single-/multi-player choice (set to 0).
       
    4. For singleplayer mode, only the client.exe is required to run, along 
       with the server_config.txt.
    
How to play:
    Press Left and Right arrow keys to steer your spaceship.
    Press Up and Down arrow keys to accelerate spaceship forward and backward.
    Press Space to shoot bullet.
    Press Esc to exit game.
    
Win Condition:
    To win, a player must reach a score of 10 points by shooting 
    destroying asteroids. Each asteroid is 1 point so whoever destroy 10
    asteroids first wins.
    
    Once the win condition is reached, the game will end with a winning screen 
    for the winning player, and a losing screen for the other players. 
    The score of every player will also be displayed on the console.
    To restart, just quit and run the application again. You can quit 
    by pressing esc button.
    
Team Contributions:
    Seah Zu Xiang - Client UDP socket and recieving thread loop
    Tan Guan Yu   - Server UDP socket, packet format, messaging protocol
    Ang Wei Hao   - Asteroid Game Logic, Singleplayer mode, messaging protocol
    Lim Jia Hao   - Asteroid Game Logic, messaging protocol
    
Protocol:
    Both server and client is using only UDP sockets.
    
    The network architecture follows a Client-Server design where server is
    in charge of receiving events from a client and dispatching them to every
    client in the game, while client is in charge of the game logic and
    rendering while also sending and receiving events to parse.
    
    Register:
        When a client runs the client.exe, if the server config is multiplayer,
        the client will send a UDP message to the server to notify that it is 
        a new player. The server will then send the client it's player number
        and the info of all the other clients that are already in the game. It
        will also forward the new client's info to all already registerd players.
    
    Creation:
        Every object has a unique id. This pool of unique id is managed by
        the server so every object created will be given an id by the server.
        
        Creation of asteroids are handled by the server. The server will keep 
        track of the number of asteroids still inside the game and create a new
        one everytime one gets destroyed. The server creates it by sending 
        a UDP message to the client with the position, velocity and scale.
        
        Creation of bullets are dependant on the client's input. When the 
        client presses spacebar, it will construct a UDP message and send 
        it to the server. The server will then send the message to all
        the clients in the game to create the bullet.
        
        We implemented rough lock step by not rendering client's input 
        immediately, instead we wait for the client to receive it's own 
        event so that all the clients will render at roughly the same time.
        
        Also, all packets needs to be acknowledged by the client upon receiving.
        The client will send a UDP message with the ack info. If the server does
        not receive the ack after the timeout, it will specifically send to that
        client the packet again. Clients will make sure not to create duplicate
        objects by checking if the game has an object with that ID already. The
        server also corrects the position of the packets being resent using the
        velocity of the game object and the timeout as delta time.
    
    Destruction:
        Destruction of objects is similar to creation. When an object is about
        to be destroyed on the client's side, it will not destroy it but instead
        send a UDP message to the server informing which game object is about
        to be destroyed. The server then dispatches that info to all the clients
        and upon receiving it, the client will then destroy that object. 
        
        The client will then acknowledge the fact that it destroyed the object by
        sending an ack back to the server. If the server does not receive it 
        after the timeout, it resends the packet back to the specific client.
        
        Destruction is done everytime a bullet collides an asteroid or when a 
        player collides into an asteroid.
        
    Movement:
        Every frame when the player pressed any of the direction key, the client
        will send the new info of the client's ship such as the velocity, position
        and orientation. The server then dispatches all the info to the 
        client. Unlike creation and destruction, the movement of the ship is 
        not required to be acknowledged as it still has the old movement.
        
        The movement of asteroids and bullets are not constantly synchronised
        by the server as they are passive and deterministic. Also the server
        already does lockstepping during creation and also synchronised 
        resent packets.


Additional Details:
    1. All implementations follow the specifications and rubrics of the assignment.
    2. Client-Server is used for this assignment.
    3. This assignment is completed under the assumption that no lag exists
       and no packets were dropped.
    4. Scores and win conditions are displayed on the console of each client.
    

    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    

    