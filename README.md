Download Link: https://assignmentchef.com/product/solved-programming-assignment-4-best-mismatch-inet-sockets
<br>
In this assignment, you will implement a network version of the <em>categorizer</em> program that you wrote for Assignment 2, but with a slight twist. You will develop an online dating service called “Best Mismatch”. Assuming that opposites attract, our program will accept user preferences as before, but will find a group of users that have exactly opposite interests. It will then recommend these users as potential best mismatches for dating or friendship. It will print the list of best mismatches and <strong>will allow the current user to send messages to any user from this group</strong>.




You will write a socket server which implements this functionality. Clients can connect to your server from any machine, and get the recommendations. For a client program, we use <em>netcat</em>. You <strong>do not</strong> have to implement a socket client for this assignment.




Below you will find a step-by-step description of the product and the code examples with (probably too detailed) explanations. Try to write and test your program on Arch linux system, for an additional bonus.




<h1>1. Reusing code from Assignment 2</h1>




The first step is to review a <em>categorizer</em> prototype you have written for Assignment 2. If your code did not work properly, as indicated by your mark for Assignment 2, it is a good time to come to office hours and fix everything with your instructor or tutor. Do not start writing a socket version before you make sure that your local code works as expected.




It is always better to work with your own code that you wrote and understood. Optionally, we provide a sample solution for Assignment 2, written by one of my best former students Gabrielle Singh Cadieux.




You have already implemented the functionality of producing a list of users based on a given path in the question tree. It is not difficult to tweak this functionality to the case when given a list of answers you move to the leaf in the direction <em>exactly opposite</em> to the answers and reach the bucket with the users who have exactly opposite interests. Add this functionality and test that it works.




Another useful feature is to be able to get a list of answers for a given user name. For this, you may want to modify your depth-first search to maintain a current path that lead to a given leaf. In case that you found a user with a given name in one of the leaves, you will now have the list of their original answers which you can pass to the function above and get the best mismatches for the current user, without asking all the questions again.




<h1>2. Socket server protocol</h1>

<h2>Users and clients</h2>

In you program, you need to collect all users with their answers into a question tree.




In addition, you need to maintain a list of currently connected clients. When user logs in, you create a new node and add it to the list of clients. When user disconnects, you remove its node from the client list.




It might be useful to allocate an array of answers and store it alongside each connected client, because clients may request the list of recommendations at any point, and in this way you would avoid traversing the question tree with each new request.




<h2>IMMP – protocol for mismatch server</h2>

<strong>Client login</strong>

You should be able to connect to your server by typing the following shell command:

nc -C server_name 12345,

where server_name is the hostname of your machine, and 12345 is your port number.




Once you, the user, are connected, you will be asked for the user name. You type the name and hit enter. You may use the same rules for the user name as in Assignment 2, but we will not test for a valid user name while marking this assignment. The only requirement is that the user name does not exceed 128 characters, and if it does, it is truncated by the server. The user name is case-sensitive, as before.

If you provide a new username (never before seen by the server) at the time you connect, the server will create a new user, but if a user by this name already exists in the question tree, it will assume that you are this user. No users are ever deleted from the question tree.

To terminate the current session, client either types quit, or terminates the <em>netcat</em> by sending an interrupt signal Ctrl+C.

When the server is ready to accept commands, it reacts to the commands listed in the table below. Any other command issued by a client should result in a “command not supported” error message returned to the client.

<strong>List of poll commands</strong>

