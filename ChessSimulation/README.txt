Both Chess server and Chess client is written in c++ using TCP sockets.
The chess game does not obey any chess rules.
The source code is unavailable as this is an assignment done for school.

------------------Protocol------------------------------------------------------
Workers:
The client itself has 1 thread for receiving and the main thread for sending 
messages. 
The server has 1 thread for listening for users, 10 threads to handle all the 
client sockets, and 1 thread for rejecting additional users.The main thread takes
in only one command, "exit", which closes the server by sending to all clients
the server_shutdown_cmd which gets echoed back to the servere.
This will unblock the recv() function getting called
by the workers so all threads can safely terminate and all clients and the server 
will exit gracefully.

Handshaking:
Invitation for games when initiated by another user will get sent to the server,
who will then check for availability of the challenged player. If the player
is in a game or is already challenged by another player or does not exist
, the challenger will automatically be rejected, with the reason sent to the challenger.
If the challenged user is available, the server will send the invitiation to the 
challenged client and the user gets to accept or reject the challenge. If the 
challenged user accepts,a game will be constructed with the game id sent to both
players, including the name of both users.

At the start of the game, the player who got challenged gets to move first.
All the game logic is handled in the client and the server is only for the player's 
availabilty and the passing of the moves. When a user sends a move command,
the server determines who sent the move command between the 2 players and send
the command to the other player. The moves are converted to hexadecimal and reconstructed
by the receiving client who then processes the move and update their own board.

A player can quit the game by keying in "quit", which will destroy the game on the server side and
updates both player's availability. The other player also get's notified that the other player left the
game. Same logic applies if the user keys in "exit", except the player also gets removed from 
the server and the client shutsdown.

Gracefully Exit:
Players will all exit gracefully if the server unexpectedly crashes or exits
on its own by typing "exit". This is done by keeping a copy of all the sockets
in the server main thread who sends the server_shutdown_cmd to all clients.
The server will continue running even if any of the players "exit" on their own 
or crash.The server will display those information on it's console. If the 
server exits, the client will be notified and they will need to key in anything 
to end.

------------------Instructions--------------------------------------------------
Initialize:
When booted up, the server will prompt the port number, and print the port and ip
address of the server. The user can then pass that info to the client when the exe
is booted up. The client will prompt the port and Ip and name. If a duplicate name
is keyed, the server will prompt the user to input another name.

Registration:
If registration is successful, the help info will be printed on the client's console,
if not an error message should show.It could be either the server does not exist or
too many users on the server.

Main menu:
All the input a user can type is the same as the one in the specification
and if a user is unsure he can type "help" to see the options again.Keying in
move will display error message notifying the player he is not in a game.

Play:
To play with another person, you need to key "play <player name>" in the console.
To find the player name, key "list users" to show all players.
The player in the "available" state can be played with, while players being
"challenged" or "in game" cannot be played with. If the player is already
in a game, the client itself will not send the invitation out and display error
message telling the client they are in a game at the moment

Commands:
The commands are different from the specification's example as I added more of it. I consulted
professor William and he is okay with it. 

quit_cmd(0)
This command is issued when the player wishes to quit the game,
the server upon receiving it will destroy the game and update player's
availablity.It will then notify the other user by echoing this message 
to it.

register_cmd(1)
This command is for telling the server the name of the
client, which the server will decide if the registration
is successful and notify the user with this command
and the result of the registration. 

list_users_cmd(2)
This command is asking the server for a list of all the users in the 
server. The server compiles it into the message with the state of the 
player right after their names.

show_users_cmd(3)
This is the command for sending the list of users.
I wanted to reuse the list_users_cmd but since the
specification did it this way I shall just follow.

play_cmd(4)
This is the command for inviting another user to 
play. The message will contain the name of the user
the client wish to challenge. The server will then send
the challenged client if it exist and is available, if not
it will send the result_cmd back to the client who challenged.

invite_cmd(5)
This the command sent by the server. The client who receives this will
be notified of the player who wishes to challenge them to a game. The 
user can then type accept or reject to the invitation. This will send
to the server the invite_result_cmd.

invite_result_cmd(6)
The server upon receiving this will know if the other client accepted or rejected the
game. If the client accepted, a game will be constructed with the game id inside sent
to both clients. If the client rejected, the server will send this command to the client.
If a client receives this, their invitation was rejected. The reason will be inside the 
message as well.

game_cmd(7)
This command is sent by the server to the client. It will contain the name
of both users and the id of the game. The client will need to store
this id and use it in the move_piece_cmd.

move_piece_cmd(8)
This command is for moving pieces. It is issued when the client keys 
in the move they wish to carry out after they move it on their own board.
The name of the clients and the id is in the message, just like the 
example in the specification. The client receiving it will extract 
only the move part and move their board.

exit_cmd(9)
This command is issued by the client when they
wish to exit the server. The server will check if they were 
in a game, if they were the game will be destroyed and the other
player will be notified. They will then be removed from the server
by deleting the player info.

server_shutdown_cmd(10)
This is issued by the server when it wishes to shutdown. All clients connected to
the server will receive this command and will echo this command back to the client 
before shuting down themselves. The server will then unblock all the receive so that the thread
can all terminate and join before the server shutsdown.