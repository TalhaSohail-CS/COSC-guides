# Part 0: The beginning (again)

If you’re reading this, you probably have finished PA1. If not, finish PA1 and come back. PA2 builds off what you did in PA1, so you need it to do PA2.

Download the [server.cpp](https://github.com/Gnome67/COSC-guides/blob/main/3360/server.cpp), [client.cpp](https://github.com/Gnome67/COSC-guides/blob/main/3360/client.cpp), and [fireman.cpp](https://github.com/Gnome67/COSC-guides/blob/main/3360/fireman.cpp) files from this repository, or UH's current education software of choice (Canvas at the time of this guide.

Add the `<strings.h>` library to both server and client, and the `<sys/wait.h>` library to the server. Does that second library look familiar? It should. We will be implementing forking into this.

But before we talk about how to do PA2, let's talk about the template files that came with it. More specifically, let's tackle what they do.

# Part 1.1: Understanding Sockets (client)

Let's start with `int main()`:

```cpp
int main(int argc, char *argv[])
{
  int sockfd, portno, n;
  std::string buffer;
  struct sockaddr_in serv_addr;
  struct hostent *server;
```

If you’ve taken Data Structures here at the University of Houston, `argc` and `argv` might look familiar to you. The integer `argc` is short for argument counter. When you run the code using `./main`, you typically use 1 argument (./main), but for both server and client, you will be using multiple arguments. The character array pointer `argv` is short for argument variables, and keeps track of what the arguments you passed in actually say.

Now we move forward to initializing variables. `sockfd` is the socket file descriptor, and `portno` is the port number. `N` is not needed, but it serves the same purpose as `pid` in forking. The string `buffer` is the string we will send to and from the server. The final two structs are the server address and server, which we will use to initiate the socket connection between client and server. These are provided by the libraries, and do not need to be initialized to work.

```cpp
    if (argc != 3) 
    {
       std::cerr << "usage " << argv[0] << " hostname port" << std::endl;
       exit(0);
    }
    portno = atoi(argv[2]);
    sockfd = socket(AF_INET, SOCK_STREAM, 0);
    if (sockfd < 0) 
    {
        std::cerr << "ERROR opening socket" << std::endl;
        exit(0);
    }
    server = gethostbyname(argv[1]);
    if (server == NULL) {
        std::cerr << "ERROR, no such host" << std::endl;
        exit(0);
    }
```

When we run the client, we will have 3 total arguments. `./client` is our first argument. The second argument depends on whether or not you are connecting to someone else’s device or your own. For your own device (which should be like 99.9% of the time for this assignment), it will be `localhost`. Keep in mind you must run the server first. For any other device, find out the device’s IP address (or something like that, not really sure) and use that as your argument. The third argument is the port number, which will be whatever you chose to initialize as when you ran the server, since it will be defined in the server’s run command.

We initialize the `portno` variable as the port number (the third argument, since `argv`, like all other indice-based data structures in C++, begin counting from 0), and initialize the socket file description sockfd as a socket with parameters `AF_INET`, `SOCK_STREAM`, and 0. `AF_INET` is an IPV4 socket, and `SOCK_STREAM` is a TCP socket. 0 is the domain, which in most cases you should leave at 0, because 0 lets it automatically determine. None of this code is important for you to remember or understand (i think) but it’s good to have an idea of what’s going on just in case. The rest of the code is just error handling, nothing crazy.

```cpp
    bzero((char *) &serv_addr, sizeof(serv_addr));
    serv_addr.sin_family = AF_INET;
    bcopy((char *)server->h_addr, 
         (char *)&serv_addr.sin_addr.s_addr,
         server->h_length);
    serv_addr.sin_port = htons(portno);
    if (connect(sockfd,(struct sockaddr *)&serv_addr,sizeof(serv_addr)) < 0) 
    {
        std::cerr << "ERROR connecting" << std::endl;
        exit(0);
    }
```

The `bzero` function initializes an array as all 0s, and `bcopy` copies the values of an array to another, to remove any residual data. I’m not sure what the `sin_ in`, `sin_family`, and `sin_port` mean but they’re just initializing stuff. Then we call `connect` which actually initializes the connection from the client to the server.

```cpp
std::cout << "Please enter the message: ";
    std::getline(std::cin,buffer);
    int msgSize = sizeof(buffer);
    n = write(sockfd,&msgSize,sizeof(int));
    if (n < 0) 
    {
        std::cerr << "ERROR writing to socket" << std::endl;
        exit(0);
    }
    n = write(sockfd,buffer.c_str(),msgSize);
    if (n < 0) 
    {
        std::cerr << "ERROR writing to socket" << std::endl;
        exit(0);
    }
    n = read(sockfd,&msgSize,sizeof(int));
    if (n < 0) 
    {
        std::cerr << "ERROR reading from socket" << std::endl;
        exit(0);
    }
    char *tempBuffer = new char[msgSize+1];
    bzero(tempBuffer,msgSize+1);
    n = read(sockfd,tempBuffer,msgSize);
    if (n < 0) 
    {
        std::cerr << "ERROR reading from socket" << std::endl;
        exit(0);
    }
    buffer = tempBuffer;
    delete [] tempBuffer;
    std::cout << "Message from server: "<< buffer << std::endl;
    close(sockfd);
    return 0;
}
```

This looks like a lot but it’s only because of all the conditionals, let's condense that a bit.

```cpp
std::cout << "Please enter the message: ";
    std::getline(std::cin,buffer);
    int msgSize = sizeof(buffer);
    if (write(sockfd,&msgSize,sizeof(int)) < 0)  { exit(0); }
    if (write(sockfd,buffer.c_str(),msgSize) < 0)  { exit(0); }
    if (read(sockfd,&msgSize,sizeof(int)) < 0)  { exit(0); }
    char *tempBuffer = new char[msgSize+1];
    bzero(tempBuffer,msgSize+1);
    if (read(sockfd,tempBuffer,msgSize) < 0) 
    { exit(0); }
    buffer = tempBuffer;
    delete [] tempBuffer;
    std::cout << "Message from server: "<< buffer << std::endl;
    close(sockfd);
    return 0;
```

Much more compact and much less readable. The first two lines are grabbing input from the user, the message we will send to the server. The next 3 lines are conditionals and error handling. As I said before, we don’t need n so I opted to remove it. However, do NOT remove the “< 0” conditional. For some inexplicable reason, C++ decides to only check if it is a negative number. I don’t know why but I removed it and it caused me errors. Anyways, `write` writes the message to the server, and `read` reads the message from the server. When we write and read to and from the server, we first need the size. This is because you can only send arrays of ints and chars using our method (you can send other data but you would have to modify the code, something outside the scope of this tutorial. I opted to send a string, and I highly recommend you do too to save yourself headaches.) and strings are actually char arrays in disguise. `tempBuffer` exists because the return message will not be the same size as the input so we have to make an array for it. We read the size so we can initialize the array with the right size, and read in the new message. We then delete the array to not have any memory left over, and output the message from the server. Make sure to close the socket when you’re done.

That's all you need to understand for the client template code. Let's look at the server.

# Part 1.2: Understanding Sockets (server)

Now let's look at the server.

```cpp
int main(int argc, char *argv[])
{
   int sockfd, newsockfd, portno, clilen;
   struct sockaddr_in serv_addr, cli_addr;

   // Check the commandline arguments
   if (argc != 2)
   {
      std::cerr << "Port not provided" << std::endl;
      exit(0);
   }

   // Create the socket
   sockfd = socket(AF_INET, SOCK_STREAM, 0);
   if (sockfd < 0)
   {
      std::cerr << "Error opening socket" << std::endl;
      exit(0);
   }
```

Professor Rincon has thankfully provided some comments for us to understand what’s going on. The `clilen` variable is short for client length. The rest is either commented for us or is explained in the client section. Before moving on, I will note that the server is created with two arguments. The first argument is `./server` and the second is the `port number`.

```cpp

   // Populate the sockaddr_in structure
   bzero((char *)&serv_addr, sizeof(serv_addr));
   portno = atoi(argv[1]);
   serv_addr.sin_family = AF_INET;
   serv_addr.sin_addr.s_addr = INADDR_ANY;
   serv_addr.sin_port = htons(portno);

   // Bind the socket with the sockaddr_in structure
   if (bind(sockfd, (struct sockaddr *)&serv_addr, sizeof(serv_addr)) < 0)
   {
      std::cerr << "Error binding" << std::endl;
      exit(0);
   }
```

Let’s talk about binding. Binding creates the socket on the server side and populates it with the proper information so the client can connect when the socket is created on its side.

```cpp
   // Set the max number of concurrent connections
   listen(sockfd, 5);
   clilen = sizeof(cli_addr);
```

5 is an arbitrary number, but this basically says you can have 1 active connection, and a queue of 5 connections waiting to connect after.

```cpp
   // Accept a new connection
   newsockfd = accept(sockfd, (struct sockaddr *)&cli_addr, (socklen_t *)&clilen);
   if (newsockfd < 0)
   {
      std::cerr << "Error accepting new connections" << std::endl;
      exit(0);
   }
   int n, msgSize = 0;
   n = read(newsockfd, &msgSize, sizeof(int));
   if (n < 0)
   {
      std::cerr << "Error reading from socket" << std::endl;
      exit(0);
   }
   char *tempBuffer = new char[msgSize + 1];
   bzero(tempBuffer, msgSize + 1);
   n = read(newsockfd, tempBuffer, msgSize + 1);
   if (n < 0)
   {
      std::cerr << "Error reading from socket" << std::endl;
      exit(0);
   }
   std::string buffer = tempBuffer;
   delete[] tempBuffer;
   std::cout << "Message from client: " << buffer << ", Message size: " << msgSize << std::endl;
   buffer = "I got your message";
   msgSize = buffer.size();
   n = write(newsockfd, &msgSize, sizeof(int));
   if (n < 0)
   {
      std::cerr << "Error writing to socket" << std::endl;
      exit(0);
   }
   n = write(newsockfd, buffer.c_str(), msgSize);
   if (n < 0)
   {
      std::cerr << "Error writing to socket" << std::endl;
      exit(0);
   }
   close(newsockfd);
   close(sockfd);
   return 0;
```

This is the receiving of the client’s message and sending the message “I got your message” back to the client.

# Part 2: Approaching PA2

Ok, let’s look at PA1 before we look at PA2. Regardless of how you did PA1, you will have:

- A call to pthread_create(), ideally in a for loop
- A call to pthread_join(), in a separate for loop
- A void* function for the threads, which i will refer to as `ThreadInstructions`
- If it is not in your void* function, a function to calculate the algorithm
- A struct to pass in data to the threading function, which I will refer to as `ThreadData`
- A data structure to hold the values

Now, we’re going to split up the code you have and the code we have from Professor Rincon. But how? Let’s discuss this in theoretical terms, because I am not here to hold your hand through the entire process. This is a really easy assignment but it is tricky if you overthink it, which is easy to do.

So we’re going to have our `threadData` struct and our `threadInstructions` function in the client. Why? We are going to be threading EXCLUSIVELY on the client. However, the algorithm will be on the server. I’ll explain how and why later, but before we do that, move your algorithm function to the server.cpp (and if it is inside your void* function take it out and make it its own function. Your void* function should now be empty, except for the casting void* to struct*, and returning nullptr. Now look at the client.cpp code, specifically lines 12 to 15, and 23 to 77. We are now going to be moving everything from lines 12 to 15 and 23 to 77 inside our void* function (if you look at Dr. Rincon’s provided client.cpp for the lines that I’m referring to. (Don’t copy over the grabbing head)

In your struct, you should add two new variables to pass in the serverIP and portno you get from `argv[1]` and `argv[2]`, respectively. Since they’re always the same, you can always just pass in `argv[1]` and `argv[2]` in your struct initializer, if you have one (no worries if not, you won't be able to use one for PA3. Just set them when you initialize the struct in `main()`). All that’s left in your main is grabbing the input, creating and joining the threads, and outputting as usual. Nothing crazy.

Let’s go over to the server. Believe it or not, that’s all we had to do for the client. Server is what tripped me up. We’re going to be keeping everything in main, but your algorithm function should be right above your int main() and no longer a void* function if you had it in your void* function. Since we’re not threading on the server, you might be asking how we will be accepting simultaneous processes at the same time. Processes… Where does that sound familiar?

That’s right, forking! We are going to be using forks on the server. But, we don’t know how many forks we need to make. Thankfully, we can take care of that using a while(true) loop just before we initialize the “newsockfd” variable, and begin the if(fork() == 0) conditional just after we initialize the “newsockfd” variable. The rest should be self explanatory.

One last thing. We need the fireman function from the fireman.cpp file from the same place we got client.cpp and server.cpp and we need to include the <sys/wait.h> library. The fireman function basically waits for our code to set itself on fire, and comes in and fixes it. (Seriously though, it will fix the issues the while(true) loop will cause when using the fork() call) Copy the function from the fireman.cpp file to the server.cpp file and remove the cout statement, leaving only the semicolon. Believe it or not, this code will still work. Crazy, right? Now we are going to call it inside our server. Just right before the while(true) loop, call signal(SIGCHLD, fireman);.

I leave the rest up to you because everyone’s code is unique, but if you did it the same way as me where you had a helper function for your algorithm and you sent a string to and from your client and server, then besides calling the appropriate functions, you should be done. See you in PA3.

# Part 3: Testing server and client

So... you want to test your server and client. There are two main ways of doing this. One is to just test it on Moodle, which if you are lazy or in a time crunch, will probably be the one that works for you. Just upload your files to Moodle, and click the check mark button to evaluate your code. If you get a proposed grade, then your code compiled, and you need to do nothing further if said grade is a 100 (unless you messed up, in which case obviously fix it).

The second version assumes you already have set up WSL and VSCode. This guide is not going to explain how to do that. I may make a guide on how to (eventually) though. You will open up VSCode outside of WSL, and press the keys `Ctrl`, `Shift`, and `P` all at the same time, and type in WSL until the option to connect to WSL shows up. If not, either your WSL or your VSCode is not set up properly. Once VSCode opens in WSL, navigate to your folder holding your server and client.cpp, which usually is held in the `mnt/c/` folder. From there, your path will be different based on where you have your Programming Assignment 2 folder installed. For me, mine is `mnt/c/Users/tzkal/Desktop/School/COSC3360/PA2`. Anyways, once you are in the folder that contains your `server.cpp` and `client.cpp` files, you will create two separate terminals, one for the server and the client first. You will always set up the server first.

For the first terminal:
- Build the server using this command `g++ server.cpp -o server`, if you get an error your code doesn't work.
- Build the client using this command `g++ client.cpp -o client -lpthread`, if you get an error your code doesn't work.
- Run the server using `./server 1234`

For the second terminal:
- Run the client using `./client localhost 1234`
- It will (hopefully) prompt you for input. Use the same inputs as PA1, and you should receive the same outputs as you did for PA1.

```
Assistant Professor tzkal
Department of Scuffed Sciences
Office: PGH 533
Phone Number: no
E-mail: no

Version 0.01 - 10/1/2023
Version 0.02 - 10/8/2023 (fixed typo)
Version 0.03 - 10/17/2023 (fixed typo)
Version 0.04 - 11/6/2023 (moved to github)
Version 0.05 - 02/17/2024 (specified instructions)
Version 0.06 - 02/17/2024 (added template files to github to link to)
Version 0.07 - 02/17/2024 (grammar tweaks)
Version 0.08 - 02/18/2024 (instructions on how to build)
```