<table width="623">

 <tbody>

  <tr>

   <td width="143">Command</td>

   <td width="480">Description</td>

  </tr>

  <tr>

   <td width="143"><em>do_test</em></td>

   <td width="480">Signifies that the current user is ready to answer questions about their preferences. Server reacts by asking each question from the provided questions file.</td>

  </tr>

  <tr>

   <td width="143"><em>yes/no</em></td>

   <td width="480">Answers to the questions of the current test. The rules are as in Assignment 2: any of YXX and NXX commands are accepted, and the yes/no answers are case-insensitive. Server collects the answers and assigns the user to the corresponding leaf list. In addition, it might choose to record user answers in a separate array, in order to use its reverse for the next command.</td>

  </tr>

  <tr>

   <td width="143"><em>get_all</em></td>

   <td width="480">At any point, the user may request the list of best mismatches to be returned. This list will only be produced if the user has already taken a test of preferences. If the user did not take the test yet, the appropriate error message is returned.</td>

  </tr>

  <tr>

   <td width="143"><em>post &lt;target_name&gt; &lt;message&gt;</em></td>

   <td width="480">Delivers <em>&lt;message&gt;</em> from the current client to the user whose name is specified as <em>&lt;target_name&gt;</em>. The message can contain several words, but has a restriction on the total length: at most 1024 characters.</td>

  </tr>

  <tr>

   <td width="143"><em>quit</em></td>

   <td width="480">Disconnects the client and removes him from the list of active clients.</td>

  </tr>

 </tbody>

</table>

<h1></h1>

<h1>3. Implementing a single-client server</h1>

<h2>3.1. Establishing communication</h2>

Set up your socket interface, bind it to the known port, and implement message passing between the server and a single client connected with <em>netcat</em>. You may start from implementing a simple echo server that echoes each message back to the client. There are plenty of echo server implementations, including the one presented in class.




The entire program should be compiled using <em>make</em>. You will create a <em>Makefile</em> that produces an executable called <em>mismatch_server</em>.




You should be able to start your server by typing the following command:

./mismatch_server &lt;questions_file_name&gt;




In addition to building your code, your <em>Makefile</em> must permit choosing a port at compile-time.




In total, there should be three alternative ways of defining the port.

First, add a #define preprocessing directive to your program to define the port number on which the server will expect connections (base the number &lt;x&gt; on unique subset of numbers in your student number, to avoid port conflicts, in case you forgot to set REUSE_ADDR option):




#ifndef PORT

#define PORT &lt;x&gt;

#endif




Secondly, in your <em>Makefile</em>, include the following code, where &lt;y&gt; should be set to your student number-based port plus 1:




PORT=&lt;y&gt;

CFLAGS+= -DPORT=$(PORT)




Now, if you type make PORT=53456, the program will be compiled with PORT defined as 53456. If you type just make, PORT will be set to y as defined in the <em>Makefile</em>.




Finally, if you use <em>gcc</em> directly and do not supply a port number, it will still have the <em>x</em> value from your source code file. This method of setting a port value will make it possible for us to test multiple submissions by compiling with our desired port numbers. (It is also useful for you to know how to use -D to define macros at command line.)




You should also make sure to add the following lines to your server code so that the port will be released as soon as your server process terminates.

int on = 1;

int status = setsockopt([sock_fd], SOL_SOCKET, SO_REUSEADDR,

(const char *) &amp;on, sizeof(on));

if(status == -1) {

perror(“setsockopt — REUSEADDR”);

}




Once you familiarize yourself with the steps needed to configure a socket server, you may start implementing the required functionality, gradually adding each new feature after you have tested the previous one.




<h2>3.2. Client connects</h2>

When a new client connects, add them to the list of active clients, store their personal file descriptor returned by <em>accept</em>, and then ask for and store their name. You can use a predefined string buffer of at most 128 characters (including the terminator ‘ ’) for the user name, and you can truncate it if the user enters longer name, but you need to notify the user about it.

You will need to maintain an independent linked list – to store all active clients, currently connected to the server.

The suggested structure of each Client node is presented below.

typedef struct client {

int fd;   //file descriptor to write into and to read from

int *answers;

//before user entered a name, he cannot issue commands

int state;

char name [MAX_NAME];

char buf [BUFFER_SIZE];  // each client has its own buffer

int inbuf; // and a pointer to the current end-of-buf position

struct client *next;

} Client;




Note that the server keeps all its data about clients and users in memory. Once the server is killed, all user information is gone.




Sample code for new connection may look like this:

int fd;

struct sockaddr_in r;

socklen_t socklen = sizeof(r);




if ((fd = accept(listenfd, (struct sockaddr *)&amp;r, &amp;socklen)) &lt; 0) {

perror(“accept”);

return;

}

add_client (fd, r.sin_addr);  //call insert into linked list




<h2>3.3. Reading client commands into a dedicated buffer</h2>

Once the connection is established and a new file descriptor for each client is added, we can use regular <em>read</em> and <em>write</em>, as with all file descriptors, to exchange messages between the server and the client.

<h4><strong>Useful note 1. Network new line convention</strong></h4>

In the case of transmitting text, the ASCII standard gives us standard byte values for just about everything <u>except newlines</u>. There is an accepted convention that the network newline is CRLF: r
.

Thus, if the user sends two commands separated by a new line, we need to be able to extract each line and process it separately. The following function suggests the simplest way to find the position of a new line in a network message:

int find_network_newline (char *buf, int inbuf) {

int i;

for (i = 0; i &lt; inbuf – 1; i++)

if ((buf[i] == ‘r’) &amp;&amp; (buf[i + 1] == ‘
’))

return i;

return -1;

}




<h4><strong>Useful note 2. Partial reads problem</strong></h4>

A single message may arrive in packets, so we should be able to read and parse everything until the new line, and then keep the remaining data in buffer until the end of the message arrives later.

This can be implemented in the following way:

char *after = buf + inbuf;

int room = BUFFER_SIZE – inbuf;

int nbytes;

//read next message into remaining room in buffer

if ((nbytes = read(fd, after, room)) &gt; 0) {

inbuf += nbytes;

int where = find_network_newline (buf, inbuf); //find new line

if (where &gt;= 0) {

buf[where] = ‘ ’; buf[where+1] = ‘ ’;

do_command (buf); //process buffer up to a new line

where+=2;  // skip over r


inbuf -= where;

memmove (buf, buf + where, inbuf);

}

}

<h2>3.4. Parsing client commands</h2>

Now, when you have the message properly extracted  from the client, you need to parse it to handle client request. To parse the message, you may use the <em>strtok</em> function. An example is presented below:




/*

** This program extracts tokens from a string using all characters specified in a delimiter.

*/

#include &lt;stdio.h&gt;

#include &lt;string.h&gt;

int main () {

char str[] =”This, a sample string! 
'”;

char * pch;

char delimiter[] = ” 
”;

printf (“Splitting string ”%s” into tokens:
”, str);




pch = strtok (str,delimiter);

while (pch != NULL) {

printf (“%s
”, pch);

pch = strtok (NULL,delimiter);

}

return 0;

}

When you have extracted the command and its arguments, you handle each command, check for a correct message format, and write the corresponding message to the client, using the same file descriptor. The <em>netcat</em> client takes care of all the problems described above, so you may use regular <em>write</em> command.




When implementing the <em>post &lt;target&gt; &lt;message&gt;</em> functionality, you need to locate the target user in the list of active clients first. If not found, you need to return the message “Your post cannot be delivered. User <em>&lt;user_name&gt;</em> is not online”. Otherwise, you post the original message by writing it to the file descriptor of a target client.




<h2>3.5. Disconnecting</h2>

When the user types <em>quit</em>, the server should remove him from the list of active clients and probably send back some sort of goodbye message. The server will also remove the clients who terminated the session without typing <em>quit</em>, by periodically checking closed file descriptors.

<h2><strong> </strong></h2>

<h2>3.6. Requirements to the client-server interaction</h2>

If the client issues an unsupported request or provides invalid parameters, he should be notified about the error. The client cannot issue the <em>get_all</em> command before he issued the <em>do_test</em> command and finished the test. You may need to store a status of each client in the current session in the field <em>status </em>(see sample definition of Client). All client commands should be handled by your code and appropriate data or error notifications should be sent back to the client.




<h1>4. Support for multiple clients</h1>

When several clients connect and issue commands, you’ll need to ensure that the server is not blocked waiting for the response from one of the clients.




The server must never block waiting for input from a particular client or the listening socket. After all, it can’t know which client will talk next or whether a new client will connect. This means that you must use <em>select</em> rather than blocking on one file descriptor.




An example of using <em>select</em> is shown below:

if (select(maxfd + 1, &amp;fdlist, NULL, NULL, NULL) &lt; 0) {

perror(“select”);

} else {

for (p = top; p; p = p-&gt;next)

if (FD_ISSET(p-&gt;fd, &amp;fdlist))

break;

if (p)  //client message received

handle(p);

if (FD_ISSET(listenfd, &amp;fdlist)) //new connection

newconnection(listenfd);

}

<h1>5. Testing</h1>

Since you’re not writing a client program, the <em>netcat</em> tool mentioned above can be used to connect clients to your server and to test your program.




To use it, type nc -C hostname yyyyy, where hostname is the full name of the machine on which your server is running, and yyyyy is the port on which your server is listening.




If you aren’t sure which machine your server is running on you can run hostname -f to find out. If you are sure that the server and client are both on the same machine, you can use localhost in place of the fully specified host name. -C specifies that the end-of-line characters of the sent message should be CRLF.

Test your final product with the following basic use cases. Each use case is accompanied by a screenshot of a running demo. Try to keep your output formats close to the ones presented in these screenshots, to avoid problems with automated testing.

<h4><strong>Case 1: We can start the server by typing:</strong></h4>

Server starts and prints the port it is listening at.

./mismatch_server interests.txt

Listening on 8888




<h4><strong>Case 2: The first client connects (in a separate window) with netcat:</strong></h4>

nc -C 127.0.0.1 8888

What is your user name?

VeryPositive

Welcome.

Go ahead and enter user commands&gt;







<h4><strong>Case 3. Performing the test and asking for recommendations:</strong></h4>

Go ahead and enter user commands&gt;

do_test

Collecting your interests

Do you like Tattoos?

y

Do you like Reality TV?

y

Do you like Justin Bieber?

y

Do you like Bill Gates?

y

Test complete.

get_all

No completing personalities found. Please try again later










<h4><strong>Case 4. The second client can connect (in a separate window) while the first one is still connected.</strong></h4>

nc -C src-code.simons-rock.edu 8888

What is your user name?

VeryNegative

Welcome.

Go ahead and enter user commands&gt;

do_test

Collecting your interests

Do you like Tattoos?

n

Do you like Reality TV?

n

Do you like Justin Bieber?

n

Do you like Bill Gates?

n

Test complete.

get_all

Here are your best mismatches:

VeryPositive







<h4><strong>Case 5. Posting message from VeryNegative to VeryPositive:</strong></h4>

post VeryPositive hello, how are you?







Message delivered:

Message from VeryNegative:  hello, how are you?







<h4><strong>Case 6. Clients disconnect with either quit command, or with an interrupt signal.</strong></h4>

quit




^C




Server handles client disconnect:

Removing client VeryNegative

Removing client VeryPositive







<strong> </strong>

<h1>6. Sample Code</h1>

You are provided with two examples: one is discussed in class (chat_server), and another is written by Alan Rosenthal (muffin_man) that you might find helpful. Feel free to yank code from there, with two important warnings:

<ul>

 <li>Don’t copy-and-paste stuff into your program and then fuss with it to make it work. You should know exactly what the code does and why. We won’t take kindly to extra stuff in your code that is unnecessary or does not work, and is clearly left over from the sample server.</li>

 <li>Clearly indicate the parts of code that you copied from the sample server programs.</li>

</ul>

<h1>7. Coding style</h1>

Coding style and code readability are very important. Use good variable names, appropriate functions, descriptive comments, and blank lines. Remember that someone needs to read your code. You may write extra helper functions. You MUST perform error-checking for all system calls (and functions which use system calls). We recommend defining a set of helper functions to wrap these calls to reduce duplication.

<strong> </strong>


